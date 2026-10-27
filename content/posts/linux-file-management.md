---
title: "Linux 文件管理：从 mkdir 到硬链接"
date: 2026-10-08
draft: false
author: "yyyw12"
summary: "整理实验二里的目录与文件操作，重点讲清楚 cp/mv 的参数陷阱，以及硬链接与软链接在 inode 层面的区别。"
description: "Linux 文件管理命令与硬链接、软链接原理"
categories: ["Linux基础"]
tags: ["linux", "命令", "文件管理", "inode"]
cover:
    image: ""
    alt: "Linux 文件管理"
    caption: ""
ShowToc: true
TocOpen: false
---

## 一、目录与文件的基本操作

Linux 里所有内容都挂在同一棵目录树上，所以「创建目录」和「创建文件」是两类不同的动作：

```bash
mkdir -p file_experiment/{dir1,dir2,dir3}   # 一次性建三个子目录
touch file1.txt file2.txt file3.log        # 建空文件
echo "Hello Linux" > file1.txt             # 覆盖写入
echo "This is file2" > file2.txt
ls -l                                       # 看文件属性
```

`-p` 是关键：没有它，父目录不存在时 `mkdir` 会直接报错；有了它就会自动补全整条路径，而且目标已存在时也不报错。

## 二、cp 与 mv 的参数陷阱

- `cp` 只能复制文件，**复制目录必须加 `-r`**，否则报 `omitting directory`。
- `mv` 既可以移动也可以重命名，`mv file3.log dir3/error.log` 就是「移动 + 改名」一步完成。
- 通配符由 shell 展开后再传给命令，所以 `mv *.txt dir1/` 实际上等价于把展开后的每个文件名都传给 `mv`。

```bash
cp -r dir1 dir1_backup      # 递归复制整个目录
mv file3.log dir3/error.log # 移动并改名
mv *.txt dir1/              # 批量移动
tree .                      # 树形查看结果
```

## 三、硬链接与软链接

这是实验里最容易混淆的一对概念，用 `ls -li` 一看就清楚了：

```bash
ln file1.txt hard_link_file1     # 硬链接
ln -s file1.txt soft_link_file1  # 软链接
ls -li
```

结果里硬链接和源文件的 **inode 号完全相同**、链接计数从 1 变成 2；软链接则有独立的 inode，内容只是一个路径字符串。

| 对比项 | 硬链接 | 软链接 |
| --- | --- | --- |
| inode | 与源文件相同 | 独立 inode |
| 能否跨文件系统 | 不能 | 能 |
| 能否指向目录 | 不能 | 能 |
| 源文件删除后 | 数据仍在 | 变成断链 |
| 占用空间 | 几乎不占 | 存一条路径 |

`file` 命令也能直接看出来：

```bash
file hard_link_file1 soft_link_file1
# hard_link_file1: ASCII text
# soft_link_file1: symbolic link to file1.txt
```

## 四、小结

文件名只是 inode 的一个「入口」，真正存数据的是 inode 指向的数据块。理解了这一点，
就能解释为什么删除源文件后硬链接还能读到数据，而软链接会失效。
