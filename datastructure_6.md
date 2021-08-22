# 数据结构与算法(六)：散列表与排序算法(Re:算法导论 & 数据结构：思想与实现)

## 散列表

在二叉查找树中，查找一个元素的效率取决于**比较的次数**。散列表提供了一种完全不同的存储和查找方法：通过**将关键字值映射到表中的某个位置**，将该关键字所对应的元素存储在该位置中。查找时可以直接根据关键值照到存储元素得位置，从而获得这个数据元素。这意味着查找时间可以从顺序查找的$O(N)$、二分查找和查找树的$O(logN)$下降到$O(1)$(经典空间换时间)。

### 1.1 散列表的定义

在计算机中任何信息都可以转换成为一个二进制的比特串，因此可以将其视为一个**整型数**，故而原则上可以用一个包含所有取值元素的数组保存查找表。但问题在于关键字的变化范围可能很大，以至于要使用这么大的数组并不现实。但尽管其变化范围可以很大，集合中并不一定有很多元素。散列表的思想是用一个**比集合规模略大**的数组来储存这个集合，将数据元素关键字映射到这个数组的下标。这个映射称为**散列函数**。

由于散列函数的定义域比值域大，故两个或更多不同的元素可能会被映射到同一个位置，这种情况称为**冲突**或**碰撞**。这种情况是不可避免的。因此，实现散列表的最基本的两个问题就是，**如何设计散列函数**，以及**如何解决碰撞**。

**散列函数：** 好的散列函数需要计算速度快，同时使得节点的散列地址尽量均匀，使得冲突的机会尽量减少。以下几种是常用的散列函数：
- 直接定址法：取关键字值或关键字值的某个线性函数的值作为散列地址
- 除留余数法：若$M$为散列表的大小，则关键字为$x$的数据元素的散列地址为$H(x) = x mod M$
- 数字分析法：在某些应用中，关键字之间的区别会集中在某几位上
- 平方取中法：如果关键字中各位分布比较均匀，但关键字取值范围较大，则可将关键字平方后，取其结果中间各位作为散列函数值。由于中间各位和每一位数字都有关系，因此均匀分布的可能性较大。
- 折叠法：如果关键字很长，则可以选取一个长度，将关键字按该长度分组相加。

**碰撞的解决：** 由于无法选择一个一一对应的散列函数，因此冲突是不可避免的。常用的冲突处理方法有以下两种：
- 闭散列表：将溢出的元素放到没有使用过的单元中去
- 开散列表：将同一地址的数据元素组织成为一个线性表

### 1.2 线性探测法

线性探测法就是**从数组散列到的位置开始顺序搜索**，直到发现一个空位置。如果需要，该搜索会从最后一个位置绕到第一个位置。在这种方法中，**标准的删除是不能实现的**，因为散列表中的每一个单元不但代表自己，而且还作为**解决碰撞时联系其它元素的占位符**。因此，闭散列表都采用**迟删除**的方法，即将这些元素标记为删除而不是物理地从表中删去。

**基于线性探测法的闭散列表的实现：**

```cpp
template<class Key, class Other>
class closeHashTable:public dynamicSearchTable<Key, Other>{
    private:
        struct node{
            Set<Key, Other> data;
            int state;      // 0--empty, 1--active, 2--delete

            node():state(0){}
        };

        node *array;
        int size;
        int (*key)(const Key &x);
        static int defaultKey(const int &x){
            return x;
        }

    public:
        closeHashTable(int length = 101, int (*f)(const Key &x) = defaultKey);
        ~closeHashTable(){
            delete[] array;
        }
        Set<Key, Other> *find(const Key &x) const;
        void insert(const Set<Key, Other> &x);
        void remove(const Key &x);
}

template<Key, Other>
closeHashTable<Key, Other>::closeHashTable(int length, int (*f) (const Key &x)):size(length), key(f){   
    array = new node[length];
}

template<class Key, class Other>
Set<Key, Other> *closeHashTable<Key, Other>::find(const Key &x) const{
    int hashcode = key(x);

    if(array[hashcode].data.Key != x || array[hashcode].data.state != 1){
        int tmp = (hashcode + 1) % size;

        while((array[hashcode].data.Key != x || array[hashcode].data.key != 1) && tmp != hashcode){
            tmp = (tmp + 1) % size;
        }

        if(tmp == hashcode){
            return NULL;
        }

        hashcode = tmp;
    }

    return static_cast<Set<Key, Other>*>(&array[hashcode].data);
}

template<class Key, class Other>
void closeHashTable<Key, Other>::insert(const Set<Key, Other> &x){
    int hashcode = key(x);

    if(array[hashcode].state == 1){
        int tmp = (hashcode + 1) % size;

        while(array[tmp].state == 1 && tmp != hashcode){
            tmp = (tmp + 1) % size;
        }

        if(tmp == hashcode){
            return;
        }

        hashcode = tmp;
    }

    array[hashcode].data = x;
    array[hashcode].state = 1;
}

template<class Key, class Other>
void closeHashTable<Key, Other>::remove(const Key &x){
    int hashcode = key(x);

    if(array[hashcode].data.Key != x || array[hashcode].data.state != 1){
        int tmp = (hashcode + 1) % size;

        while((array[hashcode].data.Key != x || array[hashcode].data.key != 1) && tmp != hashcode){
            tmp = (tmp + 1) % size;
        }

        if(tmp == hashcode){
            return;
        }

        hashcode = tmp;
    }

    array[hashcode].state = 2;
} 
```

**Remark:**
- 注意搜索一个循环后要推出
- 注意搜索时要同时检擦key和state
- 注意迟删除的设计

### 1.3 开散列表

开散列表就是将具有同一散列地址的元素都存储在一个单链表中。散列表的k号单元保存的是指向散列地址为k的链表的第一个节点地址。在散列表中插入一个元素，就是在对应单链表中插入一个元素。

**开散列表的实现：**
```cpp
template<class Key, class Other>
class openHashTable:public dynamicSearchTable<Key, Other>{
    private:
        struct node{
            Set<Key, Other> data;
            node *next;

            node(const Set<Key, Other> &d, node *n = NULL):data(d), next(n){}
            node():next(NULL){}
        };

        node **array;
        int size;
        int (*key) (const Key &x);
        static int defaultKey(const int &x){
            return x;
        }

    public: 
        openHashTable(int length = 101, int (*f) (const Key &x) = defaultKey);
        ~openHashTable();
        Set<Key, Other> *find(const Key &x) const;
        void insert(const Set<Key, Other> &x);
        void remove(const Key &x);
};

template<class Key, class Other>
openHashTable<Key, Other>::openHashTable(int length, int (*f) (const Key &x)):size(length), key(f){
    array = new *node[size];
}

template<class Key, class Other>
openHashTable<key, Other>::~openHashTable(){
    for(int i = 0; i < size; i++){
        node *tmp = array[i];

        while(tmp != NULL){
            node *n = tmp;
            tmp = tmp->next;
            delete n;
        }
    }
}

template<class Key, class Other>
Set<Key, Other> *openHashTable<Key, Other>::find(const Key &x) const{
    node *tmp = array[key(x)];

    while(tmp != NULL){
        if(tmp->data.Key != x){
            tmp = tmp->next;
        }

        return static_cast<Set<Key, Other>*>(&tmp->data);
    }

    return NULL;
}

template<class Key, class Other>
void openHashTable<Key, Other>::insert(Set<Key, Other> &x){
    node *tmp = array[key(x)];

    if(tmp == NULL){
        array[key(x)] = new node(n);
    }else{
        while(tmp->next != NULL){
            if(tmp->data.Key == x.Key){
                return;
            }

            tmp = tmp->next;
        }

        if(tmp->data.Key != x.Key){
            tmp->next = new node(x);
        }
    }
}

template<class Key, class Other>
void openHashTable<Key, Other>::remove(const Key &x){
    node *tmp = array[key(x)];

    if(tmp != NULL){
        if(tmp->next == NULL){
            if(tmp->data.Key == x.Key){
                delete tmp;
                array[key(x)] = NULL;
            }
        }else{
            while(tmp->next != NULL){
                if(tmp->next->data.Key == x.Key){
                    node *n = tmp->next->next;
                    delete tmp->next;
                    tmp->next = n;
                }

                tmp = tmp->next;
            }
        }
    }
}
```

**Remark:**
- 要考虑单链表的头节点设计

**性能分析：**
- 所有操作原则上时间复杂度均为$O(1)$

## 排序

排序是计算机的基本应用。所谓排序，就是把集合中的数据元素**按照其关键字的非递减或非递增序列排成一个序列**。假定在待排序的集合中存在许多关键字值相同的元素，经过排序后，若这些数据元素的**相对次序保持不变**，则这种排序方法是**稳定的**，否则称为**不稳定的**。按照排序过程中数据存储设备的不同，排序可以分为**内排序**和**外排序**。内排序是指被排序的数据元素全部放在计算机内存中，它适合于数据元素较少的情况。外排序是指排序过程中数据元素主要放在外存储器中，借助内存储器逐步调整元素的相对位置。

### 2.1 插入排序

插入排序的基本方法是，每次将一个待排序的数据元素按照其关键字大小插入前面已经排好的一组数据元素中的适当位置上，直到所有数据元素全部插入为止。根据在已经排好序的有序数据元素序列中不同的插入方法，可以将插入排序进一步分为直接插入排序、二分插入排序和希尔排序。

#### 2.1.1 直接插入排序

**直接插入排序的实现：**
```cpp
template<class Key, class Other>
void simpleInsertSort(Set<Key, Other> a[], int size){
    for(int i = 0; i < size; i++){
        int j = i;

        while(j > 0){
            if(a[j] < a[j - 1]){
                int tmp = a[j - 1];
                a[j - 1] = a[j];
                a[j] = tmp;
                j--;
            }else{
                break;
            }
        }
    }
}
```

**性能分析：**
- 时间复杂度为$O(n^2)$

#### 2.1.2 二分插入排序

若想改善直接插入排序的性能，可以从减少**比较次数**和**减少移位次数**进行考虑。二分插入排序是减少比较次数的有效手段。注意到在插入a[j]之前，前j-1个元素已经有序，故可以利用二分查找法，快速找到a[j]的插入位置。但二分插入排序并没有减少元素移动的次数，所以其总的时间复杂度仍然为$O(n^2)$。

#### 2.1.3 希尔排序

在直接插入排序中可以发现数据每次都是移动到它的后一个单元，若一个元素的初始位置离它的正确位置较远，则该元素需要移动多次才能到达正确位置，这就造成了直接插入排序的高代价。希尔排序的想法是**先比较那些离得较远的元素**，如果稍远的两个元素违反了排序的次序，就交换这两个元素，这样一次交换就让两个元素都移动了很长的距离。然后再比较那些离得近一点的元素，以此类推，逐步逼近直接插入排序。希尔排序选择了一个递增序列，称为增量序列$h_1$,$h_2$,...,$h_t$。首先对数组进行$h_t-$排序，然后再进行$h_{t-1}-$排序……最后进行$h_1-$排序。$h_t-$排序是使用直接插入排序使那些**间隔为$h_t$的元素成为有序序列**。

**希尔排序的实现：**
```cpp
template<class Key, class Other>
void shellSort(Set<Key, Other> a[], int size){
    for(int h = size / 2; h > 0; h /= 2){
        for(int i = 1; i < h; i++){
            for(int j = i; j < size; j += h){
                int k = j;

                while(k > i){
                    if(a[k] < a[k - h]){
                        int tmp = a[k - h];
                        a[k - h] = a[k];
                        a[k] = tmp;
                    }else{
                        break;
                    }
                }
            }
        }
    }
}
```

**性能分析：**
- 希尔排序的时间复杂度分析很困难，一般与它所选择的增量序列有关。当n很大时，Knuth统计得到其元素移动的次数大约在$n^{1.25}$到$1.6n^{1.25}$的范围内。
- 希尔排序不是稳定排序
  
### 2.2 选择排序

假定共有n个待排序的元素。首先，从n个元素中选择关键字最小的元素。再从剩下n-1个元素中选择关键字最小的元素，以此类推直至序列中最后只剩下一个元素。把每次得到的元素排成一个序列，就得到了按非递减次序排成的序列。

#### 2.2.1 直接选择排序

**直接选择排序的实现：**

```cpp
template<class Key, class Other>
void simpleSeletSort(Set<Key, Other> a[], int size){
    for(int i = 0; i < size; i++){

        for(int j = i; j < size; j++){
            if(a[i] > a[j]){
                int tmp = a[j];
                a[j] = a[i]
                a[i] = tmp;
            }
        }
    }
}
```

**性能分析：**
- 时间复杂度为$O(n^2)$
- 非稳定排序

#### 2.2.2 堆排序

对于直接选择排序而言，从n个元素中找出最小元素需要n-1次比较，这是制约直接选择排序的主要因素。如果在**选择最小元素时采用优先级队列**，那么选择排序的时间复杂度就可以下降为$O(NlogN)$，这种基于优先级队列的排序算法称为堆排序。

```cpp
template<class Key, class Other>
void heapSort(Set<Key, Other> a[], int size){
    for(int i = size  2 - 1; i >= 0; i--){
        percolateDown(a, i, size);
    }

    for(i = size - 1; i > 0; i--){
        Set<Key, Other> tmp = a[0];
        a[0] = a[i];
        a[i] = tmp;
        percolateDown(a, 0, i);
    }
}

template<class Key, class Other>
void percolateDown(Set<Key, Other> a[], int hole, int size){
    int child;
    Set<Key, Other> tmp = a[hole];

    for(;hole * 2 + 1 < size; hole = child){
        child = hole * 2 + 1;

        if(child != size - 1 && a[child + 1].key > a[child].key){
            child++;
        }

        if(a[child].key > tmp.key){
            a[hole] = a[child];
        }else{
            break;
        }
    }

    a[hole] = tmp;
}
```

**性能分析：**
- 时间复杂度为$O(NlogN)$
- 空间性能也很好，只需要提供一个交换元素所需要的辅助空间，即$O(1)$的空间性能
- 不是稳定排序

### 2.3 交换排序

交换排序，即根据序列中两个数据元素的比较结果来确定是否要交换这两个数据元素在数据中的位置。

#### 2.3.1 冒泡排序

冒泡排序的思想时从头到尾比较两个相邻的元素，将小的换到前面，大的换到后面。这样经过一趟比较，就把最大的元素放到了最后一个位置。经过n-1次起泡后，整个排序过程就完成了。但如果一趟起泡中没有发生任何数据交换，则说明这批数据中的相邻元素都满足前面小后面大的顺序，也就是说这批数据已经排好序了，此时排序可以提前终止。

```cpp
template<class Key, class Other>
void bubbleSort(Set<Key, Other> a[], int size){
    for(int i = 1; i < size; i++){
        bool flag = false;

        for(int j = 0; j < size - 1; j++){
            if(a[j].key > a[j + 1].key){
                Set<Key, Other> tmp = a[j];
                a[j] = a[j + 1];
                a[j + 1] = tmp;
                flag = true;
            }
        }
    }
}
```

**性能分析：**
- 时间复杂度$O(N^2)$
- 是稳定排序

#### 2.3.2 快速排序

目前快速排序被认为是最快的一种**内排序算法**。快速排序的基本思想是，在待排序的序列中选择一个数据元素，以该元素为标准，将所有数据分为两组，使得第一组数据元素均小于或等于标准元素，第二组数据元素均大于标准元素。将第一组的元素放在数组的前面部分，第二组的数据放在数组的后面部分，标准元素放中间，这个位置就是标准元素的最终位置。这称为一趟划分。然后对分成的两组数据重复上述过程，直到所有的元素都在适当的位置上。

很明显，快速排序算法是一个**递归算法**。除了实现递归外，要实现快速排序还需解决两个问题：一是如何选择元素，二是如何实现高效的划分。

**标准元素的选择：**
- 选择第一个元素
- 随机选择一个元素
- 采样估计中值

**如何划分：**
- 双指针向中间移动

**快速排序的实现：**
```cpp
template<class Key, class Other>
void quickSort(Set<Key, Other> a[], int size){
    quickSort(a, 0, size - 1);
}

template<class Key, class Other>
void quickSort(Set<Key, Other> a[], int low, int high){
    if(low < high){
        int key = divide(a, low, high);
        quickSort(a, low, key - 1);
        quickSort(a, key + 1, high);
    }
}

template<class Key, class Other>
int divide(Set<Key, Other> a[], int low, int high){
    int i = low, j = high;
    Set<Key, Other> tmp = a[high];
    int flag = 0;

    while(i < j){
        if(flag == 0){
            if(a[i].key < tmp.key){
                i++;
            }else{
                a[j] = a[i]
                flag = 1;
            }
        }

        if(flag == 1){
            if(a[j].key > tmp.key){
                j--;
            }else{
                a[i] = a[j];
                flag = 0;
            }
        }
    }

    a[i] = tmp;
    return i;
}
```

**性能分析：**
- 最好的情况下时间复杂度为$O(NlogN)$
- 平均情况下时间复杂度为$O(NlogN)$
- 最坏的情况下时间复杂度为$O(N^2)$

### 2.4 归并排序

归并排序的思想来源于合并两个已排序的有序表。归并两个有序数组的时间是**线性的**，如果两表加起来共有N个元素，那么归并时需要比较的次数最多为N-1次。归并排序的思想就是利用递归和归并实现排序。如果只有一个元素，那么该元素就是一个已经排好序的序列；否则将前一半序列和后一半序列分别调用递归用归并排序函数进行归并，将排好序的两个序列归并起来，这样就完成了整个排序。

```cpp
template<class Key, class Other>
void mergeSort(Set<Key, Other> a[], int size){
    mergeSort(a, 0, size - 1);
}

template<class Key, class Other>
void mergeSort(Set<Key, Other> a[], int low, int high){
    int mid = (low + high) / 2;
    mergeSort(a, low, mid);
    mergeSort(a, mid + 1, high);
    merge(a, low, mid, high);
}

template<class Key, class Other>
void merge(Set<Key, Other> a[], int low, int mid, int high){
    Set<Key, Other> *array = new Set<Key, Other>[high - low + 1];
    int i = low, j = mid, k = 0;

    while(i <= mid && j <= high){
        if(a[i].key > a[j].key){
            array[k++] = a[i++];
        }else{
            array[k++] = a[j++];
        }
    }

    while(i <= mid){
        array[k++] = a[i++];
    }

    while(j <= high){
        array[k++] = a[j++];
    }

    for(int i = 0; i <= high - low; i++){
        a[low + i] = array[i];
    }

    delete[] array;
}
```

**性能分析：**
- 时间复杂度为$O(NlogN)$
- 需要额外的$O(N)$空间
