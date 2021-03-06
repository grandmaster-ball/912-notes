Conclusions on HashTable
========================

## 散列的基本概念

> 什么是散列？为什么需要散列？

据邓公所说，散列是一种思想。与已经学过的其他数据结构相比较，向量是采用循秩访问(call by rank)的访问方式，列表是采用循位置访问(call by position)的访问方式，二叉搜索树是采用循关键码访问(call by key)的访问方式，散列与他们都不一样，是采用循值访问(call by value)的访问方式。

举个例子，你现在身处同济大学嘉定校区，四周是一片荒芒，这个时候你想回家了。沿世界上所有的街道一间一间房找过去，这是循秩访问；你记得你家是住在四川省成都市某街道多少号，然后你可以依次先到四川省，再到成都市，再到某条街道，然后找到你家，这是循关键码访问；而循值访问，则是你通常会采用的方法——你根本不用去回想我家的地址是多少，你知道它就在那里，就在家这个词刚刚出现在你的脑海中的时候。想到家乡，你想到的不是地址或者一串数字，而是一个生动的影像，包含它的环境，四周的风物，以及曾经的朋友。这就是循值访问。

可以看到，相对于其他的访问方式，循值访问是将被访问对象的数值，与它在容器中的位置之间，直接建立了一个映射关系，从而对于任何对象的基本操作（访问，插入，删除）都只需要常数$O(1)$的时间，达到了最理想的境地。这就是人类需要散列的原因，你无法不被如此的诱惑所吸引。

> 完美散列

在时间与空间性能上均达到完美的散列，称为完美散列。

也就是说，对于完美散列，其中的每一个值，都可以唯一地映射到散列表中的一个位置，既无空余，亦无重复。从映射角度来看，完美散列是一个单射，同时也是一个满射。Bitmap就是完美散列的一个例子。

可以看出，完美散列实际中并不常见，在大多数的情形下，关键码的取值是远远大于词条的个数的，设关键码的取值为$[0, R)$, 词条的个数为$N$，则$R >> N$。设散列表的大小为$M$，此时，从定义域$[0, R)$到值域$[0, M)$的映射不可能是单射，即不可避免地会出现不同的关键码映射到散列表中的同一个位置，即所谓冲突。因此就需要合理地选择这一个映射关系，即散列函数，使冲突出现的可能性最小；同时还应该事先约定好一旦出现这种冲突，应该采取的解决方案。这两个问题将在下面重点讨论，即散列函数的设计与冲突解决方案。

## 散列函数的设计

> 散列函数的设计方案？什么是好的散列函数？

前面提到，从词条空间到地址空间的映射，即散列函数，绝对不可能是单射，冲突是一定不可能避免的，但是好的散列函数应该保证尽可能地少出现冲突。由此，可以提炼出散列函数的几个设计指标。

+ 确定性。散列函数确定的条件下，同一个关键码应该总是映射到同一个地址，这样才满足一个函数的定义。
+ 快速性。是指散列地址的计算过程要尽可能快，要能在常数时间内完成。
+ 满射。好的散列函数最好是一个满射，这样可以充分利用散列空间，尽可能地减少冲突的发生。
+ 均匀性。也是为了减少冲突的发生，关键码被映射到各个散列地址的概率应该接近于$1/M$，这样可以防止汇聚(clustering)现象的发生，即关键码只被映射到少数的几个散列地址，在局部加剧散列冲突，全局的散列空间也没有得到充分地利用。

总之，为了保证冲突尽可能地少，散列函数越是随机，越是没有规律越好。

### 几个散列函数的实例

> 除余法(division method)

除余法的整体思路非常简单，即用关键码的值对散列表的长度$M$取余，即$hash(key) = key\ mod\ M$，这样可以将关键码映射到整个散列空间上。

这里问题的关键在于散列表长度$M$的选择。考虑有一组数据，其中的关键码以固定步长$S$变化（实际中的数据往往就是这种形式的，而不是随机的，例如for循环一般就是固定步长的数据）。设第$i$个数据第一次与第$j$个数据的关键码发生冲突，即
$$
S\times i \equiv S\times j\ (mod M)
$$
即$Si$与$Sj$是同余类，所以
$$
S(j - i) \equiv 0 \ (mod M)
$$
由此可解得
$$
j - i = \frac{M}{gcd(M, S)}\ gcd\ for\ Greatest\ Common\ Divisor
$$

根据上面对散列函数设计要求的分析，我们是希望散列函数可以尽可能地减少冲突，即这里的$j - i$尽可能地大，就需要保证$M$和$S$的最大公因数要尽可能小，因此$M$要和$S$互质。但是由于散列表存储的不同数据具有不同的步长$S$值，要使$M$与所有可能的步长$S$互质，只有当$M$本身就是一个素数才可能实现。

> MAD法(Multiply-add-divide method)

上面的除余法还存在着一些问题。

首先，除余法得到的散列地址，依然存在一定程度的连续性，即原来相邻的关键码对应的散列地址也仍然是相邻的；其次，在除余法中关键码较小的那些词条，始终被映射到散列表的起始区段，其中关键码为零的元素，其散列地址总是零，即是一个不动点，这显然违背了散列函数应该越随机，越没有规律越好的原则。MAD法正是对除余法上述问题的一个改进。

MAD法，顾名思义，就是对于关键码，依次执行乘法、加法和模余运算，即
$$
hash(key) = (a \times key + b)\ mod\ M
$$

这样，只要适当选择$a, b$，MAD法可以很好的克服除余法原有的连续性缺陷，其中参数$a$的作用是使相邻关键码的散列地址更加分散，$b$的作用是作为一个偏移量，去掉不动点。

> 数字分析法

遵循散列函数越是随机没有规律，就越好的原则，引入了数字分析法，即对于关键码key的特定进制展开，抽取其中的几位，映射到一个散列地址。

比较简单的情形就是取十进制展开中的奇数位，作为散列地址，例如
$$
hash(123456789) = 13579
$$

除此以外，还有平方取中法，即对于关键码$key$取平方，然后截取中间的几位来作为散列地址。之所以选择中间的几位，是因为中间的几位是受到了原来的关键码更多数位的影响；相对于取高位数字（只受到原关键码高位数字影响）或者低位数字（只受到原关键码低位数字影响），取中间位数综合了更多位数的影响，因此随机性、均匀性更强，更不易出现冲突。

此外，还有折叠法，往复折叠法。

为了保证经过这些方法得到的值仍然落在散列空间以内，通常还都需要对散列表长度$M$再取余。

> 随机数法

既然散列函数是随机性越强越好，那一个简明的思想是直接利用生成的伪随机数来构造散列地址。这样的话，任意一个伪随机数发生器本身就是一个好的散列函数了。即

$$
hash(key) = rand(key)\ mod\ M
$$
其中，$rand(key)$是系统定义的第$key$个伪随机数。

## 冲突解决方案

无论如何精心设计的散列函数，都不能完全地避免冲突的发生，随着数据量的增大，冲突的发生几乎是必然的。因此，就需要事先规定好冲突发生时的解决方案，从而保证散列表的正常工作。

### 封闭定址法(closed addressing)

> 多槽位法(multiple slots)

所谓冲突发生不过是不同的关键码被散列函数映射到同一个散列地址，既然如此，那我们事先为可能到来的、冲突的关键码预留一个位置不就可以了吗？这种简明的思想就是多槽位法。

多槽位法就类似于一山二虎，将原来对应一个关键码的桶单元，划分为若干更小的槽位，从而可以容纳后续到来的冲突的关键码。

显而易见，这种方法具有致命的缺陷，即你永远也不知道槽位应该细分到何种程度，才能保证在任何情况下都够用。槽位划分太多的话，空间利用率会非常低；槽位划分不够，又不足以应对可能出现的冲突。此外，在极端条件下，当数据量非常大的时候，无论再多的槽位，也仍然有可能会产生溢出。

> 独立链法(separate chaining)

多槽位法所面临的问题，其实就是类似于数组这种静态数据结构所面临的问题，即在实际应用之前，你不会清楚数组的大小应该划分到多大。采用链表可以有效的解决数组空间不足的问题，而将链表应用到散列表的冲突解决方案，就成为了独立链法。

独立链法与多槽位法的核心思想是完全相同的，即预备空间来应对可能出现的冲突情况。不过与多槽位法不同，独立链法是将所有冲突的关键码组织成一个列表，利用列表的动态增长特性，来规避预备的冲突空间不足的问题。

> 公共溢出区法(overflow area)

基本思想与上面两个也是相同的，即在事先预备公共的溢出区，来存储关键码冲突的词条。

上面几种方法都具有相同的思想，即在原有的散列表外还预备额外的空间来存储词条，此时散列地址不仅仅局限于散列表所覆盖的范围内，还包括这个额外的存储冲突词条的空间，故也称作开散列(open hashing)，或者封闭定址法(closed addressing)，因为任一给定的词条只可能存储在某一确定的桶单元，其他的桶单元对该词条是不开放的。

### 开放定址法(open addressing)

> 线性试探法(linear probing)

线性试探法是指，在插入关键码$key$时，若发现第$hash(key)$个桶空间已被占用，则继而试探它的下一个桶空间，如此不断，直到发现一个可用的空桶。

线性试探法的问题在于，随着散列表装填因子的增大，散列表中的查找链也会随之增长，从而降低了散列表的查找性能。另一方面，采用线性试探法时，一旦在某一局部发生冲突，极有可能后续的插入会在这里引发更多的冲突，并且多组各自冲突的查找链有可能相互重叠。

但如果散列表的装填因子不大$\lambda < 0.5$，采用线性试探法的散列表的平均效率，通常都可保持在较为理想的水平。并且线性试探法中的词条，具有良好的局部性，可以极大地降低I/O操作的开销。

> 单向平方试探法(quadratic probing)

平方试探法可以在一定程度上缓解上述的冲突聚集现象。在试探过程中发生第$j$次发生冲突时，转而试探
$$
(hash(key) + j^2) \ mod \ M, j = 0, 1, 2, \cdots
$$
由于各次试探的位置到起始位置的距离，以平方速率增长，故称为平方试探法。

与线性试探法比较，平方试探法可以理解为迅速离开发生冲突的“是非之地”，以平方的速率迅速跳离关键码聚集的区段，因此可以在一定程度上缓解汇聚的现象。

可是，关于平方试探法，我们不难提出一些问题，比如平方试探法果真可以覆盖整个散列表吗？是否存在散列表本来有空桶，却无法被探测到的现象？

这种情况是存在的，可以自己举一些例子要验证一下。不过，只要散列表长度$M$为素数，并且装填因子$\lambda \le 0.5$，则平方试探法迟早必然会终止于某个空桶，即$n^2 \ mod\ M$的取值恰好有$ceil(\frac{M}{2})$种，并且恰好由查找链的前$ceil(\frac{M}{2})$取遍。以下给出证明：

设$a < b < ceil(\frac{M}{2})$，并且第$a$次试探与第$b$次试探冲突，即$a^2, b^2$是$M$的同余类

$$
a^2 \equiv b^2\ (mod\ M)
$$
所以
$$
b^2 - a^2 \equiv 0\ (mod\ M)
$$
即$(b - a)(b + a)$可以整除$M$。但是由于$b - a < M, b + a < M$，如果$(b-a)(b+a)$可以整除$M$的话，则$M$必有不为一的因子，与$M$为素数矛盾，所以前$ceil(\frac{M}{2})$次试探必不冲突，故得证。

> 双向平方试探法

根据上面的分析，在$M$为素数并且装填因子$\lambda < 0.5$的时候，单向平方试探法可以保证试探必然终止。一个自然的想法是，另一半的桶，是否也可以利用起来呢？这就是双向平方探测法。

双向平方探测法，就是在发生冲突时，依次向前向后进行平方探测，即
$$
(hash(key) \pm j^2)\ mod \ M, j = 0, 1, 2, \cdots
$$

这种方法看似是把两侧的桶都利用起来了，但是我们也不禁产生一个问题，即这双向的探测是否是相互独立呢？它们之间除了零，是否还有其他公共的桶？

答案是，是存在不独立的情况的，并且这种情况还相当的多，也可以自己举几个例子来看一下。但是，如果散列表的长度取做素数，并且$M = 4k + 3$，则必然可以保证查找链的前$M$项都是互异的，以下来证明这个结论。

这里，我们首先需要提到费马的双平方定理，即任意素数$p$可以表示为两个正整数的平方和，当且仅当$p = 4k + 1$。

在此基础上，如果我们可以注意到
$$
(u^2 + v^2)(s^2 + t^2) = (us + vt)^2 + (ut - vs)^2
$$
即任意两个可以表示为两个正整数的平方和的正整数的乘积，也可以表示为两个正整数的平方和。
就可以推知，任意自然数$n$可以表示为一对整数的平方和，当且仅当在其素分解中，形如$M = 4k + 3$形式的每一个素因子均为偶数次方。这个推导的过程要知道啊，我这里就不写了。

从而我们假设对于$a < ceil(\frac{M}{2}), b < ceil(\frac{M}{2})$，并且$-a^2$与$b^2$是同余类，即
$$
-a^2 \equiv b^2 \ (mod \ M)
$$
所以
$$
a^2 + b^2 \equiv 0 \ (mod \ M)
$$
即$a^2 + b^2$可以整除$M$。由于$M = 4k + 3$且为素数，所以$a^2 + b^2$可以整除$M^2$，但是$a^2 + b^2 < M^2$，所以假设不成立，故原命题得证。

> 随机试探法(pseudo-random probing)

仿照散列函数中的随机数法，在发生冲突时也可以采用随机数发生器来确定试探的位置，就是随机试探法。

> 再散列法(double hashing)

顾名思义，设置一个二级散列函数来确定试探位置，即
$$
[hash(key) + j \times hash_2(key)] \ mod\ M
$$
其他的方法其实都可以视作再散列法的一个实例。
