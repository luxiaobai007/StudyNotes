# CentOS 7安装Node

## 源码安装

```shell
wget http://nodejs.org/dist/v0.10.30/node-v0.10.30.tar.gz
tar xzvf node-v* && cd node-v*
sudo yum install gcc gcc-c++
./configure
make
sudo make install
node --version	
sudo yum install npm
```

