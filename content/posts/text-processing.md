---
title: "grep / sed / awk：文本处理三剑客"
date: 2026-10-20
draft: false
author: "yyyw12"
summary: "按「找、改、算」三种需求分别介绍 grep、sed、awk，再用管道把它们串起来做组合统计。"
description: "grep、sed、awk 的用法与管道组合"
categories: ["Linux基础"]
tags: ["grep", "sed", "awk", "正则", "管道"]
cover:
    image: ""
    alt: "文本处理"
ShowToc: true
TocOpen: false
---

## 一、三个工具各管一摊

| 工具 | 侧重点 | 典型场景 |
| --- | --- | --- |
| `grep` | 找：按行筛选 | 在日志里找关键字 |
| `sed` | 改：按行替换/删除 | 批量改配置文件 |
| `awk` | 算：按字段处理 | 统计、求和、报表 |

它们都是「逐行处理」，但设计目标不同，**不能互相替代**。

## 二、grep

```bash
grep ComputerScience student_scores.txt   # 精确匹配
grep -i "geo" student_scores.txt          # 忽略大小写
grep -n Female student_scores.txt         # 显示行号
grep -v Male student_scores.txt           # 反向匹配
grep -c Geology student_scores.txt        # 只输出匹配行数
grep -E "9[0-9]" student_scores.txt       # 扩展正则
grep -rn "ssh" /etc/ssh                   # 递归搜索目录
```

## 三、sed

```bash
sed 's/ComputerScience/CS/g' student_scores.txt     # 只预览，不改原文件
sed -i.bak 's/ComputerScience/CS/g' student_scores.txt   # 修改并自动备份
sed '3d' student_scores.txt                          # 删除第 3 行
sed -n '2,4p' student_scores.txt                     # 只打印 2~4 行
sed '/Geology/d' student_scores.txt                  # 删除匹配行
```

`-i` 才会真正改文件，`-i.bak` 会先存一份 `.bak`。**改配置文件前先不加 `-i` 预览**是保命习惯。

## 四、awk

awk 以字段为单位，默认按空白切分：

```bash
awk '{print $1, $2}' student_scores.txt              # 打印前两列
awk '$2 > 80 {print $1, $2}' student_scores.txt      # 条件过滤
awk '{sum+=$2} END {printf "总分: %.2f, 平均分: %.2f\n", sum, sum/NR}' student_scores.txt
awk -F: '{print $1, $3}' /etc/passwd | head          # 指定冒号为分隔符
```

`NR` 是行号，`END` 块在所有行处理完后执行，数组 `sum[$4]` 可以按专业分组累加。

## 五、用管道组合

「小工具 + 管道」是 Linux 的经典哲学，每个命令只做好一件事，串起来就能完成复杂处理：

```bash
# 统计各专业人数
awk '{print $4}' student_scores.txt | sort | uniq -c

# 取最高分前三名
cat student_scores.txt | sort -k2 -n | tail -3

# 过滤 + 改写 + 另存
grep -E "8[0-9]|9[0-9]" student_scores.txt > high_scores.txt
```

## 六、小结

记住三个关键词就够了：**grep 找、sed 改、awk 算**。真正提高效率的是把它们用管道串起来，
一次命令完成「筛选 → 取列 → 排序 → 统计」的完整流程。
