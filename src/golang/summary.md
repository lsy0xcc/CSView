### 进程、线程、协程的区别？<Badge text="掌握" type="tip" />

**进程**：进程是每一次程序动态执行的过程，是程序运行的基本单位。进程占据独立的内存，有内存地址，有自己的堆，上级挂靠操作系统，操作系统以进程为单位分配资源(如CPU时间片、内存等)，进程是资源分配的最小单位。

**线程**：线程又叫做轻量级进程，是CPU调度的最小单元。线程从属于进程，是程序的实际执行者，一个进程至少包含一个主线程，也可以有多个子线程。线程会共享所属进程的资源，同时线程也有自己的独占资源。线程切换和线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

**协程**：协程是一种用户态的轻量级线程，协程的调度完全由用户控制。一个线程可以有多个协程，协程不是被操作系统内核所管理，而是由程序所控制。

**区别**

- **拥有资源**：进程是拥有资源的最小单位，线程不拥有资源，但是可以访问隶属进程的资源。进程所维护的是程序所包含的资源静态资源)， 如：**地址空间，打开的文件句柄集，文件系统状态，信号处理handler等**；线程所维护的运行相关的资源(动态资源)，如：**运行栈，调度相关的控制信息，待处理的信号集等**。
- **并发性**：不仅进程可以并发执行，同一进程的多个线程也可以并发执行。

- **系统开销**：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显大于创建或撤消线程时的开销。但是进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个进程死掉就等于所有的线程死掉，所以**多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些**。
- **协程和线程**：协程避免了无意义的调度，由此可以提高性能，但是用户调度过程中可能存在风险。



### goroutine相比线程的优势？<Badge text="掌握" type="tip" />

协程拥有极高的执行效率，子程序切换不是线程切换而是由程序自身控制，所以没有线程切换的开销。和多线程比，线程的数量越多，协程的性能优势就越明显。

协程不需要多线程的锁机制，因为只有一个线程，所以不存在同时写变量的冲突。在协程中控制共享资源不加锁，只需要判断状态就可以，执行效率比多线程要高。



### go与Java的区别？<Badge text="掌握" type="tip" />

**运行**：go是静态编译语言；Java基于类的面向对象语言，Java应用程序在JVM上运行。

**函数重载**：go上不允许函数重载，必须具有方法和函数的唯一名称；java允许函数重载。

**多态**：Java默认允许多态，而go没有。

**路由配置**：go语言使用HTTP协议进行路由配置；java使用Akka.routing进行路由配置。

**继承**：go的继承通过匿名组合完成，基类以Struct的方式定义，子类只需要把基类作为成员放在子类的定义中，支持多继承；Java的继承通过extends关键字完成，不支持多继承。



### go语言中是如何实现继承的？<Badge text="掌握" type="tip" />

在go中没有extends关键字，所以go并没有原生级别的继承支持。本质上，Go使用组合来代替继承：

```go
type Person struct {
	Name string
	Age  int
}

type Student struct {
	Person
	School string
}
```



### for遍历多次执行goroutine会存在什么问题？<Badge text="掌握" type="tip" />

**在协程中打印for的下标i或当前下标的元素**

会随机打印载体中的元素。

golang值拷贝传递，for循环很快就执行完了，但是创建的10个协程需要做初始化：上下文准备，堆栈，和内核态的线程映射关系的工作，是需要时间的，比for慢，等都准备好了的时候，会同时访问i。这个时候的i肯定是for执行完成后的下标(也可能有个别的协程已经准备好了，取i的时候，正好是5，或者7，就输出了这些数字)。

解决的方法就是闭包，给匿名函数增加入参，因为是值传递，所以每次for创建一个协程的时候，会拷贝一份i传到这个协程里面去，或者在开启协程之前声明一个新的变量 = i。

**for并发读取文件**

程序会panic:too many open files

解决的方法：通过带缓冲的channel和sync.waitgroup控制协程并发量。



### init函数是什么时候执行的？<Badge text="掌握" type="tip" />

**特点**：

- init函数先于main函数自动执行，不能被其他函数调用。
- init函数没有输入参数、返回值。
- 每个包可以有多个init函数，包的每个源文件也可以有多个init函数。
- go没有明确定义同一个包的init执行顺序，编程时程序不能依赖这个执行顺序。
- 不同包的init函数按照包导入的依赖关系决定执行顺序。

**作用**：

- 初始化不能采用初始化表达式初始化的变量。
- 程序运行前的注册。
- 实现sync.Once功能。

**执行顺序**：

go程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

- 初始化导入的包，包的初始化顺序并不是按导入顺序执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化
- 初始化包作用域的变量，runtime解析变量依赖关系，没有依赖的变量最先初始化
- 执行包的init函数

**最终初始化顺序：变量初始化 -> init() -> main()**
