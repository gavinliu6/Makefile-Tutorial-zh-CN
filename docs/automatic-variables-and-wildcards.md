# 自动变量和通配符

## 通配符 `*`

在 Make 中，`%` 和 `*` 都叫作通配符，但是它们是两个完全不同的东西。`*` 会搜索你的文件系统来匹配文件名。我建议你应该一直使用 `wildcard` 函数来包裹它，因为要不然你可能会掉入下述的常见陷阱中。奇怪的是，不用 `wildcard` 包裹的 `*` 没有任何好处，只能令人更加迷惑。

```makefile
# Print out file information about every .c file
print: $(wildcard *.c)
    ls -la  $?
```

`*` 可以用在 `targets`、`prerequisites` 以及 `wildcard` 函数中。

<Note type="danger">`*` 不能直接用在变量定义中。</Note>

<Note type="danger">当 `*` 匹配不到文件时，它将保持原样（除非被 `wildcard` 函数包裹）。</Note>

```makefile
thing_wrong := *.o # Don't do this! '*' will not get expanded
thing_right := $(wildcard *.o)

all: one two three four

# Fails, because $(thing_wrong) is the string "*.o"
one: $(thing_wrong)

# Stays as *.o if there are no files that match this pattern :(
two: *.o 

# Works as you would expect! In this case, it does nothing.
three: $(thing_right)

# Same as rule three
four: $(wildcard *.o)
```

## 通配符 `%`

`%` 虽然确实很有用，但是由于它的可用场景多种多样，着实有点让人迷惑。

- 在“匹配”模式下使用时，它匹配字符串中的一个或多个字符，这种匹配被称为词干（stem）匹配。
- 在“替换”模式下使用时，它会替换匹配到的词干。
- `%` 大多用在规则定义以及一些特定函数中。

有关它的用法示例可参阅以下章节：

- [静态模式规则](fancy-rules#静态模式规则)
- [模式规则](fancy-rules#模式规则)
- [字符串替换](functions#字符串替换)
- [`vpath` 指令](other-features#vpath-指令)

## 自动变量

虽然存在很多 [自动变量](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)，但是经常用到的没几个：

```makefile
hey: one two
    # Outputs "hey", since this is the first target
    echo $@

    # Outputs all prerequisites newer than the target
    echo $?

    # Outputs all prerequisites
    echo $^

    touch hey

one:
    touch one

two:
    touch two

clean:
    rm -f hey one two
```
