# Go项目部署问题



> 记Go项目部署的遇到的问题。
>
> 这是一个前后端不分离的项目，需要用静态资源打包，采用的方式是GO1.6版本以上的新功能Embed
>
> 在进行docker镜像选择上GO：1.17.4。

在进行项目部署编译打包完成之后，在目标服务器上运行失败。错误提示如下：

```shell
./project: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.28' not found (required by ./project)
```

出现这个问题是，原以为是sqlite在进行打包时出现的问题，因为在进行编程时，出现如下警告：

```shell
# github.com/mattn/go-sqlite3
sqlite3-binding.c: In function 'sqlite3SelectNew':
sqlite3-binding.c:129019:10: warning: function may return address of local variable [-Wreturn-local-addr]
129019 |   return pNew;
       |          ^~~~
sqlite3-binding.c:128979:10: note: declared here
128979 |   Select standin;
       |          ^~~~~~~
```

但是进行项目的交叉编译和Go的降版本测试，去解决这个警告显示，并不是sqlite的这个问题。

想要去掉这个警告可以在编译时加上如下命令，不提示警告信息：

```shell
CGO_CFLAGS: " -g -O2 -Wno-return-local-addr
```

==建议：最好不要这么干，万一真报错了就找不到原因了==

回到最先的话题：

网上的解释是：由于在项目编译时采用的gcc编译版本相对于目标主机的版本过低导致的。请教和兑之后，问题定位是在选择Go镜像的问题上

在版本镜像后加上如下命令：

```shell
##修改前
image: golang:1.17.4
##修改后
image: golang:1.17.4-stretch
```



## Docker镜像的版本后缀介绍

发行版本介绍：https://www.debian.org/releases/

发行版本目录：

```shell
下一代 Debian 正式发行版的代号为 bookworm — 测试（testing）版 — 发布日期尚未确定
Debian 11 (bullseye) — 当前的稳定（stable）版
Debian 10（buster） — 当前的旧的稳定（oldstable）版
Debian 9（stretch） — 更旧的稳定（oldoldstable）版，现有长期支持
Debian 8（jessie） — 已存档版本，现有扩展长期支持
Debian 7（wheezy） — 被淘汰的稳定版
Debian 6.0（squeeze） — 被淘汰的稳定版
Debian GNU/Linux 5.0（lenny） — 被淘汰的稳定版
Debian GNU/Linux 4.0（etch） — 被淘汰的稳定版
Debian GNU/Linux 3.1（sarge） — 被淘汰的稳定版
Debian GNU/Linux 3.0（woody） — 被淘汰的稳定版
Debian GNU/Linux 2.2（potato） — 被淘汰的稳定版
Debian GNU/Linux 2.1（slink） — 被淘汰的稳定版
Debian GNU/Linux 2.0（hamm） — 被淘汰的稳定版
```

