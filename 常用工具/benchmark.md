# benchmark

## BENCHMARK

`BENCHMARK` 宏用于注册一个简单的基准测试函数，不使用基准测试夹具

每一个 benchmark 测试用例都是一个类型为 `std::function<void(benchmark::State&)>` 的函数，其中 `benchmark::State&` 负责测试的运行及额外参数的传递

```cpp
#include <benchmark/benchmark.h>

static void BM_StringCreation(benchmark::State& state) {
  for (auto _ : state)
    std::string empty_string;
}
// Register the function as a benchmark
BENCHMARK(BM_StringCreation);

// Define another benchmark
static void BM_StringCopy(benchmark::State& state) {
  std::string x = "hello";
  for (auto _ : state)
    std::string copy(x);
}
BENCHMARK(BM_StringCopy);

BENCHMARK_MAIN();
```

使用 `for (auto _: state) {}` 来运行需要测试的内容，`state` 会选择合适的次数来运行循环

时间的计算从循环内的语句开始，所以可以在循环之外初始化测试环境，然后在循环体内编写需要测试的代码

测试用例编写完成后需要使用 `BENCHMARK()` 进行注册

最后使用 `BENCHMARK_MAIN()` 替代直接编写的 `main` 函数，它会处理命令行参数并运行所有注册过的测试用例生成测试结果

## BENCHMARK_REGISTER_F

`BENCHMARK_REGISTER_F` 宏用于注册一个使用基准测试夹具（Fixture）的基准测试函数。基准测试夹具允许你在基准测试运行之前和之后执行一些初始化和清理操作

```cpp
#include <benchmark/benchmark.h>
// 继承自 benchmark::Fixture
class MyFixture : public benchmark::Fixture {
public:
    void SetUp(const ::benchmark::State& state) override {
        // 初始化代码
    }

    void TearDown(const ::benchmark::State& state) override {
        // 清理代码
    }
};
// 定义基准测试函数
BENCHMARK_DEFINE_F(MyFixture, BM_FixtureFunction)(benchmark::State& state) {
    for (auto _ : state) {
        // 基准测试代码
    }
}

BENCHMARK_REGISTER_F(MyFixture, BM_FixtureFunction);

BENCHMARK_MAIN();
```

## 传递参数

测试用例只能接受一个 `benchmark::State&` 类型的参数，但可以将参数传递给 `benchmark::State`，然后在测试用例中获取

传递参数使用 `BENCHMARK` 宏生成的对象的 `Arg` 方法，`Args` 方法接受一个 `vector` 对象

传递进来的参数会被放入 `state` 对象内部存储，通过 `range` 方法获取，调用时的参数 0 是传入参数的需要，对应第一个参数

```cpp
static void bench_array_ring_insert_int(benchmark::State& state)
{
    auto length = state.range(0);
    auto ring = ArrayRing<int>(length);
    for (auto _: state) {
        for (int i = 1; i <= length; ++i) {
            ring.insert(i);
        }
        state.PauseTiming();
        ring.clear();
        state.ResumeTiming();
    }
}
// 执行一次
BENCHMARK(bench_array_ring_insert_int)->Arg(10);
// 执行多次，分别是 10、100、1000
BENCHMARK(bench_array_ring_insert_int)->Arg(10)->Arg(100)->Arg(1000);
BENCHMARK(bench_array_ring_insert_int)->Args({10, 10});
```

参数传递只接受整数

## 模板函数

`BENCHMARK` 处理不了模板，模板函数需要使用 `BENCHMARK_TEMPLATE`

```cpp
template <typename T, std::size_t length, bool is_reserve = true>
void bench_vector_reserve(benchmark::State& state)
{
	for (auto _ : state) {
		std::vector<T> container;
		if constexpr (is_reserve) {
			container.reserve(length);
		}
		for (std::size_t i = 0; i < length; ++i) {
			container.push_back(T{});
		}
	}
}

BENCHMARK_TEMPLATE(bench_vector_reserve, std::string, 100);
```

## 结果分析

- Time 表示全部时间，包括所有的等待时间、I/O 操作时间等
- CPU 表示实际执行代码的时间
- Iterations 表示基准测试运行的次数，更多的迭代次数可以提供更稳定的结果

如果墙钟时间和 CPU 时间差异较大，可能表明存在 I/O 操作或其他阻塞操作

