---
title: "Shell 脚本实战：年龄分房与自动备份"
date: 2026-10-15
draft: false
author: "yyyw12"
summary: "两个小脚本：一个用 if/elif 按年龄段分配房间，一个用 for 循环批量备份并写带时间戳的日志。附 bash -x 调试心得。"
description: "Bash 条件判断、循环、日志与调试"
categories: ["Shell脚本"]
tags: ["shell", "bash", "脚本", "自动化"]
cover:
    image: ""
    alt: "Shell 脚本"
ShowToc: true
TocOpen: false
---

## 一、脚本 1：按年龄分配房间

需求很简单：读入年龄，20~29 去 101、30~39 去 201，依次类推，超出范围就提示等待。

```bash
#!/bin/bash
read -p "Input your age！ " age

# 先校验输入是不是纯数字
if ! [[ "$age" =~ ^[0-9]+$ ]]; then
    echo "错误：请输入有效的数字！"
    exit 1
fi

if [ "$age" -ge 20 ] && [ "$age" -le 29 ]; then
    echo "Please go to room 101！"
elif [ "$age" -ge 30 ] && [ "$age" -le 39 ]; then
    echo "Please go to room 201！"
else
    echo "Please wait at the door！"
fi
```

几个容易踩的点：

1. **数值比较不能用 `>`、`<`**，因为 shell 会把它们当成重定向。要用 `-ge`、`-le`、`-eq`。
2. **变量要加双引号**，否则变量为空时 `[ -ge 20 ]` 会直接语法报错。
3. **先校验再比较**：没有正则校验时，输入 `abc` 会让 `[ "abc" -ge 20 ]` 报错。

同一个逻辑也可以用 `case` 改写，因为年龄段恰好是整十：

```bash
case $age in
    [2][0-9]) echo "Please go to room 101！" ;;
    [3][0-9]) echo "Please go to room 201！" ;;
    *)        echo "Please wait at the door！" ;;
esac
```

`case` 是字符串模式匹配而不是数值比较，所以只适合这种「整十区间」。

## 二、脚本 2：批量备份 + 日志

```bash
#!/bin/bash
LOG_DIR="./backup_logs"
mkdir -p "$LOG_DIR"
TS=$(date +"%Y-%m-%d-%H-%M-%S")
LOG_FILE="$LOG_DIR/$TS.log"

echo "Backup Process Begins" | tee -a "$LOG_FILE"

for item in "$@"; do
    if [ -e "$item" ]; then
        cp -r "$item" "${item}.bak.$TS"
        echo "$TS  SUCCESS in backup file/directory($item)" | tee -a "$LOG_FILE"
    else
        echo "$TS  FAILED: $item not found" | tee -a "$LOG_FILE"
    fi
done

echo "Backup Process Ends" | tee -a "$LOG_FILE"
```

- `$@` 是所有命令行参数，`$#` 是参数个数，`$0` 是脚本名。
- `[ -e "$item" ]` 判断路径是否存在。
- `tee -a` 把输出同时送到屏幕和文件，`-a` 表示追加；不加 `-a` 每次都会覆盖日志。
- 用 `date +"%Y-%m-%d-%H-%M-%S"` 生成时间戳做后缀，多次备份不会互相覆盖。

## 三、用 bash -x 调试

脚本不按预期执行时，`bash -x 脚本` 会逐行打印实际执行的命令和变量展开后的值，
比到处加 `echo` 高效得多：

```bash
bash -x ./test2_backup.sh test_dir
```

输出里每行前面的 `+` 就是跟踪标记。如果只想跟踪某一段，可以在脚本里用
`set -x` 和 `set +x` 成对包起来。

## 四、小结

Shell 脚本的语法坑集中在空格、引号和比较符上；写熟练之后，
批量改名、定时备份、部署检查这类重复劳动都可以几行脚本搞定。
