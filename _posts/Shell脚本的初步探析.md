title: Shell脚本的初步探析
date: 2016-03-24 13:34:08
tags:
- shell
- 脚本
---
slide]

# shell编程
## 演讲者：守望

[slide data-transition="vertical3d"]

# 什么是shell？ {:&.flexbox.vleft}
[slide]
# shell {:&.flexbox.vleft}
## [ʃel]俗称壳（命令解析器) 
[slide]
是一个命令行解释器，一种为用户提供了一个向内核发送请求以便运行程序的界面系统级程序，用户可以用shell来启动、挂起、甚至编写一些程序.例：sh是第一种Unix Shell，Windows Explorer是一个典型的图形界面Shell。 {:&.flexbox.vleft}
[slide data-transition="vertical3d"]
shell的分类 {:&.flexbox.vleft}
- Bourne shell:1979年起Unix开始使用（sh,ksh,Bash);
- c shell:在BSD版的Unix语言与c语言类似所以因此得名(csh, tcsh);
- Bourne Again shell (bash)
- Korn shell (ksh)
- POSIX shell (sh)
[slide]
#shell编程？一门语言？
[slide data-transition="vertical3d"]
## shell脚本（shell script），是一种为shell编写的脚本程序，易编写易调试灵活性，shell脚本是解释执行的脚本语言，在shell中可直接调起Linux的命令。
[slide]
# 与高级编程语言

[slide data-transition="vertical3d"]
理论上讲，只要一门语言提供了解释器（而不仅是编译器），这门语言就可以胜任脚本编程，常见的解释型语言都是可以用作脚本编程的，如：Perl、 Tcl、Python、PHP、Ruby。Perl是最老牌的脚本编程语言了，Python这些年也成了一些linux发行版的预置解释器。 {:&.flexbox.vleft}
[slide]
```php
#!/usr/bin/php
<?php
for ($i=0; $i < 10; $i++)
        echo $i . "\n";
?>


/usr/bin/php test.php
或者：
chmod +x test.php
./test.php
```
[slide data-transition="vertical3d"]
#环境
[slide]
shell编程跟java、php编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。 {:&.flexbox.vleft}
[slide style="background-image:url('/img/bg1.png')"]
## 一个简单的shell脚本
```bash
#!/bin/sh
cd ~
mkdir shell_tut
cd shell_tut

for ((i=0; i<2; i++)); do
    touch test_$i.txt
done
＃第1行：指定脚本解释器，这里是用/bin/sh做解释器的
＃第2行：切换到当前用户的home目录
＃第3行：创建一个目录shell_tu
＃第4行：循环创建文件
#运行 chmod +x ./test1.sh
```
[slide data-transition="vertical3d"]
##变量 
```bash
your_name="qinjx"
```
1. 在命名变量名时，变量名称只能是英文字母和数字，而且首字母不能是数字
2. 等号=两边不能有空格
3. 如果变量值有空格，可用双引号" "或单引号' '来包围变量值，但两者是有区别：双引号"   "内的一些特殊字符，可以保持原有的特性
##使用变量

```bash
your_name="qinjx"
echo $your_name
echo ${your_name}
```
---
```bash
for skill in Ada Coffe Action Java do
    echo "I am good at ${skill}Script"
done
```
[slide data-transition="vertical3d"]
#字符串
[slide]
```bash
str='this is a string'
#单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
#单引号字串中不能出现单引号（对单引号使用转义符后也不行）

your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"

#双引号里可以有变量
#双引号里可以出现转义字符
```
[slide]
#条件判断与流程控制
```bash
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

```bash
for var in item1 item2 ... itemN; do command1; command2… done;
```
---
```bash
while condition
do
    command
done

```
另外还有case语句；until语句
[slide]
# shell函数
[slide]
```bash
#!/bin/bash
funWithReturn(){
    echo "The function is to get the sum of two numbers..."
    echo -n "Input first number: "
    read aNum
    echo -n "Input another number: "
    read anotherNum
    echo "The two numbers are $aNum and $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "The sum of two numbers is $? !"
```
[slide]
# 重定向 Redirection {:&.flexbox.vleft}
 stdin(<或<<)、stdout(>或>>)、stderr(2>或2>>)
##将一条命令执行结果（标准输出，或者错误输出，本来都要打印到屏幕上面的）  重定向其它输出设备（文件，打开文件操作符，或打印机等等）
[slide]
![alt text](/shell_ppt/shell_redict.png "Title")
[slide data-transition="vertical3d"]
# 输出重定向
```bash
command-line1 [1-n] > file或文件操作符或设备
ls 1>suc.txt
who > users
cat users
#1,2分别是标准输出，错误输出。
```
---
## >与>>(不覆盖追加在内容后面)
例如：echo '追加在后面' >> users
[slide]
# 输入重定向
```bash
command-line [n] < file或文件描述符&设备
```
```bash
$ wc -l users
       2 users
```

```bash
command1 < infile > outfile
```
[slide data-transition="vertical3d"]
# Here Document
Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式Shell脚本或程序
```bash
command << delimiter
    document
delimite

$ wc -l << EOF
    这是第一行
    这是第二行
    这是第三行
EOF
```
---
```bash
#!/bin/bash
cat << EOF
这是第一行
这是第二行
这是第三行
EOF
```
[slide data-transition="vertical3d"]
# 管道 {:&.flexbox.vleft}
 管道命令操作符是：”|”,它仅能处理经由前面一个指令传出的正确输出信息，也就是 standard output 的信息，对于 stdandard error 信息没有直接处理能力。然后，传递给下一个命令，作为标准的输入.
![alt text](/img_shell.png "Title")
---
```bash
cat suc.txt|grep -n 'D'
```
[slide]
# grep、awk、sed
## 三剑客
### 对创建和修改文本文件的人来说是一个强大工具
[slide]
# grep
```bash
grep [-acinv] [--color=auto] '搜寻字符串' filename
选项与参数：
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行！
--color=auto ：可以将找到的关键词部分加上颜色的显示喔！
```
---
```bash
grep -n 't[ae]st' shell_ppt/regular_express.txt

```
[slide]
# awk {:&.flexbox.vleft}
读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"。 
![alt text](/shell_ppt/awk.png "awk工作流程图")
```bash
cat shell_ppt/regular_awk.txt |awk -F ':' '{println $0}'

```
[slide data-transition="vertical3d"]
# sed
##sed可以说是一个编辑器逐行处理输入，然后把结果发送到屏幕
```bash
sed 's/test/mytest/g' shell_ppt/regular_awk.txt
#如果没有g标记就只能替换每行第一个
#命令 a/ 在当前行后面加入一行文本
#命令 c/ 新文本改变本行文本等等
```
---
```bash
sed '2d' shell_ppt/regular_awk.txt
```
[slide]
# shell脚本在实际中的用处
[slide]
# That's All Thankyou!