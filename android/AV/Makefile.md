# Makefile
Makefile是一种构建工具，能够自动化编译构建出一个可执行程

## c代码编译打包过程
- gcc -c test.c 编译成二进制 test.o文件
- gcc test.o -o app 构建出可执行程序app
如果一个庞大的工程中有很多c文件，可行而知构建出一个可执行程序该是多大的工程量，所以Makefile应运而生

## Makefile 三要素
- 目标 需要编译的成的文件名称
- 依赖 编译所依赖的文件
- 命令 编译命令
```makefile 
app:test.o
  gcc test.o -o app
  
test.o:test.c
  gcc -c test.c
```

### 特殊目标
有时会需要执行没有依赖的命令，比如清楚临时文件
```makefile
.PHONY:clean //伪目标，在当前目录存在clean文件时而冲突时的解决办法
clean:
  #删除所有.o文件
  rm -f *.o 
```

## 变量
makefile文件中定义变量
```makefile
FILES=test.o test2.o test3.o

#引用变量
app:$(FILES)
  gcc $(FILES) -o app
```

### 自动化变量
- $^ 代表所有依赖
- $@ 代表目标
  
### 通配符
```makefile
#通配对应one.o:one.c  two.o:two.c
%.o:%.c 
```

### 递归展开式
可以引用还没有定义的变量，会往下找相应的变量
```makefile
str2 = $(str1)
str1 = test
```
### 直接展开式
被引用的变量需在引用之前定义
```makefile
str := test
str2 = $(str)
```
### 变量追加
```makefile

```

## 函数
- wildcard
```makefile
#所有的.c文件
SOURCES=$(wildcard *.c)
```

- patsubst 替换
```makefile
#所有的.c替换成.o
OBJECTS=$(patsubst %.c,%.o,$(SOURCES))
```
