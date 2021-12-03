# 函数

## 第 1 个函数

_函数_ 主要用于文本处理。函数调用的语法是 `$(fn, arguments)` 或 `${fn, arguments}`。你可以使用内置的函数 [`call`](https://www.gnu.org/software/make/manual/html_node/Call-Function.html#Call-Function) 来制作自己的函数。Make 拥有数量众多的 [内置函数](https://www.gnu.org/software/make/manual/html_node/Functions.html)。

```makefile
bar := ${subst not, totally, "I am not superman"}
all: 
    @echo $(bar)
```

如果你想替换空格或逗号，请使用变量：

```makefile
comma := ,
empty:=
space := $(empty) $(empty)
foo := a b c
bar := $(subst $(space),$(comma),$(foo))

all: 
    @echo $(bar)
```

**不要** 在第一个参数之后的参数中包含空格，这将被视为字符串的一部分。

```makefile
comma := ,
empty:=
space := $(empty) $(empty)
foo := a b c
bar := $(subst $(space), $(comma) , $(foo))

all: 
    # Output is ", a , b , c". Notice the spaces introduced
    @echo $(bar)
```

## 字符串替换

`$(patsubst pattern,replacement,text)` 做了下面这些事：

> “在文本中查找匹配的以空格分隔的单词，用 `replacement` 替换它们。这里的 `pattern` 可以包含一个 `%` 作为通配符以匹配单词中任意数量的任意字符。如果 `replacement` 中也包含了一个 `%`，那它表示的内容将被 `pattern` 中的 `%` 匹配的内容替换。只有 `pattern` 和 `replacement` 中的第一个 `%` 才会采取这种行为，随后的任何 `%` 都将保持不变。（摘自 [GNU 文档](https://www.gnu.org/software/make/manual/html_node/Text-Functions.html#Text-Functions)）”

`$(text:pattern=replacement)` 是一个简化写法。

还有一个仅替换后缀的简写形式：`$(text:suffix=replacement)`，这里没有使用通配符 `%`。

<Note>在简写形式中，不要添加额外的空格，它会被当作一个搜索或替换项。</Note>

```makefile
foo := a.o b.o l.a c.o
one := $(patsubst %.o,%.c,$(foo))
# This is a shorthand for the above
two := $(foo:%.o=%.c)
# This is the suffix-only shorthand, and is also equivalent to the above.
three := $(foo:.o=.c)

all:
    echo $(one)
    echo $(two)
    echo $(three)
```

## 函数 `foreach`

函数 `foreach` 看起来像这样：`$(foreach var,list,text)`，它用于将一个单词列表（空格分隔）转换为另一个。`var` 表示循环中的每一个单词，`text` 用于扩展每个单词。

在每个单词后追加一个感叹号：


```makefile
foo := who are you
# For each "word" in foo, output that same word with an exclamation after
bar := $(foreach wrd,$(foo),$(wrd)!)

all:
    # Output is "who! are! you!"
    @echo $(bar)
```

## 函数 `if`

`if` 函数用来检查它的第 1 个参数是否非空。如果非空，则运行第 2 个参数，否则运行第 3 个。

```makefile
foo := $(if this-is-not-empty,then!,else!)
empty :=
bar := $(if $(empty),then!,else!)

all:
    @echo $(foo)
    @echo $(bar)
```

## 函数 `call`

Make 支持创建基本的函数。你只需通过创建变量来“定义”函数，只是会用到参数 `$(0)`、`$(1)` 等。然后，你就可以使用专门的函数 `call` 来调用它了，语法是 `$(call variable,param,param)`。`$(0)` 是变量名，而 `$(1)`、`$(1)` 等则是参数。

```makefile
sweet_new_fn = Variable Name: $(0) First: $(1) Second: $(2) Empty Variable: $(3)

all:
    # Outputs "Variable Name: sweet_new_fn First: go Second: tigers Empty Variable:"
    @echo $(call sweet_new_fn, go, tigers)
```

## 函数 `shell`

`shell` - 调用 shell，但它在输出中会用空格替代换行。

```makefile
all: 
    @echo $(shell ls -la) # Very ugly because the newlines are gone!
```
