# hashMap实现方式及相关内容

## 实现方式
HashMap在jdk1.8中是通过**链地址法**实现的：

1. 先用一个数组作为桶存放数据结构

2. 然后桶中存放的要么是**链表**（**链表的节点个数小于等于8**），要么是**红黑树**（链表节点数大了之后变成红黑树），要么为**空**。

3. 如果计算的索引位置冲突了，就加入到链表中一直延伸，超过链表大小阈值就转换为红黑树。
 
## 插入时，计算插入到哪里(先异或后与)
实际上是通过hashCode得到哈希码，然后计算得到哈希值，再计算在桶中的位置，冲突则解决冲突，用链地址法。
```
// 给一个key，求HashCode，对Code操作得到hash，再计算存放位置，这里是哈希值hash的计算方法
return (key == null)? 0 : h = key.hashCode() ^ h >>>16;
position = (n - 1) & hash; //n是桶的数组长度
// 为什么n-1？因为n是2的幂次方，所以一个位置是1，其他全0，n-1后，所有低位都变成1，这样就可以按位与了
```
哈希表存的是key和对应的value值，需要建立key的索引位置，key的hashCode一共32位，右移16位异或可以将高低位信息整合在一起，这样就计算出哈希值了。
求哈希值（各种哈希值hash求法，称为哈希函数:模N取余（hash=K mod C），数字分析法(取key中取值较均匀的数字作为地址)，aK+C直接寻址法，平法取中法(key平方，取中间分布较均匀的几位)等等。
解决哈希冲突的比如线性探测法(一个个向后探测)，二次探测法(往左右跳着探测)，伪随机探测法(伪随机数，往边上定位置)），链地址法（后续加入链表）。
hash求出来后，实际上是做到了key->hash的映射，但是存储位置还没有定下来，所以需要继续计算存储位置position。
特别的是，**key==null的内容放在第0个桶**。

### 如果键值对多了，怎么定扩容？
在不断放入元素的时候，怎么把Map扩容？然后把数字放进去？

扩容操作一般出现在**容量\*负载因子=该容量时需要进行扩容**，HashMap每次扩容到**2的幂次方**，涉及到rehash和数据复制。

为什么容量必须为2的整数次方，并且扩容是2倍2倍地扩：使扩容前的单个节点无需与其他节点交互，只控制自己的节点和跳过去的新节点。

例如hash表有8个，要扩容为16个，那么之前放在7表的数据，可能对16取模会余15，也可能还是7；其他7种元素只能分配到7和15以外的0-15余数；这样的话，只需把部分数据往15表上移，保留一部分到原表中。

下面的方法是获得大于cap(可以理解成输入键值的大小)且最近的2的整数次幂的数。如输入10，则返回16。

其中**cap-1**的目的是另找到的目标值大于或等于原值。

```
int n = cap - 1;
n |= n >>> 1;
n |= n >>> 2;
n |= n >>> 4;
n |= n >>> 8;
n |= n >>> 16;
return (n < 0) ? 1 : (n > MAXIMUM_CAPACITY)? MAXIMUM_CAPACITY: n+1;
```

这里做了什么？

1. 把n-1，这是为了找到≥原值的扩容目标值

2. 做1 2 4 8 16 的右移位，是为了把高位1的影响依次传递到后面，找到大于cap且最近的2的整数次幂（右移1，则绑定了每2位，右移2，则在此基础上绑定了每4位，类推），最后肯定是全1，然后+1就能出2的幂次方了，对应n位。

## get方法

- 首先对key求hash后取得所定位的桶。

- 如果桶为空则直接返回 null 。

- 否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。

- 如果第一个不匹配，则判断它的下一个是红黑树还是链表。

- 红黑树就按照树的查找方式返回值。

- 不然就按照链表的方式遍历匹配返回值。
## put方法

主要通过putVal()函数实现，putVal过程中：

- 判断当前桶是否为空，空的就需要初始化（resize 中会判断是否进行初始化）。

- 根据当前 key.hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突，在当前位置创建一个新桶即可。

- 3. 如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e,在第 8 步的时候会统一进行赋值及返回。

- 3 中不等，则如果当前桶为红黑树，那就要按照红黑树的方式写入数据。

- 3.中不等，如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。

- 接着判断当前链表的大小是否大于预设的阈值（默认为8），大于时就要转换为红黑树。

- 如果在遍历过程中找到 key 相同时直接退出遍历。

- 8. 如果 e != null 就相当于存在相同的 key, 那就需要将值覆盖。

- 最后判断是否需要进行扩容。

## 扩容过程

通过resize函数将数组扩容为原来容量的2倍后，需要**重新将原有的元素映射到数组中**。

1. 如果原来的位置只有一个节点，直接通过pos = (n - 1) & hash计算其在数组中的位置。

2. 如果是链表，进行链表的rehash时，根据hash & oldCap的结果是0还是1，将链表拆分成两部分。

举个栗子：如果原数组的容量为16，那n-1=15，然后有两个Entry通过pos = (n - 1) & hash这个公式计算得到，他们在桶中的位置索引都是5。

扩容后，数组长度变成原来的两倍即32（n的大小变化了），n-1=31，用二进制表示就是1 1111，然后通过pos = (n - 1) & hash这个公式，就能把上面两个节点分成两个部分。

3. 如果是红黑树，就调用红黑树的split()方法，此处，会对红黑树也和链表一样分为高位和低位两个部分，如果树中元素的个数小于6个了，就将红黑树转换成链表。

## 移除过程

移除元素时，先通过key找到对应的结点。

然后再分为链表、红黑树两种情况去处理。

红黑树转换成链表的操作。

在resize中和remove中是不同的，resize中是当红黑树中的节点少于6个时就将红黑树转为链表，

而remove函数中，是当红黑树的根节点的左孩子或右孩子为空时，才将红黑树转换成链表的。

## 为什么HashMap线程不安全？在并发场景下使用时容易出现死循环？

put导致数据覆盖：put时如果桶中的数据为空，则直接new一个元素，放入桶中。

在并发条件下，第一个线程判断桶中无元素，则new一个新节点，如果此时该线程被挂起。

然后另一个线程直接在此处放了一个新节点进去，然后线程1被唤醒后，不知道此位置已经有新元素了，会直接放数据进去，此时会将原来线程放的数据覆盖。

此外，在put的时候涉及到size++的操作，可能会导致HashMap的size出错。

说白了就是线程1put的时候，线程1被挂起，线程2不知道new过了，重复new覆盖线程1的数据。

**1. 扩容时(不断put需要扩容时)死循环或数据丢失(jdk1.7中)**

- 假设有两个线程A和B都在进行put操作(假设当前需要扩容从2->4)。线程A在执行到transfer函数中被挂起，此时线程A还没完成扩容操作，链表还没有重连接完毕。

假设当时容量2，内容为7->5，插入一个3变成7->5->3，需要扩容，执行到下面的第二步。

```
e.next  = newTable[i];
newTable[i] = e;	//在这里被挂起
e = next.
```

此时已经变成7->5和3->null，e=3，还没完成链表调整。

- 线程A挂起后，此时线程B正常执行，并完成resize操作，把链表的重连接做好了，弄成了5 和 7->3, 有7.next = 3;

- 此时切换到线程A上，在线程A挂起时内存中存放的值和2的结构不一样。

- A继续执行，可能出现链表a.next = b, b.next = a的环形链表出现，然后进行get操作时，就可能引起死循环。

- 也有可能A继续执行，因为链表结构不同，直接跳过了某个值的连接关系，导致数据丢失。

**2. put时扩容带来数据覆盖，发生线程不安全(jdk1.8中)**

jdk1.8中，在发生hash碰撞，不再采用头插法方式，而是直接插入链表尾部，因此不会出现环形链表的情况。

put时，如果没有hash碰撞则会直接插入元素。如果**线程A和线程B同时进行put操作**，刚好**这两条不同的数据hash值一样，并且该位置数据为null**，所以这线程A、B都会进入代码put内容。

假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据。

然后线程A获取时间片，此时线程A不进行hash判断了，问题出现：线程A会把线程B插入的数据给覆盖，发生线程不安全。

### HashTable怎么就线程安全了？

HashTable通过加synchronized关键字修饰方法来做线程安全。synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性，实际上就是加了个全局锁，所以其实效率也低下。

## LinkedHashMap继承HashMap

其在HashMap的基础上，**增加了一条双向链表**，使得可以保持键值对的插入顺序，继承自原来HashMap的Node，增加了befor和after两个字段，用于维护双向链表。

除此之外，为了记录插入顺序，在LinkedHashMap中，增加了**头结点和尾节点**两个变量，每当有新键值对节点插入，新节点最终会接在 tail 引用指向的节点后面。而 tail 引用则会移动到新的节点上，这样一个双向链表就建立起来了。

相应的下面几个过程，重要的都是，在操作完之后，如果更新双向链表关系。

### 建立过程

重写了newNode方法，在这里建立链表的关系。（即生成新的节点，然后更新链表，新节点添加到链表尾部）

### 删除过程

使用的是HashMap的remove方法，但是重写了afterNodeRemoval方法，在这里更新链表。

### 访问元素

在访问元素的时候，有一个**accessOrder**属性，为**true**时，每访问一个元素，都会将这个访问的元素放到尾部，保证链表尾部是最常访问的数据，可用在缓存等场景中。

LinkedHashMap使用父类的HashMap来实现元素的访问，重写了afterNodeAccess方法，这样如果accessOrder为true时，就会调用afterNodeAccess方法，来按照访问来维护链表的顺序。

### 基于LinkedHashMap的缓存实现

基于LinkedHashMap，可以实现LRU缓存。在HashMap的putVal函数最后，会调用afterInsertion函数。而LinkedHashMap重写了这个函数：

```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 根据条件判断是否移除最近最少被访问的节点
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

	// 移除最近最少被访问条件之一，通过覆盖此方法可实现不同策略的缓存
	// 这里了也是修改缓存机制时需要重写的地方
	protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
		return false;
}
```

如果想用LinkedHashMap来实现LRU缓存，就直接继承LinkedHashMap，然后重写removeEldestEntry函数，设置在某些条件下让其返回true（比如当前集合的元素数量大于某一阈值，就返回true，否则返回false）。这样，当条件满足时，就会删除头节点。

# volatile关键字

# ConcurrentHashMap——线程安全的HashMap(jdk 1.7)

相比之下，ConcurrentHashMap 使用了分段锁技术来提高了并发度，ConcurrentHashMap引入Segment 的概念，目的是将map拆分成多个Segment(默认16个)。

操作ConcurrentHashMap细化到操作某一个Segment(槽)。在多线程环境下，不同线程操作不同的Segment，他们互不影响，这便可实现并发操作。

当我们对ConcurrentHashMap并发操作时，只要锁住一个 segment，其他剩余的Segment依然可以操作。这样只要保证每个 Segment 是线程安全的，我们就实现了全局的线程安全。而每个segment后面则是和hashmap一样，由数组+链表的结构组成。

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/807144-4db95a9fa5fedc1c?imageMogr2/auto-orient/strip|imageView2/2/w/820/format/webp" width=""> </div><br>


其主要采用**CAS操作+synchronized锁**的方式实现线程安全，其中synchronize锁的粒度为桶中头结点（包括链表Node结点，包装红黑树的TreeBin结点），底层依然由 “数组”桶+链表+红黑树 的方式实现。

## put方法

- 通过key找到承载的Segment对象位置，然后调用tryLock()竞争Segment的锁，以确保能操作线程；如果获取锁失败，执行scanAndLockForPut方法等到调用成功。

- 循环次数retries 次数大于事先设置定好的MAX_SCAN_RETRIES，就执行lock() 方法，此方法会阻塞等待，一直到成功拿到Segment锁为止。

- 调用成功后走put流程

- 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。

- 此前没有改key，即为空，则需要新建一个 HashEntry 并加入到 Segment 中，会先判断是否需要扩容。（这里就涉及到hashmap中的扩容导致线程不安全的问题了）

- 最后会解除获取当前 Segment 的锁。

### 那么在这样的设计下，是否还会有hashmap的put线程不安全问题呢？

ConcurrentHashMap的Segment槽是固定的16个，不变的

而ConcurrentHashMap的扩容实际上是Segment中的HashEntry数组扩容。当HashEntry达到某个临界点后，会扩容2为之前的2倍， 原理跟HashMap扩容类似。

进行扩容时，调用rehash方法，其实就是HashEntry扩展逻辑。

当线程执行到rehash方法时，表示当前线程已经获取到到当前Segment的锁对象，这就表示rehash方法的执行是线程安全，不会存在并发问题。

## remove

ConcurrentHashMap的remove方法跟put方法操作一样，先获取segement对象后再操作，这里就不重复了。也是找到位置，然后删除。

## get

会发现**get方法并没有获取锁的操作**，这时就得讨论下执行get操作线程安全问题啦。

只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。

get方法之所以不需要加锁，原因比较简单，get为只读操作，不会改动map数据结构，所以在操作过程中，只需要保证涉及读取数据的属性为线程可见即可，也即使用volatile修饰。

所以ConcurrentHashMap 的 get 方法是非常高效的，因为整个过程都不需要加锁。

## ConcurrentHashMap(jdk1.8版本)

jdk8版本的ConcurrentHashMap直接抛弃了Segment的设计，采用了较为轻捷的Node + CAS + Synchronized设计，保证线程安全。

具体的数据结构是这样的：

<div align="center"> <img src="https://upload-images.jianshu.io/upload_images/807144-6264960638978dff?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp" width=""> </div><br>

构建一个默认大小为16的Node数组，类似hashmap的数组桶，数组上是链表node(1.7中是HashEntry,node中把val和next都用volatile修饰，保证了可见性)或者红黑树TreeNode。

桶中元素有自己的hash值来指定他是什么内容：

	hash== -1 (MOVED) ： 当前节点是ForwardingNode节点，处于扩展中。
	
	hash== -2 (TREEBIN) ： 当前节点已经树化，为TreeBin对象，TreeBin节点代理红黑树操作
	
	hash== -3 (RESERVED) ： 临时保留的哈希
	

### 重要的内部类

1. Node节点类：**这个类是后面三个类的基类。**和HashMap中的节点类似，只是其**val变量和next指针都用volatile来修饰**。且不允许调用setValue方法修改Node的value值。

2. TreeNode：**树节点**类，指的是整个TreeBin中的节点的数据结构。

链表过长时会转换成红黑树，但是与HashMap不相同的是，**它不直接转换为红黑树**，而是把Node包装成树节点**TreeNode放在TreeBin对象**中，然后由TreeBin这个数据结构完成对红黑树的包装。

而且TreeNode在ConcurrentHashMap继承自Node类，也就是说**TreeNode带有next指针**，这样做的目的是方便基于TreeBin的访问。

3. TreeBin： 

这个类放了一个名为root的TreeNode节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别，另外这个类还带有了读写锁。

而HashMap的不足在于他的桶中存储的是TreeNode结点，这里的根本原因是并发过程中，有可能因为树的调整，形状发生错误变化，这样的话，桶中的第一个元素就变了，而使用TreeBin包装的话，就不会出现这种情况。 

这种类型的节点，我们通过hash值是否等于-2就可以判断桶中的节点是否是红黑树。

4. ForwardingNode：用于连接两个table的节点类。

它包含一个nextTable指针，用于指向下一张表。而且这个节点的key value next指针全部为null。

在扩容操作中，我们需要对每个桶中的结点进行分离和转移。如果某个桶结点中所有节点都已经迁移完成了（已经被转移到新表 nextTable 中了），那么会在原 table 表的该位置挂上一个 ForwardingNode 结点，说明此桶**已经完成迁移**。这种类型的节点的hash值是-1。

## 数组初始化

数组的初始化是在ConcurrentHashMap插入元素的时候发生的，如调用put时当前表为空时发生的。确定没有在扩容后，new一个新的表，更新sc。

初始化操作在initTable()中，没加锁，因为采取的策略是通过sizeCtl来判定是否已经有线程在扩容。

**sizeCtl(sc)**

一个控制标志符，取值不同有不同的含义：

- 负数 代表正在进行初始化或扩容操作
	
- -1 代表正在初始化
	
- -N 表示有N-1个线程正在扩容
	
- 正数或0 代表hash表还没有被初始化，这个数值表示初始化(0)或下一次进行扩容的大小(正数)，这一点类似于扩容阈值的概念。它始终是容量的0.75倍。

当sizeCtl<0时，说明已经有线程在扩容，这个线程就会调用Thread.yield()让出一次CPU执行时间。

```
if ((sc = sizeCtl) < 0)
	Thread.yield(); // lost initialization race; just spin
else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {//当前没有线程扩容，那就利用CAS方法把sizectl的值置为-1 表示本线程正在进行初始化
		try {
			if ((tab = table) == null || tab.length == 0) {
				// sc>0表明还没初始化，通过sc的值来初始化容量大小
				int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
				@SuppressWarnings("unchecked")
				// 根据sc开辟数组作为桶
				Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
				table = tab = nt;
				// 相当于0.75*n 用sizeCtl来表示阈值
				sc = n - (n >>> 2);
			}
		} finally {
			sizeCtl = sc;
		}
		break;
	}
```

## 并发扩容(记录个数+判断是否扩容+扩容步骤)

可以看到，优先级上，先判断是否初始化(初始化的时候其实也先判断了是否在扩容)，再判断是否在扩容(在扩容的话就需要去协助扩容)，最后才是put等操作。

使用addCount()记录元素个数，并检查是否需要进行扩容。

需要扩容时调用transfer()调整节点位置。

### addCount() 计数 + 扩容调用

ConcurrentHashMap使用一个long型名为baseCount变量和一个CounterCell\[]的名为counterCells的变量一起记录size。（hashmap中是单独的size变量）

- 首先通过CAS操作去直接更新baseCount。

- 如果失败，那说明有多个线程在竞争更新baseCount。则使用fullAddCount()，把x的值插入到counterCell类中，更新counterCells，等待后续更新baseCount。(为了避免更新冲突，生成随机数来生成hash，唯一标识)

- 然后检查是否需要扩容是否正在扩容。如果需要扩容，就调用扩容方法，如果正在扩容，就帮助其扩容，最后调用的都是transfer()。

fullAddCount()：

- 第一次执行的时候，counterCells这个数组应该是空的，所以先生成了一个大小为2的数组，赋值给了counterCells，然后创建一个初始值为要更新的量的CounterCell放在counterCells对应位置上。

- 如果通过(n - 1) & h计算得到的线程对应的桶为空，那直接new一个初始值为x的CounterCell，放到桶中。

- 上一步失败的话，wasUncontended=false，那就通过代码，将wasUncontended更新为true，说明已经冲突了,开启下一次循环。

### transfer进行位置调整

首先搞清楚，扩容是由线程承担的，每个线程承担不小于 16 个桶中的元素的扩容。

每当开始迁移一个桶中的元素的时候，线程会锁住当前槽中列表的头元素，扩容完成后会将这个桶中的节点设置为ForwardingNode。

假设这时候正好有 get请求过来会仍旧在旧的列表中访问，如果是插入、修改、删除、合并、compute 等操作时遇到 ForwardingNode，就表示正在扩容，那当前线程会加入扩容大军帮忙一起扩容，扩容结束后再做元素的更新操作。

- 计算线程能承担多少个桶的扩容

- 对要扩容的桶，新建2倍大小的数组(因为每次扩容都是\*2)，nextTable指向这个新数组。

- 新建一个ForwardingNode占位，这样别的线程查到这里的时候就知道正在扩容了。

- 循环处理每个槽(即槽)中的链表元素，并初始化i和bound值，i指当前槽位序号，bound指需要处理的当前槽位边界

- 当前处理槽结束了或者本来就空不用处理，那就有个ForwardingNode标明。

- 比如当前槽是需要扩容的，先定义两个变量节点ln和hn，按我的理解应该是lowNode和highNode，分别保存hash值的第X位为0和1的节点。

(这个**第X位**实际上就是**数组大小2^n的那个第n位**，因为容量肯定对应二进制某位为1，其他全0，这样也更方便了他的扩容其实)

- 使用fn&n可以快速把链表中的元素区分成两类，A类和B类节点可以分散到新数组的原槽位和后边新加的槽位中。

- 根据第X位的hash不同把原槽位切成两条(hash第X位为0和为1)，然后复制到新数组中就好了，完成扩容。

## put

- 根据 key 计算出 hashcode 。

- 判断是否需要进行初始化，初始化完毕。

- 利用 CAS 得到当前 key 定位出的 Node 记为 f ，如果为空表示是首节点插入，**可以直接写入**数据，**利用 CAS 尝试写入**，失败则一直自旋保证成功。

- 如果不为空，且当前位置的 hashcode == MOVED == -1，表示正在扩容状态，则需要进行协助扩容。

- 如果上面都走过了，则利用 synchronized 锁住 f ， 插入数据。插入的时候，要么插入到链表中，要么插入到红黑树中。我们发现，这里的锁粒度是很小的，就锁住一个桶所以并发效率更高。

- 如果是插入到链表中，那去看看链表的长度有没有超过长度阈值8，如果超过了，就要将链表转换成红黑树。

## cas写入(重点内容，实现了线程安全)

CAS, 比较交换，一种无锁原子算法。CAS是一条原子指令，因为他是一种系统原语，所以在OS中他是被开中断和关中断包起来的。

看CAS之前，先看看synchronized这个锁操作。

在并发编程中，锁是消耗性能的操作，同一时间只能有一个线程进入同步块修改变量的值，比如下面的代码:
```
synchronized void function(int b){
  a = a + b；
}
```

如果不加 synchronized 的话，这不是一个原子的操作，多线程上面时候会出问题。

多线程修改 a 的值就会导致结果不正确，出现线程安全问题。但锁又是要给耗费性能的操作。不论是拿锁，解锁，还是等待锁，阻塞，都是非常耗费性能的。那么能不能不加锁呢？

CAS过程是这样：它包含 3 个参数 CAS（V，E，N），V表示**要更新的变量的值**，E表示**预期值**，N表示**新值**，即在赋新值前判断是否变量是预期值。

仅当V==E时，才会将V的值设为N；如果V!=E，则说明已经有其他线程做两个更新，则当前线程则什么都不做。

最后，CAS 返回当前V的真实值。当多个线程同时使用CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。

失败的线程不会挂起，仅是被告知失败，并且允许再次尝试，当然也允许实现的线程放弃操作。基于这样的原理，CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰。

### JAVA如何实现原子操作

java.util.concurrent.atomic包中，所有类都是原子操作。例如：

```
public static void main(String[] args) throws InterruptedException {
    AtomicInteger integer = new AtomicInteger();
    System.out.println(integer.get());

    Thread[] threads = new Thread[1000];
	// 启动1000个线程对AtomicInteger变量做自增。
    for (int j = 0; j < 1000; j++) {
		//启动线程.start()，同时就运行了run方法了
      threads[j] = new Thread(() ->
          integer.incrementAndGet()
      );
      threads[j].start();
    }

    for (int j = 0; j < 1000; j++) {
		// join()方法会让main方法等待thread执行完毕后才继续。
      threads[j].join();
    }
	// 得到1000
    System.out.println(integer.get());
  }
}
```

### CAS 缺点

著名的ABA问题，就是A->B->A, 已经修改完了，但是CAS无法察觉到。基本类型的话没关系，但如果是引用类型，这个对象里边有多个变量，就不好办了。

这时候往往会加入版本号来标注。

### 使用CAS操作的三个核心方法

三个静态的用于CAS操作的方法：

```
//获得在i位置上的Node节点
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
	return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
	//利用CAS算法设置i位置上的Node节点。之所以能实现并发是因为他指定了原来这个节点的值是多少
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
									Node<K,V> c, Node<K,V> v) {
	return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
	//利用volatile方法设置节点位置的值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
	U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
```

## get

同1.7，因为是只读操作，所以没上锁。寻址，返回值。