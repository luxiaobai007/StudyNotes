---
Git入门级教程
---



# 远程服务器的配置

ubuntu下安装git

```shell
sudo apt-get install git
```



1、进入远程服务器,创建Git用户

```shell
useradd git
userdel git //删除用户
passwd git //设置git用户密码
```



2、创建一个git文件夹,方便将所有git项目放入其中

```shell
cd /home
mkdir git
chmod 777 git //设置git文件夹访问权限, 所有用户都可以访问
```



3、使用git用户进入git文件夹创建git仓库(即你的项目)

```shell
cd git
mkdir FirstProject
cd FirstProject
git init //git化此文件夹,此时会在该文件夹里生成三个文件
cd .git //进入该目录下对git仓库进行配置
vim config //由于git默认拒绝push操作,所以要修改config文件进行配置
添加如下内容:
	[receive]
			denyCurrentBranch = ignore
```



# 本机仓库的配置

```shell
cd /home
mkdir FirstProject
cd FirstProject
git init
git remote add origin git@[远程服务器IP地址]:/home/git/FirstProject  //绑定远程的git仓库
git remote -v //查看绑定
git remote remove origin //若绑定错误,则删除绑定git
git add . //将当前目录下所有文件交由git仓库管理
git commit -m "first commit" //commit提交本地git仓库
git push -u origin master //push上传远程git仓库 master为远程git分支(默认为主分支)
```

