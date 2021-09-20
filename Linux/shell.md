[toc]

# Shell变量

系统变量:`$HOME`、`$PWD`、`$SHELL`、`$USER`等等,比如echo  $HOME

显示当前shell中所有变量:set



## shell变量的定义

1. 定义变量: 变量=值
2. 撤销变量:unset 变量
3. 声明静态变量: readonly 变量, 注意,不能unset



## 定义变量的规则

1. 变量名称可以由字母、数字和下划线组成,但是不能已数字开头
2. 等号两侧不能有空格
3. 变量名称一般习惯为大写



## 将命令的返回值赋给变量

```shell
A=`ls -la` 反引号,运行里面的命令,并把结果返回给变量A
A=$(ls -la)等价于反引号
```



## 设置环境变量

- export  变量名=变量值 (将shell变量输出为环境变量)
- source 配置文件			(让修改后的配置信息立即生效)
- echo  $变量名			(查询环境变量的值)

在输出JAVA_HOME环境变量前,需要让其生效 

```shell
#多行注释
:<<!

!
```



# **位置参数变量**

执行shell脚本时,获取到命令行的参数信息,就可以使用到位置参数变量.如: ./myshell.sh 100 200, 在myshell脚本中获取到参数信息.

```shell
$n  (n为数字, $0代表命令本身, $1-$9代表第一道第九个参数,十以上的参数需要用大括号包含,如${10})

$*  (这个变量代表命令行中所有的参数,$*把所有的参数看成一个整体)

$@ (代表命令行中所有的参数,不过$@把每个参数区分对待)

$#  (代表命令行中所有参数的个数) 
```



## 预定义变量

- $$: 当前进程的进程号(PID)
- $! : 后台运行的最后一个进程的进程号(PID)
- $?: 最后一次执行的命令的返回状态.如果这个变量的值为0,证明上一个命令正确执行;如果这个变量的值为非0(具体时哪个数,由命令自己来决定),则证明上一个命令执行不正确.



### 运算符

```shell
"$((运算式))"或"$[运算式]"
expr m + n  ##expr运算符间要有空格
expr \*,/,% 乘, 除, 取余
```



```shell
#第一种方式
RESULT=$((2+3)*4))
#第二种方式,推荐
RESULT=$[(2+3)*4]

#expr
$TEMP=`expr 2 + 3`
RESULT=`expr $TEMP \* 4`
```



## 条件判断

[ condition ] (condition前后要有空格)

\#非空返回true, 可使用$?验证(0为true,>1为false)

[ atguigu ]	返回true

[]			返回true

[ condition ] && echo OK || echo notok  条件满足,执行 



### **常用判断条件**

1. 两个整数的比较

2. 1. = 字符串比较
   2. -lt 小于
   3. -le 小于等于
   4. -eq 等于
   5. -gt 大于
   6. -ge 大于等于
   7. -ne 不等于

3. 按照文件权限进行判断

4. 1. -r 有读的权限
   2. -w 有写的权限
   3. -x 有执行的权限

5. 按照文件类型进行判断

6. 1. -f 文件存在并且是一个常规的文件
   2. -e 文件存在
   3. -d 文件存在并且是一个目录



### **流程控制**

if [ 条件判断式 ];then

程序

fi

或者

if [ 条件判断式 ]

then

程序

elif [ 条件判断式 ]

then

程序

fi

注意事项:

1. [ 条件判断式 ],中括号和条件判断式之间必须有空格
2. 推荐使用第二种方式





### **case语句**

case $变量名 in

"值1")

如果变量的值等于值1,则执行程序1

;;

"值2")

如果变量的值等于值2,则执行程序2

;;

......

*)

如果变量的值都不是以上的值,则执行此程序

;;

esac

```shell
case $1 in
"1")
echo "周一"
;;
"2")
echo "周二"
;;
*)
echo "other"
;;
esac
```



### **for循环**

语法1:

for 变量 in 值1 值2 值3...

do

程序

done



```shell
#!/bin/sh
for i in "$*"
do
	echo "this num $i"
done

echo "================"

for j in "$@"
do
	echo "this num $j"
done

this num 10 20 30
================
this num 10
this num 20
this num 30
```

语法2:

for((初始值;循环控制条件;变量变化))

do

程序

done

```shell
#!/bin/sh

SUM=0
for((i=0;i<=100;i++))
do
	SUM=$[$SUM+$i]
done
echo "sum=$SUM"

sum=5050
```



### **while循环**

while [ 条件判断式 ]

do

程序 

done

```shell
#!/bin/sh

SUM=0
i=0
while [ $i -le $1 ]
do
	SUM=$[$SUM+$i]
	i=$[$i+1]
done
echo "sum=$SUM i=$i"
sum=5050 i=101
```



### **read读取控制台输入**

read (选项)(参数)

- -p:指定读取值的提示符
- -t: 指定读取值时等待的时间(秒),如果没有在指定的时间内输入,就不在等待

```shell
read  -p "请输入一个数num=" NUM1
echo "输入的数num=$NUM1"

read -t 10 -p "input num=" NUM1
echo "input num=$NUM1"
```



# 函数

## 系统函数

### **basename 基本语法**

返回完整路径最后/的部分,常用于获取文件名

basename [pathname][suffix]

basename [string][suffix]	basename命令会删掉所有的前缀包括最后一个(‘/’)字符,返回将字符串显示出来

suffix为后缀,如果suffix被指定来,basename会将pathname或string中的suffix去掉.

```shell
basename /home/yons/test.txt
test.txt

basename /home/yons/test.txt .txt
test
```



### dirname基本语法

返回完整路径最后/的前面部分,常用于返回路径部分

dirname 文件绝对路径 从给定的包含绝对路径的文件名中去除文件名(非目录的部分),然后返回剩下的路径(目录的部分)

```shell
dirname /home/aaa/test.txt
/home/aaa
```



### 自定义函数

[ function ] funname[()]

{

Action;

[return int;]

}

调用直接写函数名: funname

```shell
#!/bin/sh

function getSum()
{
	SUM=$[$n1 + $n2]
	echo "SUM=$SUM"
}
read -p "输入第一个数n1" n1
read -p "输入第二个数n2" n2
getSum $n1 $n2
```



# shell综合案例

```shell
#!/bin/sh

#备份路径
BACKUP=/Users/lushengyang/Desktop/DB
#当前的时间作为文件名
DATETIME=$(date +%Y_%m_%d_%H%M%S)
echo "==================开始备份===================="
echo "===========备份的路径是 $BACKUP/$DATETIME.tar.gz"

#主机
HOST=localhost
#用户名
DB_USER=root
#密码
DB_PWD=Sal123456
#备份数据库名
DATABASE=DIAN_TI
#创建备份路径
#如果备份的路径文件夹存在,就使用,否则就创建
[ ! -d "$BACKUP/$DATETIME" ] && mkdir -p "$BACKUP/$DATETIME"
#执行mysql的备份数据库的指令
mysqldump -u${DB_USER} -p${DB_PWD} --host=$HOST $DATABASE | gzip > $BACKUP/$DATETIME/$DATETIME.sql.gz
#打包备份文件
cd $BACKUP
tar -zcvf $DATETIME.tar.gz $DATETIME
#删除临时目录
rm -rf $BACKUP/$DATETIME 

#删除10天前的备份文件
find $BACKUP -mtime +10 -name "*.tar.gz" -exec rm -rf {} \;
echo "============备份文件成功======="
```



# **Ubuntu软件操作的相关命令**

- **sudo apt-get update 更新源**
-  **sudo apt-get install package 安装包**
- **sudo apt-get remove package 删除包**
- sudo apt-cache search package 搜索软件包
- **sudo apt-cache show package 获取包的相关信息,如说明、大小、版本**
- suduo apt-get install package --reinstall 重新安装包
- sudo apt-get -f install 修复安装
- sudo apt-get remove package --purge 删除包,包括配置文件等
- sudo apt-get build-dep package 安装相关的编译环境
- sudo apt-get upgrade 更新已安装的包
- sudo apt-get dist-upgrade 升级系统
- sudo apt-cache depends package 了解使用该包依赖哪些包
- sudo apt-cache rdepends package 查看该包被哪些包依赖
- **sudo apt-get source package 下载该包的源代码**



## 更新软件下载源

```shell
国内镜像源 清华:https://mirrors.tuna.tsinghua.edu.cn/
```

## 备份Ubuntu默认的源地址

```shell
sudo  cp /etc/apt/source.list /etc/apt/sources.list.backup
```

更新源地址

```shell
sudo apt-get update
```

## **使用SSH远程登录Ubuntu**

ubuntu默认没有安装sshd

安装SSH和启用

```shell
sudo  apt-get install openssh-server
service sshd restart
```

