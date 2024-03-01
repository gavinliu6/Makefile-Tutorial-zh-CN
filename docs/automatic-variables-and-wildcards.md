# 自动变量和通配符

## 通配符 `*`

在 Make 中，`%` 和 `*` 都叫作通配符，但是它们是两个完全不同的东西。`*` 会搜索你的文件系统来匹配文件名。我建议你应该一直使用 `wildcard` 函数来包裹它，要不然你可能会掉入下述的常见陷阱中。真是搞不明白，不用 `wildcard` 包裹的 `*` 除了能给人徒增迷惑，还有什么可取之处。

```makefile
# 打印出每个.c文件的文件信息
print: $(wildcard *.c)
	ls -la  $?
```

`*` 可以用在 `targets`、`prerequisites` 以及 `wildcard` 函数中。

<Note type="danger">`*` 不能直接用在变量定义中。</Note>

<Note type="danger">当 `*` 匹配不到文件时，它将保持原样（除非被 `wildcard` 函数包裹）。</Note>

```makefile
thing_wrong := *.o # 请不要这样做！'*.o' 将不会被替换为实际的文件名
thing_right := $(wildcard *.o)

all: one two three four

# 失败，因为$(thing_wrong)是字符串"*.o"
one: $(thing_wrong)

# 如果没有符合这个匹配规则的文件，它将保持为 *.o   :(
two: *.o 

# 按预期运行！在这种情况下，什么都不会执行
three: $(thing_right)

# 与规则三相同
four: $(wildcard *.o)
```

## 通配符 `%`

通配符 `%` 虽然确实很有用，但是由于它的可用场景多种多样，着实有点让人摸不着头脑。

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
	# 输出"hey"，因为这是第一个目标
	echo $@

	# 输出所有比目标新的依赖
	echo $?

	# 输出所有依赖
	echo $^

	touch hey

one:
	touch one

two:
	touch two

clean:
	rm -f hey one two
```
