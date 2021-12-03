# 变量（第 2 部分）

## 变量类型和修改

变量的类型有两种：

- 递归变量（使用 `=`）- 只有在命令执行时才查找变量，而不是在定义时
- 简单的扩展变量（使用 `:=`）- 就像普通的命令式编程一样——只有当前已经定义的变量才会得到扩展

```makefile
# Recursive variable. This will print "later" below
one = one ${later_variable}
# Simply expanded variable. This will not print "later" below
two := two ${later_variable}

later_variable = later

all:
    echo $(one)
    echo $(two)
```

扩展变量（使用 `:=`）使得你可以在一个变量的基础上追加内容，递归变量则会陷入死循环。

```makefile
one = hello
# one gets defined as a simply expanded variable (:=) and thus can handle appending
one := ${one} there

all:
    echo $(one)
```

`?=` 用于当变量还没被设置值时给它设置值，反之则忽略。

```makefile
one = hello
one ?= will not be set
two ?= will be set

all:
    echo $(one)
    echo $(two)
```

变量值尾部的空格不会被删除，但开头的空格会被删除。想要一个值为单个空格的变量请使用 `$(nullstring)`。

```makefile
with_spaces = hello   # with_spaces has many spaces after "hello"
after = $(with_spaces)there

nullstring =
space = $(nullstring) # Make a variable with a single space.

all:
    echo "$(after)"
    echo start"$(space)"end
```

一个未定义的变量实际上是一个空字符串。

```makefile
all:
    # Undefined variables are just empty strings!
    echo $(nowhere)
```

`+=` 用来追加变量的值：

```makefile
foo := start
foo += more

all:
    echo $(foo)
```

[字符串替换](functions#字符串替换) 也是一个常见且有用的修改变量的方式。其他相关可参阅 [文本函数](https://www.gnu.org/software/make/manual/html_node/Text-Functions.html#Text-Functions) 与 [文件名函数](https://www.gnu.org/software/make/manual/html_node/File-Name-Functions.html#File-Name-Functions)。

## 命令行参数与覆盖

你可以通过 `override` 来覆盖来自命令行的变量。假使我们使用下面的 makefile 执行了这样一条命令 `make option_one=hi`，那么变量 `option_one` 的值就会被覆盖掉。

```makefile
# Overrides command line arguments
override option_one = did_override
# Does not override command line arguments
option_two = not_override
all:
    echo $(option_one)
    echo $(option_two)
```

## 命令列表与 `define`

`define` 实际上就是一个命令列表，它与函数 `define` 没有任何关系。这里请注意，它与用分号分隔多个命令的场景有点不同，因为前者如预期的那样，每条命令都是在一个单独的 shell 中运行的。

```makefile
one = export blah="I was set!"; echo $$blah

define two
export blah=set
echo $$blah
endef

# One and two are different.

all: 
    @echo "This prints 'I was set'"
    @$(one)
    @echo "This does not print 'I was set' because each command runs in a separate shell"
    @$(two)
```

## 特定目标的变量

我们可以为特定目标分配变量。

```makefile
all: one = cool

all: 
    echo one is defined: $(one)

other:
    echo one is nothing: $(one)
```

## 特定模式的变量

我们可以为特定的目标 _模式_ 分配变量。

```makefile
%.c: one = cool

blah.c: 
    echo one is defined: $(one)

other:
    echo one is nothing: $(one)
```
