# mutex

`Mutex` 又称互斥量，`C++11` 中与 `Mutex` 相关的类（包括锁类型）和函数都声明在 `<mutex>` 头文件中。

在 `C++11` 中一共提供了四种 `mutex` 类：

- `std::mutex`，最基本的 `Mutex` 类
- `std::recursive_mutex`，递归 `Mutex` 类
- `std::time_mutex`，定时 `Mutex` 类
- `std::recursive_timed_mutex`，定时递归 `Mutex` 类

两种 `Lock` 类：

- `std::lock_guard`，与 `Mutex RAII` 相关，方便线程对互斥量上锁。
- `std::unique_lock`，与 `Mutex RAII` 相关，方便线程对互斥量上锁，但提供了更好的上锁和解锁控

其他类型：

- `std::once_flag`
- `std::adopt_lock_t`
- `std::defer_lock_t`
- `std::try_to_lock_t`

函数：

- `std::try_lock`，尝试同时对多个互斥量上锁
- `std::lock`，可以同时对多个互斥量上锁
- `std::call_once`，如果多个线程需要同时调用某个函数，`call_once` 可以保证多个线程对该函数只调用一次

## 1. `std::mutex`

不论是在 `C` 还是 `C++` 中，进行线程同步的处理流程基本上是一致的，`C++` 的 `mutex` 类提供了相关的 `API` 函数：

`std::mutex` 不允许拷贝构造，也不允许 `move` 拷贝，最初产生的 `mutex` 对象是处于 `unlocked` 状态的。

`lock()` 函数用于给临界区加锁，并且只能有一个线程获得锁的所有权，它有阻塞线程的作用，函数原型如下：

```cpp
void lock();
```

调用线程将锁住该互斥量。线程调用该函数会发生下面 `3` 种情况：
- (1). 如果该互斥量当前没有被锁住，则调用线程将该互斥量锁住，直到调用 `unlock` 之前，该线程一直拥有该锁。
- (2). 如果当前互斥量被其他线程锁住，则当前的调用线程被阻塞住。
- (3). 如果当前互斥量被当前调用线程锁住，则会产生死锁(`deadlock`)

除了使用 `lock()` 还可以使用 `try_lock()` 获取互斥锁的所有权并对互斥锁加锁，函数原型如下：

```cpp
bool try_lock();
```

二者的区别在于 `try_lock()` 不会阻塞线程，`lock()` 会阻塞线程：

线程调用该函数也会出现下面 `3` 种情况：
- (1). 如果当前互斥量没有被其他线程占有，则该线程锁住互斥量，直到该线程调用 `unlock` 释放互斥量
- (2). 如果当前互斥量被其他线程锁住，则当前调用线程返回 `false`，而并不会被阻塞掉
- (3). 如果当前互斥量被当前调用线程锁住，则会产生死锁(`deadlock`)

当互斥锁被锁定之后可以通过 `unlock()` 进行解锁，但是需要注意的是只有拥有互斥锁所有权的线程也就是对互斥锁上锁的线程才能将其解锁，其它线程是没有权限做这件事情的。该函数的函数原型如下：

```cpp
void unlock();
```

通过介绍以上三个函数，使用互斥锁进行线程同步的大致思路差不多就能搞清楚了，主要分为以下几步：

- 找到多个线程操作的共享资源（全局变量、堆内存、类成员变量等），也可以称之为临界资源
- 找到和共享资源有关的上下文代码，也就是临界区（下图中的黄色代码部分）
- 在临界区的上边调用互斥锁类的 `lock()` 方法
- 在临界区的下边调用互斥锁的 `unlock()` 方法

线程同步的目的是让多线程按照顺序依次执行临界区代码，这样做线程对共享资源的访问就从并行访问变为了线性访问，访问效率降低了，但是保证了数据的正确性。

当线程对互斥锁对象加锁，并且执行完临界区代码之后，一定要使用这个线程对互斥锁解锁，否则最终会造成线程的死锁。死锁之后当前应用程序中的所有线程都会被阻塞，并且阻塞无法解除，应用程序也无法继续运行。

## 2. `std::recursive_mutex`

递归互斥锁 `std::recursive_mutex` 允许同一线程多次获得互斥锁，可以用来解决**同一线程**需要多次获取互斥量时死锁的问题，在下面的例子中使用独占非递归互斥量会发生死锁：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

struct Calculate {
    Calculate() : m_i(6) {}

    void mul(int x) {
        lock_guard<mutex> locker(m_mutex);
        m_i *= x;
    }

    void div(int x) {
        lock_guard<mutex> locker(m_mutex);
        m_i /= x;
    }

    void both(int x, int y) {
        lock_guard<mutex> locker(m_mutex);
        mul(x);
        div(y);
    }

    int m_i;
    mutex m_mutex;
};

int main() {
    Calculate cal;
    cal.both(6, 3);
    return 0;
}
```

上面的程序中执行了 `cal.both(6, 3);` 调用之后，程序就会发生死锁，在 `both()` 中已经对互斥锁加锁了，继续调用 `mul()` 函数，已经得到互斥锁所有权的线程再次获取这个互斥锁的所有权就会造成死锁（在 `C++` 中程序会异常退出，使用 `C` 库函数会导致这个互斥锁永远无法被解锁，最终阻塞所有的线程）。要解决这个死锁的问题，一个简单的办法就是使用递归互斥锁 `std::recursive_mutex`，它允许一个线程多次获得互斥锁的所有权。修改之后的代码如下：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

struct Calculate {
    Calculate() : m_i(6) {}

    void mul(int x) {
        lock_guard<recursive_mutex> locker(m_mutex);
        m_i *= x;
    }

    void div(int x) {
        lock_guard<recursive_mutex> locker(m_mutex);
        m_i /= x;
    }

    void both(int x, int y) {
        lock_guard<recursive_mutex> locker(m_mutex);
        mul(x);
        div(y);
    }

    int m_i;
    recursive_mutex m_mutex;
};

int main() {
    Calculate cal;
    cal.both(6, 3);
    cout << "cal.m_i = " << cal.m_i << endl;
    return 0;
}
```

虽然递归互斥锁可以解决同一个互斥锁频繁获取互斥锁资源的问题，但是还是建议少用，主要原因如下：

- 使用递归互斥锁的场景往往都是可以简化的，使用递归互斥锁很容易放纵复杂逻辑的产生，从而导致 `bug` 的产生
- 递归互斥锁比非递归互斥锁效率要低一些
- 递归互斥锁虽然允许同一个线程多次获得同一个互斥锁的所有权，但最大次数并未具体说明，一旦超过一定的次数，就会抛出 `std::system` 错误

## 3. `std::timed_mutex`

`std::timed_mutex` 是超时独占互斥锁，主要是在获取互斥锁资源时增加了超时等待功能，因为不知道获取锁资源需要等待多长时间，为了保证不一直等待下去，设置了一个超时时长，超时后线程就可以解除阻塞去做其他事情了。

`std::timed_mutex` 比 `std::_mutex` 多了两个成员函数：`try_lock_for()` 和 `try_lock_until()`：

```cpp
void lock();
bool try_lock();
void unlock();

// std::timed_mutex比std::_mutex多出的两个成员函数
template <class Rep, class Period>
  bool try_lock_for (const chrono::duration<Rep,Period>& rel_time);

template <class Clock, class Duration>
  bool try_lock_until (const chrono::time_point<Clock,Duration>& abs_time);
```

- `try_lock_for` 函数接受一个时间范围，表示在这一段时间范围之内线程如果没有获得锁则被阻塞住（与` std::mutex` 的 `try_lock()` 不同，`try_lock` 如果被调用时没有获得锁则直接返回 `false`），如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时（即在指定时间内还是没有获得锁），则返回 `false`
- `try_lock_until` 函数则接受一个时间点作为参数，在指定时间点未到来之前线程如果没有获得锁则被阻塞住，如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时（即在指定时间内还是没有获得锁），则返回 `false`

示例：

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

timed_mutex g_mutex;

void work() {
    chrono::seconds timeout(1);
    while (true) {
        // 通过阻塞一定的时长来争取得到互斥锁所有权
        if (g_mutex.try_lock_for(timeout)) {
            cout << "当前线程ID: " << this_thread::get_id() 
                << ", 得到互斥锁所有权..." << endl;
            // 模拟处理任务用了一定的时长
            this_thread::sleep_for(chrono::seconds(10));
            // 互斥锁解锁
            g_mutex.unlock();
            break;
        } else {
            cout << "当前线程ID: " << this_thread::get_id() 
                << ", 没有得到互斥锁所有权..." << endl;
            // 模拟处理其他任务用了一定的时长
            this_thread::sleep_for(chrono::milliseconds(50));
        }
    }
}

int main() {
    thread t1(work);
    thread t2(work);

    t1.join();
    t2.join();

    return 0;
}
```

运行结果：

```cpp
当前线程ID: 140133199447808, 得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 没有得到互斥锁所有权...
当前线程ID: 140133191055104, 得到互斥锁所有权...
```

在上面的例子中，通过一个 `while` 循环不停的去获取超时互斥锁的所有权，如果得不到就阻塞 `1` 秒钟，`1` 秒之后如果还是得不到就阻塞 `50` 毫秒，然后再次继续尝试，直到获得互斥锁的所有权，跳出循环体。

## 4. `std::recursive_timed_mutex`

关于递归超时互斥锁 `std::recursive_timed_mutex` 的使用方式和 `std::timed_mutex` 是一样的，只不过它可以允许一个线程多次获得互斥锁所有权，而 `std::timed_mutex` 只允许线程获取一次互斥锁所有权。另外，递归超时互斥锁 `std::recursive_timed_mutex` 也拥有和 `std::recursive_mutex` 一样的弊端，不建议频繁使用。

## 5. `std::lock_guard`

`C++11` 标准为我们提供了两种基本的锁类型，
`std::lock_guard`，与 `std::unique_lock`。

另外还提供了几个与锁类型相关的 `Tag` 类，分别如下:

- `std::adopt_lock_t`，一个空的标记类，定义如下：
    - `struct adopt_lock_t {};`
    - 该类型的常量对象 `adopt_lock`（`adopt_lock` 是一个常量对象，定义如下：
      - `constexpr adopt_lock_t adopt_lock {};`
    - 通常作为参数传入给 `unique_lock` 或 `lock_guard` 的构造函数

- `std::defer_lock_t`，一个空的标记类，定义如下： 
  - `struct defer_lock_t {};`
  - 该类型的常量对象 `defer_lock`（`defer_lock` 是一个常量对象，定义如下：
    - `constexpr defer_lock_t defer_lock {};`
  - 通常作为参数传入给 `unique_lock` 或 `lock_guard` 的构造函数

- `std::try_to_lock_t`，一个空的标记类，定义如下：
  - `struct try_to_lock_t {};`
  - 该类型的常量对象` try_to_lock`（`try_to_lock` 是一个常量对象，定义如下：
    - `constexpr try_to_lock_t try_to_lock {};`
  - 通常作为参数传入给 `unique_lock` 或 `lock_guard` 的构造函数

`std::lock_gurad` 是 `C++11` 中定义的模板类。定义如下：

```cpp
template <class Mutex> class lock_guard;
```

`lock_guard` 对象通常用于管理某个锁对象，因此与 `Mutex RAII` 相关，方便线程对互斥量上锁，即在某个 `lock_guard` 对象的声明周期内，它所管理的锁对象会一直保持上锁状态；而 `lock_guard` 的生命周期结束之后，它所管理的锁对象会被解锁，类似 `shared_ptr` 等智能指针管理动态分配的内存资源。

模板参数 `Mutex` 代表互斥量类型，例如 `std::mutex` 类型。

在 `lock_guard` 对象构造时，传入的 `Mutex` 对象会被当前线程锁住。在 `lock_guard` 对象被析构时，它所管理的 `Mutex` 对象会自动解锁，由于不需要程序员手动调用 `lock` 和 `unlock` 对 `Mutex` 进行上锁和解锁操作，因此这也是最简单安全的上锁和解锁方式，尤其是在程序抛出异常后先前已被上锁的 `Mutex` 对象可以正确进行解锁操作，极大地简化了程序员编写与 `Mutex` 相关的异常处理代码。

值得注意的是，`lock_guard` 对象并不负责管理 `Mutex` 对象的生命周期，`lock_guard` 对象只是简化了 `Mutex` 对象的上锁和解锁操作，方便线程对互斥量上锁，即在某个 `lock_guard` 对象的声明周期内，它所管理的锁对象会一直保持上锁状态；而 `lock_guard` 的生命周期结束之后，它所管理的锁对象会被解锁。

`std::lock_guard` 构造函数：

```cpp
// locking (1)
explicit lock_guard (mutex_type& m);
// adopting (2)
lock_guard (mutex_type& m, adopt_lock_t tag);
// copy [deleted](3)
lock_guard (const lock_guard&) = delete;
```

- `locking` 初始化
  - `lock_guard` 对象管理 `Mutex` 对象 `m`，并在构造时对 `m` 进行上锁（调用 `m.lock()`）
- `adopting` 初始化
  - `lock_guard` 对象管理 `Mutex` 对象 `m`，与 `locking` 初始化 (1) 不同的是，`Mutex` 对象 `m` 已被当前线程锁住
- 拷贝构造
- `lock_guard` 对象的拷贝构造和移动构造(`move construction`) 均被禁用，因此 `lock_guard` 对象不可被拷贝构造或移动构造

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard, std::adopt_lock

std::mutex mtx;           // mutex for critical section

void print_thread_id (int id) {
    mtx.lock();
    std::lock_guard<std::mutex> lck(mtx, std::adopt_lock);
    std::cout << "thread #" << id << '\n';
}

int main () {
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_thread_id,i+1);

    for (auto& th : threads) th.join();

    return 0;
}
```

在 `print_thread_id` 中，我们首先对 `mtx` 进行上锁操作（`mtx.lock();`），然后用 `mtx` 对象构造一个 `lock_guard` 对象（`std::lock_guard<std::mutex> lck(mtx, std::adopt_lock);`），注意此时 `Tag` 参数为 `std::adopt_lock`，表明当前线程已经获得了锁，此后 `mtx` 对象的解锁操作交由 `lock_guard` 对象 `lck` 来管理，在 `lck` 的生命周期结束之后，`mtx` 对象会自动解锁。

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard
#include <stdexcept>      // std::logic_error

std::mutex mtx;

void print_even (int x) {
    if (x%2==0) std::cout << x << " is even\n";
    else throw (std::logic_error("not even"));
}

void print_thread_id (int id) {
    try {
        // using a local lock_guard to lock mtx guarantees unlocking on destruction / exception:
        std::lock_guard<std::mutex> lck (mtx);
        print_even(id);
    }
    catch (std::logic_error&) {
        std::cout << "[exception caught]\n";
    }
}

int main () {
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_thread_id, i+1);

    for (auto& th : threads) th.join();

    return 0;
}
```

## 6. `std::unique_lock`

但是 `lock_guard` 最大的缺点也是简单，没有给程序员提供足够的灵活度，因此，`C++11` 标准中定义了另外一个与 `Mutex RAII` 相关类 `unique_lock`，该类与 `lock_guard` 类相似，也很方便线程对互斥量上锁，但它提供了更好的上锁和解锁控制。

顾名思义，`unique_lock` 对象以独占所有权的方式管理 `mutex` 对象的上锁和解锁操作，所谓独占所有权，就是没有其他的 `unique_lock` 对象同时拥有某个 `mutex` 对象的所有权。

在构造（或移动赋值）时，`unique_lock` 对象需要传递一个 `Mutex` 对象作为它的参数，新创建的 `unique_lock` 对象负责传入的 `Mutex` 对象的上锁和解锁操作。

`std::unique_lock` 对象也能保证在其自身析构时它所管理的 `Mutex` 对象能够被正确地解锁（即使没有显式地调用 `unlock` 函数）。因此，和 `lock_guard` 一样，这也是一种简单而又安全的上锁和解锁方式，尤其是在程序抛出异常后先前已被上锁的 `Mutex` 对象可以正确进行解锁操作，极大地简化了程序员编写与 `Mutex` 相关的异常处理代码。

值得注意的是，`unique_lock` 对象同样也不负责管理 `Mutex` `对象的生命周期，unique_lock` 对象只是简化了 `Mutex` 对象的上锁和解锁操作，方便线程对互斥量上锁，即在某个 `unique_lock` 对象的声明周期内，它所管理的锁对象会一直保持上锁状态；而 `unique_lock` 的生命周期结束之后，它所管理的锁对象会被解锁，这一点和 `lock_guard` 类似，但 `unique_lock` 给程序员提供了更多的自由。

另外，与 `lock_guard` 一样，模板参数 `Mutex` 代表互斥量类型，例如 `std::mutex` 类型，它应该是一个基本的 `BasicLockable` 类型。

`std::unique_lock` 构造函数：

`std::unique_lock` 的构造函数的数目相对来说比 `std::lock_guard` 多，其中一方面也是因为 `std::unique_lock` 更加灵活，从而在构造 `std::unique_lock` 对象时可以接受额外的参数。总地来说，`std::unique_lock` 构造函数如下：

```cpp
// default (1)
unique_lock() noexcept;
// locking (2)
explicit unique_lock(mutex_type& m);
// try-locking (3)
unique_lock(mutex_type& m, try_to_lock_t tag);
// deferred (4)
unique_lock(mutex_type& m, defer_lock_t tag) noexcept;
// adopting (5)
unique_lock(mutex_type& m, adopt_lock_t tag);
// locking for (6)
template <class Rep, class Period>
unique_lock(mutex_type& m, const chrono::duration<Rep,Period>& rel_time);
// locking until (7)
template <class Clock, class Duration>
unique_lock(mutex_type& m, const chrono::time_point<Clock,Duration>& abs_time);
// copy [deleted] (8)
unique_lock(const unique_lock&) = delete;
// move (9)
unique_lock(unique_lock&& x);
```

下面我们来分别介绍以上各个构造函数：

- (1) 默认构造函数
  - 新创建的 `unique_lock` 对象不管理任何 `Mutex` 对象
- (2) `locking` 初始化
  - 新创建的 `unique_lock` 对象管理 `Mutex` 对象 `m`，并尝试调用 `m.lock()` 对 `Mutex` 对象进行上锁，如果此时另外某个 `unique_lock` 对象已经管理了该 Mutex 对象 `m`，则当前线程将会被阻塞。
- (3) `try-locking` 初始化
  - 新创建的 `unique_lock` 对象管理 `Mutex` 对象 `m`，并尝试调用 `m.try_lock()` 对 `Mutex` 对象进行上锁，但如果上锁不成功，并不会阻塞当前线程
- (4) `deferred` 初始化
  - 新创建的 `unique_lock` 对象管理 `Mutex` 对象 `m`，但是在初始化的时候并不锁住 `Mutex` 对象。`m` 应该是一个没有当前线程锁住的 `Mutex` 对象
- (5) `adopting` 初始化
  - 新创建的 `unique_lock` 对象管理 `Mutex` 对象 `m`，`m` 应该是一个已经被当前线程锁住的 Mutex 对象。（并且当前新创建的 `unique_lock` 对象拥有对锁(`Lock`)的所有权）。
- (6) `locking` 一段时间(`duration`)
  - 新创建的 `unique_lock` 对象管理 `Mutex` 对象 `m`，并试图通过调用 `m.try_lock_for(rel_time)` 来锁住 `Mutex` 对象一段时间(`rel_time`)
- (7) `locking` 直到某个时间点(`time point`)
  - 新创建的 `unique_lock` `对象管理` Mutex 对象 `m`，并试图通过调用 `m.try_lock_until(abs_time)` 来在某个时间点(`abs_time`)之前锁住 `Mutex` 对象
- (8) 拷贝构造 `[被禁用]`
  - `unique_lock` 对象不能被拷贝构造
- (9) 移动(`move`)构造
  - 新创建的 `unique_lock` 对象获得了由 `x` 所管理的 `Mutex` 对象的所有权(包括当前 `Mutex` 的状态)。调用 `move` 构造之后，`x` 对象如同通过默认构造函数所创建的，就不再管理任何 `Mutex` 对象了
  
综上所述，由 (2) 和 (5) 创建的 `unique_lock` 对象通常拥有 `Mutex` 对象的锁。而通过 (1) 和 (4) 创建的则不会拥有锁。通过 (3)，(6) 和 (7) 创建的 `unique_lock` 对象，则在 `lock` 成功时获得锁。

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock, std::unique_lock
                          // std::adopt_lock, std::defer_lock
std::mutex foo,bar;

void task_a () {
    std::lock (foo,bar);         // simultaneous lock (prevents deadlock)
    std::unique_lock<std::mutex> lck1 (foo,std::adopt_lock);
    std::unique_lock<std::mutex> lck2 (bar,std::adopt_lock);
    std::cout << "task a\n";
    // (unlocked automatically on destruction of lck1 and lck2)
}

void task_b () {
    // foo.lock(); bar.lock(); // replaced by:
    std::unique_lock<std::mutex> lck1, lck2;
    lck1 = std::unique_lock<std::mutex>(bar,std::defer_lock);
    lck2 = std::unique_lock<std::mutex>(foo,std::defer_lock);
    std::lock (lck1,lck2);       // simultaneous lock (prevents deadlock)
    std::cout << "task b\n";
    // (unlocked automatically on destruction of lck1 and lck2)
}


int main () {
    std::thread th1 (task_a);
    std::thread th2 (task_b);

    th1.join();
    th2.join();

    return 0;
}
```

`std::unique_lock` 移动赋值操作：

`std::unique_lock` 支持移动赋值，但是普通的赋值被禁用了。

```cpp
// move (1)
unique_lock& operator= (unique_lock&& x) noexcept;
// copy [deleted] (2)
unique_lock& operator= (const unique_lock&) = delete;
```

- 移动赋值之后，由 `x` 所管理的 `Mutex` 对象及其状态将会被新的 `std::unique_lock` 对象取代
- 如果被赋值的对象之前已经获得了它所管理的 `Mutex` 对象的锁，则在移动赋值之前会调用 `unlock` 函数释放它所占有的锁
- 调用移动赋值之后，`x` 对象如同通过默认构造函数所创建的，也就不再管理任何 `Mutex` 对象了

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::unique_lock

std::mutex mtx;           // mutex for critical section

void print_fifty (char c) {
    std::unique_lock<std::mutex> lck;         // default-constructed
    lck = std::unique_lock<std::mutex>(mtx);  // move-assigned
    for (int i=0; i<50; ++i) { std::cout << c; }
    std::cout << '\n';
}

int main () {
    std::thread th1 (print_fifty,'*');
    std::thread th2 (print_fifty,'$');

    th1.join();
    th2.join();

    return 0;
}
```
`std::unique_lock` 主要成员函数：

- 上锁/解锁操作：`lock`，`try_lock`，`try_lock_for`，`try_lock_until` 和 `unlock`
- 修改操作：移动赋值，交换(`swap`)（与另一个 `std::unique_lock` 对象交换它们所管理的 `Mutex` 对象的所有权），释放(`release`)（返回指向它所管理的 `Mutex` 对象的指针，并释放所有权）
- 获取属性操作：`owns_lock`（返回当前 `std::unique_lock` 对象是否获得了锁）、`operator bool()`（与 `owns_lock` 功能相同，返回当前 `std::unique_lock` 对象是否获得了锁）、`mutex`（返回当前 `std::unique_lock` 对象所管理的 `Mutex` 对象的指针）

`std::unique_lock::lock`：

上锁操作，调用它所管理的 `Mutex` 对象的 `lock` 函数。如果在调用  `Mutex` 对象的 `lock` 函数时该 `Mutex` 对象已被另一线程锁住，则当前线程会被阻塞，直到它获得了锁。

该函数返回时，当前的 `unique_lock` 对象便拥有了它所管理的 `Mutex` 对象的锁。如果上锁操作失败，则抛出 `system_error` 异常。

```cpp
void print_thread_id (int id) {
    std::unique_lock<std::mutex> lck (mtx,std::defer_lock);
    // critical section (exclusive access to std::cout signaled by locking lck):
    lck.lock();
    std::cout << "thread #" << id << '\n';
    lck.unlock();
}
```

`std::unique_lock::try_lock`

上锁操作，调用它所管理的 `Mutex` 对象的 `try_lock` 函数，如果上锁成功，则返回 `true`，否则返回 `false`。

```cpp
std::mutex mtx;           // mutex for critical section

void print_star () {
    std::unique_lock<std::mutex> lck(mtx,std::defer_lock);
    // print '*' if successfully locked, 'x' otherwise: 
    if (lck.try_lock())
        std::cout << '*';
    else                    
        std::cout << 'x';
}
```

`std::unique_lock::try_lock_for`

上锁操作，调用它所管理的 `Mutex` 对象的 `try_lock_for` 函数，如果上锁成功，则返回 `true`，否则返回 `false`。

```cpp
void fireworks () {
    std::unique_lock<std::timed_mutex> lck(mtx,std::defer_lock);
    // waiting to get a lock: each thread prints "-" every 200ms:
    while (!lck.try_lock_for(std::chrono::milliseconds(200))) {
        std::cout << "-";
  }
    // got a lock! - wait for 1s, then this thread prints "*"
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << "*\n";
}
```

`std::unique_lock::try_lock_until`

上锁操作，调用它所管理的 `Mutex` 对象的 `try_lock_for` 函数，如果上锁成功，则返回 `true`，否则返回 `false`。

```cpp
void fireworks () {
    std::unique_lock<std::timed_mutex> lck(mtx,std::defer_lock);
    // waiting to get a lock: each thread prints "-" every 200ms:
    while (!lck.try_lock_for(std::chrono::milliseconds(200))) {
        std::cout << "-";
  }
    // got a lock! - wait for 1s, then this thread prints "*"
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    std::cout << "*\n";
}
```

`std::unique_lock::unlock`

解锁操作，调用它所管理的 `Mutex` 对象的 `unlock` 函数。

```cpp
void print_thread_id (int id) {
    std::unique_lock<std::mutex> lck (mtx,std::defer_lock);
    // critical section (exclusive access to std::cout signaled by locking lck):
    lck.lock();
    std::cout << "thread #" << id << '\n';
    lck.unlock();
}
```

`std::unique_lock::release`

返回指向它所管理的 `Mutex` 对象的指针，并释放所有权。

```cpp
void print_count_and_unlock (std::mutex* p_mtx) {
    std::cout << "count: " << count << '\n';
    p_mtx->unlock();
}

void task() {
    std::unique_lock<std::mutex> lck(mtx);
    ++count;
    print_count_and_unlock(lck.release());
}
```

`std::unique_lock::owns_lock`

返回当前 `std::unique_lock` 对象是否获得了锁。

```cpp
void print_star () {
    std::unique_lock<std::mutex> lck(mtx,std::try_to_lock);
    // print '*' if successfully locked, 'x' otherwise: 
    if (lck.owns_lock())
    std::cout << '*';
    else                    
        std::cout << 'x';
}
```

`std::unique_lock::operator bool()`

```cpp
void print_star () {
    std::unique_lock<std::mutex> lck(mtx,std::try_to_lock);
    // print '*' if successfully locked, 'x' otherwise: 
    if (lck)
        std::cout << '*';
    else                    
        std::cout << 'x';
}
```

`std::unique_lock::mutex`

返回当前 `std::unique_lock` 对象所管理的 `Mutex` 对象的指针。

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::unique_lock, std::defer_lock

class MyMutex : public std::mutex {
    int _id;
public:
    MyMutex (int id) : _id(id) {}
    int id() {return _id;}
};

MyMutex mtx (101);

void print_ids (int id) {
    std::unique_lock<MyMutex> lck (mtx);
    std::cout << "thread #" << id << " locked mutex " << lck.mutex()->id() << '\n';
}

int main () {
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_ids,i+1);

    for (auto& th : threads) th.join();

    return 0;
}
```