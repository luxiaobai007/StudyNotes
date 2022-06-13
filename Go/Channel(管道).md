---
Go Channel详解
---

[toc]



# Channel(管道)

> Channel是Go中的一个核心类型，你可以把它看成一个管道，通过它并发核心单元就可以发送或者接收数据进行通讯(communication)。

操作符`<-`  箭头的指向就是数据的流向

```go
ch <- v  //发送值V到Channel ch中
v := <ch


###channel必须先创建再使用
ch := make(chan int)
```



## Channel类型

```go
ChannelType = ("chan" | "chan" "<-" | "<-" "chan") ElementType .
```

它包括三种类型的定义。可选的`<-`代表channel的方向。如果没有指定方向，那么Channel就是双向的，既可以接收数据，也可以发送数据。

```go
chan T			//可以接收和发送类型位T的数据
chan<- float64  //只可以用来发送float64类型的数据
<-chan int		//只可以用来接收int类型的数据
```

`<-`总是优先和最左边的类型结合

```go
chan<- chan int	//等价 chan<- (chan int)
chan<- <-chan int	//等价 chan<- (<-chan int)
<-chan <-chan int	//等价 <-chan (<-chan int)
chan (<-chan int)
```

使用`make`初始化Channel，并且可以设置容量

```go
make(chan int, 100)
```



### send

send语句用来往Channel中发送数据 `ch <- 3`

```go
SendStmt = Channel "<-" Expression .
Channel = Expression .
```

在通讯(communication)开始前channel和expression必选先求值出来(evaluated)

```go
c := make(chan int)
defer close(c)
go func() { c <- 3+4 }()
i := <- c
fmt.Println(i)
```

> send被执行前(proceed)通讯(communication)一直被阻塞着。如前所言，无缓存的channel只有在receiver准备好后send才被执行。如果有缓存，并且缓存未满，则send会被执行。往一个已经被close的channel中继续发送数据会导致**run-time panic**。
>
> 往nil channel中发送数据会一致被阻塞着。



### receive操作符

> <-ch 用来从channel ch中接收数据，这个表达式会一直被block，直到有数据可以接收
>
> 从一个nil channel中接收数据会一直被block。
>
> 从一个被close的channel中接收数据不会被阻塞，而是立即返回，接收完已发送的数据后会返回元素类型的零值(zero value)。

```go
x, ok := <-ch
x, ok = <-ch
var x, ok = <-ch
##如果OK 是false，表明接收的x是产生的零值，这个channel被关闭了或者为空。
```





### blocking

缺省情况下，发送和接收会一直阻塞着，直到另一方准备好。这种方式用来在gororutine中进行同步，而不必使用显示的锁或者条件变量

```go
func main()  {
	s := []int{7,2,8,-9,4,0}

	c := make(chan int)
	go sum(s[:len(s)/2],c)
	go sum(s[len(s)/2:],c)
	x, y := <-c, <-c
	fmt.Println(x,y,x+y)
}

func sum(s []int, c chan int)  {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum //send sum to c
}

```





### Buffered Channels

make的第二个参数指定缓存的大小： `ch := make(chan int, 100)`

通过缓存的使用，可以尽量避免阻塞，提供应用的性能



### Range

`for ...... range`语句可以处理Channel

```go
func main(){
    go func() {
		time.Sleep(1 * time.Hour)
	}()
	c := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
		close(c)
	}()

	for i := range c{
		fmt.Println(i)
	}

	fmt.Println("Finished")
}
```

`range c`产生的迭代值为Channel中发送的值，它会一直迭代直到channel被关闭。上面的例子中如果把`close(c)`注释掉，程序会一直阻塞在`for …… range`那一行。





### Select

`select`语句选择一组可能的`send`操作和`receive`操作去处理。它类似`switch`,但是只是用来处理通讯(communication)操作。
它的`case`可以是send语句，也可以是receive语句，亦或者`default`。

`receive`语句可以将值赋值给一个或者两个变量。它必须是一个receive操作。

最多允许有一个`default case`,它可以放在case列表的任何位置，尽管我们大部分会将它放在最后。

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

如果有同时多个case去处理,比如同时有多个channel可以接收数据，那么Go会伪随机的选择一个case处理(pseudo-random)。如果没有case需要处理，则会选择`default`去处理，如果`default case`存在的情况下。如果没有`default case`，则`select`语句会阻塞，直到某个case需要处理。

需要注意的是，nil channel上的操作会一直被阻塞，如果没有default case,只有nil channel的select会一直被阻塞。

`select`语句和`switch`语句一样，它不是循环，它只会选择一个case来处理，如果想一直处理channel，你可以在外面加一个无限的for循环



### timeout

超时处理，如果没有case需要处理，select语句就会一直阻塞着。这时候就需要一个超时操作，用来处理超时的情况。

```go
func main() {
	c1 := make(chan string, 1)
	go func() {
		time.Sleep(time.Second * 2)
		c1 <- "result 1"
	}()

	select {
	case res := <-c1:
		fmt.Println(res)
	case <-time.After(time.Second * 1):
		fmt.Println("timeout 1")
	}
}
```



### 同步

main goroutine通过done channel等待worker完成任务。 worker做完任务后只需往channel发送一个数据就可以通知main goroutine任务完成

```go
func worker(done chan bool)  {
	time.Sleep(time.Second * 3)

	//通知任务已完成
	done <- true
}

func main() {
	done := make(chan bool, 1)
	go worker(done)

	//等待任务完成
	<-done
}
```

























































































































































































































































































[1]: https://colobu.com/2016/04/14/Golang-Channels/	"Go Channel详解"

