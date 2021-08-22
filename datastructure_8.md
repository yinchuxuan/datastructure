# 数据结构与算法(八)：基本的图算法(Re:算法导论 & 数据结构：思想与实现)

## 图的概念与定义

图由**一个节点集**和**连接各顶点的边集**组成，通常被表示成一个二元组$G=(V,E)$，其中V表示节点集，E表示边集。每个节点是一个数据元素。**边是数据元素之间的关系**。若边是有方向的，则称为**有向图**。有向图的边用$<u,v>$表示，v是u的直接后继，u是v的直接前驱。无向图的边常常用$(u,v)$表示。有时边还有第三个属性，称为边的**代价**或者**权值**，用来表达经过这条边所花费的代价。这样的图称为**加权图**。若$|E|$远远小于$|V^2|$，则称这样的图为**稀疏图**。

**图的基本术语：**

- 邻接：若$(V_i, V_j)$是无向图中的一条边，则称$V_i$和$V_j$是邻接的
- 度：在无向图中，节点的度是与该节点关联的边数。在有向图中，度被分为**入度**和**出度**
- 子图：假设有两个图$G=(V,E)$和$G'=(V',E')$，若$V' \subseteq V$，$E' \subseteq E$，则称$G'$是$G$的子图
- 路径和路径长度：路径是指图中由边连接而成的节点序列。如果u和v之间有一条路径，则称u和v之间是**连通**的。**非加权的路径长度**就是组成路径的边数，**加权路径的长度**指路径上所有边的权值之和。如果一条路径上的所有节点，除了起始节点和终止节点可能相同外，其余的节点都不同，则称该路径为**简单路径**。一个**环**或**回路**是一条**简单路径**，其起始节点和终止节点相同，且路径长度至少为1
- 连通图和连通分量：若一个无向图$G$的任意两个节点之间都是连通的，则称$G$为连通图。每一个**极大连通的子图**称为连通分量
- 强连通图和强连通分量：若一个有向图$G$的任意两个节点之间都是连通的，则称$G$为强连通图。每个**极大的强连通子图**称为强连通分量。如果一个有向图$G$不是强连通的，但是把它看成无向图时是连通的，则称该有向图为**弱连通图**
- 完全图：每两个节点之间都有边的无向图称为**无向完全图**，每两个节点u和v之间都有两条边$<u,v>$和$<v,u>$的有向图称为**有向完全图**
- 生成树：生成树是无向连通图的**极小连通子图**

**图的基本运算：**
- 构造一个图
- 判断两节点之间是否有边存在
- 在图中添加或删除某一条边
- 返回图中的节点数或边数

```cpp
template<class Vertex, class Edge>
class Graph{
    private:
        int Vers, Edges;
    public: 
        virtual void insert(Vertex x, Vertex y, Edge w) = 0;
        virtual void remove(Vertex x, Vertex y) = 0;
        virtual bool exist(Vertex x, Vertex y) const = 0;
        int numOfVer() const{
            return Vers;
        }
        int numOfEdge() const{
            return Edges;
        }

};
```

## 图的存储

### 2.1邻接矩阵表示法

存储一个图需要存储节点和边。存储节点可以用一个节点类型的数组，存储边的一种方式是邻接矩阵，有向图和无向图都可以用邻接矩阵来表示。设有向图或无向图有n个节点，则可以用n行n列的布尔矩阵A表示。如果编号为i的节点到标号为j的节点之间存在一条自节点i到节点j的有向边或无向边，那么$A[i][j]=1$。另外，通常设$A[i][j]=0$。

邻接矩阵也可以用来存储加权图。但与非加权图的邻接矩阵不同，该矩阵不再是布尔矩阵，而是一个整型的或实型的矩阵。矩阵的类型取决于权值的类型。若i至j没有边，则$A[i][j]$为空或者其它标志，例如$\infty$。如果$i=j$，则$A[i][j]=0$

**基于邻接矩阵的图类定义：**
```cpp
template<class Vertex, class Edge>
class adjMatrixGraph:public Graph{
    private:
        Edge **edge;
        Vertex *ver;
        Edge noEdge;

        int find(Vertex x) const{
            for(int i = 0; i < Vers; i++){
                if(ver[i] == x){
                    return i;
                }
            }
        }
    public:
        adjMatrixGraph(int size, const Vertex d[], const Edge noEdgeFlag);
        void insert(Vertex x, Vertex y, Edge w);
        void remove(Vertex x, Vertex y);
        void exist(Vertex x, Vertex y) const;
        ~adjMatrixGraph();
};
```

**基于邻接矩阵的图类的实现：**
```cpp
template<class Vertex, class Edge>
adjMatrixGraph::adjMatrixGraph(int size, const Vertex d[], const Edge noEdgeFlag):Vers(size),Edges(0),noEdge(noEdgeFlag){
    ver = new Vertex[size];

    for(int i = 0; i < size; i++){
        ver = d[i]
    }

    edge = new *Edge[size];

    for(int i = 0; i < size; i++){
        edge[i] = new Edge[size];

        for(int j = 0; j < size; j++){
            if(i == j){
                edge[i][j] = 0;
            }else{
                edge[i][j] = noEdge;
            }
        }
    }
}

template<class Vertex, class Edge>
adjMatrixGraph::~adjMatrixGraph(){
    delete[] ver;

    for(int i = 0; i < Vers; i++){
        delete[] edge[i];
    }

    delete[] edge;
}

template<class Vertex, class Edge>
void adjMatrixGraph::insert(Vertex x, Vertex y, Edge w){
    edge[find(x)][find(y)] = w;
    Edge++;
}

template<class Vertex, class Edge>
void adjMatrixGraph::remove(Vertex x, Vertex y){
    edge[find(x)][find(y)] = noEdge;
    Edge--;
}

template<class Vertex, class Edge>
bool adjMatrixGraph::exist(Vertex x, Vertex y) const{
    return edge[find(x)][find(y)] != noEdge;
}
```

### 2.2邻接表表示法

对于稀疏图来说，邻接矩阵浪费了许多空间。对于稀疏图，一个更好的方法是使用**邻接表**。邻接表存储每一条存在的边，并不为不存在的边预留空间。因此只需要线性的空间量。邻接表将每一个节点的邻接节点组成一组链表，链表的每个节点表示一条边。

在邻接表示法中，保存一个图同样分为两部分：保存节点和保存边。节点集用一个数组表示，数组的每个元素由两部分组成：节点值和指向该节点对应链表的首地址。

**邻接表类的定义：**
```cpp
template<class Vertex, class Edge>
class adjListGraph:public Graph{
    private:
        struct EdgeNode{
            Vertex v;
            Edge w;
            EdgeNode *next;

            EdgeNode():next(nullptr){}

            EdgeNode(Vertex V, Edge W, EdgeNode *n = nullptr):v(V),w(W),next(n){}
        };

        struct VerNode{
            Vertex v;
            EdgeNode *ptr;

            VerNode(EdgeNode p):v(V),ptr(p){}
        };

        VerNode *verList;

        int find(Vertex v) const{
            for(int i = 0; i < Vers; i++){
                if(verList[i].v == v){
                    return i;
                }
            }
        }

    public:
        adjListGraph(int size, const Vertex d[]);
        void insert(Vertex x, Vertex y, Edge w);
        void remove(Vertex x, Vertex y);
        bool exist(Vertex x, Vertex y) const;
        ~adjListGraph();
};
```

**邻接表类的实现：**
```cpp
template<class Vertex, class Edge>
adjListGraph::adjListGraph(int size, const Vertex d[]):Vers(size),Edges(0){
    verList = new VerNode[size];

    for(int i = 0; i < size; i++){
        verList[i].v = d[i];
        verList[i].ptr = new EdgeNode;
    }
}

template<class Vertex, class Edge>
adjListGraph::~adListGraph(){
    for(int i = 0; i < Vers; i++){
        EdgeNode *tmp = verList[i].ptr;

        while(tmp->next != nullptr){
            EdgeNode *p = tmp;
            tmp = tmp->next;
            delete p;
        }

        delete verList[i].ptr;
    }

    delete[] verList;
}

template<class Vertex, class Edge>
void adjListGraph::insert(Vertex x, Vertex y, Edge w){
    EdgeNode *tmp = verList[find(x)].ptr;

    while(tmp->next != nullptr){
        tmp = tmp->next;
    }

    tmp->next = new EdgeNode(y, w);
    }
}

template<class Vertex, class Edge>
void adjListGraph::remove(Vertex x, Vertex y){
    EdgeNode *tmp = verList[find(x)].ptr;

    while(tmp->next != nullptr tmp->next->v != y){
        tmp = tmp->next;
    }

    EdgeNode *p = tmp->next;

    if(p != nullptr){
        tmp->next = p->next;
        delete p;
    }
}

template<class Vertex, class Edge>
bool adjListEdge::exist(Vertex x, Vertex y) const{
    EdgeNode *tmp = verList[find(x)].ptr;

    while(tmp->next != nullptr){
        if(tmp->next->v == y){
            return true;
        }

        tmp = tmp->next;
    }

    return false;
}
```

## 图的遍历

在图的基本操作中，**遍历**是最基本操作。对有向图和无向图进行遍历时按照某种次序系统地访问图中的所有节点，并且使得每个节点**只能被访问一次**。由于图中的某个节点可能和图中的多个节点邻接并且存在回路，因此在图中访问一个节点u后，有可能再次返回到节点u，为了避免重复访问已经访问过的节点，在图的遍历中，通常**对已经访问过的节点进行标记**。

### 3.1深度优先搜索

深度优先搜索访问方式类似于树的前序遍历，它可以用递归的方式定义如下：

- 选中第一个被访问的节点
- 对节点作已访问的标志
- 依次从节点的未被访问过的第一个、第二个、第三个……邻接节点出发，进行深度优先搜索
- 如果还有节点未被访问，则选中一个起始节点，转向第二步
- 所有节点均被访问，则结束

每个深度优先搜索过程都对应着一棵树，这些树被称为该图所对应的深度优先搜索树。

**dfs函数的实现：**

```cpp
template<class Vertex, class Edge>
void adjListGraph::dfs() const{
    int *flag = new int[Vers];

    for(int i = 0; i < Vers; i++){
        flag[i] = 0;
    }

    for(int i = 0; i < Vers; i++){
        dfs(verList[i].v, flag);
    }
}

template<class Vertex, class Edge>
void adjListGraph::dfs(const Vertex &v, int *flag){
    int i = find(v);

    if(flag[i] == 0){
        cout << v;
        flag[i] = 1;

        EdgeNode *tmp = verList[i].ptr;

        while(tmp->next != nullptr){
            dfs(tmp->next->v, flag);
            tmp = tmp->next;
        }
    }
}
```

**性能分析：**
- dfs函数将对所有的节点和边访问，因此它的时间代价和节点数$|V|$及边数$|E|$有关，即$O(|E| + |V|)$

### 3.2广度优先搜索

广度优先搜索类似于树的层次遍历，它的访问方式如下：、
- 选中第一个被访问的节点
- 对节点作已访问的标志
- 依次访问已访问节点的未被访问过的第一个、第二个、第三个、……第m个邻接节点，转向第三步
- 如果还有未被访问的节点，则再选择一个起始节点，转向第二步
- 所有节点都被访问到，结束

如上所述，在广度优先搜索中需要记住每个可以被访问的节点，然后依次访问这些节点。对于每个被访问的节点，将它二点后继节点标记为可访问。可以用一个队列来记录这些可以被访问的节点。

**bfs函数的实现：**
```cpp
template<class Vertex, class Edge>
void adjListGraph() const{
    queue<Vertex> q;
    int *flag = new int[Vers];

    for(int i = 0; i < Vers; i++){
        flag[i] = 0;
    }

    for(int i = 0; i < Vers; i++){
        if(flag[i] == 0){
            q.push(verList[i]);
        }

        while(!q.empty()){
            Vertex v = q.top();
            cout << v;
            EdgeNode *tmp = verList[find(v)].ptr;

            while(tmp->next != nullptr){
                if(flag[find(tmp->next->v)] == 0){
                    q.push(tmp->next->v);
                }
            }

            q.pop();
        }
    }
}
```

**性能分析：**
- 在广度优先搜索中，每个节点都入队一次。而对于每个出队的节点，需要查看它的所有邻接节点。假如图是用邻接表存储的，查看所有的邻接节点需要$O(|E|)$的时间，每个节点入队一次需要$O(|V|)$的时间，因此广度优先搜索的时间复杂度为$O(|E|+|V|)$

