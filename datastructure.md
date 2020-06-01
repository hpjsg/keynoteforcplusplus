算法数据结构：
判断n是不是2的整数次幂。
if(n>0 && (n&(n-1)==0)) 是整数次幂。如果0也算，那就是n>=0
10进制转2进制。
    for(int i=0;i<31;i++)
    {
        str = to_string(n&1)+str; 
        n >>= 1;
}
snprintf(c,sizeof(c),"%08x",n); 倒是可以直接转16进制。

1000个苹果分成10框，这十框可以表示1-1000的任意数
类似2进制的表示
1，2，4，8，16，32，64，128，256，489
N个数选出最小的k个
1-100的和，有多少种方法实现，分别说一下
堆排和快排复杂度。若果用堆排实现前k个最大数,复杂度
大数据去重，10亿用户对应手机号，有几个用户ID对应同一手机号。如何找到这些用户。
B+树作用，和B树区别，数据存储在哪些节点上
平衡树与B+树
map的实现原理，红黑树高读差最大是多少
排序算法/查找算法
稳定排序与不稳定排序，时间复杂度/稳定的排序算法
快排的实现，分而治之
反转链表和两个字符串最大公共子串
打印二叉树每层最右边的节点 层序遍历
查找的常用数据结构：二叉树、跳表、hash
代码题c，实现字符串比较函数 int strcmp(const char* s1, const char* s2);
string的实现 
编程题：memcpy的实现
实现单项链表的有序插入（假设value类型为unsigned int）。如何在有序链表的基础上提高查找效率？
算法题，给定uint32_t A[n](1<n)，求max(a[i] & a[j]),其中i != j。 
5 算法题，超过100亿的无序可重复的unsigned int，给定4G的内存，如果找到中位数？ 
6 给定n个样本文件（每个文件从书籍、报刊等提取），平均长度为k个字节。根据给定的样本，给出判断一个输入的数据串是否为随机串的算法。 
先简单处理，假设给定的样本都是英文的（比如泰晤士报所发行的所有内容）（马尔科夫链）
堆删除元素和插入元素/图的遍历
mmu虚拟内存映射，红黑树实现

算法题：公共结点
给你一个2G的电脑 10G的文本有1k行的字符串，要求输出所有互为逆序的字符串的组合
给个数x，一个数字n，x中删n位使得剩下的数最大
小顶堆
一亿个数字找中位数，内存比较小的情况。
讲一讲缓存算法。 
4.讲讲怎么实现一个hashmap 
5.讲讲怎么实现一个LRU
Strcpy
红黑树的时间复杂度，插入一个节点后需要几次旋转，旋转的时间复杂度。
类似抖音，腾讯视频，每个用户点击一个视频产生一条日志，求一分钟内top100点击量，假如一个点击一条日志，一分钟之类2亿条日志，每个人可以在一分钟之内点击多个视频，考虑内存放的下和放不下的情况，求top100。内存放得下采用位图，但是计数这一方面我不会。
数据结构：每次月考会更新成绩，我们需要实现三个功能:查看某位学生成绩，获取成绩前100的学生，用什么数据去存储数据会比较好

## 排序
稳定性：相同的元素在排序前和排序后的前后位置是否发生改变，没有改变则排序是稳定的，改变则排序是不稳定的
内排序：
插入排序(稳定)
冒泡排序(稳定)
选择排序(不稳定)
快速排序(不稳定)
归并排序(稳定)
shell排序(不稳定)
基数排序(稳定)
堆排序(不稳定)
外排序：
多路归并

* 插入排序：逐个处理待排序记录，每个记录与前面已排序的子序列比较，插入子序列正确位置。如果待排序数据已经“基本有序”，使用插入排序可以获得接近O(n)的性能。平均性能接近n^2;
* 冒泡排序：从数组底部到顶部，比较相邻元素。如果下面元素小则交换。上面的元素继续向上比较，每次使最小的元素被顶到数组顶部。(增加一个flag,记录一次循环是否发生交换，没有发生交换则已经有序)时间复杂度为O(n^2)。
* 选择排序
第i次选择数组中第i小的记录，并将该纪录放到数组第i个位置。时间复杂度为O(n^2)
* 快速排序：
小于轴值的元素放在数组中轴值的左侧，大于轴值的元素放在数组中轴值元素右侧，称为数组的一个partition。对轴值左右子数组分别进行类似操作。平均情况：O(nlogn)。最差情况：每次处理将所有元素划分到轴值一侧，O(n^2)
三者取中法选择轴值。首中尾三个元素中间值。
当n很小时，快排会很慢，改用插入排序。 快排恶化时，使用堆排序
* shell 排序。利用了插入排序的最佳时间代价特性，将待排序序列变成基本有序的然后在插入排序。
在执行每一次循环时，将序列分成互不相连的子序列，使每个子序列元素在数组中间距相同，每个子序列插入排序进行排序。每一次循环间距是前一次循环的1/2，子序列元素是前一次的2倍。
* 归并排序
将一个序列分成两个长度相等的子序列，每一个子序列排序，然后将它们合并成一个序列。合并两个有序子序列的过程称为归并。
时间复杂度稳定的为O(nlogn)，使用辅助数组，空间代价为O(n)
* 堆排序
根据数组构建最大堆，然后每次弹出堆顶元素。最后得到有序的序列。平均时间复杂度为O(nlogn)。注意根据已有元素建堆是很快的，如果希望找到数组中第k大的元素，可以用O(n+klogn)的时间，如果k很小，时间开销接近O(n)
多路归并：将源文件分解成多个能一次性装入内存的部分，分别把每一部分调入内存完成排序。然后对已经排序的子文件进行归并排序。
k路归并： m个子文件，每次进行k路归并，一共归并logkm次(扫描磁盘)，k个元素选取最小值需要比较k-1次。对于k值的选择，主要涉及这两个问题：
1、每一轮归并会将结果写回磁盘，那么k越小，磁盘与内存之间的数据传输就会越多，增大k可以减少扫描次数。
2、k个元素中选取最小元素，k越大，比较的次数会越多。
减少比较次数：
1、败者树
2、建堆：使用一个k个元素的数组，第一次将k个文件中最小的元素读入数组(记录每个元素来自哪个文件)，建立最小堆，弹出堆顶元素，从堆顶元素的源文件中取出下一个数，压入堆中，调整堆重复上述操作。

堆排序/快排
1、访问数据来说快排更加友好。对于快速排序来说，数据是顺序访问的，而堆排是跳着访问数据。
2、堆排的时间复杂度稳定性更好，快排如果轴值取得不好可能会恶化为n^2(STL的introsort在递归次数过多时从快排变成堆排)。堆排可以很快获得最大/最小值(如果数据都有，只是建堆是线性复杂度)。

堆是一棵完全二叉树，使用数组实现堆，堆分为两种：
* 最大堆：父节点大于任意子节点（因此堆顶为最大值）
* 最小堆：父节点小于任意子节点（因此堆顶为最小值）
## 哈希表
* 开链法：每个槽位是一个链表的表头。
链表组织方式：
1、插入次序排序
2、按关键码值次序排序：达到比要检索的关键码大的节点，说明不存在，可以停止检索。
3、按访问频率次序排序：访问较高的记录快速找到
在磁盘中用一种很有效的方式存储一个开哈希表是很困难的，因为一个链表中的多个元素能存储在不同的磁盘块中。这就会导致检索一个关键码值需要多次磁盘访问，从而抵消了哈希方法的好处。
* 闭哈希法：
闭哈希法(开地址法)把冲突记录存储在表中另一个槽内。每条记录有一个基位置，即哈希函数算出来的位置。如果要插入一条记录，而基位置已经被占据，就要把记录存储在表中的其他位置，由冲突解决策略决定是哪个位置。检索时和插入时遵循相同策略。
线性探查：基位置被占用，就继续向下一位置探查。不同基位置的关键码产生的探查序列会重合。
二次探查：探查函数为平方函数，依次探查i+1,i+4,i+9...

## 红黑树：
1.	每个节点不是红色就是黑色。
2.	根节点是黑色，外部节点(NULL)是黑色
3.	如果节点为红，子节点一定是黑色
4.	任意节点至NULL的任何路径，所含黑节点数必须相同
5.	根据规则4，插入节点必须为红，根据规则3，插入新节点的父节点必须为黑节点。
6． 实操时规定从root到新增节点A的路径上只要看到有某节点X的两个子节点皆为红色，就把X改为红色，两个子节点改为黑色。

从根到叶子的最长可能路径不多于最短可能路径的两倍长。(从根到叶每条路径上有n个黑节点，最短可能路径是连续的n-1个黑节点，最长可能路径是红-黑-红-黑….n-1个黑节点，n-1个红节点，高度差最多为n-1，也是黑高度)

1、AVL树更加严格平衡，最差情况高度1.44log(n+2)，红黑树2log(n+1)。AVL树可以提供更快的查找。
2、插入节点时两者均最多2次旋转来恢复平衡
3、删除节点时，红黑树需要常数次操作(3次o(1)),AVL树需要维护从被删除节点到root这条路径上所有节点的平衡，最差情况下需要o(logN)的操作。
(https://blog.csdn.net/yuhk231/article/details/51218244)

## 堆：
完全二叉树，vi的左子节点时v2i+1，右子节点v2i+2 父节点v[(i-1)/2],第一个非叶子节点的序号位((size-2)/2)。
底层用vector实现。插入元素，先将新元素放入vector尾部，然后push_heap(iterator,iterator,campare comp) 不断上溯与父节点比较。Comp为less是大顶堆，默认为大顶堆，为greater是小顶堆。
使用make_heap(iterator,iterator,compare comp) 将vector变成一个堆，弹出节点使用pop_heap(iterator,iterator,compare comp)将根节点放到vector尾部，取出最后一个节点的数据adjust到合适位置（根部的空洞先下溯，与较大子交换位置（大顶堆）直至叶节点，然后填入最后一个节点的数据执行一次上溯）。取出vector.back()数据然后执行vector.pop()。
* 插入节点

插入节点时，进行下列操作：
```c++
/********************************************
 * 向堆中插入元素
 *  hole：新元素所在的位置
 ********************************************/
template<class value>
void push_heap(vector<valut>&arr,int hole){
    value v = arr[hole];
    int parent = (hole-1)/2;
    while(hole>0){              //hole为当前考察位置
        if(arr[parent]>=v) break;
        arr[hole] = arr[parent];
        hole = parent;
        parent = (hole-1)/2;
    }
    arr[hole] = v;
}
```

* 删除堆顶

```c++
/********************************************
 * 删除堆顶元素
 ********************************************/
template<class value>
void _pop_heap(vector<value>&arr,int size){
    swap(arr[0],arr[size-1]); // 堆顶元素放到最后
    size--;
    pop_heap(arr,0,size);
}
template <class value>
void pop_heap(vector<value> &arr,int hole,int size)
{
    value v = arr[hole];
    int child = 2*hole+1;
    while(child<=size){
        if(child<size && arr[child]<arr[child+1]) child ++;// 选择较大子
        if(arr[child]<=v) break;
        arr[hole] = arr[child];
        hold = child;
        child = 2*hole + 1;
    }
    arr[hole] = v;
}
```

* 建堆

* 堆的大小固定(且所有元素已知)：按“序号从大到小”的顺序遍历所有非叶子节点，将这些节点与左右子节点较大者(以最大堆为例)交换，执行siftdown一直到叶子节点，因此，每遍历到一个节点时，其左子树和右子树都已经是最大堆，只需对当前节点执行siftdown操作。
* 堆的大小未知(如数据流)：可以通过插入操作来构建堆

```c++
/********************************************
 * 建堆
 *  sz：删除堆顶元素后的大小
 *  v： 被堆顶元素占据的位置原来的元素的值
 ********************************************/
template <class value>
void _make_heap(vector<value> &arr)
{
    int size = arr.size();
    int parent = (size - 2) / 2;
    while(parent >= 0){
        pop_heap(arr,parent,size);
        --parent;
    }
}
```

*复杂度

* **插入节点**：时间复杂度为O(logn)
* **删除堆顶**：时间复杂度为O(logn)
* **建堆**：
    - **堆的大小固定(且所有元素已知)**：每个siftdown操作的最大代价是节点被向下移动到树底的层数。在任意一棵完全二叉树中，大约有一半的节点是叶节点，因此不需要向下移动。四分之一的节点在叶节点的上一层，这样的节点最多只需要移动一层。每向上一层，节点的数目就为前一层的一般，而子树高度加1，因此移动层数加一。**时间复杂度为O(n)**
    - **堆的大小未知(如数据流)**：由于插入节点的时间代价为O(logn)，对于n个元素，每个执行一次插入操作，所以**时间复杂度为O(nlogn)**


## Ringbuffer

## 手写实现
实现strlen
迭代
```c++
# include<cassert>
int strlen(const char*str){
    assert(str);
    int len = 0;
    while(*str++) len++;
    return len;
}
```
递归
```c++
#include<cassert>
int strlen(const char*str)
{
    assert(str);
    return *str==0?0:1+strlen(str+1);
}
```
实现strcmp
```c++
#include<cassert>
int strcmp(const char*str1,const char*str2){
    assert(str1&&str2);
    while((*str1 == *str2) && *str1){
        str1 ++;
        str2 ++;
    }
    if(*str1 == *str2)
    return 0;
    //128种扩展ascii码使用最高位标识（8位全用了），要转为uchar。
    return *(unsigned char*)str1>*(unsigned char*)str2?1:-1; 
}
```
实现strcat
```c++
include<cassert>
char* strcat(char* strdest,char*strsrc){
    assert(strdest&&strsrc);
    char* cur = strdest;
    while(*cur) cur++;
    while(*cur++ = *strsrc++);//复制运算符返回左值引用
    return strdest;
}
```
实现strcpy
```c++
#include<cassert>
char* strcpy(char* strdest,char* strsrc){
    assert(strdest && strsrc);
    char *cur = strdest;
    while(*cur++ = *strsrc++);    
    return strdest;
}
```
实现memmove(考虑重叠)/memcpy不考虑重叠
```c++
void* memcpy(void *dst,const void*src,size_t size){
    if(dst == null || src == null) return null;
    const *pdst = (char*) dst;
    const *psrc = (char*) src;
    if(pdst>psrc && pdst<psrc+size){// 从高位开始
        pdst = pdst + size - 1;
        psrc = psrc + size - 1;
        while(size--){
            *pdst-- = *psrc--; 
        }
        return dst;
    }
    while(size--){
        *pdst++ = *src++;
    }
    return dst;
}
```
计算结构体偏移的宏
```c++
#define offset(type,mem) unsigned long(&( ((type*)0)->mem))
```
手写智能指针
```c++
template<typename T>
class shared_ptr{
    public:
        shared_ptr():ptr(nullptr),count(nullptr){}
        explicit shared_ptr(T* src):ptr(src),count(new int(1)){}
        explicit shared_ptr(const shared_ptr &rhs):ptr(rhs.ptr),count(rhs.count)
        {
            ++(*count);
        }
        shared_ptr& operator=(const shared_ptr& rhs){
            if(ptr != nullptr){
                if(--(*count) == 0){
                    delete ptr;
                    delete count;
                }
            }          // 如果已经管理对象
            ptr = rhs.ptr;
            count = rhs.count;
            ++(*count);
            return *this;
        }
        ~shared_ptr(){
            if(--(*count) == 0){
                delete ptr;
                delete count;
            }
        }
        T* operator*() const noexcept{return ptr;}
        T& operator->() const noexcept{return *ptr;}
        T *get()const{return ptr;}
    private:
        T* ptr;//原始指针
        int* count;
};
```
使用辅助类
```c++
template<typename> class shared_ptr;  
// 辅助类
template<typename T>
class RefPtr
{
    friend class shared_ptr<T>;

    RefPtr():ptr(nullptr),count(0){}
    RefPtr(T* src):ptr(src),count(1){}
    ~RefPtr(){delete ptr;}
    T* ptr;
    int count;
};
template<typename T>
class shared_ptr{
    shared_ptr()：ref(new RefPtr<T>()){}
    explictit shared_ptr(T* src):ref(new RefPtr<T>(src)) {}
    shared_ptr(const shared_ptr& rhs):ref(rhs.ref){
        ref->count ++;
    }
    shared_ptr& operator=(const shared_ptr&rhs){
        if(ref->ptr !=nullptr){
            if(--(ref->count) == 0)
                delete ref;
        }
        ref = rhs.ref;
        ++(ref->count);
        return *this;
    }
    ~shared_ptr(){
        if(ref->ptr !=nullptr){
            if(--(ref->count) == 0)
                delete ref;
        }
    }
    T* get(){return ref->ptr;}
    private:
        RefPtr<T> *ref   //辅助类指针
};
```
unique_ptr
```c++
template<typename T>
class unique_ptr
{
    public:
        explicit unique_ptr(T* p = nullptr):ptr(p){}
        ~unique_ptr(){
            if(ptr)
                delete ptr;
        }
        unique_ptr& operator=(const unique_ptr &p) = delete;
        unique_ptr(const unique_ptr&p) = delete;
        
        unique_ptr(unique_ptr&& p) noexcept:ptr(p.ptr){
            p.ptr = nullptr;
        }

        unique_ptr& operator=(unique_ptr &&p)noexcept{
            if(ptr)
                delete ptr;
            ptr = p.ptr;
            p.ptr = nullptr;
            return *this;
        }

        T* operator*() const noexcept{return ptr;}
        T& operator->() const noexcept{return *ptr;}

        explicit operator bool() const noexcept{return ptr;} // 用于bool 判断 if(unique_ptr)

        void reset(T* q = nullptr) noexcept{
            if(q != ptr){
                if(ptr)
                    delete ptr;
                ptr = q;
            }
        }
        T* release() noexcept
        {
            T* res = ptr;
            ptr = nullptr;
            return res;
        }
        T* get() const noexcept{return ptr;}
    private:
        T* ptr;
}
```

new 的异常处理

## 海量数据
1GB = 2^10/10^3 MB = 10^6KB = 10^9B;
1亿 = 10^8 
位图/hash分治/trie树hash_map统计

* 给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4GB。找出a、b文件共同的url
5*10^9*64B = 5G*64B = 320GB;
分治：
1、遍历文件a，对每个url求hash(url)%1024，根据所得值将url分别存到1024个小文件中，每个约为300MB。
2、遍历文件b，采取相同措施，将url存到1024个小文件中。a,b文件中相同的url都在对应的ai,bi文件中。
3、求出1024对小文件中相同url：把其中一个文件的url存在hash_set中。然后遍历另一个小文件的每个url，看其是否在hash_set中。

* 有10个文件，每个文件1GB，每个文件的每一行存放的都是用户的query，每个文件中的query都有可能重复。请按照query的频度排序
方案一：顺序读取10个文件，hash(query)%10写入到另外10个文件中，每个新生成文件约为1GB。
找一台内存2GB的电脑，对每个文件，使用hash_map统计query的次数。使用快速/堆/归并排序进行排序。将排序好的query和query_count输出到文件。得到10个已经排好序的文件。对这10个文件进行归并排序。
方案二：若所有query可以一次性的加入到内存中，可以使用trie树/hash_map等直接统计次数，然后快速/堆/归并排序。

* 有一个1GB大小的一个文件，里面每一行是一个词，词的大小不超过16字节，内存限制大小是1MB。返回频数最高的100个词
分治。顺序读文件，对于每个词，取hash(x)%5000,然后按照该值存到5000个小文件中，每个文件约为200KB左右。如果其中有文件超过了1M，就继续分解，直到所有文件都不超过1MB。
每个小文件采用trie树/hash_map统计词频，选出频率最高的100个词，将这100个词及相应频率写入文件。这样得到了5000个有序文件，然后将这5000个归并排序出100个。

* 海量日志数据，提取出某日访问百度次数最多的那个IP
使用hash(ip)%n将日志记录分到n个小文件中，在每个小文件中使用hash_map的方法统计每个ip的频度，再利用堆排序按频度对ip进行排序：

位图：
已知某文件内包含一些电话号码，每个号码为8位数字，统计不同号码的个数
将8为电话号码理解成为[0,99999999]的数字，用位图解决，约为12MB。

给40亿个不重复的unsigned int的整数，没排过序，然后再给一个数，如何快速判断这个数是否在40亿个数当中？
位图，uint最多有2^32个数，约为512MB内存。读入40亿个数，设置相应bit位，读入要查询的数，查看相应bit位是否为1，为1表示存在，0不存在。

40亿个不重复整数，内存限制10MB，找到一个没出现的数即可。
1、10MB内存，使用bitmap大约可以表示2^26个数，uint空间有2^32个数，先申请长度为64的数组count[64]，将uint划分为64个等长区间，每个区间2^26个数。遍历一遍40亿个数据，根据当前数在对应区间的计数上加一。
2、遍历完后必有第k个区间的count值小于2^26。建立8MB的bitmap，再遍历一次40亿个数，只关注落在第k个区间内的数，使用bitmap，将bit[num-k*2^26]置1，也就是只在这个区间上做bit映射。
3、bitmap上存在为0的一位i，返回i+k*2^26

统计40亿个数的中位数，10MB内存。
1、分区间，8MB内存存放2M个计数值，需要2^11个区间，2048个区间。申请长度为2048的无符号整数数组，遍历40亿个数据统计落入每个区间内的数据的个数。
2、找到第20亿和第20亿+1个数据落入区间k，及在区间k内的排位n，n+1。
申请长度为2M的无符号整型数组，再次遍历40亿个数，只关心落入区间k的数据，增加对应计数值。返回区间内第n,n+1个数。

在2.5亿个整数中找出只出现一次的整数，内存不足以容纳这2.5亿个整数
方案一：采用2bit位图。00不存在，01存在，10多次。2^32个数字空间，需要约1GB内存。
方案二：也可以采用Hash映射的方法，划分成多个小文件。然后在小文件中利用hash_map找出不重复的整数



## 设计数据结构
学生成绩系统 姓名查询/分值排序（前100名）/成绩更新
使用redis zset value为学生实例，score为学生成绩。可实现常规时间根据姓名查询成绩和nlogn的对成绩排名。
Zet底层使用hashtable 实现通过value来查询对应score的功能。同时使用了ziplist或skiplist用来排序（或multimap）按照score排序。Hashtable和跳表通过指针共享相同成员的元素和分值。 学生对象序列化作为value，反序列化可以得到相应的实例。更新成绩由于skiplist和muiltimap不能改key值，需要删除节点，再重新插入。这里使用跳表的好处是：红黑树如果想获得前100名需要进行中序遍历，而跳表直接用最底层的链表遍历就可以。
