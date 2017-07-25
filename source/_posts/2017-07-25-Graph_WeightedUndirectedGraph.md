---
title:         图的那点事儿(3)-加权无向图
date:       2017-07-25 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - Graph
tags:
    - Algorithms
    - Graph
    - 2017
---


> 本文作者为: [SylvanasSun][1].转载请务必将下面这段话置于文章开头处(保留超链接).
> 本文转发自[SylvanasSun Blog][2],原文链接: https://sylvanassun.github.io/2017/07/25/2017-07-25-Graph_WeightedUndirectedGraph/


### 加权无向图


----------




所谓`加权图`,即每条`边`上都有着对应的`权重`,这个`权重`是正数也可以是负数,也不一定会和距离成正比.`加权无向图`的表示方法只需要对`无向图`的实现进行一下扩展.

 - 在使用`邻接矩阵`的方法中,可以用`边`的`权重`代替布尔值来作为矩阵的元素.


 - 在使用`邻接表` 的方法中,可以在`链表`的`节点`中添加一个权重域.


 - 在使用`邻接表`的方法中,将`边`抽象为一个`Edge`类,它包含了相连的两个`顶点`和它们的`权重`,`链表`中的每个元素都是一个`Edge`.


我们使用第三种方法来实现`加权无向图`,它的数据表示如下图:

![加权无向图的表示](http://algs4.cs.princeton.edu/43mst/images/edge-weighted-graph-representation.png)


#### Edge的实现


----------


```java
public class Edge implements Comparable<Edge> {

    private final int v;

    private final int w;

    private final double weight;

    public Edge(int v, int w, double weight) {
        validateVertexes(v, w);
        if (Double.isNaN(weight)) throw new IllegalArgumentException("Weight is NaN.");

        this.v = v;
        this.w = w;
        this.weight = weight;
    }

    private void validateVertexes(int... vertexes) {
        for (int i = 0; i < vertexes.length; i++) {
            if (vertexes[i] < 0) {
                throw new IllegalArgumentException(
                        String.format("Vertex %d must be a nonnegative integer.", vertexes[i]));
            }
        }
    }

    public double weight() {
        return weight;
    }

    public int either() {
        return v;
    }

    public int other(int vertex) {
        if (vertex == v)
            return w;
        else if (vertex == w)
            return v;
        else
            throw new IllegalArgumentException("Illegal endpoint.");
    }

    @Override
    public int compareTo(Edge that) {
        return Double.compare(this.weight, that.weight);
    }

    @Override
    public String toString() {
        return String.format("%d-%d %.5f", v, w, weight);
    }

}
```

`Edge`类提供了`either()`与`other()`两个函数,在两个`顶点`都未知的情况下,可以调用`either()`获得`顶点v`,然后再调用`other(v)`来获得另一个`顶点`.

> [本文中的所有完整代码点我查看][3]



#### 加权无向图的实现


----------


```java
public class EdgeWeightedGraph {

    private static final String NEWLINE = System.getProperty("line.separator");

    private final int vertexes;

    private int edges;

    private Bag<Edge>[] adj;

    public EdgeWeightedGraph(int vertexes) {
        validateVertexes(vertexes);

        this.vertexes = vertexes;
        this.edges = 0;
        adj = (Bag<Edge>[]) new Bag[vertexes];
        for (int i = 0; i < vertexes; i++)
            adj[i] = new Bag<>();
    }


    public EdgeWeightedGraph(Scanner scanner) {
        this(scanner.nextInt());
        int edges = scanner.nextInt();
        validateEdges(edges);

        for (int i = 0; i < edges; i++) {
            int v = scanner.nextInt();
            int w = scanner.nextInt();
            double weight = scanner.nextDouble();
            addEdge(new Edge(v, w, weight));
        }
    }



    public int vertex() {
        return vertexes;
    }

    public int edge() {
        return edges;
    }

    public void addEdge(Edge e) {
        int v = e.either();
        int w = e.other(v);
        validateVertex(v);
        validateVertex(w);
        adj[v].add(e);
        adj[w].add(e);
        edges++;
    }

    public Iterable<Edge> adj(int v) {
        validateVertex(v);
        return adj[v];
    }

    public int degree(int v) {
        validateVertex(v);
        return adj[v].size();
    }

    public Iterable<Edge> edges() {
        Bag<Edge> list = new Bag<Edge>();
        for (int v = 0; v < vertexes; v++) {
            int selfLoops = 0;
            for (Edge e : adj(v)) {
				// 只添加一条边
                if (e.other(v) > v) {
                    list.add(e);
                }
                // 只添加一条自环的边
                else if (e.other(v) == v) {
                    if (selfLoops % 2 == 0) list.add(e);
                    selfLoops++;
                }
            }
        }
        return list;
    }

    public String toString() {
        StringBuilder s = new StringBuilder();
        s.append(vertexes + " " + edges + NEWLINE);
        for (int v = 0; v < vertexes; v++) {
            s.append(v + ": ");
            for (Edge e : adj[v]) {
                s.append(e + "  ");
            }
            s.append(NEWLINE);
        }
        return s.toString();
    }

    private void validateVertexes(int... vertexes) {
        for (int i = 0; i < vertexes.length; i++) {
            if (vertexes[i] < 0) {
                throw new IllegalArgumentException(
                        String.format("Vertex %d must be a nonnegative integer.", vertexes[i]));
            }
        }
    }

    private void validateEdges(int edges) {
        if (edges < 0) throw new IllegalArgumentException("Number of edges must be nonnegative.");
    }

    // throw an IllegalArgumentException unless {@code 0 <= v < V}
    private void validateVertex(int v) {
        if (v < 0 || v >= vertexes)
            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (vertexes - 1));
    }

}
```

上述代码是对`无向图`的扩展,它将`邻接表`中的元素从`整数`变为了`Edge`,函数`edges()`返回了`边`的集合,由于是`无向图`所以每条`边`会出现两次,需要注意处理.

`加权无向图`的实现还拥有以下特点: 

 - 边的比较: `Edge`类实现了`Comparable`接口,它使用了`权重`来比较两条`边`的大小,所以`加权无向图`的自然次序就是权重次序.


 - 自环: 该实现允许存在自环,并且`edges()`函数中对自环边进行了记录.


 - 平行边: 该实现允许存在平行边,但可以用更复杂的方法来消除平行边,例如只保留平行边中的权重最小者.



### 最小生成树


----------


![加权无向图的最小生成树](http://algs4.cs.princeton.edu/43mst/images/mst.png)

`最小生成树`是`加权无向图`的重要应用.**`图`的`生成树`是它的一棵含有其所有`顶点`的`无环连通子图`,`最小生成树`是它的一棵`权值`(所有边的权值之和)最小的`生成树`.**


在给定的一幅`加权无向图`$G = (V,E)$中,$(u,v)$代表连接`顶点u`与`顶点v`的`边`,也就是$(u,v) \in E$,而$w(u,v)$代表这条边的`权重`,若存在`T`为`E`的子集,也就是$T \subseteq E$,且为`无环图`,使得$w(T) = \sum_{(u,v) \in T}w(u,v)$ 的 $w(T)$ 最小,则`T`为`G`的`最小生成树`.


`最小生成树`在一些情况下可能会存在多个,例如,给定一幅图`G`,当它的所有边的`权重`都相同时,那么`G`的所有`生成树`都是`最小生成树`,当所有边的`权重`互不相同时,将会只有一个`最小生成树`.

![多个最小生成树的情况](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c9/Multiple_minimum_spanning_trees.svg/316px-Multiple_minimum_spanning_trees.svg.png)


### 切分定理


----------


**`切分定理`将图中的所有`顶点`切分为两个集合(两个非空且不重叠的集合),检查两个集合的所有边并识别哪条边应属于图的`最小生成树`.**

一种比较简单的切分方法即通过**指定一个顶点集并隐式地认为它的补集为另一个顶点集来指定一个切分.**

![白色与灰色顶点代表了不同的顶点集](http://algs4.cs.princeton.edu/43mst/images/cut-property.png)

`切分定理`也表明了对于每一种切分,`权重`最小的`横切边(一条连接两个属于不同集合的顶点的边)`必然属于`最小生成树`.

`切分定理`是解决`最小生成树`问题的所有算法的基础,**使用`切分定理`找到`最小生成树`的一条边,不断重复直到找到`最小生成树`的所有边.**

这些算法可以说都是`贪心算法`,算法的每一步都是在找最优解(`权值`最小的`横切边`),而**解决`最小生成树`的各种算法不同之处仅在于保存切分和判定`权重`最小的`横切边`的方式.**

![生成最小生成树的过程,权值最小的横切边将会被标记为黑色](http://algs4.cs.princeton.edu/43mst/images/mst-greedy.png)

### Prim算法


----------


`Prim算法`是用于解决`最小生成树`的算法之一,算法的每一步都会为一棵生长中的`树`添加一条边.一开始这棵树只有一个`顶点`,然后会一直添加到$V - 1$条边,**每次总是将下一条连接`树`中的`顶点`与不在`树`中的`顶点`且`权重`最小的边加入到`树`中(也就是由`树`中`顶点`所定义的切分中的一条`横切边`).**

![](http://algs4.cs.princeton.edu/43mst/images/prim.png)


实现`Prim算法`还需要借助以下数据结构: 

 - 布尔值数组: 用于记录`顶点`是否已在`树`中.


 - 队列: 使用一条队列来保存`最小生成树`中的边,也可以使用一个由`顶点`索引的`Edge`对象的数组.


 - 优先队列: 优先队列用于保存`横切边`,优先队列的性质可以每次取出`权值`最小的`横切边`.


#### 延时实现


----------


当我们连接新加入`树`中的`顶点`与其他已经在`树`中`顶点`的所有边都失效了(由于两个`顶点`都已在`树`中,所以这是一条失效的`横切边`).我们需要处理这种情况,**即使实现对无效边采取忽略(不加入到优先队列中),而延时实现会把无效边留在优先队列中,等到要删除优先队列中的数据时再进行有效性检查.**

![Prim延时实现的运行轨迹](http://algs4.cs.princeton.edu/43mst/images/prim-lazy.png)

上图为`Prim算法`延时实现的轨迹图,它的步骤如下: 

 - 将`顶点0`添加到`最小生成树`中,将它的`邻接表`中的所有边添加到优先队列中(将`横切边`添加到优先队列).


 - 将`顶点7`和边`0-7`添加到`最小生成树`中,将`顶点`的`邻接表`中的所有边添加到优先队列中.


 - 将`顶点1`和边`1-7`添加到`最小生成树`中,将`顶点`的`邻接表`中的所有边添加到优先队列中.


 - 将`顶点2`和边`0-2`添加到`最小生成树`中,将边`2-3`和`6-2`添加到优先队列中,边`2-7`和`1-2`失效.


 - 将`顶点3`和边`2-3`添加到`最小生成树`中,将边`3-6`添加到优先队列之中,边`1-3`失效.


 - 将`顶点5`和边`5-7`添加到`最小生成树`中,将边`4-5`添加到优先队列中,边`1-5`失效.


 - 从优先队列中删除失效边`1-3`,`1-5`,`2-7`.


 - 将`顶点4`和边`4-5`添加到`最小生成树`中,将边`6-4`添加到优先队列中,边`4-7`,`0-4`失效.


 - 从优先队列中删除失效边`1-2`,`4-7`,`0-4`.


 - 将`顶点6`和边`6-2`添加到`最小生成树`中,和`顶点6`关联的其他边失效.


 - 在添加`V`个顶点与`V - 1`条边之后,`最小生成树`就构造完成了,优先队列中剩余的边都为失效边.


```java
public class LazyPrimMST {

    private final EdgeWeightedGraph graph;

    // 记录最小生成树的总权重
    private double weight;

    // 存储最小生成树的边
    private final Queue<Edge> mst;

    // 标记这个顶点在树中
    private final boolean[] marked;

    // 存储横切边的优先队列
    private final PriorityQueue<Edge> pq;

    public LazyPrimMST(EdgeWeightedGraph graph) {
        this.graph = graph;
        int vertex = graph.vertex();
        mst = new ArrayDeque<>();
        pq = new PriorityQueue<>();
        marked = new boolean[vertex];

        for (int v = 0; v < vertex; v++)
            if (!marked[v]) prim(v);
    }

    private void prim(int s) {
        scanAndPushPQ(s);
        while (!pq.isEmpty()) {
            Edge edge = pq.poll();  // 取出权重最小的横切边
            int v = edge.either(), w = edge.other(v);  
            assert marked[v] || marked[w];

            if (marked[v] && marked[w])
                continue; // 忽略失效边

            mst.add(edge); // 添加边到最小生成树中
            weight += edge.weight(); // 更新总权重
			// 继续将非树顶点加入到树中并更新横切边
            if (!marked[v]) scanAndPushPQ(v); 
            if (!marked[w]) scanAndPushPQ(w); 
        }
    }

    // 标记顶点到树中,并且添加横切边到优先队列
    private void scanAndPushPQ(int v) {
        assert !marked[v];
        marked[v] = true;
        for (Edge e : graph.adj(v))
            if (!marked[e.other(v)]) pq.add(e);
    }

    public Iterable<Edge> edges() {
        return mst;
    }

    public double weight() {
        return weight;
    }

}
```


#### 即时实现


----------



在即时实现中,将`v`添加到树中时,对于每个`非树顶点w`,**不需要在优先队列中保存所有从`w`到`树顶点`的边,而只需要保存其中`权重`最小的边,所以在将`v`添加到`树`中后,要检查是否需要更新这条`权重`最小的边(如果`v-w`的`权重`更小的话).**

也可以认为只会在优先队列中保存每个`非树顶点w`的一条边(也是`权重`最小的那条边),将`w`和`树顶点`连接起来的其他`权重`较大的边迟早都会失效,所以没必要在优先队列中保存它们.

要实现即时版的`Prim算法`,需要使用两个顶点索引的数组`edgeTo[]`和`distTo[]`与一个索引优先队列,它们具有以下性质: 

 - 如果`顶点v`不在树中但至少含有一条边和树相连,那么`edgeTo[v]`是将`v`和树连接的最短边,`distTo[v]`为这条边的`权重`.


 - 所有这类`顶点v`都保存在索引优先队列中,索引`v`关联的值是`edgeTo[v]`的边的`权重`.


 - 索引优先队列中的最小键即是`权重`最小的`横切边`的`权重`,而和它相关联的顶点`v`就是下一个将要被添加到`树`中的`顶点`.


![即时实现Prim算法的运行轨迹](http://algs4.cs.princeton.edu/43mst/images/prim-eager.png)

 - 将`顶点0`添加到`最小生成树`之中,将它的`邻接表`中的所有边添加到优先队列中(这些边是目前唯一已知的横切边).


 - 将`顶点7`和边`0-7`添加到`最小生成树`,将边`1-7`和`5-7`添加到优先队列中,将连接`顶点4`与树的最小边由`0-4`替换为`4-7`.


 - 将`顶点1`和边`1-7`添加到`最小生成树`,将边`1-3`添加到优先队列.


 - 将`顶点2`和边`0-2`添加到最小生成树,将连接`顶点6`与树的最小边由`0-6`替换为`6-2`,将连接`顶点3`与树的最小边由`1-3`替换为`2-3`.


 - 将`顶点3`和边`2-3`添加到`最小生成树`.


 - 将`顶点5`和边`5-7`添加到`最小生成树`,将连接`顶点4`与树的最小边`4-7`替换为`4-5`.


 - 将`顶点4`和边`4-5`添加到`最小生成树`.


 - 将`顶点6`和边`6-2`添加到`最小生成树`.


 - 在添加了`V - 1`条边之后,`最小生成树`构造完成并且优先队列为空.


```java
public class PrimMST {

    private final EdgeWeightedGraph graph;

    // 存放最小生成树中的边
    private final Edge[] edgeTo;

    // 每条边对应的权重
    private final double[] distTo;

    private final boolean[] marked;

    private final IndexMinPQ<Double> pq;

    public PrimMST(EdgeWeightedGraph graph) {
        this.graph = graph;
        int vertex = graph.vertex();
        this.edgeTo = new Edge[vertex];
        this.marked = new boolean[vertex];
        this.pq = new IndexMinPQ<>(vertex);
        this.distTo = new double[vertex];
		// 将权重数组初始化为无穷大
        for (int i = 0; i < vertex; i++)
            distTo[i] = Double.POSITIVE_INFINITY;

        for (int v = 0; v < vertex; v++)
            if (!marked[v]) prim(v);
    }

    private void prim(int s) {
		// 将起点设为0.0并加入到优先队列
        distTo[s] = 0.0;
        pq.insert(s, distTo[s]);
        while (!pq.isEmpty()) {
			// 取出权重最小的边,优先队列中存的顶点是与树相连的非树顶点,
			// 同时它也是下一次要加入到树中的顶点
            int v = pq.delMin();
            scan(v);
        }
    }

    private void scan(int v) {
		// 将顶点加入到树中
        marked[v] = true;

        for (Edge e : graph.adj(v)) {
            int w = e.other(v);
			// 忽略失效边
            if (marked[w]) continue;
			// 如果w与连接树顶点的边的权重小于其他w连接树顶点的边
			// 则进行替换更新
            if (e.weight() < distTo[w]) {
                distTo[w] = e.weight();
                edgeTo[w] = e;
                if (pq.contains(w))
                    pq.decreaseKey(w, distTo[w]);
                else
                    pq.insert(w, distTo[w]);
            }
        }
    }

    public Iterable<Edge> edges() {
        Queue<Edge> mst = new ArrayDeque<>();
        for (int v = 0; v < edgeTo.length; v++) {
            Edge e = edgeTo[v];
            if (e != null) {
                mst.add(e);
            }
        }
        return mst;
    }

    public double weight() {
        double weight = 0.0;
        for (Edge e : edges())
            weight += e.weight();
        return weight;
    }

}
```

不管是`延迟实现`还是`即时实现`,`Prim算法`的规律就是: **在`树`的生长过程中,都是通过连接一个和新加入的`顶点`相邻的`顶点`.当新加入的`顶点`周围没有`非树顶点`时,树的生长又会从另一部分开始.**


### Kruskal算法


----------



`Kruskal算法`的思想是**按照边的`权重`顺序由小到大处理它们**,将边添加到`最小生成树`,加入的边不会与已经在`树`中的边构成环,直到`树`中含有`V - 1`条边为止.**这些边会逐渐由一片`森林`合并为一棵`树`**,也就是我们需要的`最小生成树`.

![Kruskal算法的运行轨迹](http://algs4.cs.princeton.edu/43mst/images/kruskal.png)


#### 与Prim算法的区别


----------


 - `Prim算法`是一条边一条边地来构造`最小生成树`,每一步都会为`树`中添加一条边.


 - `Kruskal算法`构造`最小生成树`也是一条边一条边地添加,但不同的是它寻找的边会连接一片`森林`中的两棵`树`.从一片由`V`棵单`顶点`的树构成的`森林`开始并不断地将两棵`树`合并(可以找到的最短边)直到只剩下一棵`树`,它就是`最小生成树`.


#### 实现


----------


要实现`Kruskal算法`需要借助`Union-Find`数据结构,它是一种树型的数据结构,用于处理一些不相交集合的合并与查询问题.

关于`Union-Find`的更多资料可以参考下面的链接: 

 - [Union-Find简单实现][5]


 - [Disjoint-set data structure - Wikipedia][4]


```java
public class KruskalMST {

    // 这条队列用于记录最小生成树中的边集
    private final Queue<Edge> mst;

    private double weight;
 
    public KruskalMST(EdgeWeightedGraph graph) {
        this.mst = new ArrayDeque<>();
        // 创建一个优先队列,并将图的所有边添加到优先队列中
        PriorityQueue<Edge> pq = new PriorityQueue<>();

        for (Edge e : graph.edges()) {
            pq.add(e);
        }

        int vertex = graph.vertex();
		// 创建一个Union-Find
        UF uf = new UF(vertex);
		// 一条一条地添加边到最小生成树,直到添加了 V - 1条边
        while (!pq.isEmpty() && mst.size() < vertex - 1) {
			// 取出权重最小的边
            Edge e = pq.poll();
            int v = e.either();
            int w = e.other(v);
            // 如果这条边的两个顶点不在一个分量中(对于union-find数据结构中而言)
            if (!uf.connected(v, w)) {
				// 将v和w归并(对于union-find数据结构中而言),然后将边添加进树中,并计算更新权重
                uf.union(v, w); 
                mst.add(e);
                weight += e.weight();
            }
        }
    }

    public Iterable<Edge> edges() {
        return mst;
    }

    public double weight() {
        return weight;
    }

}
```

上面代码实现的`Kruskal算法`使用了一条队列来保存`最小生成树`的边集,一条优先队列来保存还未检查的边,一个`Union-Find`来判断失效边.


#### 性能比较


----------


| 算法       | 空间复杂度 | 时间复杂度 |
| ---------- | ---------- | ---------- |
| Prim(延时) | E          | ElogE      |
| Prim(即时) | V          | ElogV      |
| Kruskal    | E          |     ElogE       |


### 参考文献


----------


 - [Minimum spanning tree - Wikipedia][6]


 - [Algorithms, 4th Edition by Robert Sedgewick and Kevin Wayne][7]


### 图的那点事儿


----------


 - [图的那点事儿(1)-无向图][8]


 - [图的那点事儿(2)-有向图][9]


 - [图的那点事儿(3)-加权无向图][10]

[1]: https://github.com/SylvanasSun
[2]: https://sylvanassun.github.io
[3]: https://github.com/SylvanasSun/algs4-study/tree/master/src/main/java/chapter4_graphs/C4_3_MinimumSpanningTrees
[4]: https://en.wikipedia.org/wiki/Disjoint-set_data_structure
[5]: https://github.com/SylvanasSun/algs4-study/blob/15ae228a1bc6a75465a96681caaa93eff3462327/src/main/java/chapter1_fundamentals/C1_5_UnionFind/UF.java
[6]: https://en.wikipedia.org/wiki/Minimum_spanning_tree
[7]: http://algs4.cs.princeton.edu/43mst/
[8]: https://sylvanassun.github.io/2017/07/18/2017-07-18-Graph_UndirectedGraph/
[9]: https://sylvanassun.github.io/2017/07/23/2017-07-23-Graph_DirectedGraphs/
[10]: https://sylvanassun.github.io/2017/07/25/2017-07-25-Graph_WeightedUndirectedGraph/