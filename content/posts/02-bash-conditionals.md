---
title: "条件执行"
date: 2026-08-11
tags: ["Bash"]
description: "if-else 家族与 case 语句：命中哪个条件就跑哪段，其余跳过。Shebang、命令速查表、练习。"
---



条件执行让你“命中哪个条件就只跑哪段代码，其余跳过；跑完这段，再回到条件外面继续”。

## 条件执行的两种形式

- `if-else` 家族
- case 语句

## Shebang

- 在脚本的第一行，以`#!` 开头，后面跟解释器的路径。
- 决定以`./script.sh` 方式执行时，内核去找谁解释这个文件。
- 不止 bash 能用，Python/Perl 也行：
  `#!/usr/bin/env python`
  `#!/usr/bin/env perl`
  （`/usr/bin/env` 表示“去环境变量 PATH 里找”）

## if 家族语法骨架

```bash
if [ condition ]   # 只有 if
then
	...
fi

if [ condition ]   # if + else
then
	...
else
	...
fi

if [ condition ]   # if + elif + else (elif 可以写多个)
then
	...
elif [ condtion ]
then
	...
else
	...
fi

```

# 命令速查表

| 命令/ 符号                | 作用                                                                                                                                | 例子                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| if / elif / else / fi | **条件分支判断**：`if`（如果）、`elif`（否则如果）、`else`（否则），最后以倒写 `fi` 闭合结束。                                                                      | if [ $# -eq 0 ]                                                                    |
| then                  | **条件触发标志**：配合 `if` 或 `elif` 使用，紧跟在条件表达式后，表示“当条件成立时执行以下代码”。                                                                        | if [ $stat -eq 1 ]; then                                                           |
| -gt                   | 数值比较：**大于**（**G**reater **T**han）                                                                                                 | if [ $hosts_total -gt 0 ]                                                          |
| {1..40}               | **生成连续序列**（Brace Expansion）：自动展开为 1 到 40 的连续数字，常用于循环。                                                                             | for i in {1..40}; do echo $i; done                                                 |
| wc                    | **文本统计工具**（**W**ord **C**ount）：用于统计行数（`-l`）、词数（`-w`）或字节/字符数（`-c`）。                                                                | cat discovered_hosts.txt \| wc -l                                                  |
| base64                | **Base64 编码/解码工具**：默认编码输入流，加上 `-d` 参数用于反向解码。                                                                                      | 编码：`echo -n "secret" \| base64`<br><br>  解码：`echo "c2VjcmV0" \| base64 -d`<br><br> |
| /usr/bin/env          | **动态解析解释器路径**：常用于脚本首行 Shebang（`#!`）。在当前用户的 `$PATH` 路径中查找指定程序，提高脚本在不同 Linux/Unix 发行版间的**跨平台移植性**（避免硬编码 `/bin/bash` 导致不同系统路径找不到报错）。 | `#!/usr/bin/env bash`                                                              |
| base64 -w             | **设置输出自动换行列数**（通常配合 `0` 使用，即 `-w 0` 或 `-w0`）<br>GNU `base64` 默认每 76 字符插一个换行符，加 `-w 0` 可以**禁用换行**，将所有编码结果输出在同一行。                   | echo -n "secret" \| base64 -w 0                                                    |
|                       |                                                                                                                                   |                                                                                    |
| echo -n               | **取消末尾换行符**：默认 `echo` 输出完内容后会自动在末尾加上一个换行符（`\n`）。使用 `-n` 可以**防止输出结尾换行**。                                                           | echo -n "my_password"                                                              |
# 练习

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -m

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}; do
  var=$(echo $var | base64)
done

echo $var | wc -m
```


## 注意

```bash
#!/bin/bash 

value=$1 
if [ $value -gt "10" ] 
then 
	echo "Given argument is greater than 10." 
elif [ $value -lt "10" ] 
then 
	echo "Given argument is less than 10." 
else 
	echo "Given argument is not a number." 
fi


----
# 返回
bash if-elif-else.sh HTB 
if-elif-else.sh: line 5: [: HTB: integer expression expected 
if-elif-else.sh: line 8: [: HTB: integer expression expected 
Given argument is not a number.

```

[ $value -gt "10" ] 拿非数字去比，test 命令报错并返回非 0（不 crash，只算"条件为假"），
报错但不崩溃 → 落入 else 兜底（bash 没有类型）。

