# 数据结构与算法(二)：栈和队列(Re:算法导论 & 数据结构：思想与实现)

## 栈

### 1.1 栈的定义

**定义：**栈是一种**特殊的线性表**，在这种线性表中的插入和删除运算**限制在某一端**来进行。进行插入和删除的一端称之为**栈顶**，另一端称之为**栈底**。处于栈顶位置的数据元素称之为**栈顶元素**。若栈中没有元素，则称为**空栈**。在栈中插入一个元素称之为**进栈**或**入栈**，在栈中删除一个元素称之为**出栈**。栈也被称之为**后进先出表**。

**基本运算：**
- 创建一个栈create()
- 进栈push(x)
- 出栈pop(x)
- 读栈顶元素top()
- 判断栈是否为空isEmpty()

**栈的抽象类定义：**

```cpp
template<class elemType>
class stack{
    public:
        virtual bool isEmpty() const = 0;
        virtual void push(const elemType &x) = 0;
        virtual elemType pop() = 0;
        virtual elemType top() const = 0;
        virtual ~stack(){}
};
```

### 1.2 栈的顺序实现

栈的顺序实现称为**顺序栈**。与顺序表类似，顺序栈也是由一个一维数组组成，栈的元素一次存放在数组中。习惯上将下标小的一端设为栈底，即将栈的元素放在数组的前面部分。一个顺序栈的存储实现需要3个变量：一个**指向动态数组的指针**、一个**表示栈大小的整型数**和一个**表示栈顶位置的整型数**。

**顺序栈类的定义：**

```cpp
template<class elemType>
class seqStack:public stack<elemType>{
    private:
        elemType *elem;
        int top_p;
        int maxSize;
        void doubleSpace();

    public:
        seqStack(int initSize=100);
        ~seqStack();
        bool isEmpty() const;
        void push(const elemType &x);
        elemType pop();
        elemType top();
};
```

**顺序栈类方法的实现：**

```cpp
template<class elemType>
seqStack<elemType>::seqStack(int initSize=100){
    maxSize = initSize;
    elem = new elemType(maxSize);
    top_p = 0;
}

template<class elemType>
seqStack<elemType>::~seqStack(){
    delete[] elem;
}

template<class elemType>
bool seqStack<elemType>::isEmpty() const{
    return top_p == -1;
}

template<class elemType>
void seqStack<elemType>::push(const elemType &x){
    if(top_p == maxSize - 1){
        doubleSpace();
    }

    elem[++top_p] = x;
}

template<class elemType>
elemType seqStack<elemType>::pop(){
    if(isEmpty()){
        return -1;
    }

    return elem[top_p--];
}

template<class elemType>
elemType seqStack<elemType>::top() const{
    if(isEmpty()){
        return -1;
    }

    return elem[top_p];
}

template<class elemType>
void seqStack<elemType>::doubleSpace(){
    maxSize *= 2;
    elemType *tmp = elem;
    elem = new elemType[maxSize];

    for(int i = 0; i < maxSize / 2; i++){
        elem[i] = tmp[i]
    }


    delete[] tmp;
}
```

**Remark:**
- top_p = -1时表示栈为空

**性能分析：**
- 除push()以外的所有操作的时间复杂度均为$O(1)$
- 一般情况下push()的时间复杂度也$O(1)$，但当需要doubleSpace()时，时间复杂度会变为$O(n)$。如果把该时间均摊到每个插入操作，则从平均意义上讲，这个操作的时间复杂度还是常数(摊还分析)。

### 1.3 栈的顺序实现

栈的链接实现称为**栈链接**。栈链接**不需要头节点**，直接在栈顶进行插入删除即可。

**链接栈类的定义:**

```cpp
template<class elemType>
class linkStack:public stack<elemType>{
    private:
        struct node{
            elemType data;
            node *next;

            node(const elemType &x, node *n=NULL){
                data = x;
                next = n;
            }

            node():next(NULL){}

            ~node(){}
        };

        node *top_p;

    public:
        linkStack();
        ~linkStack();
        bool isEmpty() const;
        void push(const elemType &x);
        elemType pop();
        elemType top() const; 
};

template<class elemType>
linkStack<elemType>::linkStack(){
    top_p = NULL;
}

template<class elemType>
linkStack<elemType>::~linkStack(){
    while(top_p != NULL){
        node *tmp = top_p->next;
        delete top_p;
        top_p = tmp;
    }

    delete top_p;
}

template<class elemType>
bool linkStack<elemType>::isEmpty() const{
    return top_p == NULL;
}

template<class elemType>
void linkStack<elemType>::push(const elemType &x){
    node *tmp = new node(x, top_p);
    top_p = tmp;
}

template<class elemType>
elemType linkStack<elemType>::pop(){
    if(isEmpty()){
        return -1;
    }

    node *tmp = top_p;
    top_p = tmp->next;
    elemType x = tmp->data;
    delete tmp;
    return x;
}

template<class elemType>
elemType linkStack<elemType>::top(){
    if(isEmpty()){
        return -1;
    }

    return top_p->data;
}
```

**Remark:**
- 栈为空时top_p=NULL

**性能分析：**
- 所有运算复杂度均为$O(1)$

## 队列

### 2.1 队列的定义

**定义:** 队列也是一种特殊的线性表。在这种线性表中，插入限定在表的**一端**，删除限定在表的**另一端**。允许插入的一端被称为**队尾**，允许删除的一端被称为**队头**。若队列中没有元素，则称其为**空队列**。在队列中插入一个元素称为**入队**，在队列中删除一个元素称为**出队**。

**基本运算:**
- 创建一个队列create()
- 入队enQueue()
- 出队deQueue()
- 读取队头元素getHead()
- 判断队列是否为空isEmpty()

**队列的抽象类定义:**
```cpp
template<class elemType>
class queue{
    public:
        virtual bool isEmpty() const = 0;
        virtual void enQueue(const elemType &x) = 0;
        virtual elemType deQueue() = 0;
        virtual elemType getHead() const = 0;
        virtual ~queue(){}       
};
```

### 2.2 队列的顺序实现

队列的顺序实现称为**顺序队列**。队列的顺序实现也由一个一维数组实现。在实现时，若固定队头，则在每次出队时都需要移动大量元素。若保持所有元素位置不变，则会使得空间利用率降低。因此，我们采用**循环队列**的方案来进行队列实现。循环队列将数组的头尾看作相连的，由front和rear来标记队头和队尾。规定front指向的单元**不能存储队列元素**，智能起到标志作用。这样当**front==rear**时队列为空，当 **(rear+1)%MaxSize==front** 时队列为满。

**顺序类队列的定义：**

```cpp
template<class elemType>
class seqQueue:public queue{
    private:
        elemType *elem;
        int maxSize;
        int front, rear;
        void doubleSpace();

    public:
        seqQueue(int initSize=10);
        ~seqQueue();
        bool isEmpty() const;
        void enQueue(const elemType &x);
        elemType deQueue();
        elemType getHead() const;
};
```

**顺序类队列方法的实现：**

```cpp
template<class elemType>
seqQueue<elemType>::seqQueue(int initSize=10){
    maxSize = initSize;
    elem = new elemType(initSize)
    front = rear = 0;
};

template<class elemType>
seqQueue<elemType>::~seqQueue(){
    delete[] elem;
}

template<class elemType>
bool seqQueue<elemType>::isEmpty() const{
    return front == rear;
}

template<class elemType>
void seqQueue<elemType>::enQueue(const elemType &x){
    if((rear + 1) % maxSize == front){
        doubleSpace();
    }

    rear = (rear + 1) % maxSize;
    elem[rear] = x;
}

template<class elemType>
elemType seqQueue<elemType>::deQueue(){
    if(isEmpty()){
        return -1;
    }

    front = (front + 1) % maxSize;
    return elem[front];
}

template<class elemType>
elemType seqQueue<elemType>::getHead() const{
    return elem[(front + 1) % maxSize];
}

template<class elemType>
void seqQueue<elemType>::doubleSpace(){
    elemType *tmp = elem;
    elem = new elemType(2 * maxSize);

    for(int i = 0; i < maxSize; i++){
        elem[i] = tmp[(front + i) % maxSize];
    }

    front = 0; 
    rear = maxSize;
    maxSize *= 2;
    delete tmp;
}
```

**Remark：**
- front和rear值溢出时需要及时取模
- doubleSpace()从front处开始复制
- 判断队空和队满的条件要注意

**性能分析：**
- 与顺序栈的情况类似

### 2.3 队列的链接实现

队列的链接实现称为**链接队列**。对单链表来说，表头的操作比较简单，但对表尾操作比较麻烦，必须从表头遍历到表尾。为方便两端操作，可采用**空间换时间**的方法，同时记住表头和表尾。

**链接队列类的定义：**

```cpp
template<class elemType>
class linkQueue:public queue<elemType>{
    private:
        struct node{
            elemType data;
            node *next;
            node(const elemType &x, node *n=NULL){
                data = x;
                next = n;
            }

            node():next(NULL){}

            ~node(){}
        };

        node *front, *rear;

    public:
        linkQueue()
        ~linkQueue()
        bool isEmpty() const;
        void enQueue(const elemType &x);
        elemType deQueue();
        elemType getHead() const;
};

template<class elemType>
linkQueue<elemType>::linkQueue(){
    front = rear = NULL;
}

template<class elemType>
linkQueue<elemType>::~linkQueue(){
    while(front != rear){
        node *tmp = front;
        front = front->next;
        delete tmp;
    }

    delete rear;
}

template<class elemType>
bool linkQueue<elemType>::isEmpty() const{
    return front == NULL;
}

template<class elemType>
void linkQueue<elemType>::enQueue(const elemType &x){
    if(rear == NULL){                               //队列为空
        front = rear = new node(x);
    }else{
        rear = rear->next = new node(x, NULL);
    }
}

template<class elemType>
elemType linkQueue<elemType>::deQueue(){
    node *tmp = front;
    front = front->next;
    elemType x = tmp->data;
    delete tmp;
    return x;
}

template<class elemType>
elemType linkQueue<elemType>::getHead() const{
    return front->data;
}
```

**Remark:**
- 两端均不需要头节点
- 队列空时两端均为NULL
- 入队时要检查队列是否为空

**性能分析：**
- 与链接栈类似
