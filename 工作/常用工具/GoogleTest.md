# GoogleTest

## TestSuite

```cpp
TEST(TestSuiteName, TestCaseName) {
    // 单测代码
    EXPECT_EQ(func(0), 0);
}
```

- `TestSuiteName` 用来汇总 test case，相关的 test case 应该是相同的 `TestSuiteName`
- 一个文件里只能有一个 `TestSuiteName`，建议命名为这个文件测试的类名
- `TestCaseName` 是测试用例的名称
- `TestSuiteName_TestCaseName` 的组合应该是唯一的
- GTest 生成的类名是带下划线的，所以上面这些名字里不建议有下划线
- `TEST` 是最基本的测试宏，代表一个最小测试单元。在执行 `TEST` 宏时，gtest 会为每个 `TEST` 定义一个独立的实例，使其互相隔离，避免对同一个变量进行修改或共享等可能带来的副作用
- `TEST_F` 是 TestFixture 的测试宏。TestFixture 是一个类，可以在多个测试用例之间共享数据结构或方法
- 对于同一个 Test Suite 的所有 Test Cases，会创建一个 TestFixture 对象，其 SetUp 函数会在每个 Test Case 执行之前被调用，而 TearDown 函数则会在每个 Test Case 执行之后被调用

```cpp
class FooTest : public ::testing::Test {
protected:
  // 在每个 Test Case 运行开始前，都会调用 SetUp，这里可以初始化
  void SetUp() override {
    ctx = RequestContext("123");
  }
  
  // 在每个 Test Case 运行结束后，都会调用 TearDown
  void TearDown() override {}
  
  // 所有 Test Case 都可以直接访问这些变量和方法
  Ad new_ad() { return Ad(ctx); }
  RequestContext ctx;
};

TEST_F(FooTest, enable_foo) { // 这里会初始化 FooTest 对象
  ctx->params.enable_foo = true; // 可以访问 FooTest 中的变量
  auto item = new_ad(); // 可以调用 FooTest 中的方法
  ...
}

// 每个 test case 都是独立的，这里会初始化另一个 FooTest 对象
TEST_F(FooTest, OnTestProgramStart) { 
  // ...
}
```

## 断言

如果某个判断不通过时，会影响后续步骤，要使用 `ASSERT`，其他情况可以使用 `EXPECT`，尽可能多测试几个用例

- `EXPECT_TRUE(foo)`、`EXPECT_FALSE(foo)`：判断一个变量是否是 true 或 false
- `EXPECT_EQ(foo, bar)`：判断两个变量是否相等，只要重载了`==`运算符的对象都可以使用
- `EXPECT_NE`、`EXPECT_LT`、`EXPECT_LE`、`EXPECT_GT`、`EXPECT_GE`
- `EXPECT_DOUBLE_EQ`、`EXPECT_FLOAT_EQ`、`EXPECT_NEAR`
- `EXPECT_STREQ`、`EXPECT_STRNE`、`EXPECT_STRCASEEQ`、`EXPECT_STRCASENE`

```cpp
double pi = 3.141592653589793238;
double approx_pi = 3.14;
EXPECT_NEAR(pi, approx_pi, 0.01);  // 检测两个 π 值，允许误差在 0.01 以内
```

默认当 EXPECT 或 ASSERT 失败时，GTest 会打印预期值和实际值，但也可以自定义输出日志，这些日志仅在失败时才打印

```cpp
for (int i = 0; i < x.size(); i++) {
  EXPECT_EQ(x[i], y[i]) << "x and y differ at index " << i;
}
```

还可以在 TestFixture 中封装 debug 函数，输出更详细的信息

```cpp
class BitsetTest : public BaseTest {
public:
  std::string debug_message() {
      stringstream ss;
      for (const auto& iter : bitset_maps) {
        ss << "bitset: name=" << iter.first << " value=" << iter.second << std::endl;
      }
      return ss.string();
  }
}

TEST_F(BitsetTest, validate) {
    // ...
    EXPECT_TRUE(validate(ad, pos)) << debug_message();
}
```



