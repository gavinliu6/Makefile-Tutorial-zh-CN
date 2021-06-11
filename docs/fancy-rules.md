# 各种规则

## 静态模式规则

Make 钟爱 C 编译，它每次表达爱意时，都会做出一些“迷惑行为”。下面是一个新的类型的规则语法，叫作静态模式：

```makefile
targets ...: target-pattern: prereq-patterns ...
   commands
```

其本质是：给定的目标由 `target-pattern` 匹配得到（利用通配符 `%`）。匹配到的任何内容都被称为 _词干 (stem)_。然后，将词干替换到 `prereq-pattern` 中去，以此生成目标的 `prerequisites` 部分。

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

下面是使用了静态模式的一种 _更加有效的方式_：

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

虽然函数功能稍后才介绍，但是我会预先向你展示你可以用函数来做什么。函数 `filter` 可以用在静态模式规则中来匹配正确的文件。在下面这个例子中，我编造了 `.raw` 和 `.result` 文件扩展名。

```makefile
obj_files = foo.result bar.o lose.o
src_files = foo.raw bar.c lose.c

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

## 隐式规则

Make 中最令人迷惑的部分可能就是魔法般的规则和变量了。以下列出了一些隐式规则：

- 编译 C 程序：构建配方（recipe）是 `$(CC) -c $(CPPFLAGS) $(CFLAGS)`，若 `n.c` 存在，会自动制作 `n.o`。
- 编译 C++ 程序：构建配方是 `$(CXX) -c $(CPPFLAGS) $(CXXFLAGS)`，若 `n.cc` 或 `n.pp` 存在，会自动制作 `n.o`。
- 链接单个目标文件：构建配方是 `$(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)`，若 `n.o` 存在，会自动制作 `n`。

如此，隐式规则使用的一些重要变量有：

- `CC`：编译 C 程序的程序，默认是 `cc`
- `CXX`：编译 C++ 程序的程序，默认是 `G++`
- `CFLAGS`：提供给 C 编译器的额外标志
- `CXXFLAGS`：提供给 C++ 编译器的额外标志
- `CPPFLAGS`：提供给 C 预处理器的额外标志
- `LDFLAGS`：当编译器应该调用链接器时提供给编译器的额外标志

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

## 模式规则

模式规则虽经常使用，但是很令人迷惑。你可以以两种方式来看待它们：

- 定义你自己的隐式规则
- 静态模式规则的简化形式

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
