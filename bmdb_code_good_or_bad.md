
# BMDB Code编码规范建议

## 总体参考

1、[Google C++ Code Style](https://google.github.io/styleguide/cppguide.html)

2、[C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)

注：请每个程序员熟悉上面这两个指南，大部分的问题都在这两个指南里有明确的说明，而且我们也基本遵从这两个指南。

我们下面列举的一些规范细节，是因为：

* 上面两个指南中没有说明的
* 上面两个指南中虽然有，但我们需要重点提出的
* 和指南有冲突的，请以下面我们自己的标准为准

## 目录

- [名字](#名字)
- [头文件](#头文件)
- [命名空间namespace](#命名空间namespace)
- [格式](#格式)
- [const](#const)
- [强类型和类型匹配](#强类型和类型匹配)
- [全局变量](#全局变量)
- [switch case](#switch-case)
- [智能指针](#智能指针)
- [enum](#enum)
- [怀疑一切越界或非法](#怀疑一切越界或非法)
- [变量的初始化](#变量的初始化)
- [auto](#auto)
- [double-m](#double-m)

## 名字

### 文件名

全部用小写，中间用_，即snake mode，如pg_sql_connector.h

同时建议：.cc，而不是.cpp。我们的项目都是.cc，而且Unix下用.cc多（cpp是Microsoft风格）

### 函数名

一般有两种，一种是snake_mode，全部小写，中间_连接。

另外一种是大写字母开头，CamelMode。

小写字母开头的CamelMode不常见，不推荐。（Java里是依此区分method和data）

另外，对于一个工程，或一个文件，尽量统一为snake_mode或CamelMode一种，但这个不强制 (特别是调用其他模块的API)。

### 变量名

建议全部用snake mode。小写开头。

class的内部变量，请加尾巴区分，即

```cpp
class A {

 private:
  int age_;
};
```

**注意：struct不需要加尾巴，而且不应该加尾巴**

### 类型名

class和struct定义的类型名，尽可能大写开头。

下面的命名不好
```cpp
struct c_biginsights {
  std::string bsqlsh;
};

struct c_log {
  std::string path;
};

struct c_sqlqueue {
  int size;
  int duration;
};

struct c_database {
  std::string host;
  int port;
  std::string dbname;
  std::string user;
  std::string password;
};
```

建议改为
```cpp
struct CBiginsights {
  std::string bsqlsh;
};

struct CLog {
  std::string path;
};

struct CSqlqueue {
  int size;
  int duration;
};

struct CDatabase {
  std::string host;
  int port;
  std::string dbname;
  std::string user;
  std::string password;
};
```

std下的类型很多都是小写，这样挺好，可以很直接区分标准的类型和自定义的类型。

而且，我们应该尽可能带入std这个命名空间，多敲几个字符是好习惯。

## 头文件

### 头文件分类

一般分三类，中间用空行分隔

* 第一类，系统头文件，如string，必须用<>
* 第二类，本文件依赖的第三方库，可<>，也可“”，根据此库的流行度或习惯
* 第三类，本文件在自己工程里定义和需要的头文件，必须用""

至于各类的前后，均可。为了下面的"头文件必须单独依赖"，有时，可能将第三类放到前面

```cpp
#include <string>
#include <memory>

#include <dep/yamal/yaml.h>
#include "dep/redis/jedis.h"

#include "my_base.h"
#include "my_derived.h"
```

### 头文件必须单独依赖

下面这个头文件test.h (注意：是错误的，随后有证明)
```cpp
class A {
public:
  A(std::string &&n) : name_(std::move(n)) {}

  std::string GetName() const { return name_; }

private:
  std::string name_;
};
```

如果我们的test.cc这样写，编译是可以通过的
```cpp
#include <string>

#include "test.h"

int main() {
  A("Tony");

  return 0;
}
```

但test.h写法是错误的，因为它没有做到头文件单独依赖，即如果我们将test.cc写成下面的写法，编译不通过
```cpp
#include "test.h"

#include <string>

int main() {
  A("Tony");

  return 0;
}
```

### 没有必要的头文件不要include

这个GoLang可以检测出来，但C++只能依赖人眼。

### 可以放到.cc的头文件，不要放到.h里

见标题。

### 特别地，.h文件里用forward class避免太多的include

比如：(坏代码)
```cpp
#pragma once

#include <crow_all.h>

#include "handle.h"

class WebServer {
 public:
  WebServer(int port);
  ~WebServer();

  void Register();

 private:
  crow::SimpleApp app_;
  Handle *handle_;
};
```

这里，handle.h没有必要include，只要用forward class替代，即

```cpp
#pragma once

#include <crow_all.h>

class Handle;

class WebServer {
 public:
  WebServer(int port);
  ~WebServer();

  void Register();

 private:
  crow::SimpleApp app_;
  Handle *handle_;
};
```

## 命名空间namespace

### 除了main，必须都有命名空间

除了main()，其他应该都在某个namespace下，引用时，带上namespace，如

坏代码
```cpp
int main(int argc, char const *argv[]) {

  // 读取配置
  config = std::make_shared<Config>("../config.yml");
```

应该改为，好代码
```cpp
int main(int argc, char const *argv[]) {

  // 读取配置
  config = std::make_shared<bi_monitor::Config>("../config.yml");
```

### std命名空间必须明示

下面的代码如果string是std::string的话，是不好的代码

坏代码
```cpp
class A {

 private:
   string name_;
};
```

应该用std::string name_，这样，大家知道name_是标准库里的类型。而如果没有std，我们很可能认为，这个string类型是本namespace自定义的string类型。

## 格式

### public/private in class

应该只比下面的method或data多一个空格出来，如下：

```cpp
class WebServer {
 public:
  WebServer(int port);
  ~WebServer();

  void Register();

 private:
  crow::SimpleApp _app;
  Handle * handle;
};
```

### {}的规范

#### 单if

如果多行，则必须有{}

```cpp
if (cond) {
  int a = 1;
  int b = 2;
}
```

如果单行，可以不用{}，但必须和下面的statement之间有空行

以下不可以 (坏代码)
```cpp
if (cond)
  a = 1;
b = 2;
```

下面可以，因为a = 1和b =2之间有空行
```cpp
if (cond)
  a = 1;

b = 2;
```

下面也可以，因为return 10下面是括号
```cpp
for (int i = 0; i < 10; ++i>) {
  if (cond)
    return 10;
}
```

#### if else

if else必须都加{}

下面是坏代码
```cpp
if (cond)
  return 10;
else
  return 20;
```

应该改为： (好代码)
```cpp
if (cond) {
  return 10;
} else {
  return 20;
}
```

#### for or while

for和while必须加{}

下面两个坏代码
```cpp
for (int i = 0; i < 100; ++i) ++cnt;

while (cnt < 100) sum += 10;
```

应该改为 (好代码)
```cpp
for (int i = 0; i < 100; ++i) {
  ++cnt;
}

while (cnt < 100) {
  sum += 10;
}
```

### parameter和返回值

一般而言，返回值优先于paramter返回

比如：(坏代码)
```cpp
// 等待消息
void wait(T &msg) {
  std::unique_lock<std::mutex> lock(mutex_);
  while (q_.empty()) {
    cv_.wait(lock);
  }

  msg = q_.front();
  q_.pop();
}
```

改为下面这种会更好
```cpp
 // 等待消息
 Msg wait() {
  std::unique_lock<std::mutex> lock(mutex_);
  while (q_.empty()) {
    cv_.wait(lock);
  }

  auto msg = q_.front();
  q_.pop();

  return msg;
}
```

### parameters的顺序

一般对于function里的parameters，我们应遵循input在前，outpu在后的原则。

## const

任何对象（parameter或variable），都要想一下，它是否需要修改。如果不需要修改，建议加上const，这是个明示，告诉阅读者，我的代码不会修改这个对象。

否则，如果没有加const，但代码里没有任何对其有修改的地方，都应该看作是代码错误。这里比较不习惯的是parameter。不强制const，但尽可能加上。

比较麻烦的是指针或智能指针，如果将指针所指类型设置为const的话，有时不能编译通过或者改动太大，所以，也不强制，但尽可能加上。

## 强类型和类型匹配

### C++里用nullptr替代NULL

如果可以，尽量用强类型

如下面的代码并不是最好

坏代码
```cpp
char *args[] = {"fluent-bit", "-c", "../fluent-bit/fluent-bit.conf", NULL};
```

改为类型明确的nullptr会更好
```cpp
char *args[] = {"fluent-bit", "-c", "../fluent-bit/fluent-bit.conf", nullptr};
```

### 不允许在一行定义两个类型

下面几个变量定义都是不允许的
```cpp
int i, *j;    // i is integer, j is pointer to integer

int* i, j;    // i is pointer to integer, j is integer
```

即只允许下面的写法
```cpp
int i, j;     // i and j are integer

int *i, *j;   // i and j are pointer to integer

int* i, *j;   // like above, but not recommend

// seperate in two lines for two definitions of different types
int i;      // i is integer
int* j;     // j is pointer to integer
int *j;     // like above
```

### 对于是与否、成功与失败，请尽可能用bool类型

下面的写法并不好
```cpp
int res = JudgeSuccess(a, b, &c);

if (res) 
  printf("We got it!!!\n");
```

上面的res改为bool，应该更好

### 对于非空指针、非零返回值的判断，尽可能类型明示

下面的代码并不好
```cpp
bool IsEmptyNode(TreeNode* node) {
  return node;
}

bool IsEmptyNode(TreeNode* node) {
  return node ? true : false;
}

bool CallFailed(Func io_func, int fd) {
  const int res = io_func(fd);
  return !res;
}

bool CallFailed(Func io_func, int fd) {
  const int res = io_func(fd);
  return res ? false : true;
}
```

除了bool类型，我们可以直接判断，其他的比较，应该明示

上面的代码更改为下面的更好
```cpp
bool IsEmptyNode(TreeNode* node) {
  return node != nullptr;
}

bool CallFailed(Func io_func, int fd) {
  const int res = io_func(fd);
  return res != 0;
}
```

### 类型转化需要明示

尽可能不要用C里的implicit type conversion

比如：(不好的代码)
```cpp
char *p = "abc";
void *addr = (void *)p;
```

改为下面C++的明示类型转换，是好代码
```cpp
char *p = "abc";
void *addr = static_cast<void *>(p);
```

C++关于类型转换还有另外两种方式，分别是reintepret_cast<>()和dynamic_cast<>()，它们有很大的区别，应该明示出来，这是C++比C先进的地方。

### size()返回的类型是size_t

下面的代码并不好
```cpp
for (int i = 0; i < nums.size(); ++i) {

}
```

改为类型一致会更好
```cpp
for (size_t i = 0; i < nums.size(); ++i) {

}
```

### 如果类型不一致，最好明示转换

上面的范例中，我们可能需要用int i做loop，特别是可能用负数判断index时，这时，建议明示类型转换
```cpp
for (int i = static_cast<int>(nums.size()); i >= 0; --i) {
  
}
```

### 如果可以，literal最好标明类型

比如下面一个代码是有bug的
```cpp
auto add_eff = [](size_t sum, const Handle* h) {
  return sum + h->eff;
};
const size_t sum = std::accumulate(samples_.begin(), samples_.end(), 0, add_eff);
```

原因是，0被当作缺省的integer，导致出现overflow

正确的写法是
```cpp
const size_t sum = std::accumulate(samples_.begin(), samples_.end(), 0ULL, add_eff);
```


## 全局变量

最好少用全局变量(含static)，如果用，尽量trivial

少用全局变量，即使是static或anonymous namespace。如果要用，让这个全局变量尽量trivially destructible

[请参考Google的定义](https://google.github.io/styleguide/cppguide.html#Static_and_Global_Variables)

## switch case

switch case，应该将所有可能进行遍历（特别是针对enum），如果被选中的类型很少，则应该加入default，并且assert(false)

如下面的代码是好代码
```cpp
enum class Fruit {
  kApple = 0,
  kOrange,
  kBanana,
};

int foo(const Fruit &fruit) {
  int price = -1;
  switch (fruit) {
  case Fruit::kApple:
    price = 10;
    break;
  case Fruit::kOrange:
    price = 20;
    break;
  case Fruit::kBanana:
    price = 30;
    break;
  default:
    assert(false);
  }

  return price;
}
```

加default和assert(false)的好处在于，如果未来在Fruit里加入了新水果，但foo()忘了返回定价，我们debug run很快就能发现bug，而不是返回我们不想要的值。

这个对于if else if和try catch，也同样类似适用（如果不影响正常逻辑的话），如果可能的话，应该尽可能罗列所有可能，并且对不应该发生的意外，进行主动报错 (如assert false)。

## 智能指针

尽可能用std::make_shared，替代std::shared_ptr s(new XXX)

比如下面的代码
不建议的代码
```cpp
void foo() { 
  std::shared_ptr<std::string> s(new std::string("abc")); 

  use(s);
}
```

改为下面的会更好
```cpp
void foo() { 
  std::shared_ptr<std::string> s = std::make_shared<std::string>("abc"); 

  use(s);
}
```

这样做的好处在于：

1、代码里看不到，或者少看到，new以及delete
2、动态创建对象尽可能统一格式，便于grep代码检索
3、如果有多个new在一个statement上，可能异常时会有内存泄漏，而make_shared可以避免这个

## enum

单纯的enum，建议改为enum class

不建议的代码
```cpp
enum State { Normal, Transaction, Error, Warning, Statement, Prepare };
```

建议改为更好的代码
```cpp
enum class State {
  kNormal,
  kTransaction,
  kError,
  kWarning,
  kStatement,
  kPrepare
};
```

这有什么好处呢？

首先，所有的enum值都必须强制加上State::，这样就明确了这个enum常量的范围（想想：如果两个enum都定义了kError，一个是State，一个是Result）

第二，这样就是强类型了，比如两个enum，一个State，一个Result，作为参数传入，这时，C++就会强制检测参数类型，类型不对，不予通过。这是我们爱C++的地方。

第三，表达更清晰，我们一眼就看出它是常量，而之前的写法并不保证。

## 怀疑一切越界或非法

如果我们的代码不确定，特别是面对外部他人的调用时，应该尽可能怀疑其值的越界或非法可能，比如：

下面的代码有可能不安全
```cpp
else {
  transaction.duration = res[0].duration;
  transaction.end_time = res[0].start_time;
}
```

这里，res这个container，我们并没有把握它是非空。

或者，我们肯定可以确认这个res是非空的，那我们如下代码会更好，既可能是检查万一的bug，也可能是边界的说明（即明示res这里绝对不可能是空的）

更严谨或更显式表达作者意图的代码
```cpp
else {
  assert(!res.empty());
  transaction.duration = res[0].duration;
  transaction.end_time = res[0].start_time;
}
```

## 变量的初始化

没有初始化的变量定义，大部分情况都不可接受

比如下面的代码并不好
```cpp
bool success;

if (foo() && cnt > 2) {
  success = true;
}

return sucess;
```

好的写法是，定义时尽可能初始化
```cpp
bool success = false;
```

对于native type，其定义初始化带来的cost忽略不计。

注意，对于复杂的对象，如class等，其初始化是default ctor完成的，如果存在的话。

## auto

复杂类型考虑用auto，简单类型尽可能不用auto

对于复杂类型，特别是需要看API才能了解类型的，建议用auto来简化之。

而对于简单类型，特别是基本类型，如size_t，int，建议明示，而不是auto。

让阅读者易阅读和理解，是我们代码的目的。


## double-m

对于class内部的mutex内部变量，加mutable关键字

即double m原则，这样，内部的方法，才能方便区分const和非const。

即不好的代码
```cpp
std::mutex mutex_;
```

改为：好代码
```cpp
mutable std::mutex mutex_;
```