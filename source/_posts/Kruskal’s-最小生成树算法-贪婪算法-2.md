---
title: Kruskal’s 最小生成树算法 | 贪婪算法-2
date: 2019-07-02 19:21:56
tags:
---

原文：[Kruskal’s Minimum Spanning Tree Algorithm | Greedy Algo-2](https://www.geeksforgeeks.org/kruskals-minimum-spanning-tree-algorithm-greedy-algo-2/)

*什么是最小生成树？*
给定一个连通的无向图，该图的生成树是一个子图，它是一棵将所有顶点连接在一起的树。单个图可以有许多不同的生成树。一个加权连通无向图的最小生成树（MST）或最小权重生成树是指其权重小于或等于所有其他生成树的权重。生成树的权重是树上每条边（edge）的权重的总和。

*最小生成树有多少条边？*
有 （V-1）条边，其中 V 是图的顶点的个数。

*最小生成树有什么应用？*
请看[这个](https://www.geeksforgeeks.org/applications-of-minimum-spanning-tree/)

下面是使用 Kruskal’s 算法找出 MST 的步骤：
1. 按权重的非降序对所有边排序。
2. 选取最小的边。检验当前生成树是否产生环（cycle），如果没有则选择，否则不选取。
3. 重复步骤2 直到生成树有（V-1）条边。

步骤2 可以使用联合查找算法（[Union-Find algorithm](https://www.geeksforgeeks.org/union-find/)）检查是否有环。我们建议先查看一下文章：
[Union-Find Algorithm | Set 1 (Detect Cycle in a Graph)](https://www.geeksforgeeks.org/union-find/)
[Union-Find Algorithm | Set 2 (Union By Rank and Path Compression)](https://www.geeksforgeeks.org/union-find-algorithm-set-2-union-by-rank/)

这个算法是贪婪算法。贪婪选择的策略是选取权重最小的边，且不会使当前构建的 MST 产生环。让我们通过一个例子来理解它：看看下面的输入图。

![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-0.jpg)

该图包括 9 个顶点和 14 个边。所以最小生成树将有（9-1）= 8 条边。
```
After sorting:
Weight   Src    Dest
1         7      6
2         8      2
2         6      5
4         0      1
4         2      5
6         8      6
7         2      3
7         7      8
8         0      7
8         1      2
9         3      4
10        5      4
11        1      7
14        3      5
```
现在一步步地从排好序的数组里选取所有边。
1. 选取边 7-6：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-1.jpg)
2. 选取边 8-2：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-2.jpg)
3. 选取边 6-5：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-3.jpg)
4. 选取边 0-1：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-4.jpg)
5. 选取边 2-5：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-5.jpg)
6. 选取边 8-6：由于包含此边会导致环，所以不选取它。
7. 选取边 2-3：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-6.jpg)
8. 选取边 7-8：由于包含此边会导致环，所以不选取它。
9. 选取边 0-7：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/Fig-7.jpg)
10. 选取边 1-2：由于包含此边会导致环，所以不选取它。
11. 选取边 3-4：没有产生环，则包含它。
![](https://www.geeksforgeeks.org/wp-content/uploads/fig8new.jpeg)

直到包含的变得数目等于（V-1），算法结束。

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 有权边
struct Edge
{
    int src, dest, weight;
};

// 连通、无向、有权图
sturct Graph
{
    // V 顶点数目，E 边数目
    int V, E;
    // 图中的所有边
    // 当图为无向图时，从 src 到 dest 还是从 dest 到 src 都只算一条边
    struct Edge* edge;
};

// 生成一个有 V 个顶点和 E 条边的图
struct Graph* createGraph(int V, int E)
{
    struct Graph* graph = new Graph;
    graph->V = V;
    graph->E = E;
    
    graph->edge = new Edge[E];
    
    return graph;
}

// 联合查找的 subset
struct subset
{
    int parent;
    int rank;
}

// 查找集合中元素 i
// 使用路径压缩技巧
int find(struct subset subsets[], int i)
{
    // 查找根节点 root，然后使 i 的父节点为 root
    // 路径压缩
    if (subsets[i].parent != i)
        subsets[i].parent = find(subsets, subsets[i].parent);
    return subsets[i].parent;
}

// 组合两个集合 x 和 y
// 根据 rank 进行联合
void Union(struct subset subsets[], int x, int y)
{
    int xroot = find(subsets, x);
    int yroot = find(subsets, y);
    
    // 将 rank 较低的树附加到 rank 较高的树下
    if (subsets[xroot].rank < subsets[yroot].rank)
        subsets[xroot].parent = yroot;
    else if (subsets[xroot].rank > subsets[yroot].rank)
        subsets[yroot].parent = xroot;
        
    // 如果 rank 相同，则使其中一个为 root，且其 rank 增加 1
    else
    {
        subsets[yroot].parent = xroot;
        subsets[xroot].rank++;
    }
}

int myComp(const void* a, const void* b)
{
    struct Edge* a1 = (struct Edge*)a;
    struct Edge* b1 = (struct Edge*)b;
    
    return a1->weight > b1->weight;
}

void KruskalMST(struct Graph* graph)
{
    int V = graph->V;
    struct Edge result[V];
    int e = 0;
    int i = 0;
    
    // 第一步：根据权值按非降序对所有边进行排序。
    // 排序不能改变原本的图结构
    qsort(graph->edge, graph->E, sizeof(graph->edge[0]), myComp);
    
    // 分配用于创建 V 个元素的 subsets 的内存
    struct subset *subsets = 
        (struct subset*) malloc( V * sizeof(struct subset) );
        
    // 初始化 V subsets 的每个元素
    for (int v = 0; v < V; ++v)
    {
        subsets[v].parent = v;
        subsets[v].rank = 0;    
    }
    
    while (e < V -1)
    {
        // 第二步：选取最小的边。
        struct Edge next_edge = graph->edge[i++];
        
        int x = find(subsets, next_edge.src);
        int y = find(subsets, next_edge.dest);
        
        // 如果选取这条边不会造成环，
        // 则包含它到结果数组里，然后增加结果数组的下标
        if (x != y)
        {
            result[e++] = next_edge;
            Union(subsets, x, y);
        }
    }
    
    printf("Following are the edges in the constructed MST\n");
    for (i = 0; i < e; ++i)
        printf("%d -- %d == %d\n", result[i].src, result[i].dest, result[i].weight);
    return;
}

int main()
{
    /* 给定如下有权图
        10 
    0--------1 
    |  \     | 
   6|   5\   |15 
    |      \ | 
    2--------3 
        4       */
    int V = 4;  // 图的顶点数目
    int E = 5;  // 图的边数目
    struct Graph* graph = createGraph(V, E);
    
    // 新增边 0-1
    graph->edge[0].src = 0;
    graph->edge[0].dest = 1;
    graph->edge[0].weight = 10;
    
    // 新增边 0-2
    graph->edge[1].src = 0;
    graph->edge[1].dest = 2;
    graph->edge[1].weight = 6;
    
    // 新增边 0-3
    graph->edge[1].src = 0;
    graph->edge[1].dest = 3;
    graph->edge[1].weight = 5;
    
    // 新增边 1-3
    graph->edge[3].src = 1;
    graph->edge[3].dest = 3;
    graph->edge[3].weight = 15;
    
    // 新增边 2-3
    graph->edge[4].src = 2;
    graph->edge[4].dest = 3;
    graph->edge[4].weight = 4;
    
    KruskalMST(graph);
    
    return 0;
}
```
