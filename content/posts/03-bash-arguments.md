---
title: "参数、变量与数组"
date: 2026-08-12
tags: ["Bash"]
description: "执行脚本时 shell 自动把参数塞进 $0-$9，最多九个，靠 IFS 切分。"
---
## 参数

**核心机制：** 执行 `./script.sh ARG1 ARG2 ... ARG9` 时，shell 自动把参数塞进预定义好的特殊变量：
`$0=script.sh $1=ARG1 ... $9=ARG9`
- 最多 9 个 （$0 被脚本名占了）
- 参数靠 `IFS`（Internal Field Separator， 默认空格）切分——“哪个字符结束一个参数、开始下一个”由它决定
**注意：** 严格说 bash 能接收超过9个参数，但第10个起需要写 `${10}` （花括号，否则 `$10` 会被解析成 `$1` 后面跟个0）

## 特殊变量

| 变量  | 含义        |
| --- | --------- |
| $#  | 参数个数      |
| $@  | 全部参数的列表   |
| $n  | 第 n 个参数   |
| $$  | 当前进程的 PID |
| $?  | 上一条命令的退出码 |

## 变量

三条铁律：
1. 赋值不带 `$` , 用值才带 `$`： `domain=$1` 是赋值； `echo $domain` 是用值。
2. `=` 两边不能有空格。`variable = "x"`会被拆成三个词，shell 把`variable` 当命令执行，`command not found`。
3. bash 没有类型：所有变量都是字符串。只是bash 遇到算术场景（如`((x++))`）会自动当数字算。

## 数组

```bash
domain=(a.com b.com)  # 声明：空格分隔元素
echo ${domain[0]}     # 取值：从下标0开始，必须花括号
```

- 为什么是 `${domain[0]}`，而不是`$domain[0]`？没有花括号的话，bash 会把`$domain`展开（数组不加下标=第0个元素），然后`[0]`就是字面量，花括号就是告诉整个shell“整个 domain[0] ”都是变量。花括号用于变量展开。
- 引号的作用：`("a.com b.com c.com" d.com)`，引号内的空格被冻结，`${domain[0]}` 输出的是一个带有空格的字符串。IFS 规则：引号能阻止按IFS切分。


# 命令速查表

| 命令 / 符号          | 作用                                                                                         | 例子                                                                                                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $#               | 传递给脚本或函数的**参数总个数**。                                                                        | if [ $# -lt 2 ]; then echo "至少需要2个参数"; fi                                                                                                                          |
| $n               | 传递给脚本的**第 n 个参数**（第 1 到 9 个写 `$1`~`$9`，**10 及以上需加花括号**如 `${10}`）。                          | `domain=$1`<br>`backup_dir=${10}`                                                                                                                                  |
| $@               | **所有位置参数的列表**。当用双引号包围 `"$@"` 时，会保留每个参数的独立性（展开为 `"$1" "$2" "$3"...`，**推荐遍历时使用**）。           | `for arg in "$@"; do echo "$arg"; done`                                                                                                                            |
| $*               | **所有位置参数合并为一个整体字符串**。当用双引号包围 `"$*"` 时，参数会用 IFS 变量（默认空格）拼接为一个单一字符串（展开为 `"$1 $2 $3..."`）。    | echo "所有输入参数：$*"                                                                                                                                                   |
| $?               | **上一条执行命令的退出状态码**（`0` 表示成功，非 `0` 表示发生错误）。                                                  | `ping -c 1 8.8.8.8 > /dev/null`<br>`if [ $? -eq 0 ]; then echo "网络正常"; fi`                                                                                         |
| ${}              | **变量边界界定与高级操作**：<br>1. 明确变量名边界（防止字符混淆）<br>2. 支持默认值、字符串截取、替换等扩展语法。                          | 边界界定：`echo "${var}_log.txt"`<br>默认值处理：`name=${USER_INPUT:-"default"}`                                                                                              |
| =                | 1. **变量赋值**（等号两边**严禁有空格**）。<br>2. **字符串判断**（在 `[ ]` 或 `[[ ]]` 中作为字符串相等比较）。                 | 变量赋值：`domain=$1`<br>字符串比较：`if [ "$opt" = "1" ]; then`                                                                                                              |
| ()               | 1. **在子 Shell（Subshell）中执行命令**：括号内的命令在独立的子进程中运行，环境变更（如 `cd`）不会影响父进程。<br>2. **定义数组**。       | 子 Shell：`(cd /tmp && ls)` _(主脚本路径不会改变)_<br>定义数组：`my_array=(apple banana orange)`                                                                                   |
| chmod +x         | **赋予可执行权限**（**ch**ange **mod**e **+**e**x**ecute）：使脚本文件具备直接执行的权限，允许通过 `./script.sh` 的方式运行。 | `chmod +x scan_network.sh`<br>`./scan_network.sh`                                                                                                                  |
| pipefail         | 改变管道退出状态码的计算规则。开启后，整个管道的返回值为**最后一个以非 0 状态退出的命令的返回值**；如果所有命令都成功（返回 0），才返回 0。                | **开启**：`set -o pipefail`<br>**关闭**：`set +o pipefail`                                                                                                               |
| ${PIPESTATUS[@]} | 查看管道里“每一个”命令的状态                                                                            | cat /non_existent_file \| grep "foo" \| wc -l<br><br># 查看管道中每个命令的退出状态<br>echo "${PIPESTATUS[@]}"<br># 输出类似: 1 1 0<br># 分别代表: cat 失败(1) \| grep 没匹配到(1) \| wc 成功(0) |


# 练习

写一个 resolver.sh，用法 `./resolver.sh inlanefreight.com hackthebox.com`，把收到的所有域名收进数组 → 逐个解析出 IP → 最后报告"共收到 N个域名，成功解析 M 个"。

```bash
#!/bin/bash

set -o pipefail
# count the argument num and asign the array
domains=("$@")
num=0

for ((i = 0; i < ($#); i++)); do
  cur_domain="${domains[i]}"
  ip_info=$(host "$cur_domain" | grep "has address" | awk '{print $4}')

  if [ $? -eq 0 ]; then
    echo "$cur_domain ip is $ip_info"
    ((num++))
  else
    echo "$cur_domain 解析失败"
  fi
done

echo "共收到 $# 个域名，成功解析 $num 个"
```

或者
```bash
#!/bin/bash

# count the argument num and asign the array
domains=("$@")
num=0

for ((i = 0; i < ($#); i++)); do
  cur_domain="${domains[i]}"
  ip_info=$(host "$cur_domain" | grep "has address" | awk '{print $4}')

  if [ -n "$ip_info" ]; then
    echo "$cur_domain ip is $ip_info"
    ((num++))
  else
    echo "$cur_domain 解析失败"
  fi
done

echo "共收到 $# 个域名，成功解析 $num 个"
```

**注意：** 管道陷阱，$? 在管道中正常会以最后一个执行的退出码。