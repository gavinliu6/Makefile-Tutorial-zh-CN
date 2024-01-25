# 起步

## Makefiles 简介

Makefiles 用于帮助决定一个大型程序的哪些部分需要重新编译。在绝大多数情况下，需要编译的只是 C 或 C++ 文件。其他语言通常有它们自己的一套与 Make 用途类似的工具。Make 的用途并不局限于编程，当你需要根据哪些文件发生了变更来运行一系列指令时也可以使用它。而本教程将只关注 C/C++ 编译用例。

这里有一个你可能会使用 Make 进行构建的依赖关系示例图。如有任何文件的依赖项发生了改变，那么该文件就会被重新编译。

<ImageZoom src="./assets/images/dependency_graph.png" :border="false" />

## Make 的替代品

除了 Make，还有一些比较流行的构建系统可选，像 [SCon](https://scons.org/)、[CMake](https://cmake.org/)、[Bazel](https://bazel.build/) 和 [Ninja](https://ninja-build.org/) 等。一些代码编辑器，像 [Microsoft Visual Studio](https://visualstudio.microsoft.com/)，内置了它们自己的构建工具。Java 语言的构建工具有 [Ant](https://ant.apache.org/)、[Maven](https://maven.apache.org/what-is-maven.html) 和 [Gradle](https://gradle.org/) 可选，其他语言像 Go 和 Rust 则都有它们自己的构建工具。

像 Python、Ruby 和 JavaScript 这样的解释型语言是不需要类似 Makefiles 的东西的。Makefiles 的目标是基于哪些文件发生了变化来编译需要被编译的一切文件。但是，当解释型语言的文件发生了变化，是不需要重新编译的，程序运行时会使用最新版的源码文件。

## Make 的不同实现与版本

虽然 Make 有多种实现，但本指南的大部分内容都是版本间无差异的。然而，需要说明的是，本指南是专门为 GNU Make 编写的，这也是 Linux 与 macOS 平台上的标准实现。文中所有的例子在 Make 的版本 3 和版本 4 上都能很好地工作，除了一些细微的差别外，它们的行为表现也几乎一致。

## 如何运行示例

为了运行文中示例，你需要一个安装了 `make` 的终端。对于每一个例子，只需要把它的内容放在一个名为 `Makefile` 的文件中，再把该文件放在运行 `make` 命令的目录下就行了。让我们从最简单的一个 Makefile 开始吧：

```makefile
hello:
    echo "hello world"
```

以下是运行上述示例的输出：

```shell
$ make
echo "hello world"
hello world
```

是的，就是这样！如果你还心存困惑，这儿有一份介绍了完整步骤的视频，同时它还描述了 Makefiles 的基本结构。

<iframe width="560" height="315" src="https://www.youtube.com/embed/zeEMISsjO38" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Makefile 语法

Makefile 文件由一系列的 _规则 (rules)_ 组成，一个规则类似下面这样：

```makefile
targets: prerequisites
    command
    command
    command
```

- `targets` 指的是文件名称，多个文件名以空格分隔。通常，一个规则只对应一个文件。
- `commands` 通常是一系列用于制作（make）一个或多个目标（targets）的步骤。它们 _需要以一个制表符开头_，而不是空格。
- `prerequisites` 也是文件名称，多个文件名以空格分隔。在运行目标（targets）的 `commands` 之前，要确保这些文件是存在的。它们也被称为 _依赖_。

## 新手示例

下面的 Makefile 有 3 个分离的 _规则 (rules)_。当你在终端运行 `make blah` 时，它会通过一系列的步骤构建一个名为 `blah` 的程序：

- `blah` 给 `make` 提供了构建目标（target）的名称，所以它会在 makefile 中优先被 `make` 程序搜索
- 构建系统发现 `blah` 依赖 `blah.o`，所以 `make` 开始搜索 `blah.o` 这个目标
- `blah.o` 又依赖 `blah.c`，所以 `make` 又开始搜索 `blah.c` 这个目标
- `blah.c` 没有依赖，直接运行 `echo` 命令
- 接着运行 `cc -c` 命令，因为 `blah.o` 的依赖的所有 `commands` 都执行完了
- 同理，接着运行顶部的 `cc` 命令
- 就这样，一个编译好的 C 程序 `blah` 就诞生了

```makefile
blah: blah.o
    cc blah.o -o blah # Runs third

blah.o: blah.c
    cc -c blah.c -o blah.o # Runs second

blah.c:
    echo "int main() { return 0; }" > blah.c # Runs first
```

<Note type="tip" label="译者注">

`-c` 选项只编译不链接，`-o file` 将其前面命令的输出内容放在文件 _file_ 中。

详情可参阅：https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#Overall-Options

</Note>

下面的这个 makefile 只有一个目标，叫作 `some_file`。因为默认的目标就是第一个目标，所以，在执行 `make` 命令时，`some_file` 目标下的命令会运行。👇

```makefile
some_file:
    echo "This line will always print"
```

若应用下面这个 makefile，在第一次构建时将创建文件 *some_file*，第二次系统就会注意到该目标文件已经创建过了，结果就会得到这样的提示信息：*make: 'some_file' is up to date.*。👇

```makefile
some_file:
    echo "This line will only print once"
    touch some_file
```

敲黑板！目标 `some_file` 依赖 `other_file`。当我们运行 `make` 时，默认目标（即 `some_file`，因为它是第一个）会被“召唤”。构建系统首先查看目标的 _依赖_ 列表，若其中有旧的目标文件，构建系统首先会为这些依赖执行目标构建，此后才轮到默认目标。第二次执行 `make` 时，默认目标和依赖目标下的命令都不会再运行了，因为二者都存在了。👇

```makefile
some_file: other_file
    echo "This will run second, because it depends on other_file"
    touch some_file

other_file:
    echo "This will run first"
    touch other_file
```

而下面这个 makefile 中的两个目标每次构建都会运行，因为 `some_file` 依赖的 `other_file` 从未被创建过。👇

```makefile
some_file: other_file
    touch some_file

other_file:
    echo "nothing"
```

<Note type="tip" label="译者注">

👆 类似上面 `other_file` 这样的目标就是俗称的 _伪目标_ 或 _虚拟目标_。

</Note>

`clean` 经常被用来作为移除其他目标的输出的目标名称，但是在 `make` 看来它并非是一个特殊用词。

```makefile
some_file: 
    touch some_file

clean:
    rm -f some_file
```

## 变量

在 Makefile 中，变量的值类型只能是字符串。下面是一个使用变量的例子：

```makefile
files = file1 file2
some_file: $(files)
    echo "Look at this variable: " $(files)
    touch some_file

file1:
    touch file1
file2:
    touch file2

clean:
    rm -f file1 file2 some_file
```

引用变量的语法是：`${}` 或 `$()`。

```makefile
x = dude

all:
    echo $(x)
    echo ${x}

    # Bad practice, but works
    echo $x
```
