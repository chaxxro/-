# pb

## 定义消息体

```pb
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

第一行必须指明当前使用 `proto3` 还是 `proto2`，默认 `proto2`

## 字段编号

- 每个字段必须有一个唯一的字段编号，范围在 1-536870911，其中 19000-19999 是 pb 内部使用的编号

- 字段编号一旦使用就不允许修改

- 不允许从保留列表中取编号出来复用编号

## 字段标签

### optional

`optional` 字段有两种状态

1. 字段被显式设置，它将被序列化

2. 字段未显式设置，它将使用默认值，并且不进行序列化

有方法知道 `optional` 字段是否被显式设置

### repeated

`repeated` 字段可以重复 0 次或多次，重复值的顺序将被保留段，类似于列表

### map

`map` 是一个 kv 结构，其编码方式跟 `repeated` 相同

`map` 的 key 只支持整形和字符串，value 不支持 `map`

```pb
message Test6 {
  map<string, int32> g = 7;
}
// same as
message Test6 {
  message g_Entry {
    optional string key = 1;
    optional int32 value = 2;
  }
  repeated g_Entry g = 7;
}
```

### implicit field presence

如果不显式指定标签则默认的标签是隐式字段，隐式字段不可显式指定

一个格式良好的消息体最多只能有一个隐式字段

## reserve

删除字段后，必须使用 `reserve` 处理被删除的字段编号，避免后续继续使用该编号

为了保证 json 和文本功能后续能正常使用，还需要 `reserved` 字段名

```pb
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

## 基础数据类型

| type     | C++ 类型 | 备注                                      |
| -------- | -------- | ----------------------------------------- |
| double   | double   |                                           |
| float    | float    |                                           |
| int32    | int32    | 变长，处理负数低效                        |
| int64    | int64    | 变长，处理负数低效                        |
| uint32   | uint32   | 变长                                      |
| uint64   | uint64   | 变长                                      |
| sint32   | int32    | 变长，处理负数高效                        |
| sint64   | int64    | 变长，处理负数高效                        |
| fixed32  | uint32   | 定长，处理大数高效                        |
| fixed64  | uint64   | 定长，处理大数高效                        |
| sfixed32 | int32    | 定长                                      |
| sfixed64 | int64    | 定长                                      |
| bool     | bool     |                                           |
| string   | string   | 必须 utf-8 或 ASCII 编码，长度不超过 2^32 |
| bytes    | string   | 长度不超过 2^32                           |

## 枚举类型

```pb
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2;
}

enum EnumNotAllowingAlias {
  ENAA_UNSPECIFIED = 0;
  ENAA_STARTED = 1;
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message.
  ENAA_FINISHED = 2;
}
```

- 枚举的第一个元素必须是 0

- 默认一个枚举里不允许有相同值的不同字段，使用 `option allow_alias = true` 时允许不同字段有相同值

- 枚举值底层使用 32 位整形，所以不推荐使用负数，并且处理负数的效率较低

- 反序列化时，不被识别的枚举值将被保留，其值取决于编程语言本身

同删除字段一样，可以将 `reserved` 应用在枚举上，来避免后续使用删除的枚举值

```pb
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

## oneof

类似于 C++ 的 `union`，但每次设置时都会清理所有数据再赋值

`oneof` 中不允许使用 `map` 和 `repeated`，它本身也不允许使用 `repeated`

```pb
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

## Any

`Any` 类型允许用户使用过一个没有 .proto 定义的类型

使用 `Any` 需要导入 google/protobuf/any.proto

```pb
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

```cpp
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

## service

使用 `service` 来定义一个 RPC 服务接口

```pb
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

## option

`option` 不会改变声明的整体含义，但可能会影响在特定上下文中的处理方式

完成的可用 `option` 在 /usr/include/google/protobuf/descriptor.proto

### optimize_for

```pb
option optimize_for = CODE_SIZE;
```

文件级，可选值有 `CODE_SIZE`、`SPEED` 和 `LITE_RUNTIME`

- `CODE_SIZE` 占空间少

- `SPEED` 运行效率高，但占更多空间

- `LITE_RUNTIME` 运行效率高，占空间少，但缺乏反射

## protoc

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

`--proto_path` 指定搜索 .proto 的文件地址，一次只能指定一个地址，可缩写成 `--I`

## protobuf API

每个 message 都生成了一个类，为每个字段提供了读写函数

```cpp
  // required uint64 id = 1;
  inline bool has_id() const;
  inline void clear_id();
  static const int kIdFieldNumber = 1;
  inline ::google::protobuf::uint64 id() const;
  inline void set_id(::google::protobuf::uint64 value);

  // required string name = 2;
  inline bool has_name() const;
  inline void clear_name();
  static const int kNameFieldNumber = 2;
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline void set_name(const char* value, size_t size);
  inline ::std::string* mutable_name();
  inline ::std::string* release_name();
  inline void set_allocated_name(::std::string* name);

  // optional string email = 3;
  inline bool has_email() const;
  inline void clear_email();
  static const int kEmailFieldNumber = 3;
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline void set_email(const char* value, size_t size);
  inline ::std::string* mutable_email();
  inline ::std::string* release_email();
  inline void set_allocated_email(::std::string* email);

  // repeated .tutorial.Student.PhoneNumber phone = 4;
  inline int phone_size() const;
  inline void clear_phone();
  static const int kPhoneFieldNumber = 4;
  inline const ::tutorial::Student_PhoneNumber& phone(int index) const;
  inline ::tutorial::Student_PhoneNumber* mutable_phone(int index);
  inline ::tutorial::Student_PhoneNumber* add_phone();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Student_PhoneNumber >& phone() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Student_PhoneNumber >* mutable_phone();
```

- getter 函数具有与字段名一模一样的名字，并且是小写的
- setter 函数都是以 set_ 前缀开头
- has_ 函数，对每一个单一的（required 或 optional 的）字段来说，如果字段被置了值，该函数会返回 true
- 每一个字段还有一个 clear_ 函数，用来将字段重置到空状态
- string 类型字段还有前缀为 mutable_ 的函数返回 string 的直接指针
- repeated 字段也有一些特殊的函数，name_size 函数得到重复字段的数量，getter 函数可以通过索引来获取一个指定的元素，mutable_name 可以通过指定的索引来更新一个已经存在的元素，add_name 可以添加一个元素并返回新元素的指针

每一个消息（message）还包含了其他一系列函数，用来检查或管理整个消息

```cpp
bool IsInitialized() const; //检查是否全部的required字段都被置（set）了值

void CopyFrom(const Person& from); //用外部消息的值，覆盖调用者消息内部的值

void Swap(Person *p);  // 交换两个对象

void Clear();	//将所有项复位到空状态（empty state）。

int ByteSize() const;	//消息字节大小
```

Debug API

```cpp
string DebugString() const;	//将消息内容以可读的方式输出

string ShortDebugString() const; //功能类似于，DebugString(),输出时会有较少的空白

string Utf8DebugString() const; //Like DebugString(), but do not escape UTF-8 byte sequences.

void PrintDebugString() const;	//Convenience function useful in GDB. Prints DebugString() to stdout.
```

解析 API

```cpp
bool SerializeToString(string* output) const; //将消息序列化并储存在指定的string中。注意里面的内容是二进制的，而不是文本；只是使用string作为一个很方便的容器。

bool ParseFromString(const string& data); //从给定的string解析消息。

bool SerializeToArray(void * data, int size) const	//将消息序列化至数组

bool ParseFromArray(const void * data, int size)	//从数组解析消息

bool SerializeToOstream(ostream* output) const; //将消息写入到给定的C++ ostream中。

bool ParseFromIstream(istream* input); //从给定的C++ istream解析消息。
```