# 数据结构与算法(五)：二叉查找树和AVL树

## 动态查找表

在大多数场合下，我们需要对集合中方的元素进行增、删的操作。既支持查找操作，又支持增删操作的集合称为**动态查找表**。

**动态查找表的抽象类定义:**

```cpp
template<class Key, class Other>
class dynamicSearchTree{
    public:
        virtual Set<Key, Other> *find(const Key &x) const = 0;
        virtual void insert(const Set<Key, Other> &x) = 0;
        virtual void remove(const Key &x) = 0;
        virtual ~dynamicSearchTable(){}
};
```

由于既需要大量的查找又需要动态地插入删除，所以用于处理线性结构的数据结构都**不适合**处理动态查找表。一棵**结构良好**的二叉树可以以$O(logN)$的时间复杂度进行插入和删除。如果该树还具有一定的**有序性**，则可以做到以$O(logN)$的时间复杂度支持查找。用于处理动态查找表的树简称为**查找树**。另一种动态查找表的实现是**散列表**，它是专门用于集合查找的数据结构。

## 二叉查找树

### 2.1 二叉查找树的定义

**定义:** 二叉查找树也称二叉排序树。它或者是一棵空树，或者是一棵同时满足以下条件的二叉树：
- 若左子树不为空，则左子树中所有元素键值均比根节点键值小
- 若右子树不为空，则右子树中所有元素键值均比根节点键值大
- 它的左右子树也是二叉查找树
  
由二叉树的有序性很容易想到其查找过程。查找从根节点开始，然后根据比较的结果决定是到左子树还是右子树中寻找。**中序遍历**一棵二叉树所得到的访问序列是按键值**依次递增**的，所以二叉查找树也能用于排序一组数据。

**基本运算：**
- 根据键值查找某一个元素find()
- 插入一个键值对insert()
- 根据键值删除某一个元素remove()

**二叉查找树类的定义:**
```cpp
template<class Key, class Other>
class BinarySearchTree:public dynamicSearchTable<Key, Other>{
    private:    
        struct BinaryNode{
            Set<Key, Other> data;
            BinaryNode *left;
            BinaryNode *right;

            BinaryNode(const Set<Key, Other> &thedata, BinaryNode *lt=NULL, BinaryNode *rt=NULL):data(thedata),left(lt),right(rt){}
        };
        BinaryNode *root;

    public:
        BinarySearchTree();
        ~BinarySearchTree();
        Set<Key,Other> *find(const Key &x) const;
        void insert(const Set<Key,Other> &x);
        void remove(const Key &x);

    private:
        void insert(const Set<Key,Other> &x, BinaryNode *&t);
        void remove(const Key &x, BinaryNode *&t);
        Set<Key,Other> *find(const Key &x, BinaryNode *t) const;
        void makeEmpty(BinaryNode *t);
};
```

### 2.2 二叉查找树的实现

```cpp
template<class Key, class Other>
BinarySearchTree<Key, Other>::BinarySearchTree(){
    root = NULL;
}

template<class Key, class Other>
BinarySearchTree<Key, Other>::~BinarySearchTree(){
    makeEmpty(root);
}

template<class Key, class Other>
Set<Key, Other>* BinarySearhTree<Key, Other>::find(const Key &x) const{
    return find(x, root);
}

template<class Key, class Other>
void BinarySearchTree<Key, Other>::insert(const Set<Key, Other> &x){
    insert(x, root);
}

template<class Key, class Other>
void BinarySearchTree<Key, Other>::remove(const Key &x){
    remove(x, root);
}

template<class Key, class Other>
void BinarySearchTree<Key, Other>::insert(const Set<Key, Other> &x, BinaryNode *&t){
    if(t == NULL){
        t = BinaryNode(x);
    }

    if(t->data.key < x.key){
        insert(x, t->left);
    }

    if(t->data.key > x.key){
        insert(x, t->right);
    }

    if(t->data.key == x.key){
        t->data.other = x.other;
    }
}

template<class Key, class Other>
void BinarySearchTree<Key, Other>::remove(const Key &x, BinaryNode *&t){
    if(t != NULL){ 
        if(t->data.key < x){
            remove(t->left, x);
        }

        if(t->data.key > x){
            remove(t->right, x);
        }

        if(t->data.key == x){
            if(t->left != NULL && t->right != NULL){
                BinaryNode *tmp = t->left;

                while(tmp->right != NULL){
                    tmp = tmp->right;
                }

                std::swap(t->data, tmp->data);
                remove(t->data.key, t->left);
            }else{
                BinaryNode *tmp = t;

                if(t->left != NULL){
                    t = t->left;
                }else{
                    t = t->right;
                }

                delete tmp;
            }
        }
    }
}

template<class Key, class Other>
Set<Key, Other>* BinarySearchTree<Key, Other>::find(const Key &x, BinaryNode *&t) const{
    if(t == NULL){
        return NULL;
    }

    if(t->data.key == x){
        return t;
    }

    if(t->data.key < x){
        find(x, t->left);
    }

    if(t->data.key > x){
        find(x, t->right);
    }
}

template<class Key, class Other>
void BinarySearchTree<Key, Other>::makeEmpty(BinaryNode *&t){
    if(t->left != NULL){
        makeEmpty(t->left);
    }

    if(t->right != NULL){
        makeEmpty(t->right);
    }

    delete t;
}
```

**Remark:**
- 注意指针的引用传递
- 注意删除时的三种情况
  
**性能分析:**
- 如果二叉树是平衡的，则所有操作均为对数级
- 在最坏的情况下，二叉查找树会退化成为单链表，此时所有操作的时间复杂度均为$O(N)$
- 平均情况下，所有操作的时间复杂度仍为$O(logN)$

## AVL树

### 3.1 AVL树的定义

为了避免二叉查找树的退化，我们需要一种方法使得它**尽可能平衡**。最基本的想法是使得这棵树尽可能矮胖，满二叉树或者完全二叉树是最好的，但想维持这种平衡很困难。AVL树就是一个**退而求其次的方案**，它将满二叉树的平衡条件稍微放宽一些。满二叉树要求左右子树的高度相同，而AVL树则要求**左右子树的高度相差不超过1**。AVL树用**平衡因子**来衡量节点的平衡度。节点的平衡度就是**左子树的高度减去右子树的高度**，所以AVL树中每个节点的平衡因子只能是+1，0或-1。

AVL树的平衡条件表明了树有**对数级的高度**。可以证明，一棵由N个节点组成的AVL树，它的高度H小于等于1.44log(N+1)-0.328。所以在最坏情况下，AVL树的高度至多比满二叉树的高度增加44%。

**AVL树类的定义:**
```cpp
template<class Key, class Other>
class AvlTree:public dynamicSearchTable<Key, Other>{
    struct AvlNode{
        Set<Key, Other> data;
        AvlNode *left;
        AvlNode *right;
        int height;

        AvlNode(const Set<Key, Other> &element, AvlNode *lt, AvlNode *rt, int h = 1):data(element), left(lt), right(rt), height(h){}
    };

    private:
        AvlNode *root;

    public:
        AvlTree(){root = NULL}
        ~AvlTree(){makeEmpty(root)}
        Set<Key, Other> *find(const Key &x) const;
        void insert(const Set<Key, Other> &x);
        void remove(const Key &x);

    private:
        void insert(const Set<Key, Other> &x, AvlNode *&t);
        bool remove(AvlNode *&t);
        void makeEmpty(AvlNode *&t);
        int height(AvlNode *t) const{
            if(t == NULL){
                return 0;
            }

            return t->height
        }
        void LL(AvlNode *&t);
        void LR(AVLNode *&t);
        void RL(AvlNode *&t);
        void RR(AvlNode *&t);
        int max(int a, int b){
            if(a > b){
                return a;
            }

            return b;
        }
        bool adjust(AvlTree *&t, int subTree);
};

template<Key, Other>
Set<Key, Other>* AvlTree<Key, Other>::find(const Key &x) const{
    AvlNode *t = root;

    while(t != NULL && t->data.key != x){
        if(t->data.key > x){
            t = t->right;
        }else{
            t = t->left;
        }
    }

    if(t == NULL){
        return NULL;
    }

    return static_cast<Set<Key, Other>*>(t);
}

template<Key, Other>
void AvlTree<Key, Other>::insert(const Set<Key, Other> &x){
    insert(x, root);
}

template<Key, Other>
void AvlTree<Key, Other>::remove(const Key &x){
    remove(x, root);
}

template<Key, Other>
void AvlTree<Key, Other>::insert(const Set<Key, Other> &x, AvlNode *&t){
    if(t != NULL){
        if(x.key < t->data.key){
            insert(x, t->left);

            if(height(t->left) - height(t->right) == 2){
                if(x.key < t->left->data.key){
                    LL(t);
                }else{
                    LR(t);
                }
            }
        }

        if(x.key > t->data.key){
            insert(x, t->right);

            if(height(t->right) - height(t->left) == 2){
                if(x.key > t->right->data.key){
                    RR(t);
                }else{
                    RL(t);
                }
            }
        }
    }else{
        t = new AvlNode(x, NULL, NULL);
    }

    t->height = max(t->left->height, t->right->height) + 1;
}

template<Key, Other>
bool AvlTree<Key, Other>::remove(const Key &x, Avl *&t){
    if(t != NULL){
        if(x. key < t->data.key){
            if(remove(x, t->left)){
                return true;
            }

            return adjust(t, 0);
        }

        if(x.key > t->data.key){
            if(remove(x, t->right)){
                return true;
            }

            return adjust(t, 1);
        }

        if(x.key == t->data.key){
            if(t->left != NULL && t->right != NULL){
                AvlNode *tmp = t->left;

                while(tmp->right != NULL){
                    tmp = tmp->right;
                }

                std::swap(t->data, tmp->data);

                if(remove(t->data, t->left){
                    return true;
                }

                return adjust(t, 0);

            }else{
                AvlNode *tmp = t;

                if(t->left != NULL){
                    t = t->left
                }else{
                    t = t->right;
                }

                delete tmp;
                return false;
            }
        }
    }else{
        return true;
    }
}

template<Key, Other>
void makeEmpty(AvlNode *&t){
    if(t != NULL){
        makeEmpty(t->left);
        makeEmpty(t->right);
        delete t;
        t = NULL;
    }
}

template<Key, Other>
void AvlTree<Key, Other>::LL(AvlNode *&t){
    AvlNode *tmp = t->left->right;
    t->left->right = t;
    t = t->left;
    t->right->left = tmp;
    tmp->height = max(height(tmp->left), height(tmp->right)) + 1;
    t->height = max(height(t->left), height(t->right)) + 1;
}

template<Key, Other>
void AvlTree<Key, Other>::LR(AvlNode *&t){
    RR(t->left);
    LL(t);
}

template<Key, Other>
void AvlTree<Key, Other>::RR(AvlNode *&t){
    AvlNode *tmp = t->right->left;
    t->right->left = t;
    t = t->right;
    t->left->right = tmp;
    tmp->height = max(height(tmp->left), height(tmp->right)) + 1;
    t->height = max(height(t->left), height(t->right)) + 1;
}

template<Key, Other>
void AvlTree<Key, Other>::RL(AvlNode *&t){
    LL(t->right);
    RR(t);
}

template<Key, Other>
bool AvlTree<Key, Other>::adjust(AvlNode *&t, int subTree){
    if(!subTree){
        if(height(t->left) == height(t->right)){
            t->height--;
            return false;
        }

        if(height(t->right) - height(t->left) == 1){
            return true;
        }

        if(height(t->right) - height(t->left) == 2){
            if(height(t->right->right) > height(t->right->left)){
                RR(t)

                if(height(t->right) == height(t->left)){
                    return false;
                }

                return true;
            }
            LR(t)
            return false;
        }
  }else{
      if(height(t->left) == height(t->right)){
            t->height--;
            return false;
        }

        if(height(t->left) - height(t->right) == 1){
            return true;
        }

        if(height(t->left) - height(t->right) == 2){
            if(height(t->left->left) > height(t->left->right)){
                LL(t)

                if(height(t->left) == height(t->right)){
                    return false;
                }

                return true;
            }
            RL(t)
            return false;
        }
  }
}
```

**Remark:**
- 在插入和删除时都要记得修改高度
- 删除时直接进行数据交换
- 删除时利用递归来访问父节点，调整平衡