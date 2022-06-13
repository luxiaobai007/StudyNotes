---
selenium编译部署
---



# 实际需求

> 将selenium能运行在不同的系统上，实现一套代码不同系统都能运行



# 思路

​	根据运行系统的不同进行条件编译，采用Go 文件后缀的方式进行不同系统的编译，同时需要对项目中引入的不同系统的驱动进行一起嵌入，使用的方式是Go embed方式。



# 问题

`panic: unknown error: unknown error: Chrome failed to start: crashed.
  (unknown error: DevToolsActivePort file doesn't exist)
  (The process started from chrome location /usr/bin/google-chrome is no longer running, so ChromeDriver is assuming that Chrome has crashed.)`

1、查看`/usr/bin/google-chrome`的权限设置（网上的普遍说法）

2、进行软连接设置正确的关联关系。如果`/usr/bin/google-chrome`如果没有进行关联也会报如上的错误。（这个问题困扰我很久，不过一般在进行安装时就已经关联上了，除非安装步骤有误）

![image-20211221181827825](http://qiliu.luxiaobai.cn/img/image-20211221181827825.png)

```shell
sudo ln -sf /etc/alternatives/google-chrome /usr/bin/google-chrome
```



`panic: unknown error: unknown error: cannot find Chrome binary
  (Driver info: chromedriver=2.41.578700 (2f1ed5f9343c13f73144538f15c00b370eda6706),platform=Linux 3.10.0-1160.36.2.el7.x86_64 x86_64)`

需要安装在Linux 上安装google-chrome浏览器

```
### Ubuntu安装websriver

google-chrome --version
sudo apt update
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i ./google-chrome-stable_current_amd64.deb
sudo apt-get install -f
google-chrome --version       //Google Chrome 96.0.4664.110
wget https://chromedriver.storage.googleapis.com/96.0.4664.45/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
```
