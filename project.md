使用非阻塞I/O，基于epoll实现I/O复用。每个I/O线程有一个eventloop(循环处理epoll上的普通I/O事件和定时事件以及任务队列里的任务)。主线程负责accept请求，接受TCP连接，然后以轮询的方式，分发给线程池里的某个普通I/O线程。 好处(nginx请求多阶段异步处理)。

 1、异步唤醒线程
 写reactor/eventloop时异步的唤醒阻塞在epoll_wait上的线程。传统方法是使用pipe（1、但是pipe是半双工的，需要两个文件描述符。2、eventfd只是单纯的用于事件通知，内核开销要低的多，不涉及缓冲区的管理），linux提供了eventfd，效率更高。写入一个uint64_t的数。 // （内核为eventfd关联一个对应的struct file结构。实际上它既不对应硬盘上的一个真实文件，也不占用inode，它是对内存中一个数据结构的抽象，这个数据结构的核心是一个64位无符号型整数。在内核中使用时，eventfd文件描述符可以提供从内核到用户空间的桥梁，例如，允许KAIO（kernel AIO）等功能向文件描述符发送信号通知一些操作已经完成了。eventfd文件描述符的一个关键点是，可以像使用select（2）、poll（2）或epoll（7）像监听普通文件描述符一样监视它。这意味着应用程序可以同时监视“普通”文件的就绪情况以及支持eventfd接口的其他内核机制的就绪情况。查看eventfd当前值 /proc/[pid]/fdinfo）
（eventfd:维护一个等待队列/count计数值/ 等待队列节点：自旋锁和struct list_head task_list以队列的形式存储阻塞task。使用自旋锁spin_lock_irq禁止中断和抢占，非阻塞读如果count大于0返回8，count读入buffer，否则返回EAGAIN。阻塞读且count=0时，将当前进程放入等待队列并阻塞。阻塞就是进入判断循环，设置task_interruptable状态，释放自旋锁，调用schedule()调度其他进程。只要count值被更新大于0且又再次获得cpu时间，那么就可以跳出循环继续执行而返回了(持有自旋锁) ,write函数也有当确认可以成功返回时，主动调用wakeup_locked的过程，这样就能实现write后立即向等待队列通知的效果了
POLL:count不为0，EPOLLIN，小于ULLONG_MAX - 1可写)。

 (管道：通过将两个文件对象file结构指向同一个临时的inode，而这个inode又指向一个物理页面而实现的。两个file struct数据结构，f_op字段指向不同文件操作函数，其中一个是向管道中写入数据的函数，而另一个是从管道中读出数据的函数。内核提供一定的机制同步二者对管道的访问，同时需要管理缓冲区，写入是否有足够空间，读取是否有足够的数据）

 2、定时任务。传统reactor利用epoll_wait的超时时间来实现定时功能。epoll_wait的超时是毫秒为单位，使用timerfd_create(2)/timerfd_settime()定时的精度是纳秒，内核管理的timerfd底层是内核中的hrtimer（高精度时钟定时器）。使用timerfd把时间变成文件描述符，定时器超时变成文件描述符可读，可以很好的融入epoll的框架，用统一的方法来处理I/O事件和超时事件。
 关于时间函数，jvm使用clock_gettime()精度更高为纳秒，但是它系统调用的开销大于gettimeofday(精确到微妙)。(nginx时间缓存，jvm没有缓存，每一次都获取时间)。(vsyscall和vDSO是用于加速某些系统调用的两种机制。Vsyscall页面在每个进程中是静态分配了相同的地址，分配的内存较小，只允许4个系统调用。vDSO提供和vsyscall相同的功能，同时解决了其局限。vDSO是动态分配的，地址是随机的。
 可以提供超过4个系统调用。virtual system call(vsyscall):内核允许包含当前时间的页面以只读方式mmap到用户空间,该页面还包含一个快速的gettimeofday()实现。使用这个虚拟系统调用，C库提供一个快速的gettimeofday()，实际上不必陷入内核态。) strace看过！
 timerfd：uint_64 clicks超时次数/expire超时标记/等待队列/ 定时器有高精度定时器hrtimer或alarm(精度为秒，发送SIGALRM信号)/自旋锁 cancel。
 timerfd_create(为上下文结构分配空间/初始化等待队列/调用hrtimer_init初始化一个hrtimer/传入timerfd的文件操作指针struct file_operations timerfd_fops，创建一个文件，返回文件描述符)
 timerfd_settime(将原有定时器取消/调用hrtimer_start启动定时器，其超时函数被设置为timerfd_tmrproc)
 timefd_tmrproc是timerfd的定时器超时函数。在timerfd超时时，该函数会设置定时器超时标记位；增加定时器超时次数（在设置定时器循环模式时，可能会出现多次超时没有被处理的情况）；唤醒等待队列，从而唤醒可能存在的正被阻塞的read、select。
 read:读到的是定时器的超时次数，将ticks和超时标记清0。该函数在阻塞模式下会把当前task放到timerfd的等待队列中，等待定时器超时时被唤醒。
 timefd_poll:poll_wait向等待队列中添加等待项，如果ticks大于0，返回events |= EPOLLIN。 

 关于定时器队列的数据结构选择：
 1、二叉堆 libevent
 2、红黑树 nginx
 两者的插入都是logn的复杂度，堆的常数项更小，插入会快。堆的内存局部性更好，速度可能略快与红黑树。但是STL现有的堆的函数不能高效的删除某个定时器，需要定时器添加一个字段记住在堆中的位置，自己实现堆的接口。(使用惰性删除，消耗内存，不立即删除，在定时器中加一个字段invalid，删除时将尾部节点与cur交换，如果小于父节点上溯，如果大于子节点下溯，需要调整堆结构，带来开销)
 使用红黑树对定时器按到时时间先后排序。最左的叶子节点就是最早到时的定时器，通过timerfd_settime将它注册到timerfd上。
 不能使用map<timestamp,timer>，因为map无法处理两个timer到期时间相同的情况。所以使用set，元素为pair{timestamp,timer*}。
 用lower_bound({now,nullptr}) 返回第一个未超时的时钟。从定时器队列begin()到第一个未超时时钟之间就是到时的时钟。

 时间轮：插入和删除定时任务都是常规时间。缺点在于定时精度和占用空间，要求足够高的定时精度时所需空间大，而且表示定时范围有有限。适用于空闲连接的超时踢除(大量定时任务，不需要非常精确的定时，定时的范围有限)。
 3、nginx文件AIO
 4、MPMC的无锁任务队列 由于每个Eventloop对应一个I/O线程。Eventloop任务队列，提供了将任务放到某一特定I/O线程中串行执行的接口。像（操作epoll添加/修改监听事件/或者对epoll上的socket的读写的任务，放到任务队列中执行，避免多线程的race condition。这些I/O任务多线程也无法提高执行效率）
 基于数组实现。维护一个入队偏移head，一个出队偏移tail。每个slot用于存储任务，持有一个原子变量turn(保护变量)，往slot中放入任务和取出任务时都将turn+1，根据slot的turn标志可以判断这个slot中是否有任务。

 阻塞接口：放任务时，原子的增加入队偏移head(fetch_add),入队偏移对数组长度取余，找到本次操作对应slot。然后load对应slot的turn标志，如果判断结果是slot中无任务，就放入任务，然后给slot的turn标志store新的值。使用同样的方法取任务。

 * 这里的关于turn标志的store和load时需要指定内存序。

 （由于原子的修改了入队/出队偏移，必须取/放任务）在load到turn的值不符合条件的情况下，只能自旋等待。

 非阻塞接口：放任务时，load入队偏移head，对数组长度取余求出对应slot，然后load对应slot的turn标志，如果可以放任务，就CAS修改入队偏移head。如果cas修改入队偏移成功，就可以放任务(compare_exchange_strong 不允许假失败)，并且给slot store的turn标志新的值。如果修改失败可以重新获取入队偏移，继续尝试。如果load到slot的turn值不符合条件，这时没有修改入队偏移，可以立马退出(可以再load一次head，如果变化重新判断，否则退出)。

 SPSC无锁队列。只需维护入队偏移head和出队偏移tail。放任务时，load入队偏移和出队偏移，根据入队偏移和出队偏移即可判断当前slot是否能放置任务。如果可以，放任务，store新的入队偏移。(入队偏移只有自己一个线程会写入，relax内存获取，load出队偏移时要和取任务的线程同步，acquire内存序)。
 
 5、RAII，在编写程序时，我们总是设法保证对象的构造和析构是成对的(栈上对象/智能指针)否则就会有内存泄漏。利用这个特性，可以使用对象来包装资源，把资源管理和对象生命期统一起来。(Tcpconnection/mutex)
智能指针/引用计数管理区域->原子操作保证对引用计数值的访问修改线程安全->shared_ptr线程安全问题
计数安全问题->enable_shared_from_this
RAII类型继承于boost::noncopyable，而后在STL容器中使用引用计数的指针。防止STL容器对Resource的复制将导致运行期错误(STL容器类是值语义的，RAII对象是对象语义)。防止拷贝继承noncopyable的类，boost提供了。noncopyable类的复制构造函数和赋值运算符函数声明为=delete，禁止拷贝和赋值。RAII类型继承noncopyable类，使用编译器生成的默认复制构造函数和赋值运算函数会调用基类的对应函数，会报错。=default显式的指定编译器为该函数自动生成默认函数体。Deleted可以禁用某些不期望的函数。Deleted函数特性还可以用来禁用某些用户自定义的类的new操作符，从而避免在自由存储区创建类的对象。


RPC是远程过程调用。在一台机器上运行的主程序，可以调用另一台机器上准备好的子程序，就像LPC(本地过程调用)。通过RPC，可以充分利用非共享内存的多处理器环境(例如通过局域网连接的多台应用服务器)，可以简便地将应用分布在多台应用服务器上，应用程序就像运行在一个多处理器的计算机上一样。方便的实现过程代码共享，提高系统资源的利用率，也可以将以大量数值处理的操作放在处理能力较强的系统上运行，从而减轻前端机的负担。RPC的主要目标是让构建分布式计算/应用更容易、透明，在提供强大的远程调用能力时不损失本地调用的语义。

服务调用者对同一个地址使用TCP长连接，单一长连接适合少量多次的调用方式，比较适合RPC的场景。
服务调用方封装调用信息(请求的service_name,method_name,消息id，和消息类型(request/response)，请求参数Request,错误信息等控制信息)，将消息序列化。并进行封包，在头部加入信息长度，用一个int长度表示。接收端根据头部长度解包，反序列化。根据service_name,method_name,请求参数Request调用相应过程，并返回响应结果。

客户端中发送到同一个地址的多个RPC调用请求复用一条TCP连接。RPCchannel为每次RPC调用生成唯一的信息id，并维护一个列表，map<id,请求对应项(响应原型和注册的回调函数等)>。响应信息中包含相同的信息id，根据信息id，在列表中找到响应对应的RPC调用请求项。（RPCchannel多线程安全。消息的发送调用Tcpconnection::send函数，发送是将任务放到它注册的Eventloop的任务队列中执行，不存在线程间的竞争）。对一个address建立一条RPC连接，发送请求前到连接池中取对应连接(以address作为key)，如果连接不存在，创建唯一连接。(map使用锁保护，lock_map每个key对应一把锁）。（连接池中还有一个lockmap。对map整体加锁，查询address对应连接，如果没有或者失效，lockmap中取address对应的lock（没有就创建），释放map的锁。获取lock_map对应锁，再次加map锁，检查是否有连接，释放map锁，创建连接，获得map锁，放入连接，释放map锁，释放lockmap锁。)

负载均衡：
1、哈希一致性：
一致性Hash常用于缓存，效果是多个节点按Hash映射到一个圆环上，每个请求按自己的Hash顺时针找到最近的节点。假如有个A节点挂了，那只是所有原先到A节点的请求需要重定位，其余节点不受影响。为了缓解节点映射到圆环上不均匀的数据倾斜问题，常使用虚拟节点。

将每个可用的server的ip+port计算hash作为节点插入map<hashval,address>中（同时插入一定的虚拟节点，在host后加入字符计算hash），然后对调用方法的参数计算哈希值hashcode（保证相同参数的请求发往同一主机，这样server端可以缓存结果，减少计算量），在map中寻找第一个大于hashcode的节点（如果没有就是map中最小的，也就是哈希值圆环环上的右侧第一个点），返回server的ip+port值。
2、leastunreplied:统计发送每个server调用{service，method}方法的请求的三种状态，有多少没有收到回复unreplied，有多少次成功收到回复success，有多少次失败failed。在LB方法leastunreplied中用这些信息来做客户端的负载均衡。每次发送请求给对应的尚未回复+1，当接收到对应回复给它-1并根据成功还是失败分别给成功数+1或是失败数+1。每个服务提供者的unreplied数可以表征这个服务提供者的繁忙程度，每次挑选尚未回复数最小的那个发送请求。也可以加权成功数和失败。


redis官方提供的C语言客户端hiredis。它有同步调用和异步调用。它的异步调用的事件是注册到第三方事件库上的，它为此写了adapter文件。这里我使用我网络库作为事件注册和通知模块。主要提供将hiredis定义的redis异步连接上下文redisAsyncContext的I/O事件注册到epoll上的接口,将hiredis提供的对应的事件处理函数绑定到相应回调函数。

Redis连接池：redis的hiredis客户端连接成功后返回连接上下文redisContext/redisAsyncContext对象。但是这个对象并不是线程安全的，(比如redisAsyncContext它把命令送入缓冲区中就返回了。但这个缓冲区是没有保护的)不能被多线程间复用。为了避免连接频繁创建销毁连接建立了连接池，每个线程在每次发送消息给redis前取出连接，使用完将连接放回连接池。

数据结构使用boost::circular_buffer。它是对vector的包装，是一个长度有限的vector。当放满元素后，push_front会用新元素覆盖队尾的元素，push_back会覆盖队首元素，可做FIFO和LIFO的cache。增加一个unread字段统计队列中元素的个数(放元素时push_front,并将unread+1,取元素时只需要调整unread-1而省去了一次pop操作。如果元素是对象，pop还会调用对象的析构函数，有一定的时间开销。
初始化连接池时，调用hiredis异步建立连接的接口，在成功建立连接的回调函数中将连接上下文对象放入连接池。

async/future接口
同步接口通过异步接口实现，异步接口返回future结构。
用标志位ready表示是否拿到了response。
get接口用于获取response。如果标志是false，则阻塞在condition上，直到标志变为true，返回response字段。
set接口用于填入response。修改标志为true,调用condition.notify通知等待线程。   


gpbcodec:将google::protobuf::Message类序列化成string，将string反序列化成Message。google::protobuf序列化不包括类型名和长度，另外加入了长度信息。
gpbcodecT:引入模板，让concrete_Message类使用gpbcodec的接口。concerete_Message是自定义message，在消息外又封装了消息类型错误码等字段，是Message类的派生类。

。
使用redis实现了简单的服务发现的功能。使用ZSET存储服务信息。key为service_name,val为provider的地址，score为信息注册时的时间戳。使用score的意义在于如果provider突然死亡，没能下线，通过score时间戳可以发现这种情况。
客户端和服务器通过Pub/Sub模型完成服务信息变更的监听。service_name是pub/sub中的一个topic。客户端建立本地cache缓存服务信息

服务端：publisher：
注册时，调用ZADD给redis添加注册信息，服务器增加定时任务，定期刷新score时间戳(修改时间戳，再次ZADD)。publish发布消息给所有subscribe了service的客户端。
取消注册时。删除对应的定时任务。调用ZREM命令删除redis的注册信息，最后调用publish发布消息给所有subscribe了service的客户端。

subscriber:
建立本地cache 缓存Service对应的可用provider
subscribe：
发送SUBSCRIBE service命令监听关于service的所有发布的消息。在subscribe_callback中，解析消息，根据是register/unregister/refresh操作，分别在本地cache中插入/删除{service，host}条目，以及向redis发送ZRANGE service 0 -1 命令获取service所有可用provider更新cache内容。
查询接口：
根据Service查询可用provider list的接口。先在本地cache中查询service，如果找不到对应项，就说明还没有subscribe这个service，先subscribe该service，然后向redis发送ZRANGE service 0 -1 WITHSCORES命令获取service所有provider刷新缓存并返回结果。如果cache中找到了service，且可用provider list不为空则直接返回，如果为空调用接口向redis查询。

monitor： 
添加加定时任务，调用redis的SCAN来遍历key值，然后按照score的范围删除过期注册信息。每一次调用SCAN cursor都会返回两个element，第一个是一个string为下一次scan的cursor，第二个是本次搜索到的key的列表。最初cursor为0，要一直检索到cursor再次为0为止(这一次返回的key列表也要处理)，然后调用zremrangebyscore来处理其中超时的provider。 PUBLISH发布“refresh”信息，给所有subscribe了service的客户端，让它们刷新本地cache。

zremrangebyscore 调用ZREMRANGEBYSCORE去掉Service的hostlist中注册时间在range中的记录。这个range为 [-inf,Timestamp::now-validate_period) validate_period是注册的有效时长。redis返回去掉的member个数，如果不为0，PUBLISH 发布"refresh"信息，让所有subscribe了Service的客户端刷新cache。 


inbuffer和outbuffer作为输入输出缓冲区，因为Tcp连接的缓冲区是有限的，防止信息较长导致读写任务阻塞用缓冲区来缓存。要一直读写到EAGIN错误才可返回。read返回0，说明对端主动close，要调用关闭连接的方法。如果输出缓冲区为空就尝试直接写fd，如果信息没有发送完就缓存到输出缓冲区的尾部，在fd的events中加入EPOLLOUT。

runinloop：将任务放到eventloop的任务队列中执行，也就是在eventloop的线程中执行，不会有线程间的race condition，任务是并发安全的。所有对epoll的操作都是放到eventloop的任务队列中执行。 

获得当前超时的Timer的列表，并将他们从定时器队列中删除。然后分别调用超时定时器对应的回调函数。最后调用reset函数检查超时列表中的定时器是否有repeat重复标志，如果有重新插入到定时器队列中，并用定时器队列中最早到时的定时器时间戳更新timerfd。 

压力测试。webbench是基于HTTP连接的，我用jmeter建立线程组，提供了TCP取样器测试过。(设置一些参数 ip/port/是否关闭连接/是否重用连接/SO_LINGER等参数)。
测试了TCP长连接模式下的吞吐量。客户端和服务端都建立echo协议。TCP连接建立时，客户端向服务器发送一些数据，服务器echo回这些数据，然后客户端再echo会服务器。直到有一方断开连接为止，用来测试吞吐量。数据是无格式的，接受多少数据就反射回多少数据，并不会拆包解析数据。
客户端和服务器线程数同时设置为2，(电脑是4核，使用epoll IO复用，每个线程一个eventloop，不会阻塞，线程多于核数再多没有意义了)并发连结数为1000时候的吞吐量。
在同一台机器上测试的，否则即使单连接cpu也很容易把网速跑满，千兆网也就是110MB/s。

测试客户端类：建立2个I/O线程，新建1000个TCPclient对象添加到I/O线程上，每个TCPclient对象和服务器建立连接，开始以echo形式通信，并统计收到message数，读取字节数和写字节数。测试客户端添加定时任务，在time时间之后，对所有的TCPclient对象主动断开连接。当1000个连接断开后，统计1000个连接上总共写入和接受到的字节数，结合time时间，可以算出吞吐量。(原子变量统计连结数numconnected，TCPclient对象建立连接后(fd可写)，在回调函数中将numconnected原子的加一。中断连接后，将numconnected原子的减一)。大致在200MB/s。

AP而不是CP
注册中心不满足一致性，会导致服务提供者的各个节点流量会不均衡，但只要注册中心满足最终一致，流量将很快趋于统计学意义上的一致。
出现网络分区时，注册中心为了数据一致性而放弃了可用性，导致了同一分区的服务之间出现了无法调用。注册中心不能因为自身的任何原因破坏服务之间本身的可连通性。

是否通过事务日志持久化
通过事务日志，持久化连续记录地址列表变化过程其实意义不大，因为在服务发现中，服务调用发起方更关注的是其要调用的服务的实时的地址列表和实时健康状态，每次发起调用时，并不关心要调用的服务的历史服务地址列表、过去的健康状态。

存储一些服务的元数据信息，例如服务的版本，分组，所在的数据中心，权重，鉴权策略信息，service label等元信息，这些数据需要持久化存储，并且注册中心应该提供对这些元信息的检索的能力。

CAS操作：
常见的方法如Hibernate对缓存的持久对象（PO）加入字段 version 值，当每次操作一次该PO，则 version=version+1，这样采用 CAS 原理探测version字段，就能在多线程的环境中，排除ABA问题，从而保证数据的一致性。

file_operations中：
一般文件的poll方法：
poll_table_struct:向等待队列中添加等待项的函数_qproc/关注的事件掩码_key
_qproc:select或poll 则是 __pollwait, 如果是 epoll 则是 ep_ptable_queue_proc

Poll：
根据事件掩码和文件实现，取得事件掩码对应的一个或多个等待队列头节点。
调用poll_wait,向获得的等待队列中添加等待队列项(当前select/epoll进程的task，设置对应回调函数)
取得文件当前就绪事件掩码保存到mask，返回mask。 

 wait_queue 头节点：
 spin_lock;
 task_list;

 wait_queue 节点：
 struct list_head task_list;
 wait_queue_func_t func;//回调函数


当文件的状态发生改变时, 文件调用 _wake_up:加锁，遍历等待队列中的节点，并调用wait_queue节点中的func函数唤醒调用poll的线程。



屏蔽SIGPIPE：信号默认结束进程
SIGPIPE：write to pipe with no readers
对一个对端已经close的socket调用两次write,会产生SIGPIPE信号。客户端发送FIN，服务端ACK确认。服务端write这个socket时, 如果发送缓冲没问题, 会返回正确写入(发送). 但发送的报文会导致对端发送RST报文, 因为客户端端的socket已经调用了close, 完全关闭, 既不发送, 也不接收数据。所以, 第二次调用write方法(假设在收到RST之后), 会生成SIGPIPE信号, 导致进程退出。

reactor 和 proactor的区别：
1、reactor模式，是同步IO模式。proactor模式，是异步IO模式。
最主要的区别是真正的IO操作是由谁来完成。二者都是事件多路的分离器和事件处理器组成。
Reactor模式，事件分离器负责等待文件描述符准备就绪，然后将就绪事件传递给对应的处理器，最后由事件处理器负责完成实际的读写工作。
Proactor模式，处理器只负责发起异步读写操作。IO操作本身由操作系统来完成。事件分离器捕获IO操作完成事件，然后将事件传递给对应处理器。
proactor的缺点：
1、编程复杂性，由于异步操作流程的事件的初始化和事件完成在时间和空间上都是相互分离的，因此开发异步应用程序更加复杂。
2、内存使用，缓冲区在读或写操作的时间段内必须保持住，可能造成持续的不确定性，并且每个并发操作都要求有独立的缓存，相比Reactor模型，在Socket已经准备好读或写前，是不要求开辟缓存的；
3、操作系统支持，Windows下通过IOCP实现了真正的异步I/O，而在Linux系统下，Linux2.6才引入，并且异步I/O使用epoll实现的，所以还不完善。






 
