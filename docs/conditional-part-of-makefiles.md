# Makefiles 的条件判断

## if/else

```makefile
foo = ok

all:
ifeq ($(foo), ok)
    echo "foo equals ok"
else
    echo "nope"
endif
```

## 检查一个变量是否为空

```makefile
nullstring =
foo = $(nullstring) # end of line; there is a space here

all:
ifeq ($(strip $(foo)),)
    echo "foo is empty after being stripped"
endif
ifeq ($(nullstring),)
    echo "nullstring doesn't even have spaces"
endif
```

## 检查一个变量是否定义

`ifdef` 不会扩展变量引用，它只会查看变量的内容究竟定义没。

```makefile
bar =
foo = $(bar)

all:
ifdef foo
    echo "foo is defined"
endif
ifdef bar
    echo "but bar is not"
endif
```

## `$(makeflags)`

此示例向你展示了如何使用 `findstring` 和 `MAKEFLAGS` 测试 `make` 标志。使用 `make -i` 来运行此例看下输出如何吧。

```makefile
bar =
foo = $(bar)

all:
# Search for the "-i" flag. MAKEFLAGS is just a list of single characters, one per flag. So look for "i" in this case.
ifneq (,$(findstring i, $(MAKEFLAGS)))
    echo "i was passed to MAKEFLAGS"
endif
```
