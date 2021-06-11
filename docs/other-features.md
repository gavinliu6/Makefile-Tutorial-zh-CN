# 其他特性

## 包含 Makefiles 文件

`include` 指令告诉 `make` 去读取其他 makefiles 文件，它是 makefile 中的一行，如下所示：

```makefile
include filenames...
```

当你使用像 `-M` 这样的编译器标志时，`include` 特别有用，它可以根据源代码创建 Makefile。例如，如果一些 c 文件包括一个头，这个头将被添加到由 gcc 编写的 Makefile 中。关于这一点，我在 [Makefile Cookbook](makefile-cookbook) 一节中有更详细的讨论。

## `vpath` 指令

`vpath` 指令用来指定某些 `prerequisites` 的位置，使用格式是 `vpath <pattern> <directories, space/colon separated>`。

`<pattern>` 中可以使用 `%`，用来匹配 0 个或多个字符。

你也可以使用变量 `VPATH` 全局执行此操作。

```makefile
vpath %.h ../headers ../other-directory

some_binary: ../headers blah.h
    touch some_binary

../headers:
    mkdir ../headers

blah.h:
    touch ../headers/blah.h

clean:
    rm -rf ../headers
    rm -f some_binary
```

## 多行处理

当命令过长时，反斜杠（`\`）可以让我们使用多行编写形式。

```makefile
some_file: 
    echo This line is too long, so \
        it is broken up into multiple lines
```

## `.PHONY`

向一个目标中添加 `.PHONY` 会避免把一个虚拟目标识别为一个文件名。在下面这个例子中，即便文件 `clean` 被创建了，`make clean` 仍会运行。`.PHONY` 非常好用，但是为了简洁起见，在其余示例中我会跳过它。

```makefile
some_file:
    touch some_file
    touch clean

.PHONY: clean
clean:
    rm -f some_file
    rm -f clean
```

## `.DELETE_ON_ERROR`

如果一个命令返回了一个非 0 的退出码，那么 `make` 会停止运行相应的规则（并会传播到它的依赖中）。

如果一个规则因为上述这种情况构建失败了，那么应用了 `.DELETE_ON_ERROR` 后，这个规则的目标文件就会被删除。不像 `.PHONY`，`.DELETE_ON_ERROR` 对所有的目标都有效。始终使用 `.DELETE_ON_ERROR` 是个不错的选择，即使由于历史原因，`make` 不支持它。

```makefile
.DELETE_ON_ERROR:
all: one two

one:
    touch one
    false

two:
    touch two
    false
```
