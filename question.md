rule of three:
如果一个类显式地定义下列其中一个成员函数，那么其他两个成员函数也应该一起被定义。也就是说这三个函数要么都不定义，要么都要定义。
析构函数
复制构造函数
重载赋值运算符
C++标准中规定如果一个类没有显式地声明以下几个函数，编译器将自动生成默认版本
构造函数
拷贝构造函数
赋值操作符
析构函数
如果对象构造函数中申请了资源，显示声明了析构函数释放资源。那么也需要显示声明拷贝构造函数和重载赋值运算符，否则将使用编译器提供的默认函数，将对象作为值语义进行浅拷贝，导致资源在拷贝中发生错误。
根据需要，显示声明拷贝构造函数和重载赋值运算符，实现深拷贝。
或者如果资源禁止复制，可以显式地禁止拷贝构造函数和赋值运算符函数。防止资源拷贝使用资源。STL是值语义，会对对象进行复制，RAII禁止复制，可以在STL中可以存储对象的shared_ptr。
C++11后改为rule of five
析构函数、拷贝构造函数、复制赋值函数三个函数中的任意一个函数，编译器不再自动生成移动构造函数及移动赋值运算符默认实现。如果一个类支持移动语义，需要显式地声明并实现这两个函数。

extern c：
extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码。加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。影响符号表的生成，表示编译生成的内部符号名使用C约定。
函数被C++编译后在符号库中的名字与C语言的不同。C++支持函数重载，编译器编译函数的过程中会将函数的参数类型也加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，只包括函数名。

移动构造函数：没有内存拷贝，只有对象资源的所有权从一个对象转移到另一个对象。避免临时对象的拷贝和析构。临时变量不需要深拷贝，如果没有移动构造函数，只有拷贝构造函数并实现了深拷贝，临时对象的拷贝和析构会带来较大的开销。

std::move():将一个左值强制转化为右值引用，继而可以通过右值引用使用该值，以用于移动语义

在泛型编程中，常常需要将参数原封不动的转发给另外一个函数。
std::forward():实现了参数在传递过程中保持其值属性的功能，即若是左值，则传递之后仍然是左值，若是右值，则传递之后仍然是右值。

对于右值引用类型变量T&&： 通过引用折叠，此参数可以匹配任何类型的变量，右值经过它传递类型保持不变还是右值，而左值经过它变为普通的左值引用。
输入参数使用T&&类型，然后再std::forward<T>。右值引用类型是独立于值的，一个右值引用参数作为函数的形参，在函数内部再转发该参数的时候它已经变成一个左值。
。
左值/纯右值/将亡值(将要被移动的对象，函数右值类型返回值/std::move的返回值)
1、左值可以寻址，而右值不可以。
2、左值可以被赋值，右值不可以被赋值，可以用来给左值赋值
3、左值：能对表达式取地址、或具名对象/变量
右值：不能对表达式取地址，或匿名对象。


多线程基础：

* cache一致性：cache coherence 作用于cpu的cache和内存memory之间
多个cpu有独立的L1/L2 cache，共享L3cache
写回策略：
write back:脏数据写回到cache，随后才更新内存
write through:脏数据写回memory，并更新cache。

write invalidate:write时发出invalidate信号，使其他所有cpu的L1/L2cache中同一cacheline失效。(实现简单/导致其他cpu再次读取时会出现cache miss)
write update:
write时，同时更新其他所有cpu的L1/L2cache中同一cacheline。

大部分cpu使用write invalidate策略。基于此开发出经典的MESI协议。
每个Cache line都有4个状态，可用2个bit来表示。在MESI协议中，每个CPU都会监听总线(bus)上的其他CPU对每个Cache line的所有操作。所有的CPU都是在一个共享的总线上，多个CPU之间需要相互通信以保证Cache line在M、E、S、I四个状态间正确的转换，从而保证数据的一致性。
MESI protocal:Invalid 无数据 shared：多节点共享/与memory一致  exclusive：单节点持有/与memory一致 modified：与memory不一致/单节点持有。

* store-buffer:cpu0发起一次对某个地址的写操作，但是local cache没有数据，该数据在CPU1的local cache中，因此，为了完成写操作，CPU0发出read invalidate的命令，invalidate其他CPU的cache数据。只有完成了这些总线上的transaction之后，CPU0才能正在发起写的操作，这是一个漫长的同步等待过程。 store-buffer把这一过程变成了异步。

cpu和cache之间加入了store-buffer。一旦增加了store buffer，那么cpu修改数据就无需等待其他CPU的响应(响应已将对应cacheline invalid了)，只需要将要修改的内容放入store buffer，然后继续执行。当cache line完成了bus transaction，并更新了cache line的状态后，要修改的数据将从store buffer进入cache line。引入了store buff，带来了一些复杂性，一不小心，会带来本地数据不一致的问题。一个变量可能在cacheline和store-buffer中有不同的值。

store forwarding，当CPU执行load操作的时候，不但要看cache，还有看store buffer是否有内容，如果store buffer有该数据，那么就采用store buffer中的值。

由于store-buffer存在，cpu修改数据无需等待其他CPU的响应，CPU在发出read invalidate message后，并没有等待CPU1收到，就继续执行后续的写入，更新了cache-line。可能导致后续写入被其他cpu先看到。

提出了memory-barrier。CPU在遇到内存屏障后，可以继续前行，但是需要记录一下当前store buffer中的数据顺序，在当前store buffer中的数据严格按顺序全部写回cache line之前，其他数据不能先更新cache line，需要按照顺序先写到store buffer才能继续前行。

* Invalidate Queue：store-buffer容易溢出。其他CPU回应invalidate acknowledge比较慢，如果能够加快这个过程，让store buffer尽快进入cache line，那么也就不会那么容易填满了。

invalidate acknowledge不能尽快回复的主要原因是invalidate cacheline的操作没有那么快完成，特别是cache比较繁忙的时候，这时，CPU往往进行密集的loading和storing的操作，而来自其他CPU的，对本CPU local cacheline的操作需要和本CPU的密集的cache操作进行竞争，只要完成了invalidate操作之后，本CPU才会发生invalidate acknowledge。此外，如果短时间内收到大量的invalidate消息，CPU有可能跟不上处理，从而导致其他CPU不断的等待。

引入一个缓冲队列，接收到invalidate请求，可以先将请求入队缓冲队列，就可以回复acknowledgement消息了，后面在异步完成invalidate操作。

没有及时处理invalidate queue中的a的invalidate消息，会导致使用本cache line中的一个已经是invalid的一个旧的值。这个时候，我们也需要一个memory barrier指令来告诉CPU，这个时候应该需要处理invalidate queue中的消息了，否则可能会读到一个invalid的旧值。

当CPU执行memory barrier指令的时候，对当前Invalidate Queue中的所有的entry进行标注，这些被标注的项被称为marked entries，而随后CPU执行的任何的load操作都需要等到Invalidate Queue中所有marked entries完成对cacheline的操作之后才能进行。

全功能的内存屏障既会mark store buffer也会mark invalidate queue。load barrier只mark invalidate queue，store barrier只需mark store buffer。

* 原子操作：
读写一个字长的数据(按字节对齐的)操作是原子的，RMW(读修改写)不是原子的
原子指令：CMPXCHG XCHG XADD
CAS(compare and swap)比较并交换 
Compare-And-Swap(CAS)被认为是最基础的一种原子性RMW操作：
c++11中compare_exchange_weak()就是最基础的CAS。
cas(volatile int *addr,int oldval,int newval)
一个原子的RWM操作 CMPXCHG(比较并交换操作数，累加器中的值于目的操作数比较，如果相等，将源操作数装载到目标操作数处。否则将目标操作的值装载到累加器) oldval 装入eax寄存器，lock 前缀 + CMPXCHG *addr newval
目标操作数在memory中，需要使用lock前缀
lock前缀 代表当前指令所操作的内存在指令执行期间只能被当前cpu使用。 

消除非原子操作：
simple read/write 字节对齐
RMW c++11 原子操作/ linux原子内存获取的内建函数。

* 内存序：
读/写重排序
单线程：重排序前后有相同的执行效果
编译器重排序/cpu内存序
编译器重排序能提高程序运行效率 -> 编译器内存屏障 compiler memory barrier 只是一个标识告诉compiler不要在指令上下部分重排序

cpu重排序：
读读乱序/读写乱序/写读乱序/写写乱序    四种乱序 X86只有写读乱序

引入cpu memory barrier：cpu内存屏障是一个cpu指令
load barrier  store barrier(store 操作不能跨过store barrier)  full barrier
read acquire    write release 两种barrier
loadload+loadstore  loadstore+storestore

程序在多核CPU中执行服从program order，需要成对使用的memory barrier。成对的memory barrier并不能提供绝对的顺序保证，只能提供有条件的顺序保证。
read acquire + write release 是所有锁的实现基础。所有被{read acquire,write release} 包含区域构成了临界区，确保中间的指令不会在临界区外执行。

内存屏障：
1、插入一个内存屏障，相当于告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行。
2、内存屏障另一个作用是强制更新一次不同CPU的缓存。一个写屏障会标记store buffer(在当前store buffer中的数据严格按顺序全部写回cache line之前，其他数据不能先更新cache line，需要按照顺序先写到store buffer才能继续前行)。一个读屏障会标记invalidate queue(屏障后CPU执行的任何的读操作都需要等到Invalidate Queue中所有标记项完成对cache line的操作之后才能进行)


* volatile 关键字：
易变性：所谓的易变性，在汇编层面反映出来，就是两条语句，下一条语句不会直接使用上一条语句对应的volatile变量的寄存器内容，而是重新从内存中读取。
不可优化：“不可优化”特性。volatile告诉编译器，不要对这个变量进行各种激进的优化，甚至将变量直接消除，保证程序员写在代码中的指令，一定会被执行。(非volatile变量可能被编译器优化掉，进行常量替换等)
顺序性：多线程中的顺序性要求。C/C++ Volatile变量与非Volatile变量间的操作顺序，有可能被编译器交换。C/C++ Volatile变量间，编译器是能够保证不交换顺序。但是cpu执行还是可能乱序的。同时，C/C++ Volatile关键词，并不能用于构建happens-before语义。
Java的Volatile也有这三个特性，但最大的不同在于：第三个特性，”顺序性”，Java的Volatile有很极大的增强，Java Volatile变量的操作，附带了Acquire与Release语义。
因此Java Volatile，能够用来构建happens-before语义。

* happens-before:
如果操作A和B是由同一个线程执行的，并且A的语句在B的语句之前（按程序顺序program order），那么A happens-before B
A和B表示由多线程进程执行的两个操作。如果A happens-before B，那么执行B的线程在执行B之前可以有效地看到A对内存的写入。


保护变量guard variable和有效的内容payload。payload是在线程之间传播的一组数据，而保护变量保护对payload的访问，保护变量用于指示所保护的内容是否准备好。

建立happens-before关系：1、单线程 2、线程间synchronizes with
synchronizes with：1、Mutex lock/unlock 2、Thread create/join 3、acquire/release 语义

提供acquire/release语义的：c++11的原子类型/acquire/release fence /jave volatile

任何时刻，只要存在两个或多个线程并发地对同一个共享变量进行操作，并且这些操作中的其中一个是执行了写操作，那么所有的线程都必须使用原子操作。
如果违反上面的规则，即存在某个线程使用了非原子操作，那么将会陷入一个在C++11标准中称之为数据竞争(data race)。如果你引发了数据竞争，那么就会得到一个"未定义行为（undefined behavior）"的结果，它们会导致torn reads(撕裂读)和torn writes(撕裂写)，也就是一个非完整的读写。

顺序一致性：顺序一致性模型要求线程的操作具有原子性，也就说线程的任何操作都是马上对其它线程可见的。


1、有些同步原语是操作系统的内核对象，调用该原语会带来昂贵的上下文切换(用户态切换到内核态)代价，同时，内核对象是一个比较有限的资源。

2、锁的不足之处：
1. 因为忘记使用锁而导致条件竞争(race condition)
2. 因为不正确的加锁顺序而导致死锁(deadlock)
3. 因为未被捕捉的异常而造成程序崩溃(corruption)
4. 因为错误地忽略了通知，造成线程无法正常唤醒(lost wakeup)

Lock Free的程序能够确保执行它的所有线程至少有一个能够继续往下执行。
3、锁可能导致线程阻塞，线程阻塞需要阻塞、重新调度一个可执行线程、唤醒、从等待队列中移除放入可执行队列中等一系列操作。线程数比较多的情况下，阻塞线程不会释放所占用的资源，影响并发性能。

4、多个线程争抢共享资源，使用锁交给操作系统来裁决。CAS操作是一个CPU级别的指令，CAS操作比锁消耗资源少的多，因为它们不牵涉操作系统，它们直接在CPU上操作。单线程无锁耗时300ms，单线程有锁耗时10000ms，单线程使用CAS耗时5700ms。内存屏障作为另一个CPU级的指令，没有锁那样大的开销。内核并没有在多个线程间干涉和调度。




```c++
uint32_t fetch_multiply(std::atomic<uint32_t>& shared, uint32_t multiplier)
{
    uint32_t oldValue = shared.load();
    while (!shared.compare_exchange_weak(oldValue, oldValue * multiplier))//cas失败会将shared的最新值放入oldvalue
return oldValue;
```
对多个变量的RMW
```c++
struct Terms
{
uint32_t x;
uint32_t y;
};
std::atomic<Terms> terms;
void atomicFibonacciStep()
{
Terms oldTerms = terms.load();
Terms newTerms;
do
{
newTerms.x = oldTerms.y;
newTerms.y = oldTerms.x + oldTerms.y;
}
while (!terms.compare_exchange_weak(oldTerms, newTerms));
}
```
C++11的原子库std::atomic<> template可以是任何类型(int、bool等buil-in type，或user-defined type)，但并不是所有的类型的原子操作是lock-free的。
redis的一个set过于大：
如果单个集合中元素过多，那么我们将其拆分成多个小集合，然后再通过路由算法找到具体的子集即可。以哈希表为例，将大哈希表拆分为5个小哈希表。那么拆分后，先计算key的哈希值，利用hash(key)%5得到key到底落在哪个哈希表上即可。同理集合，有序集合，链表的处理方式也是一样。

