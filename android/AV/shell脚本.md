# shell 脚本
shell脚本编程用于Linux执行一些命令，其实命令行中的每一个命令都可以写入shell脚本中然后执行。

## 变量
  - 用户变量
    在用户环境变量中配置的变量可以直接引用，这里要注意用户环境变量中的变量只提供该用户，其他用户不能引用到
  - 自定义变量
    在shell文件中定义变量在=号后边赋值，注意不要有空格
  - 给变量赋值为一个命令所打印的值
    使用`命令` 或 ${命令}的方式给变量赋值
```shell  
date
who
text=hello
text2="hello world"
text_date=`date`
text_who=$(who)
echo $text
echo $text2
echo $text_date
echo $text_who
```

## if-else命令
shell中的判断都是这样的格式
if 命令   
else 命令   
不是命令的话不会识别
```shell
user=har-up
if grep $user /etc/passwd
then
        echo "user har-up"
        Is -a /home/$user/
else
        echo "user not found"
fi
```
## 命令函数test
可以使用test或[]的形式，注意使用[]时里边内容两边需留一个空格
- 判断变量是否存在
```shell
var=""
if test var
then echo "var 变量已定义"

else echo "var 变量未定义"
fi
```

- -gt 判断两个数值的大小
```shell
a=10
b=20
if [ $a -gt $b ]
then echo "a greater than b"
else echo "a 小于等于 b"
fi
```
- -eq 判断两个变量是否相等
- -le 判断左值小于右值
- -ne 判断不等于
- == 字符串比较是否相等
- ！= 字符串比较是否不相等
- >/< 比较字符串的ascii顺序大小
- -n 字符串长度是否非0
- -z 字符串长度是否为0
- -d 判断目录是否存在
- -f 判断文件是否存在
- -r -w -x 判断文件是否具有这些权限
- -nt 左文件比右文件新
- -ot 左文件比右文件旧

## 双括号 （（））
在双括号中可以执行任意数学赋值和比较的命令

## case命令
```shell
var=one
case $var in
        one)
                echo "one";;
        tow)
                echo "two";;
        three)
                echo “"three";;
        *)
                echo "other";;
esac

```

## for()循环语句
```shell
for item in one two three
do echo "$item"
done

list="one two three"
for item in $list
do echo "$item"
done

str="one,two,three"
# 字段分隔符
IFS=$,
for item in $str
do echo "$item"
done
```

## while循环
```shell

n=10
while (($n > 0 ))
do echo "$n"
        (( n = $n-1 ))
done
```

## 给shell文件传参
```shell
filename=$(basename $0)
echo "参数的总数： $#"
echo $0
echo $1
echo $2
echo $filename

```

## 输入输出重定向
0-标准输入
1-标准输出
2-错误输出（注意标准输出和错误输出不能同意文件，否则会覆盖）
```shell
#标准输出定向
exec 1>log1
#自定义输出定向
exec 7>log2
echo "hello"
#自定义重定向
echo "world" >&7
```

## 函数
```
function demo
{ 
        echo "hello world"
}

demo

#带参函数
function demo2 
{
        echo $1
        echo $2
}
#调用传入参数
demo2 hello world
#函数返回值赋值
value=$(demo hello world)
echo $value

#函数引用文件的局部变量
function demo3
{
        echo "num = $num"
}

num=10
demo3

```

## 引入其他sh文件
```shell
sourch ./other.sh
#.是source的简便写法
. ./other.sh

. ./12.sh
#调用引入文件的函数 其中变量one是引入文件的变量
add $one 20
```
