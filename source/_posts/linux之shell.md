---
title: linux之shell
date: 2020-08-25 20:08:03
keywords: 'linux,shell'
description: 学习shell的一些笔记
tags:
  - linux
  - 编程
categories: shell
cover: https://pic4.zhimg.com/v2-bffa1b54778faf63a2cfc687df55b5f1_1440w.jpg?source=172ae18b
---
>$# 是传给脚本的参数个数
$0 是脚本本身的名字
$1 是传递给该shell脚本的第一个参数
$2 是传递给该shell脚本的第二个参数
$@ 是传给脚本的所有参数的列表
$* 是以一个单字符串显示所有向脚本传递的参数，与位置变量不同，参数可超过9个
$$ 是脚本运行的当前进程ID号
$? 是显示最后命令的退出状态，0表示没有错误，其他表示有错误

## 1. if-then语句
```linux
if test command
then commands
else commands
fi
```
test检测：
数字
n1 -eq n2 检查n1是否与n2相等
n1 -ge n2 检查n1是否大于或等于n2
n1 -gt n2 检查n1是否大于n2
n1 -le n2 检查n1是否小于或等于n2
n1 -lt n2 检查n1是否小于n2
n1 -ne n2 检查n1是否不等于n2

字符串
-n str1 检查str1的长度是否非0
-z str1 检查str1的长度是否为0

## 2. for循环
```linux
for var in list do
commands
done
```
例子：
```linux
file=“states”
IFS=$'\n’   #能读空行的值
for state in $(cat $file)
do
   echo "Visit beautiful $state"
done
```
跳出循环：break、continue

## 3. 创建函数
```linux
function name { commands
}
```

# 高级函数
## 4. sed命令
Sed编辑器被称为流编辑器，与普通的交互式文本编辑器想法。
1）在命令行定义编辑器
```linux
$ Echo “this is a test” | sed ’s/test/big test’
```
This is a big test
或者
```linux
Sed ’s/dog/cat/‘ data.txt
```
Sed编辑器并不会修改文本文件的数据
2）在命令行使用多个编辑器
```linux
Sed -e ’s/brown/green/; s/dog/cat’ data.txt
```
必须要用；分割
3）从文件中读取编辑器命令
```linux
Sed -f script1.sed datat.txt
```
4）写入文件
```linux
Sed ‘1,2w test.txt’ datat.txt
```
将test.txt文件的前两行打印到data.txt文件中

5、awk命令
1）格式化输出
```linux
awk '{printf "%-8s %-10s\n",$1,$4}' log.txt
``
2）使用，分割
```linux
awk 'BEGIN{FS=","} {print $1,$2}'  log.txt
```
3）条件语句
```linux
awk 'BEGIN {
    num = 11; 
    if (num % 2 == 0) printf "%d 是偶数\n", num; 
    else printf "%d 是奇数\n", num 
}'
```

