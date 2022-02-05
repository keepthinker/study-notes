# 字符串处理
假设有变量 var=http://www.aaa.com/123.htm.
## \# 号截取，删除左边字符，保留右边字符。

echo ${var#*//}

其中 var 是变量名，# 号是运算符，*// 表示从左边开始删除第一个 // 号及左边的所有字符即删除 http:// 结果是 ：www.aaa.com/123.htm

## \#\# 号截取，删除左边字符，保留右边字符。

echo ${var##*/}

##*/ 表示从左边开始删除最后（最右边）一个 / 号及左边的所有字符
即删除 http://www.aaa.com/

结果是 123.htm

## %号截取，删除右边字符，保留左边字符

echo ${var%/*}

%/* 表示从右边开始，删除第一个 / 号及右边的字符

结果是：http://www.aaa.com

## %% 号截取，删除右边字符，保留左边字符

echo ${var%%/*}

%%/* 表示从右边开始，删除最后（最左边）一个 / 号及右边的字符
结果是：http:

## 从左边第几个字符开始，及字符的个数

echo ${var:0:5}

其中的 0 表示左边第一个字符开始，5 表示字符的总个数。
结果是：http:

## 从左边第几个字符开始，一直到结束。

echo ${var:7}

其中的 7 表示左边第8个字符开始，一直到结束。
结果是 ：www.aaa.com/123.htm

## 从右边第几个字符开始，及字符的个数

echo ${var:0-7:3}

其中的 0-7 表示右边算起第七个字符开始，3 表示字符的个数。
结果是：123

## 从右边第几个字符开始，一直到结束。

echo ${var:0-7}

表示从右边第七个字符开始，一直到结束。
结果是：123.htm

注：（左边的第一个字符是用 0 表示，右边的第一个字符用 0-1 表示）


# AWK

```
/* NF: number of field
 * -F --field-separator
 */
awk -F ' ' '{print $(NF-1)}' marks.txt


/*
 * 计算第一个Field的个数
 */
ss -anp | awk '{++S[$1]} END {for(a in S) print a, S[a]}'

tailf info.log | awk -F : '{if($(NF) > 1000){ print($NF" :  "$(NF-1))}}'
```
## 参考文献
[Linux awk命令](http://www.runoob.com/linux/linux-comm-awk.html)

# 排序
```
-n: number，以数字大小进行排序
-r: reverse，倒序
-t: field separator，字段分隔符
sort -nr  -k 2 -t ' '
```

# 循环读
## 循环读文本文件的行
```
# 打印出input.txt文件
for line in $(cat input.txt); 
  do echo $line; 
done;
```

## 循环读控制盘输入字符串
```
while read word; 
  do echo $word; 
done;
```

# seq
数字叠加。
以:分割，数字从1到10，数字间隔2
```
seq -s ':' 1 2 10
```
# sed
输出a.sh并替换其中所有aaa为bbb。
```
cat a.sh | sed 's/aaa/bbb/g'
```

# shell脚本输入参数
- $* 所有参数
- $# 参数个数
- ${10} 第10个参数
- ${@:5} 第五个以及之后的所有参数


# date
date -d @1548386625 +%F

# 文件判断语句
```
-f 是不是一个普通文件
-s 表示文件是否存在并且是否为非空
-e 文件存在
-f file 是一个 regular 文件(不是目录或者设备文件)
-s 文件长度不为 0
-d 文件是个目录
-b 文件是个块设备(软盘,cdrom 等等)
-c 文件是个字符设备(键盘,modem,声卡等等)
-p 文件是个管道
-h 文件是个符号链接
-L 文件是个符号链接
-S 文件是个 socket
-r 文件具有读权限
-w 文件具有写权限
-x 文件具有执行权限
```