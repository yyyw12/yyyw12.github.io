---
title: "Makefile 入门：从三行规则到完整构建"
date: 2026-10-18
draft: false
author: "yyyw12"
summary: "从最朴素的目标/依赖/命令三段式，到变量、自动变量、模式规则和伪目标，一步步把 Makefile 写规范。"
description: "Makefile 语法、变量、自动变量与伪目标"
categories: ["构建工具"]
tags: ["makefile", "make", "gcc", "c语言"]
cover:
    image: ""
    alt: "Makefile"
ShowToc: true
TocOpen: false
---

## 一、为什么要用 make

一个 C 项目有多个源文件时，手工敲 `gcc main.c hello.c -o hello` 有两个问题：
每次都要全量重编、文件一多根本记不住依赖关系。`make` 的核心就是
**「目标 + 依赖 + 命令」** 三段式，靠比较文件时间戳来做增量编译。

## 二、基础版

```makefile
hello: main.o hello.o
	gcc -o hello main.o hello.o

main.o: main.c hello.h
	gcc -c main.c

hello.o: hello.c hello.h
	gcc -c hello.c

clean:
	rm -f hello main.o hello.o

.PHONY: clean
```

两个必须注意的点：

1. **命令行前必须是 Tab**，用空格会报 `missing separator`。
2. `.PHONY` 声明伪目标，避免目录里恰好有个叫 `clean` 的文件时规则不执行。

## 三、用变量和自动变量

```makefile
CC      = gcc
CFLAGS  = -Wall -g -O2
LDFLAGS = -lm
TARGET  = hello
OBJS    = main.o hello.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^ $(LDFLAGS)

%.o: %.c hello.h
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(TARGET) $(OBJS)
```

| 自动变量 | 含义 |
| --- | --- |
| `$@` | 当前目标 |
| `$^` | 全部依赖 |
| `$<` | 第一个依赖 |

`%.o: %.c` 是**模式规则**，所有 `.c` 到 `.o` 的编译都套用这一条，以后增加源文件
只要改 `SRCS` 变量就行。

## 四、增强版：把常用操作固化成目标

```makefile
run: $(TARGET)
	./$(TARGET)

depends:
	@echo "源文件: $(SRCS)"
	@echo "目标文件: $(OBJS)"

help:
	@echo "  make all   - 编译程序"
	@echo "  make run   - 编译并运行"
	@echo "  make clean - 清理编译文件"

.PHONY: all run clean distclean depends help
```

命令前的 `@` 表示不回显命令本身，只显示 `echo` 的输出，编译过程看起来清爽很多。

## 五、小结

Makefile 的演进路线非常清晰：

1. 先把重复的命令抽成**变量**；
2. 再把重复的规则抽象成**模式规则**；
3. 最后用多个**目标**把编译、运行、清理、帮助这些常用操作固化下来。

这套思路和实际工程里的 Makefile 组织方式是一致的。
