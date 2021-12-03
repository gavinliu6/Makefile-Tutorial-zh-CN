# 目标

## 目标 `all`

想制作多个目标，并且一次运行全部？那就使用 `all` 目标吧。

```makefile
all: one two three

one:
    touch one
two:
    touch two
three:
    touch three

clean:
    rm -f one two three
```

<Note type="tip" label="译者注">

关于目标 `all` 的更多信息可参阅 [Stack Overflow - What does “all” stand for in a makefile?](https://stackoverflow.com/questions/2514903/what-does-all-stand-for-in-a-makefile)

</Note>

## 多目标

当一个规则（rule）有多个目标时，那么对于每个目标，这个规则下面的 commands 都会运行一次。

`$@` 是一个指代目标名称的 [自动变量](automatic-variables-and-wildcards#自动变量)。

```makefile
all: f1.o f2.o

f1.o f2.o:
    echo $@
# Equivalent to:
# f1.o
#     echo $@
# f2.o
#     echo $@
```
