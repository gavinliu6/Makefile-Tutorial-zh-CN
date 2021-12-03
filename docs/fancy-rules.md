# 各种规则

## 隐式规则

Make 钟爱 C 编译，它每次表达爱意时，都会做出一些“迷惑行为”。其中最令人迷惑的部分可能就是它的那些魔法般的规则了，Make 称之为“隐式规则”。我个人不认同这个设计方案，也不推荐使用它们。然而即便如此，它们也被经常使用，所以了解一下它们也是很有用处的。下面列出了隐式规则：

- 编译 C 程序时：使用 `$(CC) -c $(CPPFLAGS) $(CFLAGS)` 形式的命令，`n.o` 会由 `n.c` 自动生成。
- 编译 C++ 程序时：使用 `$(CXX) -c $(CPPFLAGS) $(CXXFLAGS)` 形式的命令，`n.o` 会由 `n.cc` 或 `n.pp` 自动生成。
- 链接单个目标文件时：通过运行 `$(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)` 命令，`n` 会由 `n.o` 自动生成。

上述隐式规则使用的变量的含义如下所示：

- `CC`：编译 C 程序的程序，默认是 `cc`
- `CXX`：编译 C++ 程序的程序，默认是 `G++`
- `CFLAGS`：提供给 C 编译器的额外标志
- `CXXFLAGS`：提供给 C++ 编译器的额外标志
- `CPPFLAGS`：提供给 C 预处理器的额外标志
- `LDFLAGS`：当编译器应该调用链接器时提供给编译器的额外标志

现在就让我们来看一下如何在不明确告诉 Make 该如何进行编译的情况下构件一个 C 程序。

```makefile
CC = gcc # Flag for implicit rules
CFLAGS = -g # Flag for implicit rules. Turn on debug info

# Implicit rule #1: blah is built via the C linker implicit rule
# Implicit rule #2: blah.o is built via the C compilation implicit rule, because blah.c exists
blah: blah.o

blah.c:
    echo "int main() { return 0; }" > blah.c

clean:
    rm -f blah*
```

## 静态模式规则

静态模式规则是另一种可以在 Makefile 文件中“少废笔墨”的方式，但是我认为它更加有用且更容易让人理解。其语法如下：

```makefile
targets ...: target-pattern: prereq-patterns ...
   commands
```

它的本质是：给定的目标 `target` 由 `target-pattern` 在 `targets` 中匹配得到（利用通配符 `%`）。匹配到的内容被称为 _词干 (stem)_。然后，将词干替换到 `prereq-pattern` 中去，并以此生成目标的 `prerequisites` 部分。

静态模式规则的一个典型用例就是把 `.c` 文件编译为 `.o` 文件。下面是 _手动_ 方式：

```makefile
objects = foo.o bar.o all.o
all: $(objects)

# These files compile via implicit rules
foo.o: foo.c
bar.o: bar.c
all.o: all.c

all.c:
    echo "int main() { return 0; }" > all.c

%.c:
    touch $@

clean:
    rm -f *.c *.o all
```

下面则是使用了静态模式的一种 _更加有效的方式_：

```makefile
objects = foo.o bar.o all.o
all: $(objects)

# These files compile via implicit rules
# Syntax - targets ...: target-pattern: prereq-patterns ...
# In the case of the first target, foo.o, the target-pattern matches foo.o and sets the "stem" to be "foo".
# It then replaces the '%' in prereq-patterns with that stem
$(objects): %.o: %.c

all.c:
    echo "int main() { return 0; }" > all.c

%.c:
    touch $@

clean:
    rm -f *.c *.o all
```

## 静态模式规则与过滤器

虽然函数稍后才会介绍到，但是我会预先向你展示你可以用函数来做什么。函数 `filter` 可以用在静态模式规则中来匹配正确的文件。在下面这个例子中，我编造了 `.raw` 和 `.result` 文件扩展名。

```makefile
obj_files = foo.result bar.o lose.o
src_files = foo.raw bar.c lose.c

.PHONY: all
all: $(obj_files)

$(filter %.o,$(obj_files)): %.o: %.c
    echo "target: $@ prereq: $<"
$(filter %.result,$(obj_files)): %.result: %.raw
    echo "target: $@ prereq: $<" 

%.c %.raw:
    touch $@

clean:
    rm -f $(src_files)
```

## 模式规则

模式规则虽然常用，但是很令人迷惑。你可以以两种方式来看待它们：

- 一个定义你自己的隐式规则的方式
- 一个静态模式规则的简化形式

先看个例子吧：

```makefile
# Define a pattern rule that compiles every .c file into a .o file
%.o : %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

模式规则在目标中包含了一个 `%`，这个 `%` 匹配任意非空字符串，其他字符匹配它们自己。一个模式规则的 `prerequisite` 中的 `%` 表示目标中 `%` 匹配到的同一个词干。

<Note type="tip" label="译者注">

自动变量 `$<` 表示第一个 `prerequisite`。

</Note>

另一个例子：

```makefile
# Define a pattern rule that has no pattern in the prerequisites.
# This just creates empty .c files when needed.
%.c:
   touch $@
```

## 双冒号规则

双冒号规则虽然很少用到，但是它能为同一个目标定义多个规则。如果换为单冒号的话，系统会输出警告，并且只有第 2 个规则定义的命令会运行。

```makefile
all: blah

blah::
    echo "hello"

blah::
    echo "hello again"
```
