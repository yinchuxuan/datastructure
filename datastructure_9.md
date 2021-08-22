# 数据结构与算法(九)：常用的图算法(Re:算法导论 & 数据结构：思想与实现)

## 图的遍历

### 1.1 欧拉回路

**欧拉路径：** 如果能再一个图中找到一条路径，使得该路径对图的每一条边恰好经过一次，则这条路径被称为欧拉路径

**欧拉回路：** 起点与终点相同欧拉路径

存在欧拉回路的条件是：**每个节点的度均为偶数**

**EulerCircuit函数的实现:**
```cpp
template<class Vertex, class Edge>
struct EulerNode{
    int NodeNum;
    EulerNode *next;
    EulerNode(int ver):NodeNum(ver),next(NULL){}
};

template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::EulerCircuit(Vertex start){
    EulerNode *beg, *end, *p, *q, *tb, *te;
    int numOfDegree;
    edgeNode *r;
    verNode *tmp;

    if(Edges == 0){
        std::cout<< "不存在欧拉回路" << std::endl;
        return;
    }

    for(int i = 0; i < Vers; i++){
        numOfDegree = 0;
        r = verList[i].head;
        while(r != nullptr){
            numOfDegree++;
            r = r->next;
        }

        if(numOfDegree % 2){
            std::cout<< "不存在欧拉回路" << std::endl;
            return;
        }
    }

    i = find(start);
    tmp = clone();
    EulerCircuit(i, beg, end);

    while(true){
        p = beg;

        while(p->next != NULL){
            if(verList[p->next->NodeNum].head != NULL){
                break;
            }else{
                p = p->next;
            }

            if(p->next == NULL){
                break;
            }

            q = p->next;
            EulerCircuit(q->NodeNum, tb, te);
            te->next = q->next;
            p->next = tb;
            delete q;
        }

        delete[] verList;
        verList = tmp;

        std::cout << "欧拉回路是：" << std::endl;

        while(beg != NULL){
            std::cout << verList[beg->NodeNum].ver << " ";
            p = beg;
            beg = beg->next;
            delete p;
        }

        std::cout << std::endl;
    }
}

template<class Vertex, class Edge>
adjListGraph<Vertex, Edge>::verNode *adjListGraph<Vertex, Edge>::clone() const{
    verNode *tmp = new verNode[Vers];
    edgeNode *p;

    for(int i = 0; i < Vers; i++){
        tmp[i].ver = verList[i].ver;
        p = verList[i].head;

        while(p != NULL){
            tmp[i].head = new edgeNode(p->end, p->weight, tmp[i].head);
            p = p->next;
        }
    }

    return tmp;
}

template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::EulerCircuit(int start, EulerNode *&beg, EulerNode *&end){
    int nextNode;
    beg = end = new EulerNode(start);

    while(verList[start].head != NULL){
        nextNode = verList[start].head->end;
        remove(start, nextNode);
        remove(nextNode, start);
        start = nextNode;
        end->next = new EulerNode(start);
        end = end->next;
    }
}
```

### 1.2 拓扑排序

**拓扑排序**就是把**有向无环图**中的节点按下述规则排序：如果有一条从u到v的路径，那么节点v在拓扑排序的位置必须出现在节点u之后。拓扑排序可以利用**广度优先搜索实现**：首先找到入度为0的节点，将其入队。然后重复下列过程，直到队列为空：从队列中取出队头节点，访问该节点，并从逻辑上删除它以及从它出发的所有边。随后，对它的后继检查入度是否为0.如果入度为0，则表示该节点可以访问，将其入队。

**拓扑排序的实现：**
```cpp
template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::topSort() const{
    std::queue<int> q;
    int *inDegree = new int[Vers];

    for(int i = 0; i < Vers; i++){
        inDegree[i] = 0;
    }

    for(int i = 0; i < Vers; i++){
        EdgeNode *tmp = verList[i].ptr;

        while(tmp->next != nullptr){
            inDegree[find(tmp->next->v)]++;
            tmp = tmp->next;
        }
    }

    for(int i = 0; i < Vers; i++){
        if(inDegree[i] == 0){
            q.push(i);
        }
    }

    std::cout << "拓扑排序为：";
    while(!q.empty()){
        int i = q.top();
        EdgeNode *tmp = verList[i].ptr;

        while(tmp->next != nullptr){
            inDegree[find(tmp->next->v)]--;

            if(inDegree[find(tmp->next->v)] == 0){
                q.push(find(tmp->next->v));
            }

            tmp = tmp->next;
        }

        std::cout <<  verList[i].v << " ";
        q.pop();
    }

    std::cout << std::endl;
}
```

**性能分析：**
- 总时间复杂度$O(|V| + |E|)$

### 1.3 关键路径

从源点到收点为止最长的路径称为**关键路径**，关键路径的长度就是完成项目所需的最短时间。从源点到某个节点v的最长路径称为该节点所代表事件的**最早发生时间**。还可以定义一个**最迟发生时间**，这是在不推迟整个工程完成的前提下，从这个节点出发的活动的最迟开始时间。最迟开始时间和最早开始时间之差称为该活动的**时间余量**。时间余量为0的节点就是**关键活动**。由此可知找出关键路径的关键是找出各事件的最早发生时间和最迟发生时间，这两个时间相等的节点就是关键路径上的节点。寻找各节点的最早发生时间可以用松弛法，从头到尾进行遍历即可。而最晚发生时间恰好相反，可以从尾到 头用松弛法进行遍历寻找。

**寻找关键路径的实现：**
```cpp
template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::criticalPath() const{
    Edge *ee = new Edge[Vers], *le = new Edge[Vers];
    int *top = new int[Vers], *inDegree = new int[Vers];
    queue<int> q;
    int i;
    edgeNode *p;

    for(int i = 0; i < Vers; i++){
        inDegree[i] = 0;

        for(p = verList[i].ptr; p->next != nullptr; p = p->next){
            ++inDegree[find(p->v)];
        }
    }

    for(i = 0; i < Vers; i++){
        if(inDegree[i] == 0){
            q.push(i);
        }
    }

    i = 0;
    while(!q.empty()){
        top[i] = q.top();

        for(p = verList[top[i]].ptr; p->next != nullptr; p = p->next){
            if(--inDegree[find(p->v) == 0]){
                q.push(find(p->v));
            }
        }

        q.pop();
        i++;
    }

    for(i = 0; i < Vers; i++){
        ee[i] = 0;
    }

    for(i = 0; i < Vers; i++){
        for(p = verList[top[i]].ptr; p->next != nullptr; p = p->next){
            if(ee[find(p->v)] < ee[top[i]] + p->w){
                ee[find(p->v)] = ee[top[i]] + p->w;
            }
        }
    }

    for(i = 0; i < Vers; i++){
        le[i] = ee[Vers - 1];
    }

    for(i = Vers - 1; i >= 0; i--){
        for(p = verList[top[i]].ptr; p->next != nullptr; p = p->next){
            if(le[find(p->v)] - p->w < le[top[i]]){
                le[top[i]] = le[find(p->v)] - p->w;
            }
        }
    }

    for(i = 0; i < Vers; i++){
        if(le[top[i]] == ee[top[i]]){
            std::cout << "(" << verList[top[i]].v << ", " << ee[top[i]] << ")");
        }
    }
}
```

## 最小生成树

一个无向连通图的**生成树**就是该图的一个**极小连通子图**，它包括图中的全部节点，并且有尽可能少的边。加权无向连通图的众多生成树中权值之和最小的生成树称为**最小生成树**。在通常情况下，具有n个节点的无向连通图之间最多有n(n-1)/2条边，而使得n个顶点之间互相连通的最小生成树，应该包含它的n个节点和其中的n-1条边。这样，寻找最小生成树的问题九转化为**如何快速地从边地集合中选出n-1条满足条件的边**。但是这n-1条边不一定是权值最小的n-1条边，因为它们还需要保证n个节点可以连通。

### 2.1 Kruskal算法

Kruskal算法的基本思想是**从边出发**。初始时，生成树只包含了n个节点，然后依次将边加入到集合中。在加入边时，优先选择权值最小的边，如果加入该边不会导致子图中出现回路，则加入；否则考虑下一条边，直到所有的节点之间都能连通。

**Kruskal算法的实现：**
```cpp
struct edge{
    int beg, end;
    Edge w;
    bool operator<(const edge &p) const{
        return w < rp.w;
    }
}

template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::Kruskal() const{
    int edgesAccepted = 0, u, v;
    edgeNode *p;
    edge e;
    DisjointSet ds(Vers);
    priorityQueue<edge> pq;

    for(int i = 0; i < Vers; i++){
        for(p = verList[i].ptr; p->next != nullptr; p = p->next){
            if(i < p->end){
                e.beg = i;
                e.end = p->end;
                e.w = p->weight;
                pq.push(e);
            }
        }
    }

    while(edgeAccepted < Vers - 1){
        e = pq.top();
        u = ds.find(e.beg);
        v = ds.find(e.end);

        if(u != v){
            edgesAccepted++;
            ds.Union(u, v);
            std::cout << '(' << verList[e.beg].ver << ',' << verList[e.end].ver << ') ';
        }

        pq.pop();
    }
}
```

**性能分析：**
- 生成优先队列需要$O(|E|log|E|)$， 归并循环的时间复杂度为$O(|E|log|V|)$，故Kruskal算法的时间复杂度为$O(|E|log|E|)$

### 2.2 Prim算法

Prim算法的思想是**从顶点出发**。初始时，生成树中一个节点都没有，然后依次将每个节点都加入生成树。在加入节点时，优先加入两个端点分别位于U和V-U的代价最小的边(u,v)，其中U为生成树的节点集。

**Prim函数的实现：**
```cpp
template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::Prim(Edge noEdge) const{
    int *flag = new int[Vers];
    edgeNode * tree;
    Vertex v1, v2;
    int lowCost = 10000000;

    for(int i = 1; i < Vers; i++{
        flag[i] = false;
    }

    flag[0] = true;

    while(true){
        for(int i = 0; i < Vers; i++){
            if(flag[i]){
                edgeNode *tmp = verList[i].ptr;

                while(tmp->next != nullptr){
                    if(!flag[find(tmp->v)] && lowCost < tmp->w){
                        lowCost = tmp->w;
                        v1 = verList[i].v
                        v2 = tmp->v;
                    }
                }
            }
        }

        if(lowCost == 10000000){
            break;
        }

        lowCost = 10000000;

        flag[find(v2)] = true;
        std::cout << "(" << v1 << "," << v2 << ")" << std::endl;
    }
}
```

## 最短路径问题

### 3.1 单源最短路径

单源最短路径就是给出一个加权图和图上的一个节点s，找出s到图中每一节点的最短路径。对于非加权图，可以采用广度优先搜索，按层访问图中的所有节点，然后给每一层的节点标记距离，直至所有节点都被访问。

求加权图的单元最短路径要比非加权图复杂一些，因为按广度优先搜索先搜索到的路径不一定时最短路径。解决加权图的单源最短路径问题的最常用算法是**Dijkstra算法**，它采用**贪婪法**，工作方式类似于非加权图的单源最短路径问题算法。它仍然在每个节点保存一个距离，不过与之前不同的是，这个距离并不是最终的距离，它实际上是指**尝试了一部分路径后得到的最短距离**。

在Dijkstra算法中，需要保存一个节点集S，S中的节点为已经找到最短路径的节点。在加入节点时，加入在V-S中路径最短的节点，然后更新该节点周围的距离情况。按照这种方式，一直执行到V中所有节点都加入到S中。

**Dijkstra
```cpp
template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::printPath(int start, int end, int prev[]) const{
    if(start == end){
        std::cout << verList[start].v;
        return;
    }

    printPath(start, prev[end], prev);
    std::cout << "-" << verList[end].v;
}

template<class Vertex, class Edge>
void adjListGraph<Vertex, Edge>::dijsktra(Vertex start, Edge noEdge) const{
    int *distance = new Vertex[Vers];
    int *prev = new int[Vers];
    bool *known = new bool[Vers];

    int u, sNo, i, j;
    edgeNode *p;
    Edge min;

    for(i = 0; i < Vers; i++){
        known[i] = false;
        distance[i] = noEdge;
    }

    sNo = find(start);

    distance[sNo] = 0;
    prev[sNo] = sNo;
    p = verList[sNo].ptr;

    while(p->next != nullptr){
        distance[find(p->v)] = p->w;
        p = p->next;
    }

    for(i = 1; i < Vers; i++){
        min = noEdge;

        for(j = 0; j < Vers; j++){
            if(!known[j] && distance[j] < min){
                min = distance[j];
                u = j;
            }
        }

        known[u] = true;
        for(p = verList[u].ptr; p->next != nullptr; p = p->next){
            if(distance[find(p->v)] > distance[u] + p->w){
                distance[find(p->w)] = distance[u] + p->w;
                prev[p->v] = u;
            }
        }

        for(i = 0; i < Vers; i++){
            std::cout << "从" << start << verList[i].v << "的路径为：" << std::endl;
            printPath(sNo, i, prev);
            std::cout << " 长度为：" << distance[i] << std::endl;
        }
    }
}
```

**性能分析：**
- 时间复杂度为$O(|V|^2)$

### 3.2 所有顶点对的最短路径

利用Dijkstra算法可以解决所有节点对的最短路径问题，只需要对每个节点运行一下Dijkstra算法即可得到，所需要的总时间是$O(|V|^3)$。而下面介绍的Floyd算法要更直接一些，所需的总时间也是$O(|V|^3)$。

Floyd算法的基本思想是依次将每个节点作为每条路径上的一个中间节点，看看从起始点到中间节点，再从中间节点到终止节点的距离是不是比已知的距离短。如果是，则更新距离值。

