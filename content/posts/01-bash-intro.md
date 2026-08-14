---
title: "Bash 是什么？"
date: 2026-08-05
tags: ["Bash"]
description: "Bash 是 Unix 系系统的脚本语言。三种执行方式、脚本语言与编程语言、解释器与编译器。"
---


Bash 是 Unix 系系统的脚本语言，用来和系统“对话”、下命令。

## 脚本语言 VS. 编程语言

一句话：脚本语言写完就能跑，编程语言写完需要先编译。

## 存在三种执行方式

```bash
bash script.sh <可选参数> # 显示指定解释器 bash
sh script.sh <可选参数> # 指定 sh (bash 的兼容模式)
./script.sh <可选参数> # 直接执行，靠文件第一行 #!/bin/bash 指定解释器
```

前两种只需要`r`权限，第三种需要`rx`权限。
因为第三种是将脚本当作可执行二进制文件处理；而前两种是脚本作为参数让系统使用解释器执行。

## 解释器 VS. 编译器

两者都是语言翻译官，目标都是把高级语言转成可执行形式。
区别是：
- 解释器是边翻译边执行，不生成二进制文件，而是交由系统执行。
- 编译器是先翻译完再执行，产出可执行文件。

# 命令速查表

| 命令/ 符号        | 作用                                           | 例子                                                                |
| ------------- | -------------------------------------------- | ----------------------------------------------------------------- |
| $#            | 给脚本传的参数个数                                    | if [ $# -eq 0 ]                                                   |
| $0            | 当前脚本路径或文件名                                   | echo -e "\t$0 <domain>"                                           |
| $n  (n= 1-9)  | 传递给脚本的第n个参数                                  | domain=$1                                                         |
| $?            | 上条命令的退出状态码（0 表示成功，非0表示失败）                    | if [ $? -eq 0 ]                                                   |
| -eq           | 数值比较：等于 (Equal)                              | if [ $# -eq 0 ]                                                   |
| >             | 重定向符，标准输出重定向（覆写文件或设备）                        | > /dev/null                                                       |
| >& / 2>&1     | 文件描述符重定向（2>&1 表示将标准错误stderr合并重定向到标准输出stdout） | > /dev/null 2>&1                                                  |
| echo          | 在终端输出文本（-e 参数用来激活转义字符）                       | echo -e "Usage:\n"                                                |
| exit          | 退出当前脚本并返回指定退出码                               | exit 1                                                            |
| read          | 读取用户从终端输入的文本并保存到变量中（-p 用于输出提示信息）             | read -p "Select your option: " opt                                |
| case ... esac | 多分支选择结构（类似 switch-case）                      | case $opt in ... esac                                             |
| $()           | 命令替换，把命令输出当值用                                | netrange=$(whois $ip \| grep "NetRange\|CIDR" \| tee -a CIDR.txt) |
| ((...))       | 算数运算语法，用于执行整数自增/自减或计算                        | ((hosts_up++))                                                    |
| grep          | 按行筛选/检索模式字符串（使用\|可实现逻辑“或”匹配）                 | grep "NetRange\|CIDR"                                             |
| tee           | 双向重定向：既将数据输出到终端，同时又追加(-a)或写入到文件中             | tee -a CIDR.txt                                                   |
| awk           | 强大的文本格式化与处理工具（此处用于提取每一行的第2列字段）               | awk '{print $2}'                                                  |
| cut           | 按分隔符截取文本（-d" " 指定空格为分隔，-f4 提取第四个字段）          | cut -d" " -f4                                                     |
| tr            | 字符转换/替换（此处将行尾的换行符\n替换为空格，合并为单行）              | tr "\n" " "                                                       |
| whois         | 查询域名或IP地址的WHOIS 注册/网段信息                      | whois $ip                                                         |
| prips         | 将CIDR格式的网段（如192.168.1.0/24）展开为具体的IP地址列表      | prips $cidr                                                       |
| host          | DNS查询工具，用于获取域名对应的IP地址记录                      | host $domain                                                      |
| &&            | 命令链控制：只有在前一条命令执行成功（返回退出码 0）时，才执行后一条命令。       | "3") network_range && ping_host ;;                                |
## CIDR.sh

```bash
#!/bin/bash

# Check for given arguments
if [ $# -eq 0 ]
then
    echo -e "You need to specify the target domain.\n"
    echo -e "Usage:"
    echo -e "\t$0 <domain>"
    exit 1
else
    domain=$1
fi

# Identify Network range for the specified IP address(es)
function network_range {
    for ip in $ipaddr
    do
        netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
        cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
        cidr_ips=$(prips $cidr)
        echo -e "\nNetRange for $ip:"
        echo -e "$netrange"
    done
}

# Ping discovered IP address(es)
function ping_host {
    hosts_up=0
    hosts_total=0
    
    echo -e "\nPinging host(s):"
    for host in $cidr_ips
    do
        stat=1
        while [ $stat -eq 1 ]
        do
            ping -c 2 $host > /dev/null 2>&1
            if [ $? -eq 0 ]
            then
                echo "$host is up."
                ((stat--))
                ((hosts_up++))
                ((hosts_total++))
            else
                echo "$host is down."
                ((stat--))
                ((hosts_total++))
            fi
        done
    done
    
    echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

echo -e "Discovered IP address:\n$hosts\n"
ipaddr=$(host $domain | grep "has address" | cut -d" " -f4 | tr "\n" " ")

# Available options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
    "1") network_range ;;
    "2") ping_host ;;
    "3") network_range && ping_host ;;
    "*") exit 0 ;;
esac
```


## 练习：改造ping_host

```bash
stat=3
up=0
while [ $stat -gt 0 ]
do
    ping ...
    if 成功
    then
        up=1
        stat=0
    else
        ((stat--))
    fi
done
循环结束后用 up 判断 → hosts_up++ / hosts_total++
```

## 脚本流程

```
1. 检查参数（没有参数-> 报错退出）
2. 解析域名（获取IP列表）
3. 显示菜单，等待用户输入
4. 按选择执行对应函数
```

# 问题

1. $( ) 和 $1 有什么区别？
   $( )是内部的输出值作为参数，$1是在启动脚本时传入的第一个参数。
2. 为什么 bash script.sh 和 ./script.sh 两种方式执行，$1 都能拿到参数？
   因为这两种方式只是启动方式不同。
   $1 属于正在运行脚本的那个 bash 进程的位置参数。
   - bash script.sh x：你直接让 bash 跑，参数顺序 = 脚本 + 后面的 x → $0=script.sh、$1=x
   - ./script.sh x：内核看到文件头 #!/bin/bash，替你执行 bash script.sh x
   - 殊途同归，bash 拿到的参数列表一样，所以 $1 都是 x
3. "\*") 分支什么时候才轮到它执行？
   除了已有的选择（1、2、3）外，其他任意输入都执行这分支。

