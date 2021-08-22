# 数据结构与算法(四)：优先级队列与静态查找表(Re:算法导论 & 数据结构：思想与实现)

## 优先级队列

**定义(优先级队列)：** 元素之间的关系是由元素的优先级决定的，而不是由入队的先后顺序决定的队列。

### 1.1 基于树的优先级队列

基于树的优先级队列就是将队列放入一个**二叉堆**中。二叉堆是一棵满足**结构性**和**有序性**的二叉树。二叉堆的结构性是指其是一棵**完全二叉树**，因此它可以按照**层次遍历**的次序存储在**数组**中。二叉堆有序性是指最大(或最小)的元素位于根的位置，而根的子树也应该为一个二叉堆，即任何节点都应该大于(或小于)它的子孙。当根节点为最小值时，称为**最小化堆**。当根节点为最大值时，称为**最大化堆**。

**优先级队列的基本运算:**
- 创建一个队列create()
- 入队enQueue(x)
- 出队deQueue()
- 读取队头getHead()
- 判断队列是否为空isEmpty()

**优先级队列类的定义:**

```cpp
template<class T>
class priorityQueue:public queue<T>{
    private:
        int currentSize;        //队列长度
        T *array;
        int maxSize;            //容量

    public:
        priorityQueue(int capacity=100);
        priorityQueue(const T data[], int size);
        ~priorityQueue();
        bool isEmpty() const;
        void enQueue(const T &x);
        T deQueue();
        T getHead() const;

    private:
        void doubleSpace();
        void buildHeap();
        void percolateDown(int hole);
};
```

**优先级队列方法的实现:**

```cpp
template<class T>
priorityQueue<T>::priorityQueue(int capacity=100){
    maxSize = capacity;
    currentSize = 0;
    array = new T[capacity + 1];
}

template<class T>
priorityQueue<T>::priorityQueue(const T data[], int size):maxSize(size + 10), currentSize(size){
    array = new T[maxSize];

    for(int i = 0; i < size; i++){
        array[i + 1] = data[i];
    }

    buildHeap();
}

template<class T>
priorityQueue<T>::~priorityQueue(){
    delete[] array;
}

template<class T>
bool priorityQueue<T>::isEmpty() const{
    return currentSize == 0;
}

template<class T>
void priorityQueue<T>::enQueue(const T &x){
    if(currentSize == maxSize){
        doubleSpace();
    }

    array[++currentSize] = x;
    int tmp = currentSize;

    while(tmp != 1){
        if(array[tmp] > array[tmp / 2]){
            T holder = array[tmp / 2];
            array[tmp / 2] = array[tmp];
            array[tmp] = holder;
            tmp = tmp / 2;
        }else{
            break;
        }
    }
}

template<class T>
T priorityQueue<T>::deQueue(){
    T result = array[1];
    array[1] = array[currentSize--];
    percolateDown(1);
    return result;
}

template<class T>
T priorityQueue<T>::getHead() const{
    return array[1];
}

template<class T>
void priorityQueue<T>::doubleSpace(){
    T *tmp = array;
    array = new T[2 * maxSize + 1];

    for(int i = 1; i <= currentSize; i++){
        array[i] = tmp[i];
    }

    maxSize *= 2;
}

template<class T>
void priorityQueue<T>::buildHeap(){
    for(int i = currentSize / 2; i > 0; i--){
        percolateDown(i);
    }
}

template<class T>
void priorityQueue<T>::percolateDown(int hole){
    int tmp = hole;

    while(2 * tmp <= currentSize){
        if(array[tmp] < array[2 * tmp]){
            int holder = array[2 * tmp];
            array[2 * tmp] = array[tmp];
            array[tmp] = holder;
            tmp *= 2;
            percolateDown(tmp);
        }else if(2 * tmp + 1 <= currentSize && array[tmp] < array[2 * tmp + 1]){
            int holder = array[2 * tmp + 1];
            array[2 * tmp + 1] = array[tmp];
            array[tmp] = holder;
            tmp = 2 * tmp + 1;
            percolateDown(tmp);
        }
    }
}
```

**Remark:**
- 用buildHeap()而非enQueue()构造优先级队列可以节省时间
- 注意插入时上滤，删除时下滤
- 数组从1号元素开始

**性能分析:**
- enQueue()和deQueue()操作的时间复杂度均为$+O(logn)$
- 建堆的时间复杂度为$O(nlogn)$

### 1.2 D堆

显然，如果能进一步降低堆的高度，就可以提高堆的性能。**D堆**就是一个选择。在D堆中，每个节点**有D个儿子**。当D大于2时，生成的堆就比二叉堆矮。它可以将入队的时间复杂度降为$O(log_DN)$。然而对于大的D值，**出队操作就费时得多**，因为在下滤时需要找出所有儿子中最小者。如果采用简答的选择算法，则需要进行D-1次比较，需要操作的时间复杂度提高到$O(Dlog_DN)$。因此D堆主要用于插入操作比删除操作多的场景中。当优先级队列太大无法完全装入内存，需要放入外存时，D堆也很有用，因为它可以**减少内外存的交换**。

### 1.3 归并优先级队列

#### 1.3.1 左堆

左堆是满足**有序性**，却在**平衡性上稍微弱一些**的堆。如果定义节点的空路径长度npl(x)为到不满两个孩子的节点的最短路径，具有0个或1个孩子的节点的npl为0，空节点的npl为-1，那么左堆就是对于堆中的每个节点x，左孩子的npl不小于右孩子的npl的堆。显然左堆是向左倾斜的堆。左堆不再是一棵完全二叉树，因此**不能用顺序存储**的方式来存储，而只能用二叉树的标准形式存储。左堆的主要操作是归并。由于左堆堆平衡性要求不高，而左堆是左高右低的堆，因此往根的右子树插入是一个很好的想法。左堆的插入采用如下递归的方法：
- 将根节点稍大的堆与另一个堆的右子树归并
- 如产生的堆违法了左堆的定义，则交换左右子树
- 递归终止条件：当某个堆为空时，另一个堆就是归并结果

#### 1.3.2 斜堆

斜堆是满足堆的有序性，但**没有任何平衡条件限制的二叉堆**。在斜堆中，不能保证树的深度是对数的。但**从平均的意义上**可以保证所有操作都有对数性能。在斜堆的归并中，最简单的策略也是将根节点的值比较大的堆与根节点比较小的堆的右子堆归并。实际的效果可能会使得**堆的右路径变得很长**。只需要在完成每一个归并前，对产生的临时树的**右路径上的每个节点交换它们的左右孩子**即可避免这一问题。

#### 1.3.3 二项堆(斐波那契堆)

二项堆不是用一棵满足堆的有序性的二叉树表示，而是用一个**森林**表示。森林中的每一棵树都有一定的约束，称为**二项树**。二项树是满足堆的**有序性**的树。高度为0的二项树是只有根节点的树，高度为k的二项树$B_k$是将一棵$B_{k-1}$加到另一棵$B_{k-1}$的根上形成的。在这片森林中，**每个高度的二项树最多只有一棵**。可知$B_k$有$2^k$个节点。由二项树的特性，对于给定的元素个数，我们可以挑选森林二项树构造**唯一的一个二项堆**。二项堆的归并类似于二进制加法，**由低到高依次归并两个优先级队列中高度相同的树**。在归并两棵相同的树时，只需要将根节点值大的树作为根节点值小的树的子树即可。

**斐波那契堆的基本运算：**
- 创建空堆create()
- 入队insert(x)
- 读取最小元素getHead()
- 出队()
- 合并堆union(H)

**斐波那契堆类的定义:**

```cpp
template<class T>
class FibHeap:public queue<T>{
    private:
        int size;
        
};
```
## 集合

### 2.1 集合的定义

在逻辑上，集合中的数据元素之间的关系非常松散，除了属于同一集合之外，它们之间没有任何逻辑联系。集合中每个元素有一个区别于其它元素的**唯一标识**，通常称为**键值**或**关键字**。所以每一个集合元素可以看成由两部分构成：关键字和其它信息。

**集合元素的类型：**
```cpp
template<class Key, class Other>
struct Set{
    Key key;
    Other other;
};
```

### 2.2 查找的基本概念

通常将用于查找的数据结构称为**查找表**。查找就是要确定指定关键字值得数据结构在查找表中是否存在。通常，每个数据元素得关键字值时不同的，但在某些场合下也可能有少量的数据元素有相同的关键字。有重复关键字的集合称为**多重集**。

如果查找表中的数据元素的个数和每个数据元素的值是不变的，这样的查找表称为**静态查找表**。如果对查找表不但要进行查找操作，还要经行插入、删除操作，这样的查找表称为**动态查找表**。

若被查找的所有数据全部存放在内存中，则称为**内部查找**。若有部分数据需要放到外存储器中，则查找它们的操作称为**外部查找**。在外存储器中，每个数据元素通常称为**记录**。在内部查找中，以**比较次数**作为衡量时间性能的标准。在外部查找中，以**外存储器的访问次数**为衡量标准。

### 2.3 静态有序表的查找

在无序静态查找表中，我们只能采用顺序查找的方法来查找。而在有序表中，我们可以进一步提高查找效率。

#### 2.3.1 顺序查找

在有序表中，我们可以提前结束顺序查找。当一旦发现数组元素小于(大于)被查找的元素，我们可以立即确定需要查找的元素在集合中不存在，查找过程就结束了。不过顺序查找的时间复杂度仍然为$O(n)$。

**有序表的顺序查找:**

```cpp
template<class Key, class Other>
int seqSearch(Set<Key, Other> data[], int size, const Key &x){
    data[0].key = x;

    for(int i = size; x < data[i].key; --i){
        if(x == data){
            return i;
        }
    }

    return 0;
}
```

#### 2.3.2 二分查找

若待查数据是已排序的，我们可以利用**排序的信息来加速查找**，最常用的查找方法就是二分查找。我们每次都从待查数据的最中间元素开始检查，若中间元素是待查元素，则查找完成。否则，确定待查数据是在前半部分还是后半部分，然后缩小范围在前一半或者后一半中继续查找。

**二分查找的实现：**
```cpp
template<class Key, class Other>
int binarySearch(Set<Key, Other> data[], int size, const Key &x){
    int low = 1, high = size, mid;

    while(low <= high){
        mid = (low + high) / 2;

        if(data[mid].key == x){
            return mid;
        }

        if(data[mid].key < x){
            high = mid - 1;
        }

        if(data[mid].key > x){
            low = mid + 1;
        }
    }

    return 0;
}
```