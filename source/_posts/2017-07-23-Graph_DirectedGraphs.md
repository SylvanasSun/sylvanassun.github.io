---
title:         图的那点事儿(2)-有向图
date:       2017-07-23 11:00
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
> 本文转发自[SylvanasSun Blog][4],原文链接: https://sylvanassun.github.io/2017/07/23/2017-07-23-Graph_DirectedGraphs/


### 有向图的性质


----------



`有向图`与`无向图`不同,**它的`边`是单向的,每条边所连接的两个顶点都是一个有序对,它们的邻接性是单向的.**

在`有向图`中,一条`有向边`**由第一个`顶点`指出并指向第二个`顶点`**,**一个`顶点`的`出度`为由该`顶点`指出的`边`的总数;一个`顶点`的`入度`为指向该`顶点`的边的总数**.

![有向图的解析](http://algs4.cs.princeton.edu/42digraph/images/digraph-anatomy.png)

`v->w`表示一条由`v`指向`w`的边,在一幅`有向图`中,两个`顶点`的关系可能有以下四种(特殊图除外): 

 1. 没有`边`相连.


 2. 存在一条从`v`到`w`的`边`: `v->w`.


 3. 存在一条从`w`到`v`的`边`: `w->v`.


 4. 既存在`v->w`,也存在`w->v`,也就是一条`双向边`.


当存在从`v`到`w`的`有向路径`时,称`顶点w`能够由`顶点v`达到.但在`有向图`中,由`v`能够到达`w`并不意味着由`w`也能到达`v`(但每个`顶点`都是能够到达它自己的).


### 有向图的实现


----------


`有向图`的实现与`无向图`差不多,只不过在`边`的方向上有所不同.(本文中的所有完整代码可以在我的[GitHub][3]中查看)

![有向图的API](http://algs4.cs.princeton.edu/42digraph/images/digraph-api.png)


```java
public class Digraph implements Graph {

    private static final String NEWLINE = System.getProperty("line.separator");

    // number of vertices in this digraph
    private final int vertex;

    // number of edges in this digraph
    private int edge;

    // adj[v] = adjacency list for vertex v
    private Bag<Integer>[] adj;

    // indegree[v] = indegree of vertex v
    private int[] indegree;

    public Digraph(int vertex) {
        validateVertex(vertex);

        this.vertex = vertex;
        this.edge = 0;
        this.indegree = new int[vertex];
        this.adj = (Bag<Integer>[]) new Bag[vertex];
        for (int i = 0; i < vertex; i++)
            adj[i] = new Bag<Integer>();
    }

    public Digraph(Scanner scanner) {
        if (scanner == null)
            throw new IllegalArgumentException("Scanner must be not null.");

        try {
            int vertex = scanner.nextInt();
            validateVertex(vertex);
            this.vertex = vertex;
            this.indegree = new int[vertex];
            this.adj = (Bag<Integer>[]) new Bag[vertex];
            for (int i = 0; i < vertex; i++)
                adj[i] = new Bag<Integer>();

            int edge = scanner.nextInt();
            validateEdge(edge);
            for (int i = 0; i < edge; i++) {
                int v = scanner.nextInt();
                int w = scanner.nextInt();
                addEdge(v, w);
            }
        } catch (NoSuchElementException e) {
            throw new IllegalArgumentException("Invalid input format in Digraph constructor", e);
        }
    }

    public Digraph(Digraph digraph) {
        this(digraph.vertex);
        this.edge = digraph.edge;

        for (int v = 0; v < vertex; v++)
            this.indegree[v] = digraph.indegree(v);

        for (int v = 0; v < vertex; v++) {
            Stack<Integer> reverse = new Stack<>();
            for (int w : digraph.adj(v))
                reverse.push(w);
            for (int w : reverse)
                this.adj[v].add(w);
        }
    }

    @Override
    public int vertex() {
        return vertex;
    }

    @Override
    public int edge() {
        return edge;
    }

    /**
     * 注意这里与无向图不同,只在v的邻接表中添加了w
     */
    @Override
    public void addEdge(int v, int w) {
        validateVertex(v);
        validateVertex(w);
        adj[v].add(w);
		// w的入度+ 1
        indegree[w]++;
        edge++;
    }

    @Override
    public Iterable<Integer> adj(int v) {
        validateVertex(v);
        return adj[v];
    }

    public int indegree(int v) {
        validateVertex(v);
        return indegree[v];
    }

    /**
	 * v的出度就是它邻接表中的顶点数
     */
    public int outdegree(int v) {
        validateVertex(v);
        return adj[v].size();
    }

    @Override
    @Deprecated
    public int degree(int v) {
        validateVertex(v);
        return adj[v].size();
    }

    /**
     * 它返回该有向图的一个副本,但所有边的方向都会被反转.
     */
    public Digraph reverse() {
        Digraph reverse = new Digraph(vertex);
        for (int v = 0; v < vertex; v++) {
            for (int w : adj[v]) {
                reverse.addEdge(w, v);
            }
        }
        return reverse;
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(String.format("Vertexes: %s, Edges: %s", vertex, edge));
        sb.append(NEWLINE);
        for (int v = 0; v < vertex; v++) {
            sb.append(String.format("vertex %d, ", v));
            sb.append(String.format("indegree: %d, outdegree: %d", indegree(v), outdegree(v)));
            sb.append(NEWLINE);
            sb.append("adjacent point: ");
            for (int w : adj[v])
                sb.append(w).append(" ");

            sb.append(NEWLINE);
        }
        return sb.toString();
    }

    private void validateEdge(int edge) {
        if (edge < 0)
            throw new IllegalArgumentException("Number of edges in a Digraph must be nonnegative.");
    }

    private void validateVertex(int vertex) {
        if (vertex < 0)
            throw new IllegalArgumentException("Number of vertex in a Digraph must be nonnegative.");
    }

}
```


### 可达性


----------


对于"是否存在一条从集合中的任意`顶点`到达给定`顶点v`的有向路径?"等类似问题,可以使用`深度优先搜索`或`广度优先搜索`(与`无向图`的实现一致,只不过传入的`图`的类型不同),`有向图`生成的搜索轨迹甚至要比`无向图`还要简单.

对于`可达性分析`的一个典型应用就是内存管理系统.例如,`JVM`使用`多点可达性分析`的方法来判断一个`对象`是否可以进行回收: 所有`对象`组成一幅`有向图`,其中有多个`Root顶点`(它是由`JVM`自己决定的)作为`起点`,如果一个`对象`从`Root顶点`不可达,那么这个`对象`就可以进行回收了.


### 环


----------


在与`有向图`相关的实际应用中,`有向环`特别的重要.我们需要知道一幅`有向图`中是否包含`有向环`.在任务调度问题或其他许多问题中会不允许存在`有向环`,所以对于`环`的检测是很重要的.

使用`深度优先搜索`解决这个问题并不困难,递归调用隐式使用的栈表示的正是"当前"正在遍历的`有向路径`,一旦找到了一条`边v->w`且`w`已经存在于栈中,就等于找到了一个`环`(栈表示的是一条由`w`到`v`的`有向路径`,而`v->w`正好补全了这个`环`).

```java.
public class DirectedCycle {

    private final Digraph digraph;

    // marked[v] = has vertex v been marked?
    private final boolean[] marked;

    // edgeTo[v] = previous vertex on path to v
    private final int[] edgeTo;

    // onStack[v] = is vertex on the stack?
    private final boolean[] onStack;

    // directed cycle (or null if no such cycle)
    private Stack<Integer> cycle;

    public DirectedCycle(Digraph digraph) {
        this.digraph = digraph;
        int vertex = digraph.vertex();
        this.marked = new boolean[vertex];
        this.edgeTo = new int[vertex];
        this.onStack = new boolean[vertex];

        for (int v = 0; v < vertex; v++) {
			// 已经找到环时就不再需要继续搜索了
            if (!marked[v] && cycle == null)
                dfs(v);
        }
    }

    public boolean hasCycle() {
        return cycle != null;
    }

    public Iterable<Integer> cycle() {
        return cycle;
    }

    private void dfs(int vertex) {
        marked[vertex] = true;
        onStack[vertex] = true; // 用于模拟递归调用栈

        for (int w : digraph.adj(vertex)) {
            if (cycle != null)
                return;
            else if (!marked[w]) {
                edgeTo[w] = vertex;
                dfs(w);
            } else if (onStack[w]) {
				// 当w已被标记且在栈中时: 找到环
                cycle = new Stack<>();
                for (int x = vertex; x != w; x = edgeTo[x])
                    cycle.push(x);
                cycle.push(w);
                cycle.push(vertex);
                assert check();
            }
        }
		// 这条路径已经到头,从栈中弹出
        onStack[vertex] = false;
    }

    // certify that digraph has a directed cycle if it reports one
    private boolean check() {
        if (hasCycle()) {
            // verify cycle
            int first = -1, last = -1;
            for (int v : cycle()) {
                if (first == -1) first = v;
                last = v;
            }
            if (first != last) {
                System.err.printf("cycle begins with %d and ends with %d\n", first, last);
                return false;
            }
        }

        return true;
    }

}
```

### 拓扑排序


----------


`拓扑排序`等价于计算优先级限制下的调度问题的,所谓优先级限制的调度问题即是在给定一组需要完成的任务与关于任务完成的先后次序的优先级限制,需要在满足限制条件的前提下来安排任务.

`拓扑排序`需要的是一幅`有向无环图`,如果这幅`图`中含有`环`,那么它肯定不是`拓扑有序`的(一个带有环的调度问题是无解的).

在学习`拓扑排序`之前,需要先知道`顶点`的排序.


#### 顶点排序


----------


使用`深度优先搜索`来记录`顶点排序`是一个很好的选择(正好只会访问每个`顶点`一次),我们借助一些`数据结构`来保存`顶点排序`的顺序: 

 - 前序: 在递归调用之前将`顶点`加入队列.


 - 后序: 在递归调用之后将`顶点`加入队列.


 - 逆后序: 在递归调用之后将`顶点`压入栈.


![顶点排序的轨迹](http://algs4.cs.princeton.edu/42digraph/images/depth-first-orders.png)

```java
public class DepthFirstOrder {

    private final Graph graph;

    // marked[v] = has v been marked in dfs?
    private final boolean[] marked;

    // pre[v]    = preorder  number of v
    private final int[] pre;

    // post[v]   = postorder number of v
    private final int[] post;

    // vertices in preorder
    private final Queue<Integer> preorder;

    // vertices in postorder
    private final Queue<Integer> postorder;

    // counter or preorder numbering
    private int preCounter;

    // counter for postorder numbering
    private int postCounter;

    public DepthFirstOrder(Graph graph) {
        this.graph = graph;
        int vertex = graph.vertex();
        this.pre = new int[vertex];
        this.post = new int[vertex];
        this.preorder = new ArrayDeque<>();
        this.postorder = new ArrayDeque<>();
        this.marked = new boolean[vertex];

        for (int v = 0; v < vertex; v++)
            if (!marked[v]) dfs(v);

        assert check();
    }

    public int pre(int v) {
        validateVertex(v);
        return pre[v];
    }

    public int post(int v) {
        validateVertex(v);
        return post[v];
    }

    public Iterable<Integer> post() {
        return postorder;
    }

    public Iterable<Integer> pre() {
        return preorder;
    }

	// 逆后序,遍历后序队列并压入栈中
    public Iterable<Integer> reversePost() {
        Stack<Integer> reverse = new Stack<Integer>();
        for (int v : postorder)
            reverse.push(v);
        return reverse;
    }

    private void dfs(int vertex) {
        marked[vertex] = true;
		// 前序
        pre[vertex] = preCounter++;
        preorder.add(vertex);
        for (int w : graph.adj(vertex)) {
            if (!marked[w])
                dfs(w);
        }
		// 后序
        post[vertex] = postCounter++;
        postorder.add(vertex);
    }

    // check that pre() and post() are consistent with pre(v) and post(v)
    private boolean check() {

        // check that post(v) is consistent with post()
        int r = 0;
        for (int v : post()) {
            if (post(v) != r) {
                System.out.println("post(v) and post() inconsistent");
                return false;
            }
            r++;
        }

        // check that pre(v) is consistent with pre()
        r = 0;
        for (int v : pre()) {
            if (pre(v) != r) {
                System.out.println("pre(v) and pre() inconsistent");
                return false;
            }
            r++;
        }

        return true;
    }

    // throw an IllegalArgumentException unless {@code 0 <= v < V}
    private void validateVertex(int v) {
        int V = marked.length;
        if (v < 0 || v >= V)
            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (V - 1));
    }

}
```

#### 拓扑排序的实现


----------


所谓`拓扑排序`就是`无环有向图`的`逆后序`,现在已经知道了如何检测`环`与`顶点排序`,那么实现`拓扑排序`就很简单了.

![](http://algs4.cs.princeton.edu/42digraph/images/topological-sort.png)

```java
public class Topological {

    // topological order
    private Iterable<Integer> order;

    // rank[v] = position of vertex v in topological order
    private int[] rank;

    public Topological(Digraph digraph) {
        DirectedCycle directedCycle = new DirectedCycle(digraph);
		// 只有这幅图没有环时,才进行计算拓扑排序
        if (!directedCycle.hasCycle()) {
            DepthFirstOrder depthFirstOrder = new DepthFirstOrder(digraph);
			// 拓扑排序即是逆后序
            order = depthFirstOrder.reversePost();
            rank = new int[digraph.vertex()];
            int i = 0;
            for (int v : order)
                rank[v] = i++;
        }
    }

    public Iterable<Integer> order() {
        return order;
    }

    public boolean hasOrder() {
        return order != null;
    }

    public int rank(int v) {
        validateVertex(v);
        if (hasOrder()) return rank[v];
        else return -1;
    }

    // throw an IllegalArgumentException unless {@code 0 <= v < V}
    private void validateVertex(int v) {
        int V = rank.length;
        if (v < 0 || v >= V)
            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (V - 1));
    }

}
```


### 强连通性


----------


在一幅`无向图`中,如果有一条路径连接顶点`v`和`w`,则它们就是`连通`的(既可以从`w`到达`v`,也可以从`v`到达`w`).但在`有向图`中,如果从顶点`v`有一条有向路径到达`w`,则`w`是从`v`可达的,但从`w`到达`v`的路径可能存在也可能不存在.

**`强连通性`就是两个顶点`v`和`w`是互相可达的.**`有向图`中的`强连通性`具有以下性质: 

 - 自反性: 任意`顶点v`和自己都是`强连通性`的(`有向图`中顶点都是自己可达的).


 - 对称性: 如果`v`和`w`是强连通的,那么`w`和`v`也是强连通的.


 - 传递性: 如果`v`和`w`是强连通的且`w`和`x`也是强连通的,那么`v`和`x`也是强连通的.


`强连通性`将所有`顶点`分为了一些等价类,每个等价类都是由相互为强连通的`顶点`的最大子集组成的.这些子集称为`强连通分量`,它的定义是基于顶点的,而非边.

一个含有`V`个顶点的`有向图`含有`1 ~ V`个`强连通分量`.一个`强连通图`只含有一个`强连通分量`,而一个`有向无环图`中则含有`V`个`强连通分量`.


#### Kosaraju算法


----------


[Kosaraju][7]算法是用于枚举图中每个`强连通分量`内的所有顶点,它主要有以下步骤: 

 - 在给定一幅`有向图`$G$中,取得它的反向图$G^R$.


 - 利用`深度优先搜索`得到$G^R$的逆后序排列.


 - 按照上述逆后序的序列进行`深度优先搜索`


 - 同一个`深度优先搜索`递归子程序中访问的所有`顶点`都在同一个`强连通分量`内.


```java
public class KosarajuSharirSCC {

    private final Digraph digraph;

    // marked[v] = has vertex v been visited?
    private final boolean[] marked;

    // id[v] = id of strong component containing v
    private final int[] id;

    // number of strongly-connected components
    private int count;

    public KosarajuSharirSCC(Digraph digraph) {
        this.digraph = digraph;
        int vertex = digraph.vertex();

        // compute reverse postorder of reverse graph
        DepthFirstOrder depthFirstOrder = new DepthFirstOrder(digraph.reverse());

        // run DFS on G, using reverse postorder to guide calculation
        marked = new boolean[vertex];
        id = new int[vertex];
        for (int v : depthFirstOrder.reversePost()) {
            if (!marked[v]) {
                dfs(v);
                count++;
            }
        }

        // check that id[] gives strong components
        assert check(digraph);
    }

    private void dfs(int v) {
        marked[v] = true;
        id[v] = count;

        for (int w : digraph.adj(v)) {
            if (!marked[w])
                dfs(w);
        }
    }

    public int count() {
        return count;
    }

    public boolean stronglyConnected(int v, int w) {
        validateVertex(v);
        validateVertex(w);
        return id[v] == id[w];
    }

    public int id(int v) {
        validateVertex(v);
        return id[v];
    }

    // does the id[] array contain the strongly connected components?
    private boolean check(Digraph G) {
        TransitiveClosure tc = new TransitiveClosure(G);
        for (int v = 0; v < G.vertex(); v++) {
            for (int w = 0; w < G.vertex(); w++) {
                if (stronglyConnected(v, w) != (tc.reachable(v, w) && tc.reachable(w, v)))
                    return false;
            }
        }
        return true;
    }

    // throw an IllegalArgumentException unless {@code 0 <= v < V}
    private void validateVertex(int v) {
        int V = marked.length;
        if (v < 0 || v >= V)
            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (V - 1));
    }

}
```


### 传递闭包


----------


![](http://algs4.cs.princeton.edu/42digraph/images/transitive-closure.png)

在一幅有向图`G`中,`传递闭包`是由相同的一组`顶点`组成的另一幅`有向图`,在`传递闭包`中存在一条从`v`指向`w`的边且仅当在`G`中`w`是从`v`可达的.

由于`有向图`的性质,每个`顶点`对于自己都是可达的,所以`传递闭包`会含有`V`个自环.

通常将`传递闭包`表示为一个布尔值矩阵,其中`v`行`w`列的值为`true`代表当且仅当`w`是从`v`可达的.

`传递闭包`不适合于处理`大型有向图`,因为构造函数所需的空间与$V^2$成正比,所需的时间和$V(V+E)$成正比.

```java
public class TransitiveClosure {

    private DirectedDFS[] tc;  // tc[v] = reachable from v

    public TransitiveClosure(Digraph G) {
        tc = new DirectedDFS[G.vertex()];
        for (int v = 0; v < G.vertex(); v++)
            tc[v] = new DirectedDFS(G, v);
    }

    public boolean reachable(int v, int w) {
        validateVertex(v);
        validateVertex(w);
        return tc[v].marked(w);
    }

    // throw an IllegalArgumentException unless {@code 0 <= v < V}
    private void validateVertex(int v) {
        int V = tc.length;
        if (v < 0 || v >= V)
            throw new IllegalArgumentException("vertex " + v + " is not between 0 and " + (V - 1));
    }

}
```


### 参考文献


----------


 - [Algorithms, 4th Edition by Robert Sedgewick and Kevin Wayne][2]


 - [Kosaraju's algorithm - Wikipedia][7]


 - [Transitive closure - Wikipedia][8]


### 图的那点事儿


----------


 - [图的那点事儿(1)-无向图][5]


 - [图的那点事儿(2)-有向图][6]


[1]: https://github.com/SylvanasSun
[2]: http://algs4.cs.princeton.edu/42digraph/
[3]: https://github.com/SylvanasSun/algs4-study/tree/master/src/main/java/chapter4_graphs/C4_2_DirectedGraphs
[4]: https://sylvanassun.github.io/
[5]: https://sylvanassun.github.io/2017/07/18/2017-07-18-Graph_UndirectedGraph/
[6]: https://sylvanassun.github.io/2017/07/23/2017-07-23-Graph_DirectedGraphs/
[7]: https://en.wikipedia.org/wiki/Kosaraju%27s_algorithm
[8]: https://en.wikipedia.org/wiki/Transitive_closure