# 数据结构与算法(三)：树结构(Re:算法导论 & 数据结构：思想与实现)

## 树

### 1.1 树的定义

树是表示**层次关系**的数据结构，树状结构的特点就是一个数据元素可以有很**多个直接后继**，但**只有一个直接前驱**，因此树状结构可以表达更加复杂的关系。

**定义(树的递归定义)：**树是n个节点的**有限**集合，它或者是**空集**(注意递归)，或者满足以下条件：
1. 有一个被称为根的节点
2. 其余节点可以分为$m(m \geq 0)$个互不相交的集合$T_1,T_2,...,T_m$，这些集合本身也是一种树，并把它们称为根节点的子树(subtree)

**树的基本用语:**
- 树中唯一一个没有直接前驱的节点称为**根节点**，树中没有后继的节点称为**叶节点**，除根以外的非叶节点也称为**内部节点**(同样有拓扑内味了……)。
- 一个节点的直接后继数目称为节点的**度**。树中所有节点的度的**最大值**称为这棵树的度。
- 节点的直接后继称为节点的**子节点**，而节点的直接前驱称为它的**父节点**。在树中，每个节点都**存在着唯一一条到根节点的路径**，路径上的所有节点都是该节点的**祖先节点**。**子孙节点**是指该节点的所有子树中的全部节点。同一个节点的子节点互为**兄弟节点**。
- 结构的层次也称为深度。一棵树中节点的最大层次称为树的**高度**。节点的高度是指以该节点为根的子树的高度。
- 若将树中每个节点的子树看成是自左向右有序的，则称该树为**有序树**，否则称为**无序树**。
- M棵互不相交的树的集合称为**森林**。

**树的基本运算：**
- 建树create()
- 清空clear()
- 判空isEmpty()
- 找根节点root()
- 找父节点parent(x)
- 找子节点child(x, i)
- 剪枝remove(x, i)
- 遍历traverse()

**树的抽象类定义：**
```cpp
template<class T>
class Tree{
    public:
        virtual void clear() = 0;
        virtual bool isEmpty() const = 0;
        virtual T root(T flag) const = 0;
        virtual T parent(T x, T flag) const = 0;
        virtual T child(T x, int i, T flag) const = 0;
        virtual void remove(T x, int i) = 0;
        virtual void traverse() const = 0;
}
```

## 二叉树

### 2.1 二叉树的定义

**定义:** 二叉树是节点的 **有限集合**，它或者为空，或者由一个根节点及两棵**互不相交**的左、右子树构成，而左、右子树又都是二叉树。注意二叉树是**有序树**，其必须严格区分左右子树。

如果一颗二叉树中的任意一层的节点个数都达到了最大值，则这棵二叉树被称为**满二叉树**。**完全二叉树**是在满二叉树的最底层自右至左依次去掉若干个节点。

**二叉树的常用性质：**
- 一棵非空二叉树的第i层上最多有$2^{i-1}$个节点$(i \geq 1)$
- 一棵高度为k的二叉树，最多有$2^k - 1$个节点
- 对于一棵非空二叉树，如果叶子节点数为$n_0$，度为2的节点数为$n_2$，则有$n_0 = n_2 + 1$。
- 具有n个节点的完全二叉树的高度$k = \lfloor log_2 n \rfloor + 1$

**二叉树的基本运算：**
- 建树create()
- 清空clear()
- 判空isEmpty()
- 找根节点root()
- 找父节点parent(x)
- 找左孩子lchild(x)
- 找右孩子rchild(x)
- 删除左子树delLeft(x)
- 删除右子树delRight(x)
- 遍历traverse()

**遍历方法：**
- 前序遍历：访问根节点 $\rightarrow$ 前序遍历左子树 $\rightarrow$ 前序遍历右子树
- 中序遍历：中序遍历左子树 $\rightarrow$ 访问根节点 $\rightarrow$ 中序遍历右子树
- 后序遍历：后序遍历左子树 $\rightarrow$ 后序遍历右子树 $\rightarrow$ 访问根节点
- 层次遍历：访问根节点 $\rightarrow$ 从左至右访问第二层 $\rightarrow$ ... $\rightarrow$ 从左至右访问第k层
  
已知二叉树的前(后)序遍历序列和中序遍历序列可以**唯一地确定**一棵二叉树

**二叉树的抽象类定义：**
```cpp
template<class T>
class bTree{
    public:
        virtual void clear() = 0;
        virtual bool isEmpty() const = 0;
        virtual T Root(T flag) const = 0;
        virtual T parent(T x, T flag) const = 0;
        virtual T lchild(T x, T flag) const = 0;
        virtual T rchild(T x, T flag) const = 0;
        virtual void delLeft(T x) = 0;
        virtual void delRight(T x) = 0;
        virtual void preOrder() const = 0;
        virtual void midOrder() const = 0;
        virtual void postOrder() const = 0;
        virtual void levelOrder() const = 0;
};
```

### 2.2 二叉树的链接实现
二叉树的主要存储方式为**链接存储**，用链接存储结构存储二叉树是很自然的方式，所谓的二叉树链接存储就是用指针指出父子关系。类似于单链表和双链表，二叉树也有两种链接存储模式：**标准存储结构**和**广义的标准存储结构**。

**标准存储结构：** 标准存储结构也称为**二叉链表**。在二叉链表中，每个存储节点由三个字段组成：存储数据元素的值以及左、右儿子的指针字段。以二叉链表的方式存储一棵二叉树只需要一个指向根节点的指针root，称为根指针。

**广义标准存储结构：** 在标准存储结构的基础上，再添加一个指向其父节点的指针，也被称为**三叉链表**。

#### 2.2.1 二叉链表类

**二叉链表类的定义：**
```cpp
template<class T>
class binaryTree:public bTree<T>{
    private:
        struct Node{
            Node *left, *right;
            T data;

            Node():left(NULL),right(NULL){}
            Node(T item, Node *L=NULL, Node *R=NULL):data(item),left(L),right(R){}
            ~Node(){}
        };

        Node *root;

    public:
        binaryTree():root(NULL){}
        binaryTree(T x);
        ~binaryTree();
        void clear();
        bool isEmpty() const;
        T Root(T flag) const;
        T lchild(T x, T flag) const;
        T rchild(T x, T flag) const;
        void delRight(T x);
        void delLeft(T x);
        void preOrder() const;
        void midOrder() const;
        void postOrder() const;
        void levelOrder() const;
        T parent(T x, T flag) const;
    
    private:
        Node *find(T x, Node *t) const;
        void clear(Node *&t);
        void preOrder(Node *t) const;
        void midOrder(Node *t) const;
        void postOrder(Node *t) const;
};

```

**二叉链表类的方法实现：**
```cpp
template<class T>
binaryTree<T>::binaryTree(T x){
    root = new Node(x);
}

template<class T>
binaryTree<T>::~binaryTree(){
    clear();
}

template<class T>
void binaryTree<T>::clear(){
    clear(root);
}

template<class T>
bool binaryTree<T>::isEmpty() const{
    return root == NULL
}

template<class T>
T binaryTree<T>::Root(T flag) const{
    if(isEmpty()){
        return flag;
    }

    return root->data;
}

template<class T>
T binaryTree<T>::lchild(T x, T flag) const{
    Node *tmp = find(x, root);

    if(tmp == NULL || tmp->left == NULL){
        return flag;
    }

    return tmp->left->data;
}

template<class T>
T binaryTree<T>::rchild(T x, T flag) const{
    Node *tmp = find(x, root);

    if(tmp == NULL || tmp->right == NULL){
        return flag;
    }

    return tmp->right->data;
}

template<class T>
void binaryTree<T>::delLeft(T x){
    Node *tmp = find(x ,root);

    if(tmp != NULL && tmp->left != NULL){
        clear(tmp->left);
    }
}

template<class T>
void binaryTree<T>::delRight(T x){
    Node *tmp = find(x ,root);

    if(tmp != NULL && tmp->right != NULL){
        clear(tmp->right);
    }
}

template<class T>
void binaryTree<T>::preOrder() const{
    preOrder(root);
}

template<class T>
void binaryTree<T>::midOrder() const{
    midOrder(root);
}

template<class T>
void binaryTree<T>::postOrder() const{
    postOrder(root);
}

template<class T>
void binaryTree<T>::levelOrder() const{
    std::queue<Node*> q;
    Node *tmp;
    q.push(root);

    while(!q.empty()){
        tmp = q.pop();
        std::cout << tmp->data;

        if(tmp->left != NULL){
            q.push(tmp->left);
        }

        if(tmp->right != NULL){
            q.push(tmp->right);
        }
    }
}

template<class T>
T parent(T x, T flag) const{
    return flag;
}

template<class T>
binary<T>::Node* find(T x, Node *t) const{
    if(t == NULL){
        return NULL;
    }

    if(t->data == x){
        return t;
    }

    lr = find(x, t->left);
    rr = find(x, t->right);

    if(lr != NULL){
        return lr;
    }

    if(rr != NULL){
        return rr;
    }

    return NULL
}

template<class T>
void binaryTree<T>::clear(Node *&t){
    if(t != NULL){
        clear(t->left);
        clear(t->right);
        Node *tmp = t;
        delete tmp;
        t = NULL;
    }
}

template<class T>
void binary<T>::preOrder(Node *t) const{
    if(t != NULL){
        std::cout << t->data << " ";
        preOrder(t->left);
        preOrder(t->right);
    }
}

template<class T>
void binary<T>::midOrder(Node *t) const{
    if(t != NULL){
        preOrder(t->left);
        std::cout << t->data << " ";
        preOrder(t->right);
    }
}

template<class T>
void binary<T>::postOrder(Node *t) const{
    if(t != NULL){
        preOrder(t->left);
        preOrder(t->right);
        std::cout << t->data << " ";
    }
}    
```

**Remark:**
- 注意私有函数的设计，尤其是find()和clear()
- 注意递归终止条件

**性能分析：**
- 树上的动态操作均与树高有关，即时间复杂度为$O(h)$。在一般情况下，h的期望值为$log(n)$，最坏情况为n
  
### 2.3 二叉树遍历的非递归实现：
递归虽然使程序结构清晰、明了、美观，但递归程序也有其缺点，即它的时间、空间利用率比较低。所以在实际使用中，常常希望使用**非递归版本**。

前序遍历的非递归实现比较简单。设置一个**栈**，保存将要访问的树的**树根**。访问时从栈中取出一个节点，按照前序遍历先遍历该节点，再将左子树和右子树的节点依次放入栈中。当栈为空时，表示所有子树已经被遍历，函数结束(函数递归在内存中也是通过栈进行模拟的)。

**前序遍历的非递归实现：**
```cpp
template<class T>
void BinaryTree<T>::preOrder() const{
    std::stack<Node*> s;
    s.put(root);
    Node *tmp;

    while(!s.empty()){
        tmp = s.pop();
        std::cout << tmp->data << " ";
        
        if(tmp->left != NULL){
            s.push(tmp->left);
        }

        if(tmp->right != NULL){
            s.push(tmp->right);
        }
    }
}
```

中序遍历较前序遍历稍微复杂一些。和前序遍历一样，还是采用栈来保存需要访问的节点的树根。但在中序遍历中，**先要遍历左子树**，接下来才能访问根节点。因此，当根节点**出栈时不能立即访问它**，而是要等左子树访问完毕后才能访问它，故此时需要把根节点暂存一下。由于左子树访问完后还需要访问根节点，故此时**仍然可以把它存在栈中**，接着左子树也进栈。在左子树访问完毕后，再次将根节点出栈，此时根节点可以被访问。从上述过程中可以看到根节点**需要被访问两次**，故节点内部需要有一个标记记录节点是第几次进栈。为此，在binaryTree类中定义一个私有的内嵌类StNode，表示栈元素类型。TimesPop是一个整型数，表示元素第几次进栈。

**StNode类定义:**

```cpp
struct StNode{
    Node *node;
    int TimesPop;

    StNode(Node *N=NULL):node(N),TimesPop(0){}
};
```

**中序遍历的非递归实现：**

```cpp
template<class T>
void BinaryTree<T>::midOrder() const{
    std::stack<StNode> s;
    StNode Root(root);
    s.push(Root);
    StNode tmp;

    while(!s.empty()){
        tmp = s.pop();

        if(tmp->TimesPop == 0){
            tmp->TimesPop++;

            if(tmp->node->left != NULL){
                StNode Left(tmp->node->left);
                s.push(Left);
            }

            s.push(tmp);
        }else{
            std::cout << tmp->node->data << " ";

            if(tmp->node->right != NULL){
                StNode Right(tmp->node->right);
                s.push(Right);
            }
        }
    }
}
```

将中序遍历的非递归实现思想进一步延伸，可以得到后序遍历的非递归实现。同样用栈来保存当前需要访问的树节点，当左子树和右子树都访问完毕，节点**第三次出栈**时，才能被访问。

**后序遍历的非递归实现：**

```cpp
template<class T>
void BinaryTree<T>::postOrder() const{
    std::stack<StNode> s;
    StNode Root(root);
    s.push(Root);
    StNode tmp;

    while(!s.empty()){
        tmp = s.pop();

        if(tmp->TimesPop == 0){
            tmp->TimesPop++;

            if(tmp->node->left != NULL){
                StNode Left(tmp->node->left);
                s.push(Left);
            }

            s.push(tmp);
        }else if(tmp->TimesPop == 1){
            tmp->TimesPop++;

            if(tmp->node->right != NULL){
                StNode Right(tmp->node->left);
                s.push(Right);
            }

            s.push(tmp);
        }else{
            std::cout << tmp->node->data << " ";
        }
    }
}
```

**Remark:**
- 注意前、中、后序遍历的非递归实现均使用栈来实现
- 注意对节点状态的标记
- 后序遍历时，为什么不将左右子树同时入栈？

## 哈夫曼树和哈夫曼编码

### 3.1 文本编码 

**motivation(与信息论有关)：** 文本处理时，为提高存储和处理文本的频率，在某些场合下希望采用**不等长的编码**。让使用频率高的字符有较短的编码，使用频率低的字符有较长的编码。总体上，这样会使保存文本所需的空间变少。我们可以通过构建一棵**哈夫曼树**，再从哈夫曼树中获得**哈夫曼编码**。由信息论可知这样的编码是**最优的**。

一个**等长的二进制编码**可以由一棵**完全二叉树**进行表示。在这棵树中，**字符仅被存储在叶节点**中，每个字符的编码是**从根节点到叶节点的路径**，用0表示左子树，用1表示右子树。若某字符$c_i$到到根节点的枝数为$L_i$，并在文件中出现$w_i$次，则**所有字符所需的总存储量**$\Sigma L_i \times w_i$称为这个编码的代价。

**前缀编码：** 字符编码可以有不同的长度，只要每个字符的编码和其它字符编码的前缀不同即可，这种编码被称为**前缀编码**。相反，如果某个字符被包含在一个非叶节点中，则不可能保证无二义性解码。

所以，基本问题就是**找到一棵最小代价的二叉树**，在这棵树上，所有**字符都包含在叶节点**上。要使得整棵树代价最小，显然**权值(字符在文本中的出现频率)** 越大的叶子节点应该越靠近树根，使得它的编码尽可能短。

### 3.2 哈夫曼算法

哈夫曼(Huffman)提出了一种算法，即哈夫曼算法，来构造一棵最优的二叉树。哈夫曼算法需要维护一棵**森林**，其中每棵树的**权值**由其**所有叶节点的出现频率之和**组成。每次选出**权值最小的两棵树$T_1$和$T_2$**(当有树权值相同时，可任意选择一棵)，将$T_1$和$T_2$作为两棵子树构建一棵新树，很明显新树的权值为两棵子树的权值之和。在算法开始时，有n棵单节点构成的树，**每一棵树对应一个字符**。经过**n-1次归并**后，所剩下的这一棵树就是我们所需要的哈夫曼树。

更形式化地说，哈夫曼算法构造最优二叉树的步骤如下：
1. 给定一个具有n个权值$\{ w_1, w_2, ... , w_n \}$的节点的集合，构造一片森林$F = \{ T_1, T_2, ... , T_n \}$。F中的每棵树都是只有树根节点的二叉树。$T_i$的权值为$w_i$。
2. 执行i=1至n-1次循环，在每次循环中，执行如下操作：
   从当前森林中选择两棵权值最小的树，以这两棵树为左右子树构建一棵新树，其权值是两棵子树的权值之和。在森林中将两棵子树去除并加入新树。
3. 经过n-1次循环后，森林F中只剩下一棵树，这棵树就是我们需要的哈夫曼树。

### 3.3 哈夫曼树类的实现

由于哈夫曼编码的生成过程是从**树叶到树根的回溯过程**。而二叉树的标准存储仅适合于从父节点找子节点，并不适合从子节点找父节点。而在编码生成时，需要知道某个节点是它的父节点的左孩子还是右孩子。故二叉树的标准存储并不适合实现哈夫曼树。

在哈夫曼树中，每个带编码的元素都是一个叶节点，其它节点都是度数为2的节点。只要给定了带编码的元素个数n，**哈夫曼树的规模就固定为2n-1**。而一旦生成了哈夫曼树，这颗树就**不会被修改**了。鉴于这些特性，我们可以用静态存储的方式来实现哈夫曼树的广义标准存储。因此，哈夫曼树用一棵**规模为2n的数组**来存储。**下标为0的数组元素不用**，根存放在下标为1的数组元素中，每个节点中保存自己的**父节点**以及**左、右孩子节点**。

**哈夫曼树类的定义：**

```cpp
template<class T>
class hfTree{
    private:
        struct Node{
            T data;
            int weight;
            int parent, left, right;
        };

        Node *elem;
        int length;

    public:
        struct hfCode{
            T data;
            std::string code;
        };

        hfTree(const T *x, const int *w, int size);
        void getCode(hfCode result[]);
        ~hfTree()
};
```

**哈夫曼树类方法的实现：**

```cpp
template<class T>
hfTree<T>::hfTree(const T *x, const int *w, int size){
    length = 2 * size;
    elem = new Node(length);

    for(int i = size; i < length; i++){
        elem[i]->data = x[i - size];
        elem[i]->weight = w[i - size];
        elem[i]->parent = elem[i]->left = elem[i]->right = -1;
    }

    elem[0]->weight = 10000000;
    int tmp1, tmp2;

    for(int i = size - 1; i > 0; i--){
        tmp1 = tmp2 = 0;

        for(int j = size; j < length; j++){
            if(elem[j] < elem[tmp1] && elem[j]->parent == -1){
                tmp1 = j;
            }else if(elem[j] < elem[tmp2] && elem[j]->parent == -1){
                tmp2 = j;
            }
        }

        elem[i]->weight = elem[tmp1]->weight + elem[tmp2]->weight;
        elem[i]->left = tmp1;
        elem[i]->right = tmp2;
        elem[i]->parent = -1;
        elem[tmp1]->parent = elem[tmp2]->parent = i;
    }
}

template<class T>
hfTree<T>::~hfTree(){
    delete[] elem;
}

template<class T>
void hfTree<T>::getCode(hfCode result[]){
    result = new hfCode(length / 2);

    for(int i = length / 2; i < length; i++){
        result[i].data = elem[i].data;
        result[i].code = "";
        int tmp = i;

        while(tmp != 1){
            int child = tmp;
            tmp = elem[child]->parent;

            if(elem[tmp]->left == child){
                result[i].code = "0" + result[i].code;
            }else{
                result[i].code = "1" + result[i].code;
            }
        }
    }
}
```

**Remark:**
- 在数组元素0处标一个flag可以方便寻找最小的两个元素
- 暂时没有父节点的元素用-1进行标记

**性能分析：**
- 数组长度为2n，每次循环寻找最小元素需要$O(n)$，故构造函数时间复杂度为$O(n^2)$
- getCode()循环n次，每次从叶节点回溯深度的期望为logn，故平均时间复杂度为$O(nlogn)$，最坏情况为$O(n^2)$

## 树和森林

### 4.1 树的存储实现

**孩子链表示法:** 将**每个节点的孩子组织成一个链表**。因此，树中的的每个节点由两部分组成：存储数据元素值的数据部分和指向孩子链的指针(这不就是用邻接表表示树嘛……)。

**孩子兄弟链表示法:** 孩子兄弟链表示法是最常用的数据结构，它实际上是用二叉树表示一般树的方法。在孩子兄弟链表示法中，节点的形式和二叉树是一致的，但有不同的含义:**左指针指向它第一个孩子，右指针指向它兄弟**。

**双亲表示法:** 由于树上的节点可以有任意多个儿子，确只有一个父亲，故通过指向父节点的指针来将所有节点组织在一起是十分简洁的(类似于哈夫曼树)。在双亲表示法中，每个节点由两部分组成：存储数据元素的数据字段，以及存储父节点位置的父指针字段。双亲表示法中指针是向上的，因此**它对求节点祖先的操作非常方便，但对求指定节点子孙的操作很不方便**。在某些情况下，这种表示法会非常有用。

### 4.2 树的遍历

一般而言，树的遍历有三种方法：
- 前序遍历：先访问根节点，再依次前序遍历每一棵子树
- 后序遍历：先依次后序遍历每一棵子树，再访问根节点
- 层次遍历：先访问根节点，再从左至右逐层访问

### 4.3 树、森林与二叉树的转换

树的孩子兄弟链表示法就是将一棵树表示成二叉树的形态。注意，在这些二叉树表示中，**根节点是没有右儿子的**。反之，将一棵二叉树的左孩子看成第一个儿子，右孩子看作兄弟，可以将一棵二叉树还原为普通的树。

**森林**通常定义为树的集合。将森林转化为树的主要思想是将所有树的树根互相之间看作为兄弟，用根指针的右指针连起来。反过来，将一棵二叉树转化为森林，只要将从根节点右链开始的连续右链全部断开，分解出每一棵二叉树，再将二叉树还原为树即可。


