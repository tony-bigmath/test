# BMDB Code Format and Tools

## 目录

* 工具Clang Format：安装和配置
* Clang Format: C++
* Clang Format: 其他语言（Java、Python、前端或其他语言）
* BMDB Copyright头部自动插入工具

## Clang Format：安装和配置

我们采用统一的Clang Format处理我们统一的code style和code format。

### 安装步骤

```cpp
if 你已有最新(或较新)的Visual Studio Code {
    // Do Nothing，因为VS Code已自带了Clang Format
} else if (VS Code太老) {
    到Extensions MarketPlace，下载一款Clang Format
} else {
    // 你可以安装命令行工具
    switch (your_os) {
        case Ubuntu:
            apt install clang-format;
            break;
        case CentOS:
            yum install clang;
            break;
        case MacOS:
            brew install clang-format;
            break;
        case Windows:
            Install LLVM
            Install Clang-Format extension
            break;
        default;
            assert(false);
    }
}
```

### 关于配置

同时，为了保证我们所有代码用统一的规格，请将bmdb根目录下的文件 **.clang-format**

copy到你的新的project的VS Code根目录下

## C++

### 一个范例

我们以VSCode里自带的Clang Format为例，其热键是：**SHIFT-ALT-F**

比如下面的代码：
```cpp
#include <iostream>
#include <string>

void foo (std::string* to_print) {
    int b = 1;
    std::cout << *to_print << '\n';
}

int main(int argc, char ** argv) 
{
    int a = 1;  // one table ahead

    std::cout << "This is a test for code format\n";    // four spaces ahead

        std::string to_print = "I want to print something"; // two tables ahead
        foo(&to_print);

    return 0;
}
```

热键后，得到
```cpp
#include <iostream>
#include <string>

void foo(std::string *to_print) {
  int b = 1;
  std::cout << *to_print << '\n';
}

int main(int argc, char **argv) {
  int a = 1; // one table ahead

  std::cout << "This is a test for code format\n"; // four spaces ahead

  std::string to_print = "I want to print something"; // two tables ahead
  foo(&to_print);

  return 0;
}
```

### 范例的总结

1、不管是table被定义为4个，还是前面使用了两个table、多个space，都会被统一为2个space这个规范上

2、{}的使用，会被统一，我们采用同行scope

3、指针的定义，也会被统一为 Type *var

4、Clang Format不解决编码的错误或不当逻辑，比如：foo()中，没有对to_print进行nullptr判断，这个clang format不负责报错或提示，需要code reviewer严格把关

5、社区的代码如果用Clang Format去检查，基本都有不一致的地方，但都是微小地方，所以，不强制用Clang Format作为标准，但可以作为工具，具体可参考下面的编码规范建议。

### 要养成不依赖Clang Format的习惯

我们不应该养成依赖Clang Format的习惯，即平时随便写代码，然后最后提交前用clang format格式一下。

**记住：我们平常写的代码应该和Clang Format要求我们的一样，即Clang Format仅用于补漏**

下面这个链接是我们常见的一些代码不规范地方，请参考并养成好习惯。

[BMDB Code编码规范建议](bmdb_code_good_or_bad.md)

## 其他（Java、Python、前端或其他语言）

### 前端或其他语言

前端，当前没有自己的code style，如果我们需要创建自己的某个编程语言，或者某个子项目（不是上面的C++）的code style时，我们可以采用Clang Format工具，具体操作如下：

安装Clang Format工具，然后到自己做好的工程或范例文件，用Clang Format输出某种格式配置文件.clang-format，具体如下

```bash
clang-format -dump-config > .clang-format
```

### Java & Python

对于Java，请将bmdb根目录下的code_style.xml用于你的IDE，如JetBrains IntelliJ IDEA。

对于Python，请使用bmdb根目录下的.editorconfig，如JetBrains PyCharm。


## BMDB Copyright头部自动插入工具

我们以VSCode为例，首先安装一个Extension，[psioniq File Header](https://marketplace.visualstudio.com/items?itemName=psioniq.psi-header&ssr=false#user-content-configuration)。它支持本地以及Linux服务器安装。

首次安装后，请重启VSCode，然后Installed Extension里找到这个psioniq File Header，点击其配置图标，选择Extension Settings。

我们需要修改settings.json里的下面两项，一个是：psi-header.config，一个是：psi-header.templates

如下：

```
    "psi-header.config": {
        "forceToTop": true,
    },
    "psi-header.templates": [
        {
            "language": "*",
            "template": [
                "BIGMATH INSIGHTS CONFIDENTIAL",
                "",
                "Copyright (C) 2021-2023 BIGMATH CORPORATION, All Rights Reserved.",
                "",
                "@Version: V1.0",
                "",
                "These source codes are subject to the terms and conditions defined",
                "in 'LICENSE', which is part of this source code package. Write to",
                "LICENSE@BIGMATH.COM for more authorization requirements, or obtain",
                "an entire copy of license agreement at http://bigmath.com/license.",
                "",
                "THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS AS \"IS\" AND",
                "ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED",
                "WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE",
                "DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER BE LIABLE FOR ANY",
                "DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES",
                "(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;",
                "LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND",
                "ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT",
                "(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS",
                "SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.",
            ]
        }
    ]
```

然后打开你想增加Copyright的任何source file，然后连续两次按ctrl-alt-H。