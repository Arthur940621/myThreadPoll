# chrono

`C++11` 中提供了日期和时间相关的库 `chrono`，定义于头文件 `<chrono>`，通过 `chrono` 库可以很方便地处理日期和时间，为程序的开发提供了便利。`chrono` 库主要包含三种类型的类：时间间隔 `duration`、时钟 `clocks`、时间点 `time point`。

## 1. 时间间隔 `duration`

常用类成员

`duration` 表示一段时间间隔，用来记录时间长度，可以表示几秒、几分钟、几个小时的时间间隔。`duration` 的原型如下：

```cpp
// 定义于头文件 <chrono>
template<
    class Rep,
    class Period = std::ratio<1>
> class duration;
```

- Rep：这是一个数值类型，表示时钟数（周期）的类型（默认为整形）。若 Rep 是浮点数，则 duration 能使用小数描述时钟周期的数目。
- Period：表示时钟的周期，它的原型如下：
```cpp
  // 定义于头文件 <ratio>
template<
    std::intmax_t Num,
    std::intmax_t Denom = 1
> class ratio;
```

`ratio` 类表示每个时钟周期的秒数，其中第一个模板参数 `Num` 代表分子，`Denom` 代表分母，该分母值默认为 `1`，因此，`ratio `代表的是一个分子除以分母的数值，比如：`ratio<2>` 代表一个时钟周期是 `2` 秒，`ratio<60>` 代表一分钟，`ratio<60*60>` 代表一个小时，`ratio<60*60*24>` 代表一天。而 `ratio<1, 1000>` 代表的是 `1/1000 `秒，也就是 `1` 毫秒，`ratio<1, 1000000>` 代表一微秒，`ratio<1, 1000000000>` 代表一纳秒。

为了方便使用，在标准库中定义了一些常用的时间间隔，比如：时、分、秒、毫秒、微秒、纳秒，它们都位于 `chrono` 命名空间下，定义如下：

|类型|
|-|
|纳秒：std::chrono::nanoseconds|
|微秒：std::chrono::microseconds|
|毫秒：std::chrono::milliseconds|
|秒：std::chrono::seconds|
|分钟：std::chrono::minutes|
|小时：std::chrono::hours|

`duration` 类的构造函数原型如下：

```cpp
// 1. 拷贝构造函数
duration(const duration&) = default;
// 2. 通过指定时钟周期的类型来构造对象
template<class Rep2>
constexpr explicit duration(const Rep2& r);
// 3. 通过指定时钟周期类型，和时钟周期长度来构造对象
template<class Rep2, class Period2>
constexpr duration(const duration<Rep2, Period2>& d);
```

为了更加方便的进行 `duration` 对象之间的操作，类内部进行了操作符重载：

|操作符重载|描述|
|-|-|
|operator=|赋值内容 (公开成员函数)|
|operator+ operator-|实现一元 + 和一元 - (公开成员函数)|
operator++ operator++(int) operator– operator–(int)|递增或递减周期计数 (公开成员函数)|
|operator+= operator-= operator*= operator/= operator%=|实现二个时长间的复合赋值 (公开成员函数)|

`duration` 类还提供了获取时间间隔的时钟周期数的方法 `count()`，函数原型如下：

```cpp
constexpr rep count() const;
```

类的使用

通过构造函数构造事件间隔对象示例代码如下：

```cpp
#include <iostream>
#include <chrono>

int main() {
    std::chrono::hours h(1); // 1小时
    std::chrono::milliseconds ms{3}; // 3毫秒
    std::chrono::duration<int, std::ratio<1000>> ks(3); // 3000秒
    // std::chrono::duration<int, std::ratio<1000>> d3(3.5); // error
    std::chrono::duration<double> dd(6.6); // 6.6秒
    // 使用小数表示时钟周期的次数
    std::chrono::duration<double, std::ratio<1, 30>> hz(3.5);
    return 0;
}
```

- h(1) 时钟周期为 1 小时，共有 1 个时钟周期，所以 h 表示的时间间隔为 1 小时
- ms(3) 时钟周期为 1 毫秒，共有 3 个时钟周期，所以 ms 表示的时间间隔为 3 毫秒
- ks(3) 时钟周期为 1000 秒，一共有三个时钟周期，所以 ks 表示的时间间隔为 3000 秒
- d3(3.5) 时钟周期为 1000 秒，时钟周期数量只能用整形来表示，但是此处指定的是浮点数，因此语法错误
- dd(6.6) 时钟周期为默认的 1 秒，共有 6.6 个时钟周期，所以 dd 表示的时间间隔为 6.6 秒
- hz(3.5) 时钟周期为 1/30 秒，共有 3.5 个时钟周期，所以 hz 表示的时间间隔为 1/30*3.5 秒

`chrono` 库中根据 `duration` 类封装了不同长度的时钟周期（也可以自定义），基于这个时钟周期再进行周期次数的设置就可以得到总的时间间隔了（时钟周期 * 周期次数 = 总的时间间隔）。

示例代码如下：

```cpp
#include <iostream>
#include <chrono>

int main() {
    std::chrono::milliseconds ms{3}; // 3 毫秒
    std::chrono::microseconds us = 2*ms; // 6000 微秒
    // 时间间隔周期为 1/30 秒
    std::chrono::duration<double, std::ratio<1, 30>> hz(3.5);
    std::cout <<  "3 ms duration has " << ms.count() << " ticks\n"
              <<  "6000 us duration has " << us.count() << " ticks\n"
              <<  "3.5 hz duration has " << hz.count() << " ticks\n";    
    return 0;
}
```

输出结果如下：

```
3 ms duration has 3 ticks
6000 us duration has 6000 ticks
3.5 hz duration has 3.5 ticks
```

- ms 时间单位为毫秒，初始化操作 ms{3} 表示时间间隔为 3 毫秒，一共有 3 个时间周期，每个周期为 1 毫秒
- us 时间单位为微秒，初始化操作 2*ms 表示时间间隔为 6000 微秒，一共有 6000 个时间周期，每个周期为 1 微秒
- hz 时间单位为秒，初始化操作 hz(3.5) 表示时间间隔为 1/30*3.5 秒，一共有 3.5 个时间周期，每个周期为 1/30 秒

由于在 `duration` 类内部做了操作符重载，因此时间间隔之间可以直接进行算术运算，比如我们要计算两个时间间隔的差值，就可以在代码中做如下处理：

```cpp
#include <iostream>
#include <chrono>

int main() {
    std::chrono::minutes t1(10);
    std::chrono::seconds t2(60);
    std::chrono::seconds t3 = t1 - t2;
    std::cout << t3.count() << " second" << std::endl;
    return 0;
}
```

输出结果：

```
540 second
```

在上面的测试程序中，`t1` 代表 `10` 分钟，`t2` 代表 `60` `秒，t3` 是 `t1` 减去 `t2，也就是` `60*10-60=540`，这个 `540` 表示的时钟周期，每个时钟周期是 `1` 秒，因此两个时间间隔之间的差值为 `540` 秒。

注意事项：`duration` 的加减运算有一定的规则，当两个 `duration` 时钟周期不相同的时候，会先统一成一种时钟，然后再进行算术运算，统一的规则如下：假设有 `ratio<x1,y1>` 和 `ratio<x2,y2>` 两个时钟周期，首先需要求出 `x1`，`x2` 的最大公约数 `X`，然后求出 `y1`，`y2` 的最小公倍数 `Y`，统一之后的时钟周期 `ratio` 为 `ratio<X,Y>`。

```cpp
#include <iostream>
#include <chrono>

int main() {
    std::chrono::duration<double, std::ratio<9, 7>> d1(3);
    std::chrono::duration<double, std::ratio<6, 5>> d2(1);
    // d1 和 d2 统一之后的时钟周期
    std::chrono::duration<double, std::ratio<3, 35>> d3 = d1 - d2;
    return 0;
}
```

对于分子 `6`、`9` 最大公约数为 `3`，对于分母 `7`、`5` 最小公倍数为 `35`，因此推导出的时钟周期为 `ratio<3, 35>`。

## 2. 时间点 `time point`

`chrono` 库中提供了一个表示时间点的类 `time_point`，该类的定义如下：

```cpp
// 定义于头文件 <chrono>
template<
    class Clock,
    class Duration = typename Clock::duration
> class time_point;
```

它被实现成如同存储一个 `Duration` 类型的自 `Clock` 的纪元起始开始的时间间隔的值，通过这个类最终可以得到时间中的某一个时间点。

- Clock：此时间点在此时钟上计量
- Duration：用于计量从纪元起时间的 `std::chrono::duration` 类型

`time_point` 类的构造函数原型如下：

```cpp
// 1. 构造一个以新纪元(epoch，即：1970.1.1)作为值的对象，需要和时钟类一起使用，不能单独使用该无参构造函数
time_point();
// 2. 构造一个对象，表示一个时间点，其中d的持续时间从epoch开始，需要和时钟类一起使用，不能单独使用该构造函数
explicit time_point(const duration& d);
// 3. 拷贝构造函数，构造与t相同时间点的对象，使用的时候需要指定模板参数
template<class Duration2>
time_point(const time_point<Clock, Duration2>& t);
```

在这个类中除了构造函数还提供了另外一个 `time_since_epoch()` 函数，用来获得 `1970` 年 `1` 月 `1` 日到 `time_point` 对象中记录的时间经过的时间间隔（`duration`），函数原型如下：

```cpp
duration time_since_epoch() const;
```

除此之外，时间点 `time_point` 对象和时间段对象 `duration` 之间还支持直接进行算术运算（即加减运算），时间点对象之间可以进行逻辑运算，具体细节可以参考下面的表格：

其中 `tp` 和 `tp2` 是 `time_point` 类型的对象， `dtn` 是 `duration` 类型的对象。

|描述|操作|返回值|
|-|-|-|
|复合赋值 (成员函数) operator+=|tp += dtn|*this|
|复合赋值 (成员函数)  operator-=|tp -= dtn|*this|
|算术运算符 (非成员函数) operator+|tp + dtn|a time_point value|
|算术运算符 (非成员函数)  operator+|dtn + tp|a time_point value|
|算术运算符 (非成员函数) operator-|tp - dtn|a time_point value|
|算术运算符 (非成员函数) operator-|tp - tp2|aduration value|
|关系操作符 (非成员函数) operator==|tp == tp2|a bool value|
|关系操作符 (非成员函数) operator!=|tp != tp2|a bool value|
|关系操作符 (非成员函数) operator<|tp < tp2|a bool value|
|关系操作符 (非成员函数) operator>|tp > tp2|a bool value|
|关系操作符 (非成员函数) operator>=|tp >= tp2|a bool value|
|关系操作符 (非成员函数) operator<=|tp <= tp2|a bool value|

## 3. 时钟 `clocks`

`chrono` 库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种：

`system_clock`：系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准。

`steady_clock`：是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，适合用于记录程序耗时。

`high_resolution_clock`：和时钟类 `steady_clock` 是等价的（是它的别名）。

在这些时钟类的内部有 `time_point`、`duration、Rep`、`Period` 等信息，基于这些信息来获取当前时间，以及实现 `time_t` 和 `time_point` 之间的相互转换。

|时钟类成员类型|描述|
|-|-|
|rep|表示时钟周期次数的有符号算术类型|
|period|表示时钟计次周期的 std::ratio 类型|
|duration|时间间隔，可以表示负时长|
|time_point|表示在当前时钟里边记录的时间点|

在使用 `chrono` 提供的时钟类的时候，不需要创建类对象，直接调用类的静态方法就可以得到想要的时间了。

`system_clock`

具体来说，时钟类 `system_clock` `是一个系统范围的实时时钟。system_clock` 提供了对当前时间点 `time_point` 的访问，将得到时间点转换为 `time_t` 类型的时间对象，就可以基于这个时间对象获取到当前的时间信息了。

`system_clock` 时钟类在底层源码中的定义如下：

```cpp
struct system_clock { // wraps GetSystemTimePreciseAsFileTime/GetSystemTimeAsFileTime
    using rep                       = long long;
    using period                    = ratio<1, 10'000'000>; // 100 nanoseconds
    using duration                  = chrono::duration<rep, period>;
    using time_point                = chrono::time_point<system_clock>;
    static constexpr bool is_steady = false;

    _NODISCARD static time_point now() noexcept 
    { // get current time
        return time_point(duration(_Xtime_get_ticks()));
    }

    _NODISCARD static __time64_t to_time_t(const time_point& _Time) noexcept 
    { // convert to __time64_t
        return duration_cast<seconds>(_Time.time_since_epoch()).count();
    }

    _NODISCARD static time_point from_time_t(__time64_t _Tm) noexcept 
    { // convert from __time64_t
        return time_point{seconds{_Tm}};
    }
};
```

通过以上源码可以了解到在 `system_clock` 类中的一些细节信息：

- rep：时钟周期次数是通过整形来记录的 long long
- period：一个时钟周期是 100 纳秒 ratio<1, 10'000'000>
- duration：时间间隔为 rep*period 纳秒 chrono::duration<rep, period>
- time_point：时间点通过系统时钟做了初始化 chrono::time_point<system_clock>，里面记录了新纪元时间点

另外还可以看到 `system_clock` 类一共提供了三个静态成员函数：

```cpp
// 返回表示当前时间的时间点。
static std::chrono::time_point<std::chrono::system_clock> now() noexcept;
// 将 time_point 时间点类型转换为 std::time_t 类型
static std::time_t to_time_t(const time_point& t) noexcept;
// 将 std::time_t 类型转换为 time_point 时间点类型
static std::chrono::system_clock::time_point from_time_t(std::time_t t) noexcept;
```

比如，我们要获取当前的系统时间，并且需要将其以能够识别的方式打印出来，示例代码如下：

```cpp
#include <chrono>
#include <iostream>
using namespace std;
using namespace std::chrono;
int main() {
    // 新纪元1970.1.1时间
    system_clock::time_point epoch;

    duration<int, ratio<60 * 60 * 24>> day(1);
    // 新纪元1970.1.1时间 + 1天
    system_clock::time_point ppt(day);

    using dday = duration<int, ratio<60 * 60 * 24>>;
    // 新纪元1970.1.1时间 + 10天
    time_point<system_clock, dday> t(dday(10));

    // 系统当前时间
    system_clock::time_point today = system_clock::now();
    
    // 转换为time_t时间类型
    time_t tm = system_clock::to_time_t(today);
    cout << "今天的日期是:    " << ctime(&tm);

    time_t tm1 = system_clock::to_time_t(today+day);
    cout << "明天的日期是:    " << ctime(&tm1);

    time_t tm2 = system_clock::to_time_t(epoch);
    cout << "新纪元时间:      " << ctime(&tm2);

    time_t tm3 = system_clock::to_time_t(ppt);
    cout << "新纪元时间+1天:  " << ctime(&tm3);

    time_t tm4 = system_clock::to_time_t(t);
    cout << "新纪元时间+10天: " << ctime(&tm4);
    return 0;
}
```

输出结果：
```
今天的日期是:    Wed Mar 16 15:56:18 2022
明天的日期是:    Thu Mar 17 15:56:18 2022
新纪元时间:      Thu Jan  1 08:00:00 1970
新纪元时间+1天:  Fri Jan  2 08:00:00 1970
新纪元时间+10天: Sun Jan 11 08:00:00 1970
```

`steady_clock`

如果我们通过时钟不是为了获取当前的系统时间，而是进行程序耗时的时长，此时使用 `syetem_clock` 就不合适了，因为这个时间可以跟随系统的设置发生变化。在 `C++11` 中提供的时钟类 `steady_clock` 相当于秒表，只要启动就会进行时间的累加，并且不能被修改，非常适合于进行耗时的统计。

`steady_clock` 时钟类在底层源码中的定义如下：

```cpp
struct steady_clock { // wraps QueryPerformanceCounter
    using rep                       = long long;
    using period                    = nano;
    using duration                  = nanoseconds;
    using time_point                = chrono::time_point<steady_clock>;
    static constexpr bool is_steady = true;

    // get current time
    _NODISCARD static time_point now() noexcept 
    { 
        // doesn't change after system boot
        const long long _Freq = _Query_perf_frequency(); 
        const long long _Ctr  = _Query_perf_counter();
        static_assert(period::num == 1, "This assumes period::num == 1.");
        const long long _Whole = (_Ctr / _Freq) * period::den;
        const long long _Part  = (_Ctr % _Freq) * period::den / _Freq;
        return time_point(duration(_Whole + _Part));
    }
};
```

通过以上源码可以了解到在 `steady_clock` 类中的一些细节信息：

- rep：时钟周期次数是通过整形来记录的 long long
- period：一个时钟周期是 1 纳秒 nano
- duration：时间间隔为 1 纳秒 nanoseconds
- time_point：时间点通过系统时钟做了初始化 chrono::time_point<steady_clock>

另外，在这个类中也提供了一个静态的 `now()` 方法，用于得到当前的时间点，函数原型如下：

```cpp
static std::chrono::time_point<std::chrono::steady_clock> now() noexcept;
```

假设要测试某一段程序的执行效率，可以计算它执行期间消耗的总时长，示例代码如下：

```cpp
#include <chrono>
#include <iostream>
using namespace std;
using namespace std::chrono;
int main() {
    // 获取开始时间点
    steady_clock::time_point start = steady_clock::now();
    // 执行业务流程
    cout << "print 1000 stars ...." << endl;
    for (int i = 0; i < 1000; ++i) {
        cout << "*";
    }
    cout << endl;
    // 获取结束时间点
    steady_clock::time_point last = steady_clock::now();
    // 计算差值
    auto dt = last - start;
    cout << "总共耗时: " << dt.count() << "纳秒" << endl;
}
```

`high_resolution_clock`

`high_resolution_clock` 提供的时钟精度比 `system_clock` 要高，它也是不可以修改的。在底层源码中，这个类其实是 `steady_clock` 类的别名。

```cpp
using high_resolution_clock = steady_clock;
```

## 4. 转换函数

`duration_cast`

`duration_cast` 是 `chrono` 库提供的一个模板函数，这个函数不属于 `duration` 类。通过这个函数可以对 `duration` 类对象内部的时钟周期 `Period`，和周期次数的类型 `Rep` 进行修改，该函数原型如下：

```cpp
template <class ToDuration, class Rep, class Period>
  constexpr ToDuration duration_cast (const duration<Rep, Period>& dtn);
```

- 如果是对时钟周期进行转换：源时钟周期必须能够整除目的时钟周期（比如：小时到分钟）。
- 如果是对时钟周期次数的类型进行转换：低等类型默认可以向高等类型进行转换（比如：int 转 double）。
- 如果时钟周期和时钟周期次数类型都变了，根据第二点进行推导（也就是看时间周期次数类型）。
- 以上条件都不满足，那么就需要使用 `duration_cast` 进行显示转换。

```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace std::chrono;

void f() {
    cout << "print 1000 stars ...." << endl;
    for (int i = 0; i < 1000; ++i) {
        cout << "*";
    }
    cout << endl;
}

int main() {
    auto t1 = steady_clock::now();
    f();
    auto t2 = steady_clock::now();

    // 整数时长：时钟周期纳秒转毫秒，要求 duration_cast
    auto int_ms = duration_cast<chrono::milliseconds>(t2 - t1);

    // 小数时长：不要求 duration_cast
    duration<double, ratio<1, 1000>> fp_ms = t2 - t1;

    cout << "f() took " << fp_ms.count() << " ms, "
        << "or " << int_ms.count() << " whole milliseconds\n";
}
```

`time_point_cast`
`time_point_cast` 也是 `chrono` 库提供的一个模板函数，这个函数不属于 `time_point` 类。函数的作用是对时间点进行转换，因为不同的时间点对象内部的时钟周期 `Period`，和周期次数的类型 `Rep` 可能也是不同的，一般情况下它们之间可以进行隐式类型转换，也可以通过该函数显示的进行转换，函数原型如下：

```cpp
template <class ToDuration, class Clock, class Duration>
time_point<Clock, ToDuration> time_point_cast(const time_point<Clock, Duration> &t);
```
关于函数的使用，示例代码如下：

```cpp
#include <chrono>
#include <iostream>
using namespace std;

using Clock = chrono::high_resolution_clock;
using Ms = chrono::milliseconds;
using Sec = chrono::seconds;
template<class Duration>
using TimePoint = chrono::time_point<Clock, Duration>;

void print_ms(const TimePoint<Ms>& time_point) {
    std::cout << time_point.time_since_epoch().count() << " ms\n";
}

int main() {
    TimePoint<Sec> time_point_sec(Sec(6));
    // 无精度损失, 可以进行隐式类型转换
    TimePoint<Ms> time_point_ms(time_point_sec);
    print_ms(time_point_ms); // 6000 ms

    time_point_ms = TimePoint<Ms>(Ms(6789));
    // error，会损失精度，不允许进行隐式的类型转换
    // TimePoint<Sec> sec(time_point_ms);

    // 显示类型转换,会损失精度。6789 truncated to 6000
    time_point_sec = std::chrono::time_point_cast<Sec>(time_point_ms);
    print_ms(time_point_sec); // 6000 ms
    return 0;
}
```

关于时间点的转换如果没有没有精度的损失可以直接进行隐式类型转换，如果会损失精度只能通过显示类型转换，也就是调用 `time_point_cast` 函数来完成该操作。

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


# mutex

`Mutex` 又称互斥量，`C++11` 中与 `Mutex` 相关的类（包括锁类型）和函数都声明在 `<mutex>` 头文件中。

在 `C++11` 中一共提供了四种 `mutex` 类：

- std::mutex，最基本的 Mutex 类。
- std::recursive_mutex，递归 Mutex 类。
- std::time_mutex，定时 Mutex 类。
- std::recursive_timed_mutex，定时递归 Mutex 类。

两种 `Lock` 类：

- std::lock_guard，与 Mutex RAII 相关，方便线程对互斥量上锁。
- std::unique_lock，与 Mutex RAII 相关，方便线程对互斥量上锁，但提供了更好的上锁和解锁控

其他类型：

- std::once_flag
- std::adopt_lock_t
- std::defer_lock_t
- std::try_to_lock_t

函数：

- std::try_lock，尝试同时对多个互斥量上锁。
- std::lock，可以同时对多个互斥量上锁。
- std::call_once，如果多个线程需要同时调用某个函数，call_once 可以保证多个线程对该函数只调用一次。

## 1. `std::mutex`

不论是在 `C` 还是 `C++` 中，进行线程同步的处理流程基本上是一致的，`C++` 的 `mutex` 类提供了相关的 `API` 函数：

`std::mutex` 不允许拷贝构造，也不允许 `move` 拷贝，最初产生的 `mutex` 对象是处于 `unlocked` 状态的。

`lock()` 函数用于给临界区加锁，并且只能有一个线程获得锁的所有权，它有阻塞线程的作用，函数原型如下：

```cpp
void lock();
```

调用线程将锁住该互斥量。线程调用该函数会发生下面 `3` 种情况：
- (1). 如果该互斥量当前没有被锁住，则调用线程将该互斥量锁住，直到调用 unlock之前，该线程一直拥有该锁。
- (2). 如果当前互斥量被其他线程锁住，则当前的调用线程被阻塞住。
- (3). 如果当前互斥量被当前调用线程锁住，则会产生死锁(deadlock)。

除了使用 `lock()` 还可以使用 `try_lock()` 获取互斥锁的所有权并对互斥锁加锁，函数原型如下：

```cpp
bool try_lock();
```

二者的区别在于 `try_lock()` 不会阻塞线程，`lock()` 会阻塞线程：

线程调用该函数也会出现下面 `3` 种情况：
- (1). 如果当前互斥量没有被其他线程占有，则该线程锁住互斥量，直到该线程调用 unlock 释放互斥量。
- (2). 如果当前互斥量被其他线程锁住，则当前调用线程返回 false，而并不会被阻塞掉。
- (3). 如果当前互斥量被当前调用线程锁住，则会产生死锁(deadlock)。

当互斥锁被锁定之后可以通过 `unlock()` 进行解锁，但是需要注意的是只有拥有互斥锁所有权的线程也就是对互斥锁上锁的线程才能将其解锁，其它线程是没有权限做这件事情的。该函数的函数原型如下：

```cpp
void unlock();
```

通过介绍以上三个函数，使用互斥锁进行线程同步的大致思路差不多就能搞清楚了，主要分为以下几步：

- 找到多个线程操作的共享资源（全局变量、堆内存、类成员变量等），也可以称之为临界资源
- 找到和共享资源有关的上下文代码，也就是临界区（下图中的黄色代码部分）
- 在临界区的上边调用互斥锁类的 lock() 方法
- 在临界区的下边调用互斥锁的 unlock() 方法

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

- 使用递归互斥锁的场景往往都是可以简化的，使用递归互斥锁很容易放纵复杂逻辑的产生，从而导致 bug 的产生
- 递归互斥锁比非递归互斥锁效率要低一些。
- 递归互斥锁虽然允许同一个线程多次获得同一个互斥锁的所有权，但最大次数并未具体说明，一旦超过一定的次数，就会抛出 std::system 错误。

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

- try_lock_for 函数接受一个时间范围，表示在这一段时间范围之内线程如果没有获得锁则被阻塞住（与 std::mutex 的 try_lock() 不同，try_lock 如果被调用时没有获得锁则直接返回 false），如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时（即在指定时间内还是没有获得锁），则返回 false。
- try_lock_until 函数则接受一个时间点作为参数，在指定时间点未到来之前线程如果没有获得锁则被阻塞住，如果在此期间其他线程释放了锁，则该线程可以获得对互斥量的锁，如果超时（即在指定时间内还是没有获得锁），则返回 false。

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

- std::adopt_lock_t，一个空的标记类，定义如下：
    - struct adopt_lock_t {};
    - 该类型的常量对象 adopt_lock（adopt_lock 是一个常量对象，定义如下：
      - constexpr adopt_lock_t adopt_lock {};
    - 通常作为参数传入给 unique_lock 或 lock_guard 的构造函数。

- std::defer_lock_t，一个空的标记类，定义如下： 
  - struct defer_lock_t {};
  - 该类型的常量对象 defer_lock（defer_lock 是一个常量对象，定义如下：
    - constexpr defer_lock_t defer_lock {};
  - 通常作为参数传入给 unique_lock 或 lock_guard 的构造函数。

- std::try_to_lock_t，一个空的标记类，定义如下：
  - struct try_to_lock_t {};
  - 该类型的常量对象 try_to_lock（try_to_lock 是一个常量对象，定义如下：
    - constexpr try_to_lock_t try_to_lock {};
  - 通常作为参数传入给 unique_lock 或 lock_guard 的构造函数。

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

- locking 初始化
  - lock_guard 对象管理 Mutex 对象 m，并在构造时对 m 进行上锁（调用 m.lock()）。
- adopting 初始化
  - lock_guard 对象管理 Mutex 对象 m，与 locking 初始化 (1) 不同的是， Mutex 对象 m 已被当前线程锁住。
- 拷贝构造
- lock_guard 对象的拷贝构造和移动构造(move construction) 均被禁用，因此 lock_guard 对象不可被拷贝构造或移动构造。

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
  - 新创建的 unique_lock 对象不管理任何 Mutex 对象。
- (2) locking 初始化
  - 新创建的 unique_lock 对象管理 Mutex 对象 m，并尝试调用 m.lock() 对 Mutex 对象进行上锁，如果此时另外某个 unique_lock 对象已经管理了该 Mutex 对象 m，则当前线程将会被阻塞。
- (3) try-locking 初始化
  - 新创建的 unique_lock 对象管理 Mutex 对象 m，并尝试调用 m.try_lock() 对 Mutex 对象进行上锁，但如果上锁不成功，并不会阻塞当前线程。
- (4) deferred 初始化
  - 新创建的 unique_lock 对象管理 Mutex 对象 m，但是在初始化的时候并不锁住 Mutex 对象。 m 应该是一个没有当前线程锁住的 Mutex 对象。
- (5) adopting 初始化
  - 新创建的 unique_lock 对象管理 Mutex 对象 m， m 应该是一个已经被当前线程锁住的 Mutex 对象。(并且当前新创建的 unique_lock 对象拥有对锁(Lock)的所有权)。
- (6) locking 一段时间(duration)
  - 新创建的 unique_lock 对象管理 Mutex 对象 m，并试图通过调用 m.try_lock_for(rel_time) 来锁住 Mutex 对象一段时间(rel_time)。
- (7) locking 直到某个时间点(time point)
  - 新创建的 unique_lock 对象管理 Mutex 对象 m，并试图通过调用 m.try_lock_until(abs_time) 来在某个时间点(abs_time)之前锁住 Mutex 对象。
- (8) 拷贝构造 [被禁用]
  - unique_lock 对象不能被拷贝构造。
- (9) 移动(move)构造
  - 新创建的 unique_lock 对象获得了由 x 所管理的 Mutex 对象的所有权(包括当前 Mutex 的状态)。调用 move 构造之后， x 对象如同通过默认构造函数所创建的，就不再管理任何 Mutex 对象了。
  
综上所述，由 (2) 和 (5) 创建的 unique_lock 对象通常拥有 Mutex 对象的锁。而通过 (1) 和 (4) 创建的则不会拥有锁。通过 (3)，(6) 和 (7) 创建的 unique_lock 对象，则在 lock 成功时获得锁。

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

- 移动赋值之后，由 `x` 所管理的 `Mutex` 对象及其状态将会被新的 `std::unique_lock` 对象取代。
- 如果被赋值的对象之前已经获得了它所管理的 `Mutex` 对象的锁，则在移动赋值之前会调用 `unlock` 函数释放它所占有的锁。
- 调用移动赋值之后， `x` 对象如同通过默认构造函数所创建的，也就不再管理任何 `Mutex` 对象了。

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

- 上锁/解锁操作：lock，try_lock，try_lock_for，try_lock_until 和 unlock
- 修改操作：移动赋值，交换(swap)（与另一个 std::unique_lock 对象交换它们所管理的 Mutex 对象的所有权），释放(release)（返回指向它所管理的 Mutex 对象的指针，并释放所有权）
- 获取属性操作：owns_lock（返回当前 std::unique_lock 对象是否获得了锁）、operator bool()（与 owns_lock 功能相同，返回当前 std::unique_lock 对象是否获得了锁）、mutex（返回当前 std::unique_lock 对象所管理的 Mutex 对象的指针）。

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


# condition_variable

`<condition_variable>` 头文件主要包含了与条件变量相关的类和函数。相关的类包括 `std::condition_variable` 和 `std::condition_variable_any`，还有枚举类型 `std::cv_status`。另外还包括函数 `std::notify_all_at_thread_exit()`，下面分别介绍一下以上几种类型：

`std::condition_variable` 类

当 `std::condition_variable` 对象的某个 `wait` 函数被调用的时候，它使用 `std::unique_lock`（通过 `std::mutex`）来锁住当前线程。当前线程会一直被阻塞，直到另外一个线程在相同的 `std::condition_variable` 对象上调用了 `notification` 函数来唤醒当前线程。

`std::condition_variable` 对象通常使用 `std::unique_lock<std::mutex>` 来等待，如果需要使用另外的 `lockable` 类型，可以使用 `std::condition_variable_any` 类。

```cpp
#include <iostream>                // std::cout
#include <thread>                // std::thread
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx; // 全局互斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false; // 全局标志位.

void do_print_id(int id) {
    std::unique_lock <std::mutex> lck(mtx);
    while (!ready) // 如果标志位不为 true, 则等待...
        cv.wait(lck); // 当前线程被阻塞, 当全局标志位变为 true 之后,
    // 线程被唤醒, 继续往下执行打印线程编号id.
    std::cout << "thread " << id << '\n';
}

void go() {
    std::unique_lock <std::mutex> lck(mtx);
    ready = true; // 设置全局标志位为 true.
    cv.notify_all(); // 唤醒所有线程.
}

int main() {
    std::thread threads[10];
    // spawn 10 threads:
    for (int i = 0; i < 10; ++i)
        threads[i] = std::thread(do_print_id, i);

    std::cout << "10 threads ready to race...\n";
    go(); // go!

  for (auto & th:threads)
        th.join();

    return 0;
}
```

`std::condition_variable` 构造函数

```cpp
// default (1)
condition_variable();
// copy [deleted] (2)
condition_variable (const condition_variable&) = delete;
```

`std::condition_variable` 的拷贝构造函数被禁用，只提供了默认构造函数。

`std::condition_variable::wait()`

```cpp
// unconditional (1)
void wait (unique_lock<mutex>& lck);
// predicate (2)
template <class Predicate>
  void wait (unique_lock<mutex>& lck, Predicate pred);
```

`std::condition_variable` 提供了两种 `wait()` 函数。当前线程调用 `wait()` 后将被阻塞（此时当前线程应该获得了锁（`mutex`），不妨设获得锁 lck），直到另外某个线程调用 `notify_*` 唤醒了当前线程。

在线程被阻塞时，该函数会自动调用 `lck.unlock()` 释放锁，使得其他被阻塞在锁竞争上的线程得以继续执行。另外，一旦当前线程获得通知（`notified`，通常是另外某个线程调用 `notify_*` 唤醒了当前线程），`wait()` 函数也是自动调用 `lck.lock()`，使得 `lck` 的状态和 `wait` 函数被调用时相同。

在第二种情况下（即设置了 `Predicate`），只有当 `pred` 条件为 `false` 时调用 `wait()` 才会阻塞当前线程，并且在收到其他线程的通知后只有当 `pred` 为 `true` 时才会被解除阻塞。因此第二种情况类似以下代码：

```cpp
while (!pred()) wait(lck);
```

```cpp
#include <iostream>                // std::cout
#include <thread>                // std::thread, std::this_thread::yield
#include <mutex>                // std::mutex, std::unique_lock
#include <condition_variable>    // std::condition_variable

std::mutex mtx;
std::condition_variable cv;

int cargo = 0;
bool shipment_available() {
    return cargo != 0;
}

// 消费者线程.
void consume(int n) {
    for (int i = 0; i < n; ++i) {
        std::unique_lock <std::mutex> lck(mtx);
        cv.wait(lck, shipment_available);
        std::cout << cargo << '\n';
        cargo = 0;
    }
}

int main() {
    std::thread consumer_thread(consume, 10); // 消费者线程.

    // 主线程为生产者线程, 生产 10 个物品.
    for (int i = 0; i < 10; ++i) {
        while (shipment_available())
            std::this_thread::yield();
        std::unique_lock <std::mutex> lck(mtx);
        cargo = i + 1;
        cv.notify_one();
    }

    consumer_thread.join();

    return 0;
}
```

`std::condition_variable::wait_for`

```cpp
// unconditional (1)
template <class Rep, class Period>
  cv_status wait_for (unique_lock<mutex>& lck,
                      const chrono::duration<Rep,Period>& rel_time);
// predicate (2)
template <class Rep, class Period, class Predicate>
       bool wait_for (unique_lock<mutex>& lck,
                      const chrono::duration<Rep,Period>& rel_time, Predicate pred);
```
与 `std::condition_variable::wait()` 类似，不过 `wait_for` 可以指定一个时间段，在当前线程收到通知或者指定的时间 `rel_time` 超时之前，该线程都会处于阻塞状态。而一旦超时或者收到了其他线程的通知，`wait_for` 返回，剩下的处理步骤和 `wait()` 类似。

`另外，wait_for` 的重载版本（`predicte(2)`）的最后一个参数 `pred` 表示 `wait_for` 的预测条件，只有当 `pred` 条件为 `false` 时调用 `wait()` 才会阻塞当前线程，并且在收到其他线程的通知后只有当 `pred` 为 `true` 时才会被解除阻塞。

下面的例子中，主线程等待 `th` 线程输入一个值，然后将 `th` 线程从终端接收的值打印出来，在 `th` 线程接受到值之前，主线程一直等待，每隔一秒超时一次，并打印一个 "."：

```cpp
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <chrono>             // std::chrono::seconds
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable, std::cv_status

std::condition_variable cv;

int value;

void do_read_value() {
    std::cin >> value;
    cv.notify_one();
}

int main () {
    std::cout << "Please, enter an integer (I'll be printing dots): \n";
    std::thread th(do_read_value);

    std::mutex mtx;
    std::unique_lock<std::mutex> lck(mtx);
    while (cv.wait_for(lck, std::chrono::seconds(1)) == std::cv_status::timeout) {
        std::cout << '.';
        std::cout.flush();
    }

    std::cout << "You entered: " << value << '\n';

    th.join();
    return 0;
}
```

`std::condition_variable::wait_until`

```cpp
// unconditional (1)
template <class Clock, class Duration>
  cv_status wait_until (unique_lock<mutex>& lck,
                        const chrono::time_point<Clock,Duration>& abs_time);
// predicate (2)
template <class Clock, class Duration, class Predicate>
       bool wait_until (unique_lock<mutex>& lck,
                        const chrono::time_point<Clock,Duration>& abs_time,
                        Predicate pred);
```

与 `std::condition_variable::wait_for` 类似，但是 `wait_until` 可以指定一个时间点，在当前线程收到通知或者指定的时间点 `abs_time` 超时之前，该线程都会处于阻塞状态。而一旦超时或者收到了其他线程的通知，`wait_until` 返回。

另外，`wait_until` 的重载版本（`predicte(2)`）的最后一个参数 `pred` 表示 `wait_until` 的预测条件，只有当 `pred` 条件为 `false` 时调用 `wait()` 才会阻塞当前线程，并且在收到其他线程的通知后只有当 `pred` 为 `true` 时才会被解除阻塞，因此相当于如下代码：

```cpp
while (!pred())
    if (wait_until(lck, abs_time) == cv_status::timeout)
        return pred();
return true;
```
`std::condition_variable::notify_one()`

唤醒某个等待线程。如果当前没有等待线程，则该函数什么也不做，如果同时存在多个等待线程，则唤醒某个线程是不确定的。

`std::condition_variable::notify_all()`

唤醒所有的等待线程。如果当前没有等待线程，则该函数什么也不做。

`std::condition_variable_any`

与 `std::condition_variable` 类似，只不过 `std::condition_variable_any` 的 `wait` 函数可以接受任何 `lockable` 参数，而 `std::condition_variable` 只能接受 `std::unique_lock<std::mutex>` 类型的参数，除此以外，和 `std::condition_variable` 几乎完全一样。

`std::cv_status` 枚举类型
|||
|-|-|
|cv_status::no_timeout|wait_for 或者 wait_until 没有超时，即在规定的时间段内线程收到了通知。|
|cv_status::timeout|wait_for 或者 wait_until 超时。|

`std::notify_all_at_thread_exit`

函数原型为：

```cpp
void notify_all_at_thread_exit (condition_variable& cond, unique_lock<mutex> lck);
```

当调用该函数的线程退出时，所有在 `cond` 条件变量上等待的线程都会收到通知。

```cpp
#include <iostream>           // std::cout
#include <thread>             // std::thread
#include <mutex>              // std::mutex, std::unique_lock
#include <condition_variable> // std::condition_variable

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void print_id (int id) {
    std::unique_lock<std::mutex> lck(mtx);
    while (!ready) cv.wait(lck);
    // ...
    std::cout << "thread " << id << '\n';
}

void go() {
    std::unique_lock<std::mutex> lck(mtx);
    std::notify_all_at_thread_exit(cv, std::move(lck));
    ready = true;
    std::cout << "go" << std::endl;
}

int main () {
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_id, i);
    std::cout << "10 threads ready to race...\n";

    std::thread(go).detach(); // go!

    for (auto& th : threads) th.join();

    return 0;
}
```


# future

<future> 头文件中包含了以下几个类和函数：

- Providers 类：std::promise, std::packaged_task
- Futures 类：std::future, shared_future
- Providers 函数：std::async()
- 其他类型：std::future_error, std::future_errc, std::future_status, std::launch

## 1. `std::promise`

`promise` 对象可以保存某一类型 `T` 的值，该值可被 `future` 对象读取（可能在另外一个线程中），因此 `promise` 也提供了一种线程同步的手段。在 `promise` 对象构造时可以和一个共享状态（通常是 `std::future`）相关联，并可以在相关联的共享状态（`std::future`）上保存一个类型为 `T` 的值。

可以通过 `get_future` 来获取与该 `promise` 对象相关联的 `future` 对象，调用该函数之后，两个对象共享相同的共享状态（`shared state`）

`promise` 对象是异步 `Provider`，它可以在某一时刻设置共享状态的值。
`future` 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 `ready`，然后才能获取共享状态的值。

```cpp
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

void print_int(std::future<int>& fut) {
    int x = fut.get(); // 获取共享状态的值.
    std::cout << "value: " << x << '\n'; // 打印 value: 10.
}

int main () {
    std::promise<int> prom; // 生成一个 std::promise<int> 对象.
    std::future<int> fut = prom.get_future(); // 和 future 关联.
    std::thread t(print_int, std::ref(fut)); // 将 future 交给另外一个线程t.
    prom.set_value(10); // 设置共享状态的值, 此处和线程t保持同步.
    t.join();
    return 0;
}
```
`std::promise` 构造函数

```cpp
// default (1)
promise();
// with allocator (2)
template <class Alloc> promise (allocator_arg_t aa, const Alloc& alloc);
// copy [deleted] (3)
promise (const promise&) = delete;
// move (4)
promise (promise&& x) noexcept;
```

- 默认构造函数，初始化一个空的共享状态。
- 带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态。
- 拷贝构造函数，被禁用。
- 移动构造函数。

另外，`std::promise` 的 `operator=` 没有拷贝语义，即 `std::promise` 普通的赋值操作被禁用，`operator=` 只有 `move` 语义，所以 `std::promise` 对象是禁止拷贝的。

```cpp
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

std::promise<int> prom;

void print_global_promise () {
    std::future<int> fut = prom.get_future();
    int x = fut.get();
    std::cout << "value: " << x << '\n';
}

int main () {
    std::thread th1(print_global_promise);
    prom.set_value(10);
    th1.join();

    prom = std::promise<int>();    // prom 被move赋值为一个新的 promise 对象.

    std::thread th2 (print_global_promise);
    prom.set_value (20);
    th2.join();

  return 0;
}
```

`std::promise::get_future`

该函数返回一个与 `promise` 共享状态相关联的 `future`。返回的 `future` 对象可以访问由 `promise` 对象设置在共享状态上的值或者某个异常对象。只能从 `promise` 共享状态获取一个 `future` 对象。在调用该函数之后，`promise` 对象通常会在某个时间点准备好（设置一个值或者一个异常对象），如果不设置值或者异常，`promise` 对象在析构时会自动地设置一个 `future_error` 异常（`broken_promise`）来设置其自身的准备状态。。

`std::promise::set_value`

```cpp
// generic template (1)
void set_value (const T& val);
void set_value (T&& val);
// specializations (2)
void promise<R&>::set_value (R& val);   // when T is a reference type (R&)
void promise<void>::set_value (void);   // when T is void
```

设置共享状态的值，此后 `promise` 的共享状态标志变为 `ready`。

`std::promise::set_exception`

为 `promise` 设置异常，此后 `promise` 的共享状态变标志变为 `ready`，例子如下，`线程1` 从终端接收一个整数，`线程2` 将该整数打印出来，如果 `线程1` 接收一个非整数，则为 `promise` 设置一个异常，`线程2` 在`std::future::get` 是抛出该异常。

```cpp
#include <iostream>       // std::cin, std::cout, std::ios
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
#include <exception>      // std::exception, std::current_exception

void get_int(std::promise<int>& prom) {
    int x;
    std::cout << "Please, enter an integer value: ";
    std::cin.exceptions (std::ios::failbit);   // throw on failbit
    try {
        std::cin >> x;                         // sets failbit if input is not int
        prom.set_value(x);
    } catch (std::exception&) {
        prom.set_exception(std::current_exception());
    }
}

void print_int(std::future<int>& fut) {
    try {
        int x = fut.get();
        std::cout << "value: " << x << '\n';
    } catch (std::exception& e) {
        std::cout << "[exception caught: " << e.what() << "]\n";
    }
}

int main () {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread th1(get_int, std::ref(prom));
    std::thread th2(print_int, std::ref(fut));

    th1.join();
    th2.join();
    return 0;
}
```

`std::promise::set_value_at_thread_exit`

设置共享状态的值，但是不将共享状态的标志设置为 `ready`，当线程退出时该 `promise` 对象会自动设置为 `ready`。如果某个 `std::future` 对象与该 `promise` 对象的共享状态相关联，并且该 `future` 正在调用 `get`，则调用 `get` 的线程会被阻塞，当线程退出时，调用 `future::get` 的线程解除阻塞，同时 `get` 返回 `set_value_at_thread_exit` 所设置的值。注意，该函数已经设置了 `promise` 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 `future_error`。

`std::promise::swap`

交换 `promise` 的共享状态。

## 2. `std::packaged_task`

`std::packaged_task `包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，从包装可调用对象意义上来讲，`std::packaged_task` 与 `std::function` 类似，只不过 `std::packaged_task` 将其包装的可调用对象的执行结果传递给一个 `std::future` 对象（该对象通常在另外一个线程中获取 `std::packaged_task` 任务的执行结果）。

`std::packaged_task` 对象内部包含了两个最基本元素，一、被包装的任务，任务是一个可调用的对象，如函数指针、成员函数指针或者函数对象，二、共享状态，用于保存任务的返回值，可以通过 `std::future` 对象来达到异步访问共享状态的效果。

可以通过 `std::packged_task::get_future` 来获取与共享状态相关联的 `std::future` 对象。在调用该函数之后，两个对象共享相同的共享状态，具体解释如下：

- `std::packaged_task` 对象是异步 `Provider`，它在某一时刻通过调用被包装的任务来设置共享状态的值。
`std::future` 对象是一个异步返回对象，通过它可以获得共享状态的值，在必要的时候需要等待共享状态标志变为 `ready`。

- `std::packaged_task` 的共享状态的生命周期一直持续到最后一个与之相关联的对象被释放或者销毁为止。

```cpp
#include <iostream>     // std::cout
#include <future>       // std::packaged_task, std::future
#include <chrono>       // std::chrono::seconds
#include <thread>       // std::thread, std::this_thread::sleep_for

// count down taking a second for each value:
int countdown (int from, int to) {
    for (int i=from; i!=to; --i) {
        std::cout << i << '\n';
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Finished!\n";
    return from - to;
}

int main () {
    std::packaged_task<int(int,int)> task(countdown); // 设置 packaged_task
    std::future<int> ret = task.get_future(); // 获得与 packaged_task 共享状态相关联的 future 对象.

    std::thread th(std::move(task), 10, 0);   //创建一个新线程完成计数任务.

    int value = ret.get();                    // 等待任务完成并获取结果.

    std::cout << "The countdown lasted for " << value << " seconds.\n";

    th.join();
    return 0;
}
```

构造函数

```cpp
// default (1)
packaged_task() noexcept;
// initialization (2)
template <class Fn>
  explicit packaged_task (Fn&& fn);
// with allocator (3)
template <class Fn, class Alloc>
  explicit packaged_task (allocator_arg_t aa, const Alloc& alloc, Fn&& fn);
// copy [deleted] (4)
packaged_task (const packaged_task&) = delete;
// move (5)
packaged_task (packaged_task&& x) noexcept;
```

`std::packaged_task` 构造函数共有 `5` 种形式，不过拷贝构造已经被禁用了。下面简单地介绍一下上述几种构造函数的语义：

- 默认构造函数，初始化一个空的共享状态，并且该 `packaged_task` 对象无包装任务。
- 初始化一个共享状态，并且被包装任务由参数 `fn` 指定。
- 带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态。
- 拷贝构造函数，被禁用。
- 移动构造函数。

```cpp
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

int main () {
    std::packaged_task<int(int)> foo; // 默认构造函数.

    // 使用 lambda 表达式初始化一个 packaged_task 对象.
    std::packaged_task<int(int)> bar([](int x){return x*2;});

    foo = std::move(bar); // move-赋值操作，也是 C++11 中的新特性.

    // 获取与 packaged_task 共享状态相关联的 future 对象.
    std::future<int> ret = foo.get_future();

    std::thread(std::move(foo), 10).detach(); // 产生线程，调用被包装的任务.

    int value = ret.get(); // 等待任务完成并获取结果.
    std::cout << "The double of 10 is " << value << ".\n";

return 0;
}
```
`std::promise` 类似，`std::packaged_task` 也禁用了普通的赋值操作运算，只允许 `move` 赋值运算。

`std::packaged_task::valid`

检查当前 `packaged_task` 是否和一个有效的共享状态相关联，对于由默认构造函数生成的 `packaged_task` 对象，该函数返回 `false`，除非中间进行了 `move` 赋值操作或者 `swap` 操作。

```cpp
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// 在新线程中启动一个 int(int) packaged_task.
std::future<int> launcher(std::packaged_task<int(int)>& tsk, int arg)
{
    if (tsk.valid()) {
        std::future<int> ret = tsk.get_future();
        std::thread (std::move(tsk),arg).detach();
        return ret;
    }
    else return std::future<int>();
}

int main () {
    std::packaged_task<int(int)> tsk([](int x){return x*2;});

    std::future<int> fut = launcher(tsk,25);

    std::cout << "The double of 25 is " << fut.get() << ".\n";

    return 0;
}
```

`std::packaged_task::get_future`

返回一个与 `packaged_task` 对象共享状态相关的 `future` 对象。返回的 `future` 对象可以获得由另外一个线程在该 `packaged_task` 对象的共享状态上设置的某个值或者异常。

```cpp
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

int main () {
    std::packaged_task<int(int)> tsk([](int x) { return x * 3; })); // package task

    std::future<int> fut = tsk.get_future();   // 获取 future 对象.

    std::thread(std::move(tsk), 100).detach();   // 生成新线程并调用packaged_task.

    int value = fut.get();                     // 等待任务完成, 并获取结果.

    std::cout << "The triple of 100 is " << value << ".\n";

    return 0;
}
```

`std::packaged_task::operator()(Args... args)`

调用该 `packaged_task` 对象所包装的对象（通常为函数指针，函数对象，`lambda` 表达式等），传入的参数为 `args`，调用该函数一般会发生两种情况：

- 如果成功调用 packaged_task 所包装的对象，则返回值（如果被包装的对象有返回值的话）被保存在 packaged_task 的共享状态中。
- 如果调用 packaged_task 所包装的对象失败，并且抛出了异常，则异常也会被保存在 packaged_task 的共享状态中。

以上两种情况都使共享状态的标志变为 `ready`，因此其他等待该共享状态的线程可以获取共享状态的值或者异常并继续执行下去。

共享状态的值可以通过在 `future` 对象（由 `get_future` 获得）上调用 `get` 来获得。

由于被包装的任务在 `packaged_task` 构造时指定，因此调用 `operator()` 的效果由 `packaged_task` 对象构造时所指定的可调用对象来决定：

- 如果被包装的任务是函数指针或者函数对象，调用 `std::packaged_task::operator()` 只是将参数传递给被包装的对象。

- 如果被包装的任务是指向类的非静态成员函数的指针，那么 `std::packaged_task::operator()` 的第一个参数应该指定为成员函数被调用的那个对象，剩余的参数作为该成员函数的参数。
如果被包装的任务是指向类的非静态成员变量，那么 `std::packaged_task::operator()` 只允许单个参数。


`std::packaged_task::make_ready_at_thread_exit`

该函数会调用被包装的任务，并向任务传递参数，类似 `std::packaged_task` 的 `operator()` 成员函数。但是与 `operator()` 函数不同的是，`make_ready_at_thread_exit` 并不会立即设置共享状态的标志为 `ready`，而是在线程退出时设置共享状态的标志。

如果与该 `packaged_task` 共享状态相关联的 `future` 对象在 `future::get` 处等待，则当前的 `future::get` 调用会被阻塞，直到线程退出。而一旦线程退出，`future::get` 调用继续执行，或者抛出异常。

注意，该函数已经设置了 `promise` 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 `future_error`。

`std::packaged_task::reset()`

重置 `packaged_task` 的共享状态，但是保留之前的被包装的任务。

```cpp
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// a simple task:
int triple (int x) { return x*3; }

int main () {
    std::packaged_task<int(int)> tsk (triple); // package task


    std::future<int> fut = tsk.get_future();
    std::thread (std::move(tsk), 100).detach();
    std::cout << "The triple of 100 is " << fut.get() << ".\n";


    // re-use same task object:
    tsk.reset();
    fut = tsk.get_future();
    std::thread(std::move(tsk), 200).detach();
    std::cout << "Thre triple of 200 is " << fut.get() << ".\n";

    return 0;
}
```

`std::packaged_task::swap()`

交换 `packaged_task` 的共享状态。

## 3. `std::future`

前面已经多次提到过 `std::future`，那么 `std::future` 究竟是什么呢？简单地说，`std::future` 可以用来获取异步任务的结果，因此可以把它当成一种简单的线程间同步的手段。`std::future` 通常由某个 `Provider` 创建，可以把 `Provider` 想象成一个异步任务的提供者，`Provider` 在某个线程中设置共享状态的值，与该共享状态相关联的 `std::future` 对象调用 `get`（通常在另外一个线程中） 获取该值，如果共享状态的标志不为 `ready`，则调用 `std::future::get` 会阻塞当前的调用者，直到 `Provider` 设置了共享状态的值（此时共享状态的标志变为 `ready`），`std::future::get` 返回异步任务的值或异常（如果发生了异常）。


一个有效（`valid`）的 `std::future` 对象通常由以下三种 `Provider` 创建，并和某个共享状态相关联。`Provider` 可以是函数或者类，其实我们前面都已经提到了，他们分别是：

- std::async 函数。
- std::promise::get_future，get_future 为 promise 类的成员函数。
- std::packaged_task::get_future，此时 get_future为 packaged_task 的成员函数。

一个 `std::future` 对象只有在有效的情况下才有用，由 `std::future` 默认构造函数创建的 `future` 对象不是有效的（除非当前非有效的 `future` 对象被 `move` 赋值另一个有效的 `future` 对象）。

在一个有效的 `future` 对象上调用 `get` 会阻塞当前的调用者，直到 `Provider` 设置了共享状态的值或异常（此时共享状态的标志变为 `ready`），`std::future::get` 将返回异步任务的值或异常（如果发生了异常）。

构造函数

`std::future` 一般由 `std::async`, `std::promise::get_future`, `std::packaged_task::get_future` 创建，不过也提供了构造函数，如下表所示：

```cpp
// default (1)
future() noexcept;
// copy [deleted] (2)
future (const future&) = delete;
// move (3)
future (future&& x) noexcept;
```

不过 `std::future` 的拷贝构造函数是被禁用的，只提供了默认的构造函数和 `move` 构造函数。另外，`std::future` 的普通赋值操作也被禁用，只提供了 `move` 赋值操作。如下代码所示：

```cpp
std::future<int> fut;           // 默认构造函数
fut = std::async(do_some_task);   // move-赋值操作。
```

`std::future::share()`

返回一个 `std::shared_future` 对象，调用该函数之后，该 `std::future` 对象本身已经不和任何共享状态相关联，因此该 `std::future` 的状态不再是 `valid` 的了。

```cpp
#include <iostream>       // std::cout
#include <future>         // std::async, std::future, std::shared_future

int do_get_value() { return 10; }

int main () {
    std::future<int> fut = std::async(do_get_value);
    std::shared_future<int> shared_fut = fut.share();

    // 共享的 future 对象可以被多次访问.
    std::cout << "value: " << shared_fut.get() << '\n';
    std::cout << "its double: " << shared_fut.get()*2 << '\n';

    return 0;
}
```

`std::future::get()`

`std::future::get` 一共有三种形式，如下表所示：

|||
|-|-|
|generic template (1)|T get();|
|reference specialization (2)|R& future<R&>::get(); // when T is a reference type (R&)|
|void specialization (3)|void future<void>::get(); // when T is void|

当与该 `std::future` 对象相关联的共享状态标志变为 `ready` 后，调用该函数将返回保存在共享状态中的值，如果共享状态的标志不为 `ready`，则调用该函数会阻塞当前的调用者，而此后一旦共享状态的标志变为 `ready`，`get` 返回 `Provider` 所设置的共享状态的值或者异常（如果抛出了异常）。

`std::future::valid()`

检查当前的 `std::future` 对象是否有效，即释放与某个共享状态相关联。一个有效的 `std::future` 对象只能通过 `std::async()`, `std::future::get_future` 或者 `std::packaged_task::get_future` 来初始化。另外由 `std::future` 默认构造函数创建的 `std::future` 对象是无效的，当然通过 `std::future` 的 `move` 赋值后该 `std::future` 对象也可以变为 `valid`。

```cpp
#include <iostream>       // std::cout
#include <future>         // std::async, std::future
#include <utility>        // std::move

int do_get_value() { return 11; }

int main () {
    // 由默认构造函数创建的 std::future 对象,
    // 初始化时该 std::future 对象处于为 invalid 状态.
    std::future<int> foo, bar;
    foo = std::async(do_get_value); // move 赋值, foo 变为 valid.
    bar = std::move(foo); // move 赋值, bar 变为 valid, 而 move 赋值以后 foo 变为 invalid.

    if (foo.valid())
        std::cout << "foo's value: " << foo.get() << '\n';
    else
        std::cout << "foo is not valid\n";

    if (bar.valid())
        std::cout << "bar's value: " << bar.get() << '\n';
    else
        std::cout << "bar is not valid\n";

    return 0;
}
```

`std::future::wait()`

等待与当前 `std::future `对象相关联的共享状态的标志变为 `ready`。

如果共享状态的标志不是 `ready`（此时 `Provider` 没有在共享状态上设置值（或者异常）），调用该函数会被阻塞当前线程，直到共享状态的标志变为 `ready`。
一旦共享状态的标志变为 `ready`，`wait()` 函数返回，当前线程被解除阻塞，但是 `wait()` 并不读取共享状态的值或者异常。

```cpp
#include <iostream>                // std::cout
#include <future>                // std::async, std::future
#include <chrono>                // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool do_check_prime(int x) { // 为了体现效果, 该函数故意没有优化.
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int main() {
    // call function asynchronously:
    std::future < bool > fut = std::async(do_check_prime, 194232491);

    std::cout << "Checking...\n";
    fut.wait();

    std::cout << "\n194232491 ";
    if (fut.get()) // guaranteed to be ready (and not block) after wait returns
        std::cout << "is prime.\n";
    else
        std::cout << "is not prime.\n";

    return 0;
}
```

`std::future::wait_for()`

与 `std::future::wait()` 的功能类似，即等待与该 `std::future` 对象相关联的共享状态的标志变为 `ready`，该函数原型如下：

```cpp
template <class Rep, class Period>
  future_status wait_for (const chrono::duration<Rep,Period>& rel_time) const;
```

而与 `std::future::wait()` 不同的是，`wait_for()` 可以设置一个时间段 `rel_time`，如果共享状态的标志在该时间段结束之前没有被 `Provider` 设置为 `ready`，则调用 `wait_for` 的线程被阻塞，在等待了 `rel_time` 的时间长度后 `wait_for()` 返回，返回值如下：

|返回值|描述|
|-|-|
|future_status::ready|共享状态的标志已经变为 ready，即 Provider 在共享状态上设置了值或者异常|。
|future_status::timeout|超时，即在规定的时间内共享状态的标志没有变为 ready。|
|future_status::deferred|共享状态包含一个 deferred 函数。|

```cpp
#include <iostream>                // std::cout
#include <future>                // std::async, std::future
#include <chrono>                // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool do_check_prime(int x) { // 为了体现效果, 该函数故意没有优化.
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int main() {
    // call function asynchronously:
    std::future < bool > fut = std::async(do_check_prime, 194232491);

    std::cout << "Checking...\n";
    std::chrono::milliseconds span(1000); // 设置超时间隔.

    // 如果超时，则输出"."，继续等待
    while (fut.wait_for(span) == std::future_status::timeout)
        std::cout << '.';

    std::cout << "\n194232491 ";
    if (fut.get()) // guaranteed to be ready (and not block) after wait returns
        std::cout << "is prime.\n";
    else
        std::cout << "is not prime.\n";

    return 0;
}
```

`std::future::wait_until()`

与 `std::future::wait()` 的功能类似，即等待与该 `std::future` 对象相关联的共享状态的标志变为 `ready`，该函数原型如下：

```cpp
template <class Rep, class Period>
  future_status wait_until (const chrono::time_point<Clock,Duration>& abs_time) const;
```

而与 `std::future::wait()` 不同的是，`wait_until()` 可以设置一个系统绝对时间点 `abs_time`，如果共享状态的标志在该时间点到来之前没有被 `Provider` 设置为 `ready`，则调用 `wait_until` 的线程被阻塞，在 `abs_time` 这一时刻到来之后 `wait_until()` 返回，返回值如下：

|返回值|描述|
|-|-|
|future_status::ready|共享状态的标志已经变为 ready，即 Provider 在共享状态上设置了值或者异常。|
|future_status::timeout|超时，即在规定的时间内共享状态的标志没有变为 ready。|
|future_status::deferred|共享状态包含一个 deferred 函数。|

`std::shared_future`
`std::shared_future` 与 `std::future` 类似，但是 `std::shared_future` 可以拷贝、多个 `std::shared_future` 可以共享某个共享状态的最终结果（即共享状态的某个值或者异常）。`shared_future` 可以通过某个 `std::future` 对象隐式转换（参见 `std::shared_future` 的构造函数），或者通过 `std::future::share()` 显示转换，无论哪种转换，被转换的那个 `std::future` 对象都会变为 `not-valid`。

构造函数

`std::shared_future` 共有四种构造函数：

```cpp
// default (1)|
shared_future() noexcept;
// copy (2)|
shared_future (const shared_future& x);
// move (3)|
shared_future (shared_future&& x) noexcept;
// move from future (4)|
shared_future (future<T>&& x) noexcept;
```

最后 `move from future(4)` 即从一个有效的 `std::future` 对象构造一个 `std::shared_future`，构造之后 `std::future` 对象 `x` 变为无效。

`std::shared_future` 其他成员函数

`std::shared_future` 的成员函数和 `std::future` 大部分相同，如下：

`operator=`
- 赋值操作符，与 std::future 的赋值操作不同，std::shared_future 除了支持 move 赋值操作外，还支持普通的赋值操作。
- get
获取与该 std::shared_future 对象相关联的共享状态的值（或者异常）。
- valid
有效性检查。
- wait
等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。
- wait_for
等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。（等待一段时间，超过该时间段wait_for 返回。）
- wait_until
等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。（在某一时刻前等待，超过该时刻 wait_until 返回。）

`std::future_error`

```cpp
class future_error : public logic_error;
```

`std::future_error` 继承自 `C++` 标准异常体系中的 `logic_error`。

其他与 `std::future` 相关的函数介绍

与 `std::future` 相关的函数主要是 `std::async()`，原型如下：

```cpp
// unspecified policy (1)|
template <class Fn, class... Args>
  future<typename result_of<Fn(Args...)>::type>
    async(Fn&& fn, Args&&... args);
// specific policy (2)|
template <class Fn, class... Args>
  future<typename result_of<Fn(Args...)>::type>
    async(launch policy, Fn&& fn, Args&&... args);
```

上面两组 `std::async()` 的不同之处是第一类 `std::async` 没有指定异步任务（即执行某一函数）的启动策略，而第二类函数指定了启动策略，指定启动策略的函数的 `policy` 参数可以是 `launch::async`，`launch::deferred`，以及两者的按位或( | )。

`std::async()` 的 `fn` 和 `args` 参数用来指定异步任务及其参数。另外，`std::async()` 返回一个 `std::future` 对象，通过该对象可以获取异步任务的值或异常（如果异步任务抛出了异常）。

```cpp
#include <stdio.h>
#include <stdlib.h>

#include <cmath>
#include <chrono>
#include <future>
#include <iostream>

double ThreadTask(int n) {
    std::cout << std::this_thread::get_id()
        << " start computing..." << std::endl;

    double ret = 0;
    for (int i = 0; i <= n; i++) {
        ret += std::sin(i);
    }

    std::cout << std::this_thread::get_id()
        << " finished computing..." << std::endl;
    return ret;
}

int main() {
    std::future<double> f(std::async(std::launch::async, ThreadTask, 100000000));

#if 0
    while(f.wait_until(std::chrono::system_clock::now() + std::chrono::seconds(1))
            != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#else
    while(f.wait_for(std::chrono::seconds(1))
            != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#endif

    std::cout << f.get() << std::endl;

    return 0;
}
```

其他与 `std::future` 相关的枚举类

与 `std::future` 相关的枚举类型包括：

```cpp
enum class future_errc;
enum class future_status;
enum class launch;
```

下面分别介绍以上三种枚举类型：

std::future_errc 类型：

|类型|取值|描述|
|-|-|-|
|broken_promise|0|与该 std::future 共享状态相关联的 std::promise 对象在设置值或者异常之前一被销毁。|
|future_already_retrieved|1|与该 std::future 对象相关联的共享状态的值已经被当前 Provider 获取了，即调用了 std::future::get 函数。|
|promise_already_satisfied|2|std::promise 对象已经对共享状态设置了某一值或者异常。|
|no_state|3|无共享状态。|

`std::future_status` 类型：

`std::future_status` 类型主要用在 `std::future`（或`std::shared_future`）中的 `wait_for` 和 `wait_until` 两个函数中的。

|类型|取值|描述|
|-|-|-|
|future_status::ready|0|wait_for(或wait_until) 因为共享状态的标志变为 ready 而返回。|
|future_status::timeout|1|超时，即 wait_for(或wait_until) 因为在指定的时间段（或时刻）内共享状态的标志依然没有变为 ready 而返回。|
|future_status::deferred|2|共享状态包含了 deferred 函数。|

`std::launch` 类型：

该枚举类型主要是在调用 `std::async` 设置异步任务的启动策略的。

|类型|描述|
|-|-|
|launch::async|Asynchronous: 异步任务会在另外一个线程中调用，并通过共享状态返回异步任务的结果（一般是调用 std::future::get() 获取异步任务的结果）。|
|launch::deferred|Deferred: 异步任务将会在共享状态被访问时调用，相当与按需调用（即延迟(deferred)调用）。|

```cpp
#include <iostream>                // std::cout
#include <future>                // std::async, std::future, std::launch
#include <chrono>                // std::chrono::milliseconds
#include <thread>                // std::this_thread::sleep_for

void
do_print_ten(char c, int ms) {
    for (int i = 0; i < 10; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(ms));
        std::cout << c;
    }
}

int main() {
    std::cout << "with launch::async:\n";
    std::future < void >foo =
        std::async(std::launch::async, do_print_ten, '*', 100);
    std::future < void >bar =
        std::async(std::launch::async, do_print_ten, '@', 200);
    // async "get" (wait for foo and bar to be ready):
    foo.get();
    bar.get();
    std::cout << "\n\n";

    std::cout << "with launch::deferred:\n";
    foo = std::async(std::launch::deferred, do_print_ten, '*', 100);
    bar = std::async(std::launch::deferred, do_print_ten, '@', 200);
    // deferred "get" (perform the actual calls):
    foo.get();
    bar.get();
    std::cout << '\n';

    return 0;
}
```

总结：

- 使用 async () 函数，是多线程操作中最简单的一种方式，不需要自己创建线程对象，并且可以得到子线程函数的返回值。
- 使用 std::promise 类，在子线程中可以传出返回值也可以传出其他数据，并且可选择在什么时机将数据从子线程中传递出来，使用起来更灵活。
- 使用 std::packaged_task 类，可以将子线程的任务函数进行包装，并且可以得到子线程的返回值。

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

# 线程池

## 1. 线程池原理

我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。打个比方：

阶段一：

一个医院，每天面对成千上万的病人，处理方式是：来一个病人找来一个医生处理，处理完了医生也走了。当看病时间较短的时候，医生来去的时间，显得尤为费时了。

阶段二：

医院引进了线程池的概念。设置门诊，把医生全派出去坐诊，病人来看病先挂号排队，医生根据病人队列顺序依次处理各个病人，这样就省去医生来来去去的时间了。但是，很多时候病人不多，医生却很多导致很多医生空闲浪费水电资源撒。

阶段三：

医院引进了可伸缩性线程池的概念，如阶段二，但是门诊一开始只派出了部分医生，但是增加了一个领导，病人依旧是排队看病，领导负责协调整个医院的医生。当病人很多医生忙不过来的时候，领导就去多叫几个医生来帮忙；当病人不多医生太多的时候，领导就叫一些医生回家休息去免得浪费医院资源。

阶段三就是一个线程池的例子。

