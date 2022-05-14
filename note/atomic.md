# atomic

## `atomic_flag`

`atomic_flag` 是一种简单的原子布尔类型，只支持两种操作，`test-and-set` 和 `clear`。

构造函数：
```cpp
atomic_flag() noexcept = default;
atomic_flag (const atomic_flag&T) = delete;
```

`std::atomic_flag` 只有默认构造函数，拷贝构造函数已被禁用，因此不能从其他的 `std::atomic_flag` 对象构造一个新的 `std::atomic_flag` 对象。

如果在初始化时没有明确使用 `ATOMIC_FLAG_INIT` 初始化，那么新创建的 `std::atomic_flag` 对象的状态是未指定的（`unspecified`）（既没有被 `set` 也没有被 `clear`）另外，`atomic_flag` 不能被拷贝，也不能 `move` 赋值。

`ATOMIC_FLAG_INIT`：如果某个 `std::atomic_flag` 对象使用该宏初始化，那么可以保证该 `std::atomic_flag` 对象在创建时处于 `clear` 状态。

下面先看一个简单的例子，`main()` 函数中创建了 `10` 个线程进行计数，率先完成计数任务的线程输出自己的 `ID`，后续完成计数任务的线程不会输出自身 `ID`：

```cpp
#include <iostream>              // std::cout
#include <atomic>                // std::atomic, std::atomic_flag, ATOMIC_FLAG_INIT
#include <thread>                // std::thread, std::this_thread::yield
#include <vector>                // std::vector

std::atomic<bool> ready(false);    // can be checked without being set
std::atomic_flag winner = ATOMIC_FLAG_INIT;    // always set when checked

void count1m(int id) {
    while (!ready) {
        std::this_thread::yield();
    } // 等待主线程中设置 ready 为 true.

    for (int i = 0; i < 1000000; ++i) {
    } // 计数.

    // 如果某个线程率先执行完上面的计数过程，则输出自己的 ID.
    // 此后其他线程执行 test_and_set 是 if 语句判断为 false，
    // 因此不会输出自身 ID.
    if (!winner.test_and_set()) {
        std::cout << "thread #" << id << " won!\n";
    }
};

int main() {
    std::vector<std::thread> threads;
    std::cout << "spawning 10 threads that count to 1 million...\n";
    for (int i = 1; i <= 10; ++i)
        threads.push_back(std::thread(count1m, i));
    ready = true;

    for (auto & th:threads)
        th.join();

    return 0;
}
```

`std::atomic_flag::test_and_set`

```cpp
bool test_and_set (memory_order sync = memory_order_seq_cst) volatile noexcept;
bool test_and_set (memory_order sync = memory_order_seq_cst) noexcept;
```

`test_and_set()` 函数检查 `std::atomic_flag` 标志，如果 `std::atomic_flag` 之前没有被设置过，则设置 `std::atomic_flag` 的标志，并返回先前该 `std::atomic_flag` 对象是否被设置过，如果之前 `std::atomic_flag` 对象已被设置，则返回 `true`，否则返回 `false`。

`test-and-set` 操作是原子的（因此 `test-and-set` 是原子 `read-modify-write ` 操作）。

`test_and_set` 可以指定 `Memory Order`，取值如下：

|Memory Order 值|Memory Order 类型|
|-|-|
|memory_order_relaxed|Relaxed|
|memory_order_consume|Consume|
|memory_order_acquire|Acquire|
|memory_order_release|Release|
|memory_order_acq_rel|Acquire/Release|
|memory_order_seq_cst|Sequentially consistent|

`std::atomic_flag::clear()`

清除 `std::atomic_flag` 对象的标志位，即设置 `atomic_flag` 的值为 `false`。`clear` 函数原型如下：

```cpp
void clear (memory_order sync = memory_order_seq_cst) volatile noexcept;
void clear (memory_order sync = memory_order_seq_cst) noexcept;
```

清除 `std::atomic_flag` 标志使得下一次调用 `std::atomic_flag::test_and_set` 返回 `false`。可以指定 Memory Order。

结合 `std::atomic_flag::test_and_set()` 和 `std::atomic_flag::clear()`，`std::atomic_flag` 对象可以当作一个简单的自旋锁使用：

```cpp
#include <thread>
#include <vector>
#include <iostream>
#include <atomic>

std::atomic_flag lock = ATOMIC_FLAG_INIT;

void f(int n) {
    for (int cnt = 0; cnt < 10; ++cnt) {
        while (lock.test_and_set(std::memory_order_acquire))  // acquire lock
             ; // spin
        std::cout << "Output from thread " << n << '\n';
        lock.clear(std::memory_order_release);               // release lock
    }
}

int main() {
    std::vector<std::thread> v;
    for (int n = 0; n < 10; ++n) {
        v.emplace_back(f, n);
    }
    for (auto& t : v) {
        t.join();
    }
}
```

在上锁的时候，如果 `lock.test_and_set` 返回 `false`，则表示上锁成功（此时 `while` 不会进入自旋状态），但调用 `test_and_set` 后 `lock` 的标志位为 `true`，说明某一线程已经成功获得了 `lock` 锁。

如果在该线程解锁（即调用 `lock.clear(std::memory_order_release)`） 之前，另外一个线程也调用 `lock.test_and_set(std::memory_order_acquire)` 试图获得锁，则 `test_and_set(std::memory_order_acquire)` 返回 `true`，则 `while` 进入自旋状态。如果获得锁的线程解锁（即调用了 `lock.clear(std::memory_order_release)`）之后，某个线程试图调用 `lock.test_and_set(std::memory_order_acquire)` 并且返回 `false`，则 `while` 不会进入自旋，此时表明该线程成功地获得了锁。

## `std::atomic`

`std::atomic` 是模板类，一个模板类型为 `T` 的原子对象中封装了一个类型为 `T` 的值。

```cpp
template <class T> struct atomic;
```

通过这个原子类型管理的内部变量就可以称之为原子变量，我们可以给原子类型指定 `bool`、`char`、`int`、`long`、指针等类型作为模板参数（不支持浮点类型和复合类型）。

原子类型对象的主要特点就是从不同线程访问不会导致数据竞争。因此从不同线程访问某个原子对象是良性行为，而通常对于非原子类型而言，并发访问某个对象（如果不做任何同步操作）会导致未定义行为发生。

构造函数：

```cpp
// default (1)
atomic() noexcept = default;
// initialization (2)
constexpr atomic (T val) noexcept;
// copy [deleted] (3)
atomic (const atomic&) = delete;
```

- 默认构造函数，由默认构造函数创建的 std::atomic 对象处于未初始化状态，对处于未初始化状态 std::atomic对象可以由 atomic_init 函数进行初始化。
- 初始化构造函数，由类型 T 初始化一个 std::atomic对象。
- 拷贝构造函数被禁用。

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic, std::atomic_flag, ATOMIC_FLAG_INIT
#include <thread>         // std::thread, std::this_thread::yield
#include <vector>         // std::vector
 
// 由 false 初始化一个 std::atomic<bool> 类型的原子变量
std::atomic<bool> ready(false);
std::atomic_flag winner = ATOMIC_FLAG_INIT;
 
void do_count1m(int id) {
    while (!ready) { std::this_thread::yield(); } // 等待 ready 变为 true.
 
    for (volatile int i=0; i<1000000; ++i) {} // 计数
 
    if (!winner.test_and_set()) {
      std::cout << "thread #" << id << " won!\n";
    }
}
 
int main () {
    std::vector<std::thread> threads;
    std::cout << "spawning 10 threads that count to 1 million...\n";
    for (int i=1; i<=10; ++i) threads.push_back(std::thread(do_count1m,i));
    ready = true;
 
    for (auto& th : threads) th.join();
    return 0;
}
```

`std::atomic::operator=()`

```cpp
// set value (1)|
T operator= (T val) noexcept;
T operator= (T val) volatile noexcept;
// copy [deleted] (2)|
atomic& operator= (const atomic&) = delete;
atomic& operator= (const atomic&) volatile = delete;
```

可以看出，普通的赋值拷贝操作已经被禁用。但是一个类型为 `T` 的变量可以赋值给相应的原子类型变量（相当与隐式转换），该操作是原子的，内存序默认为顺序一致性，如果需要指定其他的内存序，需使用 `std::atomic::store()`。

```cpp
#include <iostream>             // std::cout
#include <atomic>               // std::atomic
#include <thread>               // std::thread, std::this_thread::yield
 
std::atomic <int> foo(0);

void set_foo(int x) {
    foo = x; // 调用 std::atomic::operator=().
}
 
void print_foo() {
    while (foo == 0) { // wait while foo == 0
        std::this_thread::yield();
    }
    std::cout << "foo: " << foo << '\n';
}

int main() {
    std::thread first(print_foo);
    std::thread second(set_foo, 10);
    first.join();
    second.join();
    return 0;
}
```

成员函数：

`std::atomic` 是模板类，一个模板类型为 `T` 的原子对象中封装了一个类型为 `T` 的值。`std::atomic` 类模板除了基本类型以外，还针对整形和指针类型做了特化。 特化的 `std::atomic` 类型支持更多的操作，如 `fetch_add`, `fetch_sub`, `fetch_and` 等：

`is_lock_free`

```cpp
bool is_lock_free() const volatile noexcept;
bool is_lock_free() const noexcept;
```

判断该 `std::atomic` 对象是否具备 `lock-free` 的特性。如果某个对象满足 `lock-free` 特性，在多个线程访问该对象时不会导致线程阻塞。

`store`

```cpp
void store (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
void store (T val, memory_order sync = memory_order_seq_cst) noexcept;
```

修改被封装的值，`std::atomic::store` 函数将类型为 `T` 的参数 `val` 复制给原子对象所封装的值。`T` 是 `std::atomic` 类模板参数。另外参数 `sync` 指定内存序，可能的取值如下：

|Memory Order 值|Memory Order 类型|
|-|-|
|memory_order_relaxed|Relaxed|
|memory_order_release|Release|
|memory_order_seq_cst|Sequentially consistent|

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic, std::memory_order_relaxed
#include <thread>         // std::thread
 
std::atomic<int> foo(0); // 全局的原子对象 foo
 
void set_foo(int x) {
    foo.store(x, std::memory_order_relaxed); // 设置(store) 原子对象 foo 的值
}
 
void print_foo() {
    int x;
    do {
        x = foo.load(std::memory_order_relaxed); // 读取(load) 原子对象 foo 的值
    } while (x == 0);
    std::cout << "foo: " << x << '\n';
}
 
int main () {
    std::thread first(print_foo); // 线程 first 打印 foo 的值
    std::thread second(set_foo, 10); // 线程 second 设置 foo 的值
    first.join();
    second.join();
    return 0;
}
```

`operator T`

```cpp
operator T() const volatile noexcept;
operator T() const noexcept;
```

与 `load` 功能类似，也是读取被封装的值，`operator T()` 是类型转换操作，默认的内存序是 `std::memory_order_seq_cst`，如果需要指定其他的内存序，应使用 `load()` 函数。

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic
#include <thread>         // std::thread, std::this_thread::yield
 
std::atomic<int> foo(0);
std::atomic<int> bar(0);
 
void set_foo(int x) {
    foo = x;
}
 
void copy_foo_to_bar() {
 
    // 如果 foo == 0，则该线程 yield,
    // 在 foo == 0 时, 实际也是隐含了类型转换操作,
    // 因此也包含了 operator T() const 的调用.
    while (foo == 0) std::this_thread::yield();
 
    // 实际调用了 operator T() const, 将foo 强制转换成 int 类型,
    // 然后调用 operator=().
    bar = static_cast<int>(foo);
}
 
void print_bar() {
    // 如果 bar == 0，则该线程 yield,
    // 在 bar == 0 时, 实际也是隐含了类型转换操作,
    // 因此也包含了 operator T() const 的调用.
    while (bar == 0) std::this_thread::yield();
    std::cout << "bar: " << bar << '\n';
}
 
int main () {
    std::thread first(print_bar);
    std::thread second(set_foo, 10);
    std::thread third(copy_foo_to_bar);
 
    first.join();
    second.join();
    third.join();
    return 0;
}
```

`exchange`

```cpp
T exchange (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T exchange (T val, memory_order sync = memory_order_seq_cst) noexcept;
```

读取并修改被封装的值，`exchange` 会将 `val` 指定的值替换掉之前该原子对象封装的值，并返回之前该原子对象封装的值，整个过程是原子的。`sync` 参数指定内存序。

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic
#include <thread>         // std::thread
#include <vector>         // std::vector
 
std::atomic<bool> ready(false);
std::atomic<bool> winner(false);
 
void count1m (int id) {
    while (!ready) {}                  // wait for the ready signal
    for (int i = 0; i < 1000000; ++i) {}   // go!, count to 1 million
    if (!winner.exchange(true)) { std::cout << "thread #" << id << " won!\n"; }
};

int main () {
    std::vector<std::thread> threads;
    std::cout << "spawning 10 threads that count to 1 million...\n";
    for (int i = 1; i <= 10; ++i) threads.push_back(std::thread(count1m,i));
    ready = true;
    for (auto& th : threads) th.join();
 
    return 0;
}
```

`compare_exchange_weak`

```cpp
// (1)
bool compare_exchange_weak (T& expected, T val,
           memory_order sync = memory_order_seq_cst) volatile noexcept;
bool compare_exchange_weak (T& expected, T val,
           memory_order sync = memory_order_seq_cst) noexcept;
// (2)
bool compare_exchange_weak (T& expected, T val,
           memory_order success, memory_order failure) volatile noexcept;
bool compare_exchange_weak (T& expected, T val,
           memory_order success, memory_order failure) noexcept;
```

比较并交换被封装的值与参数 `expected` 所指定的值是否相等，如果：
- 相等，则用 val 替换原子对象的旧值。
- 不相等，则用原子对象的旧值替换 expected ，因此调用该函数之后，如果被该原子对象封装的值与参数 expected 所指定的值不相等，expected 中的内容就是原子对象的旧值。

该函数通常会读取原子对象封装的值，如果比较为 `true`（即原子对象的值等于 `expected`），则替换原子对象的旧值，但整个操作是原子的，在某个线程读取和修改该原子对象时，另外的线程不能对读取和修改该原子对象。

在第(2)种情况下，内存序的选择取决于比较操作结果，如果比较结果为 `true`（即原子对象的值等于 `expected`），则选择参数 `success` 指定的内存序，否则选择参数 `failure` 所指定的内存序。

注意，该函数直接比较原子对象所封装的值与参数 `expected` 的物理内容，所以某些情况下，对象的比较操作在使用 `operator==()` 判断时相等，但 `compare_exchange_weak` 判断时却可能失败，因为对象底层的物理内容中可能存在位对齐或其他逻辑表示相同但是物理表示不同的值（比如 `true` 和 `2` 或 `3`，它们在逻辑上都表示"真"，但在物理上两者的表示并不相同）。

`weak` 版本允许偶然出乎意料的返回（比如在字段值和期待值一样的时候却返回了false），不过在一些循环算法中，这是可以接受的。通常它比起 `strong` 有更高的性能。

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic
#include <thread>         // std::thread
#include <vector>         // std::vector
 
// a simple global linked list:
struct Node { int value; Node* next; };
std::atomic<Node*> list_head(nullptr);
 
void append(int val) {
    // append an element to the list
    Node* newNode = new Node{val, list_head};
 
    // next is the same as: list_head = newNode, but in a thread-safe way:
    while (!list_head.compare_exchange_weak(newNode->next,newNode)) {}
    // (with newNode->next updated accordingly if some other thread just appended another node)
}

int main () {
    // spawn 10 threads to fill the linked list:
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) threads.push_back(std::thread(append, i));
    for (auto& th : threads) th.join();
 
    // print contents:
    for (Node* it = list_head; it!=nullptr; it=it->next)
        std::cout << ' ' << it->value;
 
    std::cout << '\n';
 
    // cleanup:
    Node* it; while (it=list_head) {list_head=it->next; delete it;}
 
    return 0;
}
```

`compare_exchange_strong`

```cpp
// (1)
bool compare_exchange_strong (T& expected, T val,
           memory_order sync = memory_order_seq_cst) volatile noexcept;
bool compare_exchange_strong (T& expected, T val,
           memory_order sync = memory_order_seq_cst) noexcept;
// (2)
bool compare_exchange_strong (T& expected, T val,
           memory_order success, memory_order failure) volatile noexcept;
bool compare_exchange_strong (T& expected, T val,
           memory_order success, memory_order failure) noexcept;
```

`C++11` 标准库中的 `std::atomic` 针对整形和指针类型的特化版本新增了一些算术运算和逻辑运算操作。具体如下：

`fetch_add`

```cpp
// if T is integral (1)
T fetch_add (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_add (T val, memory_order sync = memory_order_seq_cst) noexcept;
// if T is pointer (2)
T fetch_add (ptrdiff_t val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_add (ptrdiff_t val, memory_order sync = memory_order_seq_cst) noexcept;
```

将原子对象的封装值加 `val`，并返回原子对象的旧值（适用于整形和指针类型的 `std::atomic` 特化版本），整个过程是原子的。`sync` 参数指定内存序。

另外，如果第二个参数不指定（取默认参数 `memory_order_seq_cst`），则 `fetch_add` 相当与 `std::atomic::operator+=`。

`fetch_sub`

```cpp
// if T is integral (1)
T fetch_sub (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_sub (T val, memory_order sync = memory_order_seq_cst) noexcept;
// if T is pointer (2)
T fetch_sub (ptrdiff_t val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_sub (ptrdiff_t val, memory_order sync = memory_order_seq_cst) noexcept;
```

将原子对象的封装值减 `val`，并返回原子对象的旧值，整个过程是原子的。`sync` 参数指定内存序。如果第二个参数不指定，则 `fetch_sub` 相当与 `std::atomic::operator-=`。

`fetch_and`

```cpp
T fetch_and (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_and (T val, memory_order sync = memory_order_seq_cst) noexcept;
```

将原子对象的封装值按位与 `val`，并返回原子对象的旧值。

`fetch_or`

```cpp
T fetch_or (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_or (T val, memory_order sync = memory_order_seq_cst) noexcept;
```

将原子对象的封装值按位或 `val`，并返回原子对象的旧值。

`fetch_xor`

```cpp
T fetch_xor (T val, memory_order sync = memory_order_seq_cst) volatile noexcept;
T fetch_xor (T val, memory_order sync = memory_order_seq_cst) noexcept;
```

将原子对象的封装值按位异或 `val`，并返回原子对象的旧值。

`operator++`

```cpp
// pre-increment (1)
T operator++() volatile noexcept;
T operator++() noexcept;
// post-increment (2)
T operator++ (int) volatile noexcept;
T operator++ (int) noexcept;
```

自增运算符重载， 第一种形式 (1) 返回自增后的值（即前缀++），第二种形式(2) 返回自增前的值（即后缀++）。

`operator--`

自减运算符重载，同自增。

`atomic::operator (comp. assign.)`

复合赋值运算符重载，主要包含以下形式：

```cpp
// T is integral (1)
T operator+= (T val) volatile noexcept;
T operator+= (T val) noexcept;
T operator-= (T val) volatile noexcept;
T operator-= (T val) noexcept;
T operator&= (T val) volatile noexcept;
T operator&= (T val) noexcept;
T operator|= (T val) volatile noexcept;
T operator|= (T val) noexcept;
T operator^= (T val) volatile noexcept;
T operator^= (T val) noexcept;
// if T is pointer (2)
T operator+= (ptrdiff_t val) volatile noexcept;
T operator+= (ptrdiff_t val) noexcept;
T operator-= (ptrdiff_t val) volatile noexcept;
T operator-= (ptrdiff_t val) noexcept;
```