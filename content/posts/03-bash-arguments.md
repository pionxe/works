---
title: "参数、变量与数组"
date: 2026-08-12
tags: ["Bash"]
description: "执行脚本时 shell 自动把参数塞进 $0-$9，最多九个，靠 IFS 切分。"
---


**核心机制：** 执行 `./script.sh ARG1 ARG2 ... ARG9` 时，shell 自动把参数塞进预定义好的特殊变量：
`$0=script.sh $1=ARG1 ... $9=ARG9`
- 最多 9 个 （$0 被脚本名占了）
- 参数靠 IPS（Internal Field Separator， 默认空格）切分——“哪个字符结束一个参数、开始下一个”有它决定
**注意：** 
