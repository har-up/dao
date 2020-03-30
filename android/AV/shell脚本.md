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
- > / < 比较字符串的ascii顺序大小
- -n 字符串长度是否非0
- -z 字符串长度是否为0


