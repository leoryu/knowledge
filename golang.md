# foreach中的指针问题

```go
type student struct {
    Name string
    Age  int
}

func main() {
    m := make(map[string]*student)
    stus := []student{
        {Name: "zhou", Age: 24},
        {Name: "li", Age: 23},
        {Name: "wang", Age: 22},
    }
    // 错误写法，stu的指针不会变，到这m中的每一项都指向同一个指针
    for _, stu := range stus {
        m[stu.Name] = &stu
    }
     // 正确写法
    for i:=0;i<len(stus);i++  {
        m[stus[i].Name] = &stus[i]
    }
}
```

# 执行随机性和闭包

```go
func main() {
    runtime.GOMAXPROCS(1)
    wg := sync.WaitGroup{}
    // add 限制wg的wait总数
    wg.Add(20)
    // 两个for循环输出顺序无法确定，因为协程的执行顺序是随机的
    // A都会输出10因为i是外部for的一个变量，地址不变化。遍历完成后，最终i=10。 故go func执行时，i的值始终是10
    for i := 0; i < 10; i++ {
        go func() {
            fmt.Println("A: ", i)
            wg.Done()
        }()
    }
    // B会正常输出，因为i是函数参数，与外部for中的i完全是两个变量。 尾部(i)将发生值拷贝，go func内部指向值拷贝地址
    for i := 0; i < 10; i++ {
        go func(i int) {
            fmt.Println("B: ", i)
            wg.Done()
        }(i)
    }
    wg.Wait()
}
```

# 常量的指针

```go
package main
const cl  = 100

var bl    = 123

func main()  {
    println(&bl,bl)
    // 常量在预编译阶段就会被展开作为指令数据使用，不是在运行期分配内存，无法获取指针
    println(&cl,cl)
}
```

# goto
```go
package main

func main() {

	for i := 0; i < 10; i++ {
	loop:
		println(i)
    }
    // goto不能跳转到其他函数或者内层代码
	goto loop
}
```

# 别名和定义

```go
package main

import "fmt"

func main() {
    // 定义了一个新类型
    type MyInt1 int
    // int类型的别名
	type MyInt2 = int
    var i int = 9
    // 会报错类型不一致
	var i1 MyInt1 = i
	var i2 MyInt2 = i
	fmt.Println(i1, i2)
}
```


# 作用域的遮罩

```go
package main

import (
    "errors"
    "fmt"
)

var ErrDidNotWork = errors.New("did not work")

func DoTheThing(reallyDoIt bool) (err error) {
    if reallyDoIt {
        result, err := tryTheThing()
        if err != nil || result != "it worked" {
            err = ErrDidNotWork
        }
    }
    return err
}

func tryTheThing() (string,error)  {
    return "",ErrDidNotWork
}

func main() {
    fmt.Println(DoTheThing(true))
    fmt.Println(DoTheThing(false))
}
```

# 单例与锁

```go
package main

import "sync"

// 实现一个单例

type singleton struct{}

var ins *singleton
var mu sync.Mutex

//懒汉加锁:虽然解决并发的问题，但每次加锁是要付出代价的
func GetIns() *singleton {
	mu.Lock()
	defer mu.Unlock()

	if ins == nil {
		ins = &singleton{}
	}
	return ins
}

//双重锁:避免了每次加锁，提高代码效率
func GetIns1() *singleton {
	if ins == nil {
		mu.Lock()
		defer mu.Unlock()
		if ins == nil {
			ins = &singleton{}
		}
	}
	return ins
}

//sync.Once实现
var once sync.Once

func GetIns2() *singleton {
	once.Do(func() {
		ins = &singleton{}
	})
	return ins
}

3
```

# 算法整数反转

```go
func reverse(x int) (num int) {
	for x != 0 {
		num = num*10 + x%10
		x = x / 10
	}
	// 使用 math 包中定义好的最大最小值
	if num > math.MaxInt32 || num < math.MinInt32 {
		return 0
	}
	return
}
```
```go
package main

import "fmt"
import "math"

//-2147483648
//2147483647

func reverse(v int) int {
	max10:= math.MaxInt32 / 10
	rv := 0
	for v/10 != 0 {
		rv = rv*10 + (v % 10)
		v /= 10
        }
        //和214748364比较
	if rv > max10 || rv < 0-max10 || (rv == max10 && v > 7) || (rv <= 0-max10 && v < -8) {
		return 0
	} else {
		return rv*10 + v
	}
}
func main() {
	fmt.Println(reverse(-123))
	fmt.Println(reverse(1463847412))
	fmt.Println(reverse(7463847412))
	fmt.Println(reverse(7463847413))
	fmt.Println(reverse(8463847412))
	fmt.Println(reverse(-8463847412))
	fmt.Println(reverse(-8463847413))
	fmt.Println(reverse(-9463847412))
}
```

# map中的指针
```go
package main

import "fmt"

type Test struct {
	Name string
}

var list map[string]Test

func main() {

	list = make(map[string]Test)
	name := Test{"xiaoming"}
    list["name"] = name
    //  因为list[“name”]不是一个普通的指针值，map的value本身是不可寻址的，因为map中的值会在内存中移动，并且旧的指针地址在map改变时会变得无效。 定义的是var list map[string]Test，注意哦Test不是指针，而且map我们都知道是可以自动扩容的，那么原来的存储name的Test可能在地址A，但是如果map扩容了地址A就不是原来的Test了，所以go就不允许我们写数据。你改为var list map[string]*Test试试看。
	list["name"].Name = "Hello"
	fmt.Println(list["name"])
}
```

# bitmap的实现，解决空间复杂问题
```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	b := New()
	b.Add(1)
	b.Add(10000)
	fmt.Println(b.String(), b.Len(), b.Has(2), b.Has(10000))
}

type Bitmap struct {
	words  []uint64
	length int
}

func New() *Bitmap {
	return &Bitmap{}
}
func (bitmap *Bitmap) Has(num int) bool {
	word, bit := num/64, uint(num%64)
	return word < len(bitmap.words) && (bitmap.words[word]&(1<<bit)) != 0
}

func (bitmap *Bitmap) Add(num int) {
	word, bit := num/64, uint(num%64)
	for word >= len(bitmap.words) {
		bitmap.words = append(bitmap.words, 0)
	}
	// 判断num是否已经存在bitmap中
	if bitmap.words[word]&(1<<bit) == 0 {
		bitmap.words[word] |= 1 << bit
		bitmap.length++
	}
}

func (bitmap *Bitmap) Len() int {
	return bitmap.length
}

func (bitmap *Bitmap) String() string {
	var buf bytes.Buffer
	buf.WriteByte('{')
	for i, v := range bitmap.words {
		if v == 0 {
			continue
		}
		for j := uint(0); j < 64; j++ {
			if v&(1<<j) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				fmt.Fprintf(&buf, "%d", 64*uint(i)+j)
			}
		}
	}
	buf.WriteByte('}')
	fmt.Fprintf(&buf, "\nLength: %d", bitmap.length)
	return buf.String()
}
```

https://www.kancloud.cn/cserli/golang/530431
https://pioneerlfn.github.io/2019/01/21/interview-golang/
网络
https://blog.csdn.net/lengxiao1993/article/details/82771768
进程
https://www.cnblogs.com/lxmhhy/p/6212405.html
缓存穿透、缓存击穿、缓存雪崩
https://blog.csdn.net/zeb_perfect/article/details/54135506
综合知识
https://hit-alibaba.github.io/interview/basic/network/TCP.html

# 一次面试有前后两个面试官，面试官问的问题如下，其中还带有面试官的提示：

一面：自我介绍一下吧

网络相关1. TCP 和 UDP 的区别是什么？

2. TCP 的三次握手解释一下，为什么是三次握手？（答三点）

3. UDP 在什么时候使用？

4. 浏览器输入一个域名之后会发生什么？

5. 登陆 baidu 之后，下次打开浏览器就不用再登陆了，怎么做到？

6. https 和 http 的区别是什么？公钥私钥是怎么来的？浏览器如何判断公钥的正确性？

数据库相关1. 事务属性 ACID 都是什么？

2. 乐观锁和悲观锁的区别？乐观锁怎么判断一致性？

3. 用过哪些存储引擎？Innodb 和 Myiasm 的区别是什么？

4. 你们有没有用到其他类型的数据库？（比如mongodb，我说没有，面试官就没让我回答，应该多看看）

其他1. 如何防止死锁，可以举例各种语言中都是怎么做的？（1. 不可以复制锁，2. 还有其他的么）

算法题1. 蛇形打印一棵二叉树上的 value

二面：1. 自我介绍一下

2. 说一下有挑战的项目

RPC 相关：

1. RPC 调用中，客户端都发生了什么，要完成哪些工作？（serialize，connection，parse，我主要说了 serialize 中的 protobuf）

2. RPC 客户端如何处理请求失败？（面试官想问客户端如何实现失败重试机制，我引申了 nginx 中的重试方式，提到了幂等性重试）

架构相关：

1. 水平拓展一个新的机器时，都有哪些操作或步骤？（拉取机器的镜像，下载代码，启动服务）

2. 启动服务的时候，都有个启动的时间，这段时间机器可以有流量么？如果没有流量，什么时候开始可以有流量？用什么机制让 Haproxy 开始给新机器发流量？（应该就是服务发现机制）

3. 同上，更新代码部署时，有个重新启动的时间？这段时间有流量么？如果没有，什么时候恢复流量，怎么恢复流量？比如如何在 Haproxy 注册部署好的机器的 ip ？（我谈到了蓝绿部署，在 proxy 中删掉再恢复机器的 ip）

算法题：有 10 万个数，我们要找出最大的 1000 个数，如何做？（因为只要 1000 个数，甚至不关心它们的顺序，可以用比 nlogn 排序更快的方法，因为可以不用对 10 万个数全都排序） 面试官提示，可以用小根堆，取出前 1000 个数组成一个小根堆，后面不断跟 root 比较，小的抛弃，大的插入堆，并替换 root。


# 我觉得如果要考察一个人对一门语言的熟悉程度，少说也要涵盖这些方面：

语言基础是否学习完整？例：除了 mutex 以外还有那些方式安全读写共享变量？
语言基础是否足够扎实？例：无缓冲 chan 的发送和接收是否同步？
语言细节是否有过了解？例：golang 采用什么并发模型？体现在哪里？
语言生态是否进行关注？例：在 Vendor 特性之前包管理工具是怎么实现的？
考察基础经验，例：说说 golang 中常用的并发模式？
考察实际经验，例：JSON 标准库对 nil slice 和 空 slice 的处理是一致的吗？
考察学习历程，例：学习 Go 的主要途径？读过那些书？

1. 在go语言中，new和make的区别？

new 的作用是初始化一个指向类型的指针(*T)

new函数是内建函数，函数定义：func new(Type) *Type

使用new函数来分配空间。传递给new 函数的是一个类型，不是一个值。返回值是 指向这个新分配的零值的指针。

make 的作用是为 slice，map 或 chan 初始化并返回引用(T)。

make函数是内建函数，函数定义：func make(Type, size IntegerType) Type

-        第一个参数是一个类型，第二个参数是长度

-        返回值是一个类型

make(T, args)函数的目的与new(T)不同。它仅仅用于创建 Slice, Map 和 Channel，并且返回类型是 T（不是T*）的一个初始化的（不是零值）的实例。

2. 在go语言中，Printf()、Sprintf()、Fprintf()函数的区别用法是什么？

都是把格式好的字符串输出，只是输出的目标不一样：

Printf()，是把格式字符串输出到标准输出（一般是屏幕，可以重定向）。

Printf() 是和标准输出文件(stdout)关联的,Fprintf 则没有这个限制.

Sprintf()，是把格式字符串输出到指定字符串中，所以参数比printf多一个char*。那就是目标字符串地址。

Fprintf()， 是把格式字符串输出到指定文件设备中，所以参数笔printf多一个文件指针FILE*。主要用于文件操作。Fprintf()是格式化输出到一个stream，通常是到文件。

3. 说说go语言中，数组与切片的区别？

(1). 数组
数组是具有固定长度且拥有零个或者多个相同数据类型元素的序列。
数组的长度是数组类型的一部分，所以[3]int 和 [4]int 是两种不同的数组类型。

  数组需要指定大小，不指定也会根据初始化的自动推算出大小，不可改变 ;

  数组是值传递;

  数组是内置(build-in)类型,是一组同类型数据的集合，它是值类型，通过从0开始的下标索引访问元素值。在初始化后长度是固定的，无法修改其长度。当作为方法的参数传入时将复制一份数组而不是引用同一指针。数组的长度也是其类型的一部分，通过内置函数len(array)获取其长度。

  数组定义：var array [10]int

            var array = [5]int{1,2,3,4,5}

 

(2). 切片
切片表示一个拥有相同类型元素的可变长度的序列。
切片是一种轻量级的数据结构，它有三个属性：指针、长度和容量。

切片不需要指定大小;

切片是地址传递;

切片可以通过数组来初始化，也可以通过内置函数make()初始化 .初始化时len=cap,在追加元素时如果容量cap不足时将按len的2倍扩容；

切片定义：var slice []type = make([]type, len)

 

4. 解释以下命令的作用？

go env:   #用于查看go的环境变量

go run:   #用于编译并运行go源码文件

go build:  #用于编译源码文件、代码包、依赖包

go get:   #用于动态获取远程代码包

go install:  #用于编译go文件，并将编译结构安装到bin、pkg目录

go clean:  #用于清理工作目录，删除编译和安装遗留的目标文件

go version:  #用于查看go的版本信息

 

5. 说说go语言中的协程？

协程和线程都可以实现程序的并发执行；

通过channel来进行协程间的通信；

只需要在函数调用前添加go关键字即可实现go的协程，创建并发任务；

关键字go并非执行并发任务，而是创建一个并发任务单元；

 

6. 说说go语言中的for循环？

for循环支持continue和break来控制循环，但是它提供了一个更高级的break，可以选择中断哪一个循环
for循环不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量 

7. 说说go语言中的switch语句？

单个case中，可以出现多个结果选项

只有在case中明确添加fallthrough关键字，才会继续执行紧跟的下一个case

 

8. go语言中没有隐藏的this指针，这句话是什么意思？

方法施加的对象显式传递，没有被隐藏起来

golang的面向对象表达更直观，对于面向过程只是换了一种语法形式来表达

方法施加的对象不需要非得是指针，也不用非得叫this

 

9. go语言中的引用类型包含哪些？

数组切片、字典(map)、通道（channel）、接口（interface）

 

10. go语言中指针运算有哪些？

可以通过“&”取指针的地址

可以通过“*”取指针指向的数据

 

11. 说说go语言的main函数

main函数不能带参数

main函数不能定义返回值

main函数所在的包必须为main包

main函数中可以使用flag包来获取和解析命令行参数

 

12. 说说go语言的同步锁？

(1) 当一个goroutine获得了Mutex后，其他goroutine就只能乖乖的等待，除非该goroutine释放这个Mutex

(2) RWMutex在读锁占用的情况下，会阻止写，但不阻止读

(3) RWMutex在写锁占用情况下，会阻止任何其他goroutine（无论读和写）进来，整个锁相当于由该goroutine独占

 

13. 说说go语言的channel特性？

A. 给一个 nil channel 发送数据，造成永远阻塞

B. 从一个 nil channel 接收数据，造成永远阻塞

C. 给一个已经关闭的 channel 发送数据，引起 panic

D. 从一个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回一个零值

E. 无缓冲的channel是同步的，而有缓冲的channel是非同步的

 

14. go语言触发异常的场景有哪些？

A. 空指针解析

B. 下标越界

C. 除数为0

D. 调用panic函数

 

15. 说说go语言的beego框架？

A. beego是一个golang实现的轻量级HTTP框架

B. beego可以通过注释路由、正则路由等多种方式完成url路由注入

C. 可以使用bee new工具生成空工程，然后使用bee run命令自动热编译

 

16. 说说go语言的goconvey框架？

A. goconvey是一个支持golang的单元测试框架

B. goconvey能够自动监控文件修改并启动测试，并可以将测试结果实时输出到web界面

C. goconvey提供了丰富的断言简化测试用例的编写

 

17. go语言中，GoStub的作用是什么？

A. GoStub可以对全局变量打桩

B. GoStub可以对函数打桩

C. GoStub不可以对类的成员方法打桩

D. GoStub可以打动态桩，比如对一个函数打桩后，多次调用该函数会有不同的行为

 

18. 说说go语言的select机制？

A. select机制用来处理异步IO问题

B. select机制最大的一条限制就是每个case语句里必须是一个IO操作

C. golang在语言级别支持select关键字

 

19. 说说进程、线程、协程之间的区别？

进程是资源的分配和调度的一个独立单元，而线程是CPU调度的基本单元；

同一个进程中可以包括多个线程；

进程结束后它拥有的所有线程都将销毁，而线程的结束不会影响同个进程中的其他线程的结束；

线程共享整个进程的资源（寄存器、堆栈、上下文），一个进程至少包括一个线程；

进程的创建调用fork或者vfork，而线程的创建调用pthread_create；

线程中执行时一般都要进行同步和互斥，因为他们共享同一进程的所有资源；

进程是资源分配的单位 

线程是操作系统调度的单位 

进程切换需要的资源很最大，效率很低 
线程切换需要的资源一般，效率一般 
协程切换任务资源很小，效率高 
多进程、多线程根据cpu核数不一样可能是并行的 也可能是并发的。协程的本质就是使用当前进程在不同的函数代码中切换执行，可以理解为并行。 协程是一个用户层面的概念，不同协程的模型实现可能是单线程，也可能是多线程。

进程拥有自己独立的堆和栈，既不共享堆，亦不共享栈，进程由操作系统调度。（全局变量保存在堆中，局部变量及函数保存在栈中）

线程拥有自己独立的栈和共享的堆，共享堆，不共享栈，线程亦由操作系统调度(标准线程是这样的)。

协程和线程一样共享堆，不共享栈，协程由程序员在协程的代码里显示调度。

一个应用程序一般对应一个进程，一个进程一般有一个主线程，还有若干个辅助线程，线程之间是平行运行的，在线程里面可以开启协程，让程序在特定的时间内运行。

协程和线程的区别是：协程避免了无意义的调度，由此可以提高性能，但也因此，程序员必须自己承担调度的责任，同时，协程也失去了标准线程使用多CPU的能力。

# k8s
https://blog.csdn.net/huakai_sun/article/details/82378856
https://zhuanlan.zhihu.com/p/58548703
https://www.bbsmax.com/A/Vx5M6G2mdN/
