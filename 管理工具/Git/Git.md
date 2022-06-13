# 安装

http://npm.tobao.org/mirrors/git-for-windows/



# 配置用户名和邮箱

```shell
git config --global user.name "Luxiaobai"
git config --global user.email "1234567@qq.com"
```





# Git基本命令

```shell
#查看指定文件状态
git status [filename]

#查看所有文件状态
git status

#添加所有文件到暂存区
git add.

#提交暂存区中的内容到本地仓库 -m 提交信息
git commit -m "消息内容"
```

> 忽略文件 .gitignore

1. 忽略文件中的空行或以井号(#)开始的行将会被忽略
2. 使用Linux通配符. 例如: (*) 代表任意多个字符,(?)代表一个字符,([abc])代表可选字符范围,大括号({string1,string2,...})代表可选的字符串等.
3. 如果名称的最前面有一个感叹号(!),表示例外规则,将不被忽略
4. 如果名称的最前面是一个路径分隔符(/),表示要忽略的文件在此目录下,而子目录中的文件不忽略.
5. 如果名称的最后面是一个路径分隔符(/),表示要忽略的是此目录下该名称的子目录,而非文件(默认文件或目录都忽略)

```shell
#为注释
*.txt		#忽略所有.txt结尾的文件
!lib.txt	#但lib.txt除外
/temp		#仅忽略项目根目录下的TODO文件,不包括其他目录temp
build/	#忽略build/目录下的所有文件
doc/*.txt 	#会忽略doc/notes.txt 但不包括doc/server/arch.txt
```

 

```gitignore
*.class
*.log
*.lock

#Package FIles #
*.jar
*.war
*.ear
target/

# idea
.idea/
*.iml

*velocity.log*

### STS ###
.apt_generated
.factorypath
.springBeans

###IntelliJ IDEA #####
*.iml
*.ipr
*.iws
.idea
.classpath
.project
.settings/
bin/

*.log
tmp/

#rebel
*rebel.xml*

```



# GIT分支

```shell
# 列出所有本地分支
git branch

#列出所有远程分支
git branch -r 

#新建一个分支,但依然停留在当前分支
git branch [branch-name]

#新建一个分支,并切换到该分支
git checkout -b [branch]

#合并指定分支到当前分支
git merge [branch]
 
 #删除分支
 git branch d [branch-name]
 
 #删除远程分支
 git push origin --delete [branch-name]
 git branch -dr [remote/branch]
```





# 现实场景

### Git 提交(commit)之后,想要撤回怎么办

```shell
git reset --soft HEAD^
```



### 合并分支

有两个分支master和dev

dev分支版本高于master分支.如何进行合并

```shell
##切换为master
git checkout master
##进行合并
git merge dev
##最后推送远程
git push origin master
```

