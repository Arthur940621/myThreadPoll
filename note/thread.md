# thread

`C++11` 中提供的线程类叫做 `std::thread`，基于这个类创建一个新的线程非常的简单，只需要提供线程函数或者函数对象即可，并且可以同时指定线程函数的参数。我们首先来了解一下这个类提供的一些常用 `API`：

## 1. 构造函数

```cpp
// default (1)
thread() noexcept;
// initialization (2)
template <class Fn, class... Args>
explicit thread (Fn&& fn, Args&&... args);
// copy [deleted] (3)
thread (const thread&) = delete;
// move (4)
thread (thread&& x) noexcept;
```

- (1). 默认构造函数，创建一个空的 thread 执行对象。
- (2). 初始化构造函数，创建一个 thread对象，该 thread对象可被 joinable，新产生的线程会调用 fn 函数，该函数的参数由 args 给出。
  - 任务函数 fn 的可选类型有很多，具体如下：
    - 普通函数，类成员函数，匿名函数，仿函数（这些都是可调用对象类型）
    - 可以是可调用对象包装器类型，也可以是使用绑定器绑定之后得到的类型（仿函数）
- (3). 拷贝构造函数(被禁用)，意味着 thread 不可被拷贝构造。
- (4). move 构造函数，move 构造函数，调用成功之后 x 不代表任何 thread 执行对象。

```cpp
#include <iostream>
#include <thread>
#include <chrono>
 
void f1(int n) {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread " << n << " executing\n";
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
void f2(int& n) {
    for (int i = 0; i < 5; ++i) {
        std::cout << "Thread 2 executing\n";
        ++n;
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}
 
int main() {
    int n = 0;
    std::thread t1; // t1 is not a thread
    std::thread t2(f1, n + 1); // pass by value
    std::thread t3(f2, std::ref(n)); // pass by reference
    std::thread t4(std::move(t3)); // t4 is now running f2(). t3 is no longer a thread
    t2.join();
    t4.join();
    std::cout << "Final value of n is " << n << '\n';
}
```

## 2. `move` 赋值操作

```cpp
// move (1)
thread& operator= (thread&& rhs) noexcept;
// copy [deleted] (2)
thread& operator= (const thread&) = delete;
```

- (1). move 赋值操作，如果当前对象不可 joinable，需要传递一个右值引用(rhs)给 move 赋值操作；如果当前对象可被 joinable，则 terminate() 报错。
- (2). 拷贝赋值操作被禁用，thread 对象不可被拷贝。

```cpp
#include <stdio.h>
#include <stdlib.h>

#include <chrono>    // std::chrono::seconds
#include <iostream>  // std::cout
#include <thread>    // std::thread, std::this_thread::sleep_for

void thread_task(int n) {
    std::this_thread::sleep_for(std::chrono::seconds(n));
    std::cout << "hello thread "
        << std::this_thread::get_id()
        << " paused " << n << " seconds" << std::endl;
}

int main() {
    std::thread threads[5];
    std::cout << "Spawning 5 threads...\n";
    for (int i = 0; i < 5; i++) {
        threads[i] = std::thread(thread_task, i + 1);
    }
    std::cout << "Done spawning threads! Now wait for them to join\n";
    for (auto& t: threads) {
        t.join();
    }
    std::cout << "All threads joined.\n";

    return 0;
}
```

## 3. 其他成员函数

### 3.1 get_id()

应用程序启动之后默认只有一个线程，这个线程一般称之为主线程或父线程，通过线程类创建出的线程一般称之为子线程，每个被创建出的线程实例都对应一个线程 `ID`，这个 `ID` 是唯一的，可以通过这个 `ID` 来区分和识别各个已经存在的线程实例，这个获取线程 `ID` 的函数叫做 `get_id()`，函数原型如下：

```cpp
std::thread::id get_id() const noexcept;
```

### 3.2 `join()`

`join()` 字面意思是连接一个线程，意味着主动地等待线程的终止（线程阻塞）。在某个线程中通过子线程对象调用 `join()` 函数，调用这个函数的线程被阻塞，但是子线程对象中的任务函数会继续执行，当任务执行完毕之后 `join()` 会清理当前子线程中的相关资源然后返回，同时，调用该函数的线程解除阻塞继续向下执行。

再次强调，我们一定要搞清楚这个函数阻塞的是哪一个线程，函数在哪个线程中被执行，那么函数就阻塞哪个线程。该函数的函数原型如下：

```cpp
void join();
```

### 3.3 `detach()`

`detach()` 函数的作用是进行线程分离，分离主线程和创建出的子线程。在线程分离之后，主线程退出也会一并销毁创建出的所有子线程，在主线程退出之前，它可以脱离主线程继续独立的运行，任务执行完毕之后，这个子线程会自动释放自己占用的系统资源。该函数函数原型如下：

```cpp
void detach();
```

线程分离函数没有参数也没有返回值，只需要在线程成功之后，通过线程对象调用该函数即可。

线程分离函数 `detach()` 不会阻塞线程，子线程和主线程分离之后，在主线程中就不能再对这个子线程做任何控制了，比如：通过 `join()` 阻塞主线程等待子线程中的任务执行完毕，或者调用 `get_id()` 获取子线程的线程 `ID`。

### 3.5 `joinable()`
`joinable()` 函数用于判断主线程和子线程是否处理关联（连接）状态，一般情况下，二者之间的关系处于关联状态，该函数返回一个布尔类型：

- 返回值为 true：主线程和子线程之间有关联（连接）关系。
- 返回值为 false：主线程和子线程之间没有关联（连接）关系。

```cpp
bool joinable() const noexcept;
```

- 在创建的子线程对象的时候，如果没有指定任务函数，那么子线程不会启动，主线程和这个子线程也不会进行连接
- 在创建的子线程对象的时候，如果指定了任务函数，子线程启动并执行任务，主线程和这个子线程自动连接成功
- 子线程调用了 detach() 函数之后，父子线程分离，同时二者的连接断开，调用 joinable() 返回 false
- 在子线程调用了 join() 函数，子线程中的任务函数继续执行，直到任务处理完毕，这时 join() 会清理（回收）当前子线程的相关资源，所以这个子线程和主线程的连接也就断开了，因此，调用 join() 之后再调用 joinable() 会返回 false。


### 3.6 `swap`

```cpp
void swap(thread& t) noexcept
```

交换两个线程。

### 3.7 静态函数

`thread` 线程类还提供了一个静态方法，用于获取当前计算机的 `CPU` 核心数，根据这个结果在程序中创建出数量相等的线程，每个线程独自占有一个 `CPU` 核心，这些线程就不用分时复用 `CPU` 时间片，此时程序的并发效率是最高的。

```cpp
static unsigned hardware_concurrency() noexcept;
```

示例代码如下：

```cpp
#include <iostream>
#include <thread>
using namespace std;

int main() {
    int num = thread::hardware_concurrency();
    cout << "CPU number: " << num << endl;
}
```

## 4. 命名空间 - `this_thread`

在 `C++11` 中不仅添加了线程类，还添加了一个关于线程的命名空间 `std::this_thread`，在这个命名空间中提供了四个公共的成员函数，通过这些成员函数就可以对当前线程进行相关的操作了。

调用命名空间 `std::this_thread` 中的 `get_id()` 方法可以得到当前线程的线程 ID，函数原型如下：

```cpp
thread::id get_id() noexcept;
```

### 4.1 `get_id()`
 
调用命名空间 `std::this_thread` 中的 `get_id()` 方法可以得到当前线程的线程 `ID`，函数原型如下：

```cpp
thread::id get_id() noexcept;
```

关于函数使用对应的示例代码如下：

```cpp
#include <iostream>
#include <thread>
using namespace std;

void func() {
    cout << "子线程: " << this_thread::get_id() << endl;
}

int main() {
    cout << "主线程: " << this_thread::get_id() << endl;
    thread t(func);
    t.join();
}
```

程序启动，开始执行 `main()` 函数，此时只有一个线程也就是主线程。当创建了子线程对象 `t` 之后，指定的函数 `func()` 会在子线程中执行，这时通过调用 `this_thread::get_id()` 就可以得到当前线程的线程 `ID` 了。

### 4.2 `sleep_for()`

进程被创建后一共有五种状态，同样地线程被创建后也有这五种状态：创建态，就绪态，运行态，阻塞态（挂起态），退出态（终止态）。

线程和进程的执行有很多相似之处，在计算机中启动的多个线程都需要占用 `CPU` 资源，但是 `CPU` 的个数是有限的并且每个 `CPU` 在同一时间点不能同时处理多个任务。为了能够实现并发处理，多个线程都是分时复用 `CPU` 时间片，快速的交替处理各个线程中的任务。因此多个线程之间需要争抢 `CPU` 时间片，抢到了就执行，抢不到则无法执行（因为默认所有的线程优先级都相同，内核也会从中调度，不会出现某个线程永远抢不到 `CPU` 时间片的情况）。

命名空间 `this_thread` 中提供了一个休眠函数 `sleep_for()`，调用这个函数的线程会马上从运行态变成阻塞态并在这种状态下休眠一定的时长，因为阻塞态的线程已经让出了 `CPU` 资源，代码也不会被执行，所以线程休眠过程中对 `CPU` 来说没有任何负担。这个函数是函数原型如下，参数需要指定一个休眠时长，是一个时间段：

```cpp
template <class Rep, class Period>
  void sleep_for (const chrono::duration<Rep,Period>& rel_time);
```

示例程序如下：

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

void func() {
    for (int i = 0; i < 10; ++i) {
        this_thread::sleep_for(chrono::seconds(1));
        cout << "子线程: " << this_thread::get_id() << ", i = " << i << endl;
    }
}

int main() {
    thread t(func);
    t.join();
    return 0;
}
```

在 `func()` 函数的 `for` 循环中使用了 `this_thread::sleep_for(chrono::seconds(1));` 之后，每循环一次程序都会阻塞 `1` 秒钟，也就是说每隔 `1` 秒才会进行一次输出。需要注意的是：程序休眠完成之后，会从阻塞态重新变成就绪态，就绪态的线程需要再次争抢 `CPU` 时间片，抢到之后才会变成运行态，这时候程序才会继续向下运行。

### 4.3 `sleep_until()`

命名空间 `this_thread` 中提供了另一个休眠函数 `sleep_until()`，和 `sleep_for()` 不同的是它的参数类型不一样。

- `sleep_until()`：指定线程阻塞到某一个指定的时间点 `time_point` 类型，之后解除阻塞

- `sleep_for()`：指定线程阻塞一定的时间长度 duration 类型，之后解除阻塞

该函数的函数原型如下：

```cpp
template <class Clock, class Duration>
  void sleep_until (const chrono::time_point<Clock,Duration>& abs_time);
```

示例程序如下：

```cpp
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

void func() {
    for (int i = 0; i < 10; ++i) {
        // 获取当前系统时间点
        auto now = chrono::system_clock::now();
        // 时间间隔为2s
        chrono::seconds sec(2);
        // 当前时间点之后休眠两秒
        this_thread::sleep_until(now + sec);
        cout << "子线程: " << this_thread::get_id() << ", i = " << i << endl;
    }
}

int main() {
    thread t(func);
    t.join();
    return 0;
}
```

`sleep_until() `和 `sleep_for()` 函数的功能是一样的，只不过前者是基于时间点去阻塞线程，后者是基于时间段去阻塞线程，项目开发过程中根据实际情况选择最优的解决方案即可。

### 4.4 `yield()`

命名空间 `this_thread` 中的函数 `yield()`，在线程中调用这个函数之后，处于运行态的线程会主动让出自己已经抢到的 `CPU` 时间片，最终变为就绪态，这样其它的线程就有更大的概率能够抢到 `CPU` 时间片了。使用这个函数的时候需要注意一点，线程调用了 `yield()` 之后会主动放弃 `CPU` 资源，但是这个变为就绪态的线程会马上参与到下一轮 `CPU` 的抢夺战中，不排除它能继续抢到 `CPU` 时间片的情况，这是概率问题。

```cpp
void yield() noexcept;
```

函数对应的示例程序如下：

```cpp
#include <iostream>
#include <thread>
using namespace std;

void func() {
    for (int i = 0; i < 100000000000; ++i) {
        cout << "子线程: " << this_thread::get_id() << ", i = " << i << endl;
        this_thread::yield();
    }
}

int main() {
    thread t(func);
    thread t1(func);
    t.join();
    t1.join();
    return 0;
}
```

在上面的程序中，执行 `func()` 中的 `for` 循环会占用大量的时间，在极端情况下，如果当前线程占用 `CPU` 资源不释放就会导致其他线程中的任务无法被处理，或者该线程每次都能抢到 `CPU` 时间片，导致其他线程中的任务没有机会被执行。解决方案就是每执行一次循环，让该线程主动放弃 `CPU` 资源，重新和其他线程再次抢夺 `CPU` 时间片，如果其他线程抢到了 `CPU` 时间片就可以执行相应的任务了。

- `std::this_thread::yield()` 的目的是避免一个线程长时间占用 `CPU` 资源，从而导致多线程处理性能下降
- `std::this_thread::yield()` 是让当前线程主动放弃了当前自己抢到的 `CPU` 资源，但是在下一轮还会继续抢。