# lock_guard

## lock_guard

`std::lock_guard` 利用了 C++ RAII 的特性，在构造函数中上锁，析构函数中解锁，从而保证了一个已锁的互斥量总是会被正确的解锁

`std::lock_guard` 没有任何成员函数，没有额外的开销，只包含一个互斥量的引用

不能复制或移动`lock_guard`对象，确保锁的所有权单一

```cpp
template <class Mutex> class lock_guard {
private:
  lock_guard(lock_guard const&) = delete;
  lock_guard& operator=(lock_guard const&) = delete;
}

/*
Mutex 代表互斥量，可以是 std::mutex、std::timed_mutex、std::recursive_mutex、std::recursive_timed_mutex 中的任何一个，也可以是 std::unique_lock，都提供了 lock 和 unlock 的能力

lock_guard 仅用于上锁、解锁，不对 mutex 承担供任何生周期的管理，因此在使用的时候，请确保 lock_guard 管理的 mutex 一直有效
*/
```

```cpp
std::mutex my_lock;

void add(int &num, int &sum) {
  while (true) {
    std::lock_guard<std::mutex> lock(my_lock);
    if (num < 100) {
      num += 1;
      sum += num;
    } else {
      break;
    }
  }
}

int main() {
  int sum = 0;
  int num = 0;
  std::vector<std::thread> ver; // 保存线程的vector
  for (int i = 0; i < 20; ++i) {
    std::thread t = std::thread(add, std::ref(num), std::ref(sum));
    ver.emplace_back(std::move(t)); // 保存线程
  }

  std::for_each(ver.begin(), ver.end(),
                std::mem_fn(&std::thread::join)); // join
  std::cout << sum << std::endl;
}
```

## unique_lock

`std::unique_lock` 有更多成员函数函数，可以手动释放锁，一般配合条件变量使用

- 可以在需要时手动锁定和解锁互斥量
- 可以在构造时不立即锁定互斥量，稍后再锁定
- 可以与`std::condition_variable`一起使用，因为条件变量需要手动解锁和重新锁定
- 可以通过移动转移所有权，但不能复制

```cpp
void flexible_function() {
    std::unique_lock<std::mutex> lock(mtx); // 构造时加锁
    
    // 手动解锁（临时释放锁）
    lock.unlock();
    
    // 执行非临界区代码
    non_critical_work();
    
    // 重新加锁
    lock.lock();
    
    // 继续临界区操作
} // 作用域结束时自动解锁

void defer_lock() {
  // 延迟加锁构造
  std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
  // ...其他操作...
  lock.lock(); // 需要时手动加锁
}
```

