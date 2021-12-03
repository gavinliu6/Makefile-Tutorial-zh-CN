# 命令与执行

## 回显/静默命令

在一个命令前添加一个 `@` 符号就会阻止该命令输出内容。

你也可以使用 `make -s` 在每个命令前添加 `@`。

```makefile
all: 
    @echo "This make line will not be printed"
    echo "But this will"
```

## 命令的执行

每个命令都运行在一个新的 shell 中（或者说运行效果等同于运行在一个新 shell 中）。

```makefile
all: 
    cd ..
    # The cd above does not affect this line, because each command is effectively run in a new shell
    echo `pwd`

    # This cd command affects the next because they are on the same line
    cd ..;echo `pwd`

    # Same as above
    cd ..; \
    echo `pwd`
```

## 默认的 Shell

系统默认的 shell 是 `/bin/sh`，你可以通过改变 `SHELL` 变量的值来改变它：

```makefile
SHELL=/bin/bash

cool:
    echo "Hello from bash"
```

## 错误处理：`-k`，`-i` 和 `-`

`make -k` 会使得即便遇到错误，构建也会继续执行下去。如果你想一次查看 Make 的所有错误，这会很有帮助。

在一个命令前添加 `-` 会抑制错误。

`make -i` 等同于在每个命令前添加 `-`。

```makefile
one:
    # This error will be printed but ignored, and make will continue to run
    -false
    touch one
```

## 中断或杀死 `make`

<Note>如果你在 `make` 的过程中，使用了 `ctrl+c`，那么刚刚制作的新目标会被删除。</Note>

## `make` 的递归用法

为了递归应用一个 makefile，请使用 `$(MAKE)` 而不是 `make`，因为它会为你传递构建标志，而使用了 `$(MAKE)` 变量的这一行命令不会应用这些标志。

```makefile
new_contents = "hello:\n\ttouch inside_file"
all:
    mkdir -p subdir
    printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
    cd subdir && $(MAKE)

clean:
    rm -rf subdir
```

## 在递归 make 中使用 `export`

指令 `export` 携带了一个变量，并且对子 `make` 命令可见。在下面的例子中，变量 `cooly` 被导出以便在子目录中的 makefile 可以使用它。

<Note>`export` 的语法与 sh 相同，但二者并不相关（虽然功能类似）。</Note>

```makefile
new_contents = "hello:\n\\techo \$$(cooly)"

all:
    mkdir -p subdir
    echo $(new_contents) | sed -e 's/^ //' > subdir/makefile
    @echo "---MAKEFILE CONTENTS---"
    @cd subdir && cat makefile
    @echo "---END MAKEFILE CONTENTS---"
    cd subdir && $(MAKE)

# Note that variables and exports. They are set/affected globally.
cooly = "The subdirectory can see me!"
export cooly
# This would nullify the line above: unexport cooly

clean:
    rm -rf subdir
```

在 shell 中运行的变量也需要导出。

```makefile
one=this will only work locally
export two=we can run subcommands with this

all: 
    @echo $(one)
    @echo $$one
    @echo $(two)
    @echo $$two
```

`.EXPORT_ALL_VARIABLES` 可以为你导出所有变量。

```makefile
.EXPORT_ALL_VARIABLES:
new_contents = "hello:\n\techo \$$(cooly)"

cooly = "The subdirectory can see me!"
# This would nullify the line above: unexport cooly

all:
    mkdir -p subdir
    echo $(new_contents) | sed -e 's/^ //' > subdir/makefile
    @echo "---MAKEFILE CONTENTS---"
    @cd subdir && cat makefile
    @echo "---END MAKEFILE CONTENTS---"
    cd subdir && $(MAKE)

clean:
    rm -rf subdir
```

## 给 `make` 传递参数

[这里](http://www.gnu.org/software/make/manual/make.html#Options-Summary) 很好地列出了可以用于 `make` 的选项。看下 `--dry-run`，`--touch` 和 `--old-file` 选项吧。

你可以同时传递多个目标给 `make`，例如 `make clean run test` 会先后运行 `clean`、`run`、`test`。
