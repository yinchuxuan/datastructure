# 数据结构与算法(一)：线性表(Re:算法导论 & 数据结构：思想与实现)

## 亿点啰嗦……

- **算法(algorithm):** 任何良定义的计算过程，该过程去某个值或值的集合作为**输入**并产生某个值或值得集合作为**输出**
- 若对于每个输出实例，算法都以正确的输出停机，则称该算法**正确**
- **数据结构(data structure):** 一种存储和组织数据的方式，旨在便于修改和访问。**没有一种单一的数据结构对所有的用途均有效，重要的是知道数据结构的优点和局限性**。
- 一般来说，输入算法所需时间与输入规模同步增长，所以通常把一个程序的运行时间描述成其输入规模的函数。**输入规模的最佳概念依赖于研究问题** (所以这种描述并非很严格)。
- 我们一般集中于只求**最坏情况的运行时间**，但在某些特定的情况下，我们会对**平均运行时间**感兴趣。 

## 函数的增长

当输入规模足够大，使得只有运行时间的增长量级是重要的时，我们需要研究算法的渐近效率。

- **$\Theta$记号**：对于一个给定的函数$g(n)$,我们用$\Theta(g(n))$来表示以下函数的集合:
    $$ \Theta(g(n)) = \{f(n): \exist c_1,c_2,n_0, s.t.  \forall n \geq n_0, 0 \leq c_1 g(n) \leq f(n) \leq c_2 g(n) \} $$
  即存在正常量$c_1$, $c_2$, 使得对于足够大的n, $f(n)$能够“夹入”$c_1 g(n)$与$c_2 g(n)$之间。我们称$g(n)$为$f(n)$的一个**渐近紧确界(asympotically tight bound)**。在这里，$\Theta(g(n))$的定义要求每个$f(n)$都**渐近非负**(显然为了研究时间复杂度……)，这个假设对即将介绍的其它复杂度也成立。

- **$O$记号**：对于一个给定的函数$g(n)$,我们用$O(g(n))$来表示以下函数的集合:
    $$ O(g(n)) = \{f(n): \exist c, n_0, s.t.  \forall n \geq n_0, 0 \leq f(n) \leq cg(n) \} $$
    我们称$g(n)$为$f(n)$的一个**渐近上界**,它为我们提供了一个算法在最坏情况下运行时间的描述。

- **$\Omega$记号**：对于一个给定的函数$g(n)$,我们用$\Omega(g(n))$来表示以下函数的集合:
    $$ \Omega(g(n)) = \{f(n): \exist c, n_0, s.t.  \forall n \geq n_0, 0 \leq c g(n) \leq f(n) \} $$
    我们称$g(n)$为$f(n)$的一个**渐近下界**。

**定理2.1**: $\forall f(n), g(n)$, $f(n) = \Theta(g(n)$ 当且仅当 $f(n) = O(g(n))$ 且 $f(n) = \Omega(g(n))$

- **$o$记号**: 由$O$记号提供的渐近上界有可能不是渐近紧确的。我们可以用$o$记号来表示一个非渐近紧确的上界。对于一个给定的函数$g(n)$,我们用$o(g(n))$来表示以下函数的集合:
    $$ o(g(n)) = \{f(n): \exist c, n_0, s.t.  \forall n \geq n_0, 0 \leq f(n) < cg(n) \} $$

- **$\omega$记号**: 类似地，我们使用$\omega$记号来表示一个非渐近紧确的下界。对于一个给定的函数$g(n)$,我们用$\omega(g(n))$来表示以下函数的集合:
    $$ \omega(g(n)) = \{f(n): \exist c, n_0, s.t.  \forall n \geq n_0, 0 \leq cg(n) < f(n) \} $$

## 线性表

### 3.1 线性表定义

**定义**：线性结构是n(n $\geq$ 0)个节点的有穷序列$(a_0, a_1,...,a_n )$。$a_0$称为起始节点，$a_n$称为终止节点，i称为$a_i$的序号或者位置。对任意一对相邻节点$a_i$和$a_{i+1}$,$a_i$称为$a_{i+1}$的前驱，$a_{i+1}$称为$a_i$的后继。(有拓扑的感觉在里面，不要被表面结构限制)

**基本运算**:
- 创建一个空线性表create
- 删除所有元素clear
- 求线性表的长度length
- 在第i个位置插入一个元素insert(x)
- 删除第i个位置的元素remove(i, x)
- 搜索元素search(x)
- 访问第i个元素visit(i)
- 顺序访问每一个元素traverse()

**线性表的抽象类定义**:
```cpp
template<class elemType>
class list{
    public:
        virtual void clear() = 0
        virtual int length() const = 0
        virtual void insert(int i, elemType &x) = 0
        virtual void remove(int i) = 0
        virtual int search(const elemType &x) const = 0
        virtual elemType visit(int i) = 0
        virtual void traverse() const = 0
        virtual ~list() {};
}
```

### 3.2 线性表的顺序实现

所谓线性表的顺序实现，就是将线性表的数据元素存储在**一块连续的空间**中，用存储位置反映数据元素之间的关系(在内存中的存储方式才是本质)。要实现顺序表，需要在计算机内部开辟一块连续的存储空间。**数组**是所有高级语言都支持的数据类型，它在内存中占用了一块连续的空间，具有天然的线性结构，只要将数组的线性关系和线性表中的线性关系一一对应就可以实现线性表的顺序存储。

线性表经常要执行插入、删除操作，而这两个操作会引起表长的变化，因此数组中需要留有一定的空余空间(用数组的麻烦就在这里，数据在内存中的位置固定，查询方便但是难以修改)。数组的规模称为顺序表的**容量**。当执行插入操作时，如果表长小于容量，则直接插入；当表长等于容量时，需要先扩大动态数组的空间，再执行插入。

**顺序表类的定义：**
```cpp
template<class elemType>
class seqList:public list<elemType>{
    private:
        elemType *data;
        int currentLength;
        int maxSize;
        void doubleSize()

    public:
        seqList(int initSize=100);
        ~seqList();
        void clear();
        int Length() const;
        void insert(int i, const elemType &x);
        void remove(int i);
        int search(const elemType &x) const;
        elemType visit(int i) const;
        void traverse() const;
};
```

**顺序表方法实现**:

```cpp
template<elemType>
seqList<elemType>::seqList(int initSize=100){
    data = new int(initSize);
    currentSize = 0;
    maxSize = 100;
}

template<elemType>
seqList<elemType>::~seqList(){
    delete[] data;
}

template<elemType>
void seqList<elemType>::clear(){
    currentLength = 0;
}

template<elemType>
int seqList<elemType>::Length(){
    return currentLength;
}

template<elemType>
void seqList<elemType>::insert(int i, const elemType &x){
    if(currentLength == maxSize){
        doubleSize();
    }

    for(j = currentLength; j > i; j--){
        data[j] = data[j - 1];
    }

    data[i] = x
}

template<elemType>
void seqList<elemType>::remove(int i){
    if(currentLength != 0){
        currentLength--;

        for(int j = i; j < currentLength; j++){
            data[j] = data[j + 1];
        }
    }
}

template<elemType>
int seqList<elemType>::search(const elemType &x) const{
    for(int i = 0; i < currentLength; i++){
        if(data[i] == x){
            return i;
        }
    }

    return -1;
}

template<elemType>
elemType seqList<elemType>::visit(int i) const{
    return data[i]
}

template<elemType>
void seqList<elemType>::traverse(){
    for(int i =0; i < currentLength; i++){
        std::cout << i << " ";
    }

    std::cout << std::endl;
}

template<elemType>
void seqList<elemType>::doubleSize(){
    maxSize = maxSize * 2;
    elemType *newdata = data;
    data = new elemType[maxSize];

    for(int i = 0; i < currentLength; i++){
        data[i] = newdata[i]
    }

    delete[] newdata;
}

```

**Remark:**
- clear()直接设currentLength为0是最骚的(把currentLength考虑成指针！)
- 在插入删除前都要check一下当前的大小

**性能分析**
- length(),visit(),clear()时间复杂度为$O(1)$
- traverse()时间复杂度为$O(n)$
- 构造函数时间复杂度为$O(1)$
- insert()和remove()平均时间复杂度为$O(n)$
- 不适合大规模插入删除，适合定位访问
- 当表长小于数组规模时有一部分闲置空间被浪费

### 3.3 线性表的链接实现

为了克服顺序表的缺点，我们可以采用**链接**的方式来实现线性表。链接存储是一种常用的**表示关系**的方法，它通过让每个节点保存与它有关系的节点的**地址**来保存节点之间的联系。链接方式不仅可以用来表示线性表，还可以用来表示各种非线性的数据结构(各种多元表，跳表，etc)。

线性表的链接存储是指将每个数据元素存放在一个**独立的存储单元**中。这些独立的存储单元在物理上可以是相邻的，也可以是不相邻的(在内存结构上区别于顺序结构)。在每个节点中除了存放数据元素之外还存放了邻接节点的地址。由于无须保持逻辑次序和物理次序的一致性，插入删除时只需要修改节点中保存的邻接节点的地址，不会引起大量的数据移动。

#### 3.2.1 单链表

在单链表中，每个节点由一个数据元素和一个后继指针组成。后继指针指向该节点的后继节点，终端节点的后继指针为空指针。为了保证链中每个元素都有一个前驱，我们在链表中引入一个头节点(头节点中不存放数据)。

**单链表类定义：**
```cpp
template<class elemType>
class sLinkList:public list<elemType>{
    private:
        struct node{                //单链表中的节点类
            elemType data;
            node *next;

            node(const elemType &x, node *n=NULL){
                data = x;
                next = n;
            };

            node():next(NULL){}

            ~node(){}
        }

        node *head                  //头指针
        int currentLength           //表长

        node *move(int i) const     //返回第i个节点的地址

    public:
        sLinkList();
        ~sLinkList();
        void clear();
        int length() const;
        void insert(int i, const elemType &x);
        void remove(int i);
        int search(const elemType &x) const;
        elemType visit(int i) const;
        void traverse() const;
        void toggle();              //表演一下翻转链表
};

```

**单链表类方法实现：**
```cpp
template<elemType>
sLinkList<elemType>::sLinkList(){
    head = new node;
    currentLength = 0;
}

template<elemType>
sLinkList<elemType>::~sLinkList(){
    clear()
    delete[] head;
}

template<elemType>
void sLinkList<elemType>::clear(){
    while(head->next != NULL){
        node *tmp = head->next;
        head->next = tmp->next;
        delete tmp;
    }
}

template<elemType>
int sLinkList<elemType>::Length() const{
    return currentLength;
}

template<elemType>
void sLinkList<elemType>::insert(int i, const elemType &x){
    node *precursor = move(i);
    node *tmp = new node(x, precursor->next);
    precursor->next = tmp;
    currentLength++;
}

template<elemType>
void sLinkList<elemType>::remove(int i){
    node *precursor = move(i - 1);
    tmp = precursor->next;
    precursor->next = tmp->next;
    delete tmp;
    currentLength--;
}

template<elemType>
int sLinkList<elemType>::search(const elemType &x) const{
    node* tmp = head->next;

    for(i = 1; i <= currentLength; i++){
        if(tmp == NULL){
            break;
        }

        if(tmp->data == x){
            return i;
        }

        tmp = tmp->next;
    }

    return -1;
}

template<elemType>
elemType sLinkList<elemType>::visit(int i) const{
    return move(i)->data;
}

template<elemType>
void sLinkList<elemType>::traverse() const{
    node *tmp = head->next;

    while(tmp != NULL){
        std::cout << tmp->data << " ";
        tmp = tmp->next;
    }

    std::cout << std::endl;
}

template<elemType>
void sLinkList<elemType>::toggle(){
    node *tmp = head->next;
    node *res;

    if(tmp != NULL){
        res = tmp->next;
    }

    while(res != NULL){
        node *tmp1 = res;
        res = res->next;
        tmp1->next = tmp;
        tmp = tmp1;
    }

    head->next = tmp;
}

template<elemType>
sLinkList<elemType>::node* sLinkList<elemType>::move(int i) const{
    tmp = head;

    for(int j = 0; j < i; i++){
        tmp = tmp->next
    }

    return tmp;
}
```

**Remark:**
- 在链表中保存currentLength可以在很多地方提供便利
- 在插入和删除的时候记得修改currentLength
- node结构体中要有默认构造函数！

#### 3.2.2 双链表

在链接实现中，若每个节点既保存了**前驱**的地址又保存了**后继**的地址，则该链表称为双链表。为方便插入和删除，双链表除了需要添加头节点外，还需要添加一个**尾节点**，保存一个双链表就是保存头尾两个节点的地址。

**双链表类定义：**
```cpp
template<class elemType>
class dLinkList:public list<elemType>{
    private:
        struct node{                    //双链表中的节点类
            elemType data;
            node *prev, node *next;
        };

        node(const elemType &x, node *p=NULL, node *n=NULL){
            data = x;
            prev = p;
            nexxt = n;
        }

        node():prev(NULL), next(NULL){}

        ~node(){}

        node *head, *tail;              //头尾指针
        int currentLength;

        node *move(int i) const         //移动到第i个节点的地址
    
    public:
        dLinkList();
        ~dLinkList();
        void clear();
        int length() const;
        void insert(int i, const elemType &x);
        void remove(int i);
        int search(const elemType &x) const;
        elemType visit(int i) const;
        void traverse() const;
};

template<class elemType>
dLinkList<elemType>::dLinkList(){
    head = new node();
    tail = new node();
    currentLength = 0;
    head->next = tail;
    tail->prev = head;
}

template<class elemType>
dLinkList<elemType>::~dLinkList(){
    clear();
    delete head; 
    delete tail;
}

template<class elemType>
dLinkList<elemType>::clear(){
    while(head->next != tail){
        node *tmp = head->next;
        head->next = tmp->next;
        delete tmp;
    }
}

template<class elemType>
int dLinkList<elemType>::length() const{
    return currentLength;
}

template<class elemType>
void dLinkList<elemType>::insert(int i, const elemType &x){
    prev = move(i);
    node *tmp = new node(x, prev, prev->next);
    prev->next = tmp;
    tmp->next->prev = tmp;
    currentLength++;
}

template<class elemType>
void dLinkList<elemType>::remove(int i){
    prev = move(i - 1);
    node *tmp = prev->next;
    prev->next = tmp->next;
    tmp->next->prev = prev;
    delete tmp;
    currentLength--;
}

template<class elemType>
int dLinkList<elemType>::search(const elemType &x){
    node *tmp = head->next;

    for(i = 1; i <= currentLength; i++){
        if(tmp->data == x){
            return i;
        }
    }

    return -1;
}

template<class elemType>
elemType dLinkList<elemType>::visit(int i){
    node *tmp = head;

    for(int j = 1; j <= i; j++){
        tmp = tmp->next;
    }

    return tmp->data;
}

template<class elemType>
void dLinkList<elemType>::traverse(){
    return move(i)->data;
}

template<class elemType>
dLinkList<elemType>::node* dLinkList<elemType>::move(int i) const{
    node *tmp = head;

    for(int j = 1; j <= i; j++){
        tmp = tmp->next;
    }

    return tmp;
}
```

**Remark:**
- 在插入删除时需要仔细处理指针，其它操作与单链表几乎一样

**性能分析**
- visit()的时间复杂度为$O(n)$
- insert()和remove()的时间复杂度均为$O(n)$，但比顺序表的性能好
- 适合经常执行插入删除但很少访问指定位置元素的线性表
- 动态分配内存，不会出现空间浪费

  

