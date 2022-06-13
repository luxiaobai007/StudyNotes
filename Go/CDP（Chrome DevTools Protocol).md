---
Go chromedp库的使用 ---CDP
---

[toc]

# CDP（Chrome DevTools Protocol)

**Chrome DevTools Protocol** 是基于websocket的一种协议，可以运行其他工具或者库来通过CDP协议进行检查、调试Chrome和其他基于Blink的浏览器。



## 基于CDP协议的相关语言操作

- **Pyppeteer**，它是谷歌chrome官方无头框架puppeteer的python版本 https://miyakogi.github.io/pyppeteer/reference.html
- **Puppeeer**，它是一个Node库，可以使用Node.js来实现控制Chrome或Chromiumhttp://www.puppeteerjs.com/
- **Chromedp**,  是由Go语言编写，支持Chrome DevTools Protocol的一个库，不需要依赖其他的外界服务（如Selenium和PhantomJS） https://github.com/chromedp/chromedp





## Chromedp使用

- chromedp.NewContext() 初始化chromedp的上下文，后续这个页面都使用这个上下文进行操作
- chromedp.Run() 运行一个chrome的一系列操作
- chromedp.Navigate() 将浏览器导航到某个页面
- chromedp.WaitVisible() 等候某个元素可见，再继续执行。
- chromedp.Click() 模拟鼠标点击某个元素
- chromedp.Value() 获取某个元素的value值
- chromedp.ActionFunc() 再当前页面执行某些自定义函数
- chromedp.Text() 读取某个元素的text值
- chromedp.Evaluate() 执行某个js，相当于控制台输入js
- network.SetExtraHTTPHeaders() 截取请求，额外增加header头
- chromedp.SendKeys() 模拟键盘操作，输入字符
- chromedp.Nodes() 根据xpath获取某些元素，并存储进入数组
- chromedp.NewRemoteAllocator
- chromedp.OuterHTML() 获取元素的outer html
- chromedp.Screenshot() 根据某个元素截图
- page.CaptureScreenshot() 截取整个页面的元素
- chromedp.Submit() 提交某个表单
- chromedp.WaitNotPresent() 等候某个元素不存在，比如“正在搜索。。。”
- chromedp.Tasks{} 一系列Action组成的任务



## 示例

### 本地界面浏览器操作

==默认情况下chromedp使用的浏览器为headless-chrome==

```go
func main() {
    //禁用headless-chrome 无头浏览器  推荐debug模式下使用
	opts := append(chromedp.DefaultExecAllocatorOptions[:], chromedp.Flag("headless", false))
	alloctx, cancel := chromedp.NewExecAllocator(context.Background(), opts...)
	defer cancel()

	ctx, cancel := chromedp.NewContext(alloctx, chromedp.WithErrorf(log.Printf))
	defer cancel()
	var nodes []*cdp.Node
	err := chromedp.Run(ctx,
		chromedp.Navigate("https://www.cnblogs.com/"),
		chromedp.WaitVisible(`#footer`, chromedp.ByID),
		chromedp.Nodes(`//*[@id="footer_bottom"]/div[2]/*`,&nodes),
	)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("get nodes:", len(nodes))
	// print titles
	for _, node := range nodes {
		fmt.Println(node.Children[0].NodeValue, ":", node.AttributeValue("href"))
	}
}

```



## 远程操作示例（无头浏览器）

### Centos 安装chronium-headless

```shell
###搜索chrome的yum源
yum search chromium
##选择chromium-headless.x86_64
sudo yum install chromium-headless.x86_64
###查看chrome二进制文件位置
rpm -ql chromium-headless.x86_64     ///usr/lib64/chromium-browser/headless_shell
nohup /usr/lib64/chromium-browser/headless_shell --no-first-run --no-default-browser-check --headless --disable-gpu --remote-debugging-port=9222 --no-sandbox --disable-plugins --remote-debugging-address=0.0.0.0 --window-size=1920,1080 &

###查看端口服务器是否开启
netstat -lntp
```

headless_shell（chrome）Flag参数说明

- `--no-first-run` 第一次不运行
- `---default-browser-check` 不检查默认浏览器
- `--headless` 不开启图像界面
- `--disable-gpu` 关闭gpu,服务器一般没有显卡
- `remote-debugging-port` chrome-debug工具的端口(golang chromepd 默认端口是9222,建议不要修改)
- `--no-sandbox` 不开启沙盒模式可以减少对服务器的资源消耗,但是服务器安全性降低,配和参数 `--remote-debugging-address=127.0.0.1` 一起使用
- `--disable-plugins` 关闭chrome插件
- `--remote-debugging-address` 远程调试地址 `0.0.0.0` 可以外网调用但是安全性低,建议使用默认值 `127.0.0.1`
- `--window-size` 窗口尺寸

[其他headless-chrome参数说明官方文档](https://developers.google.com/web/updates/2017/04/headless-chrome)

![image-20211224153734427](http://qiliu.luxiaobai.cn/img/image-20211224153734427.png)



### 示例代码

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"github.com/chromedp/cdproto/cdp"
	"github.com/chromedp/chromedp"
	"log"
)

func main() {
	devtoolsWsURL := flag.String("devtools-ws-url", "http://192.168.3.177:9222/json", "DevTools WebSsocket URL")
	flag.Parse()
	if *devtoolsWsURL == "" {
		log.Fatal("must specify -devtools-ws-url")
	}
	fmt.Println("----", *devtoolsWsURL)
	// create allocator context for use with creating a browser context later
	allocatorContext, cancel := chromedp.NewRemoteAllocator(context.Background(), *devtoolsWsURL)
	defer cancel()

	// create context
	ctxt, cancel := chromedp.NewContext(allocatorContext)
	defer cancel()
	var nodes []*cdp.Node
	err := chromedp.Run(ctxt,
		chromedp.Navigate("https://www.cnblogs.com/"),
		chromedp.WaitVisible(`#footer`, chromedp.ByID),
		chromedp.Nodes(`//*[@id="footer_bottom"]/div[2]/*`, &nodes),
	)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("get nodes:", len(nodes))
	// print titles
	for _, node := range nodes {
		fmt.Println(node.Children[0].NodeValue, ":", node.AttributeValue("href"))
	}

}
```



# Browsesrless/chrome部署

[官方操作文档](https://docs.browserless.io/docs/docker.html)

注：部署Browserless需要有docker环境

```shell
####拉取镜像
docker pull browserless/chrome
###运行 设置最大并发会话数。默认情况下，未指定时设置为 5。一旦达到限制，就会开始排队，请求将等到更多工作人员准备就绪
 docker run -e "MAX_CONCURRENT_SESSIONS=3" -p 10003:3000 --restart always -d browserless/chrome

```



[相关配置](https://docs.browserless.io/docs/docker.html)

- **连接超时**：连接超时是设置任何会话可以运行多长时间的参数。这是为了防止脚本没有正确清理，或者遇到导致它们挂起的错误。其值可以以毫秒为单位设置，默认为`30000`, 或 30 秒。 `CONNECTION_TIMEOUT=60000`

- **最大队列长度**：此值确定在发出`429`响应代码并关闭请求之前队列中允许的项目数。这种机制是为了防止消费者意外（或故意）触发拒绝服务。默认情况下，在开始失败之前，图像只允许 5 个请求的队列。例如，如果您的 a`MAX_CONCURRENT_SESSIONS`为 5，a`MAX_QUEUE_LENGTH`为 5，则允许 10 个并发连接（5 个正在运行，5 个待处理）。`MAX_QUEUE_LENGTH=10`

- **预启动Chrome**: 可以选择预启动 Chrome 并将其保存在实例池中（由 确定`MAX_CONCURRENT_SESSIONS`），以缩短启动时间。`PREBOOT_CHROME=true`

- **定义主机绑定**： 默认情况下，当没有提供主机时，browserless 将绑定到 localhost。如果想绑定到另一个 IP 或域，请传入一个`HOST`变量来执行此操作。`HOST=192.168.1.1`

- **禁用下载行为**: 默认情况下，browserless 会告诉 Chrome 使用一个特殊的目录`/tmp`来存储文件。如果您想退出此行为，请使用以下标志启动 docker 镜像：  `DISABLE_AUTO_SET_DOWNLOAD_BEHAVIOR=true`

- **禁用调试器**: 要禁用调试器（和所有accompnaying HTML），您可以设置`ENABLE_DEBUGGER`以`false`只允许`puppeteer.connect`调用成功。`ENABLE_DEBUGGER=false`

- **保护实例**: 如果您要向全世界公开您的实例，但不希望任何人使用它，您可以选择应用一个`TOKEN`参数来限制没有`token`查询字符串参数的调用。当存在时，browserless **将拒绝任何没有匹配令牌的调用**

  ```shell
  docker run -e "TOKEN=2cbc5771-38f2-4dcf-8774-50ad51a971b8" -p 3000:3000 --restart always -d --name browserless browserless/chrome
  
  ###在应用查询中使用查询参数来传递改令牌
  const browser = await puppeteer.connect({
      browserWSEndpoint: 'ws://localhost:3000?token=2cbc5771-38f2-4dcf-8774-50ad51a971b8',
  });
  ```



### Linux-Centos安装NodeJS

```shell
su root
yum install -y wget
####官方下载地址：https://nodejs.org/en/download/
wget https://nodejs.org/dist/v12.18.1/node-v12.18.1-linux-x64.tar.xz
tar -xvf node-v12.18.1-linux-x64.tar.xz
ln -s /root/node-v12.18.1-linux-x64/bin/node /usr/bin/node
ln -s /root/node-v12.18.1-linux-x64/bin/npm /usr/bin/npm
node -v
npm -v
```







## 参考

[1]: https://miyakogi.github.io/pyppeteer/reference.html
[2]: https://github.com/pyppeteer/pyppeteer
[3]: https://github.com/Pines-Cheng/blog/issues/82	"Chrome DevTools Protocol协议详解"
[4]: https://www.html.cn/doc/chrome-devtools/devtools-extensions/remote-debugging-protocol/	"远程调试协议"
[5]: https://www.cnblogs.com/bigben0123/p/15241062.html	"深入浅出CDP"
[6]: https://blog.csdn.net/qq_24564445/article/details/110632347	"nodejs使用Puppeteer简单实现爬虫"
[7]: https://www.jianshu.com/p/f1a8fb7037d7	"Pyppeteer入门教程"
[8]: https://www.keepnight.com/archives/1446/	"pyppeteer的使用总结"
[9]: https://github.com/chromedp/examples	"chromedp基本操作"
[10]: http://www.puppeteerjs.com/	"Puppeteer"
[11]: https://github.com/chromedp/chromedp	"chromedp"
[12]: https://www.cnblogs.com/yjf512/p/13181998.html	"chromedp入门"
[13]: https://blog.csdn.net/wanwuguicang/article/details/79751571	"Chrome启动参数大全"

