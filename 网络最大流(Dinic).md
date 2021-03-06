## 网络最大流



### 基本概念

```markdown
对于任意一张有向图（也就是网络），其中有N个点、M条边以及源点S和汇点T
	c(x,y)称为边的容量 
	f(x,y)称为边的流量

流函数f(x,y)三大性质:
	容量限制：每条边的流量总不可能大于该边的容量的（不然水管就爆了）
	斜对称：正向边的流量=反向边的流量（反向边后面会具体讲）
	流量守恒：正向的所有流量和=反向的所有流量和（就是总量始终不变）
最大流
	使得整个网络流量之和最大的流函数称为网络的最大流，此时的流量和被称为网络的最大流量
增广路	
	若一条从S到T的路径上所有边的剩余容量都大于0，则称这样的路径为一条增广路
反向边
	因为可能一条边可以被包含于多条增广路径，所以为了寻找所有的增广路经我们就要让这一条边有多次被选择的机会
	构建反向边则是这样一个机会，相当于给程序一个反悔的机会。
残量网络	
	在任意时刻，网络中所有节点以及剩余容量大于0的边构成的子图被称为残量网络

邻接表“成对存储”
	将正向边和反向边存在“2和3”、“4和5”、“6和7”。因为在更新边权的时候，我们就可以直接使用xor的方
	式，找到对应的正向边和反向边
```



### 知识点梳理

```markdown
1. 基本概念
    1.1 流网络，不考虑反向边
    1.2 可行流，不考虑反向边
        1.2.1 两个条件：容量限制、流量守恒
        1.2.2 可行流的流量指从源点流出的流量 - 流入源点的流量
        1.2.3 最大流是指最大可行流
    1.3 残留网络，考虑反向边，残留网络的可行流f' + 原图的可行流f = 原题的另一个可行流
        (1) |f' + f| = |f'| + |f|
        (2) |f'| 可能是负数
    1.4 增广路径
    1.5 割
        1.5.1 割的定义
        1.5.2 割的容量，不考虑反向边，“最小割”是指容量最小的割。
        1.5.3 割的流量，考虑反向边，f(S, T) <= c(S, T)
        1.5.4 对于任意可行流f，任意割[S, T]，|f| = f(S, T)
        1.5.5 对于任意可行流f，任意割[S, T]，|f| <= c(S, T)
        1.5.6 最大流最小割定理
            (1) 可以流f是最大流
            (2) 可行流f的残留网络中不存在增广路
            (3) 存在某个割[S, T]，|f| = c(S, T)
    1.6. 算法
        1.6.1 EK O(nm^2)
        1.6.2 Dinic O(n^2m)
    1.7 应用
        1.7.1 二分图
            (1) 二分图匹配
            (2) 二分图多重匹配
        1.7.2 上下界网络流
            (1) 无源汇上下界可行流
            (2) 有源汇上下界最大流
            (3) 有源汇上下界最小流
        1.7.3 多源汇最大流
```




### Dinic算法

#### 思路

- *D**i**n**i**c*算法框架

1. 在残量网络上 BFS 求出节点的层次，构造分层图
2. 在分层图上 DFS 寻找增广路，在回溯时同时更新边权




### 代码实现(固定风格)

```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 10010, M = 200010, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx; // f[N] 为 容量
int q[N], d[N], cur[N]; // 队列   深度   当前弧

void add(int a, int b, int c) {
    e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
    e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx ++ ;
}

bool bfs() {
    int hh = 0, tt = 0;
    memset(d, -1, sizeof d);
    q[0] = S, d[S] = 0, cur[S] = h[S];
    
    while (hh <= tt) {
        int t = q[hh ++ ];
        for (int i = h[t]; ~i; i = ne[i]) {
            int ver = e[i];
            if (d[ver] == -1 && f[i]) {
                d[ver] = d[t] + 1;
                cur[ver] = h[ver];
                if (ver == T)  return true;
                q[ ++ tt] = ver;
            }
        }
    }
    
    return false;
}

int find(int u, int limit) {
    if (u == T) return limit;
    int flow = 0;
    
    for (int i = cur[u]; ~i && flow < limit; i = ne[i]) {
        cur[u] = i;  // 当前弧优化
        int ver = e[i];
        if (d[ver] == d[u] + 1 && f[i]) {
            int t = find(ver, min(f[i], limit - flow));
            if (!t) d[ver] = -1;
            f[i] -= t, f[i ^ 1] += t, flow += t;
        }
    }
    
    return flow;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) {  // 存在增广路时循环继续
        while (flow = find(S, INF)) r += flow;
    }
    return r;
}

int main() {
    scanf("%d%d%d%d", &n, &m, &S, &T);
    memset(h, -1, sizeof h);
    while (m -- ) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c);
    }

    printf("%d\n", dinic());

    return 0;
}
```

[最大流模板题](https://www.luogu.com.cn/problem/P3376)

[参考题解](https://www.luogu.com.cn/problem/solution/P3376)





### 无源汇上下界可行流

```markdown
问题描述
    给定一个包含 n 个点 m 条边的有向图，每条边都有一个流量下界和流量上界。
    求一种可行方案使得在所有点满足流量平衡条件的前提下，所有边满足流量限制。

输入格式
	第一行包含两个整数 n 和 m。接下来 m 行，每行包含四个整数 a,b,c,d 表示点 a 和 b 之间存在一条有向边，该边的流量下界为 c，流量上界为 d。点编号从 1 到 n。

输出格式
	如果存在可行方案，则第一行输出 YES，接下来 m 行，每行输出一个整数，其中第 i 行的整数表示输入的第 i 条边的流量。
	如果不存在可行方案，直接输出一行 NO。
	如果可行方案不唯一，则输出任意一种方案即可
```



```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 211, M = (10200 + N) * 2, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], l[M], ne[M], idx;   // l[M] 存储容量下界
int q[N], d[N], cur[N], A[N];  // A[N] 存储第 i 个点的出入度容量差

void add(int a, int b, int c, int d) {   // c : 容量下界   d : 容量上界
    e[idx] = b, f[idx] = d - c, l[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
    e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx ++ ;
}

bool bfs() {
    int hh = 0, tt = 0;
    memset(d, -1, sizeof d);
    q[0] = S, d[S] = 0, cur[S] = h[S];
    while (hh <= tt) {
        int t = q[hh ++ ];
        for (int i = h[t]; ~i; i = ne[i]) {
            int ver = e[i];
            if (d[ver] == -1 && f[i]) {
                d[ver] = d[t] + 1;
                cur[ver] = h[ver];
                if (ver == T) return true;
                q[ ++ tt] = ver;
            }
        }
    }
    return false;
}

int find(int u, int limit) {
    if (u == T) return limit;
    int flow = 0;
    for (int i = cur[u]; ~i && flow < limit; i = ne[i]) {
        cur[u] = i;
        int ver = e[i];
        if (d[ver] == d[u] + 1 && f[i]) {
            int t = find(ver, min(f[i], limit - flow));
            if (!t) d[ver] = -1;
            f[i] -= t, f[i ^ 1] += t, flow += t;
        }
    }
    return flow;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) while (flow = find(S, INF)) r += flow;
    return r;
}

int main() {
    scanf("%d%d", &n, &m);
    S = 0, T = n + 1;
    memset(h, -1, sizeof h);
    for (int i = 0; i < m; i ++ ) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, c, d);
        A[a] -= c, A[b] += c;
    }

    int tot = 0;
    for (int i = 1; i <= n; i ++ )   // 注意流网络的处理   减去下界需要补边来保持流量守恒
        if (A[i] > 0) add(S, i, 0, A[i]), tot += A[i];
        else if (A[i] < 0) add(i, T, 0, -A[i]);

    if (dinic() != tot) puts("NO");
    else {
        puts("YES");
        for (int i = 0; i < m * 2; i += 2)
            printf("%d\n", f[i ^ 1] + l[i]);
    }
    return 0;
}
```





### 有源汇上下界最大流

```markdown
问题描述
    给定一个包含 n 个点 m 条边的有向图，每条边都有一个流量下界和流量上界。
    给定源点 S 和汇点 T，求源点到汇点的最大流。

输入格式
	第一行包含四个整数 n,m,S,T。接下来 m 行，每行包含四个整数 a,b,c,d 表示点 a 和 b 之间存在一条有向边，该边的流量下界为 c，流量上界为 d。点编号从 1 到 n。

输出格式
	输出一个整数表示最大流。如果无解，则输出 No Solution。
```

```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 210, M = (N + 10000) * 2, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N], A[N];

void add(int a, int b, int c) {
    e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
    e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx ++ ;
}

bool bfs() {
    int hh = 0, tt = 0;
    memset(d, -1, sizeof d);
    q[0] = S, d[S] = 0, cur[S] = h[S];
    while (hh <= tt) {
        int t = q[hh ++ ];
        for (int i = h[t]; ~i; i = ne[i]) {
            int ver = e[i];
            if (d[ver] == -1 && f[i]) {
                d[ver] = d[t] + 1;
                cur[ver] = h[ver];
                if (ver == T) return true;
                q[ ++ tt] = ver;
            }
        }
    }
    return false;
}

int find(int u, int limit) {
    if (u == T) return limit;
    int flow = 0;
    for (int i = cur[u]; ~i && flow < limit; i = ne[i]) {
        cur[u] = i;
        int ver = e[i];
        if (d[ver] == d[u] + 1 && f[i]) {
            int t = find(ver, min(f[i], limit - flow));
            if (!t) d[ver] = -1;
            f[i] -= t, f[i ^ 1] += t, flow += t;
        }
    }
    return flow;
}

int dinic() {
    int r = 0, flow;
    while (bfs()) while (flow = find(S, INF)) r += flow;
    return r;
}

int main() {
    int s, t;
    scanf("%d%d%d%d", &n, &m, &s, &t);
    S = 0, T = n + 1;
    memset(h, -1, sizeof h);
    while (m -- ) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, d - c);
        A[a] -= c, A[b] += c;
    }

    int tot = 0;
    for (int i = 1; i <= n; i ++ )
        if (A[i] > 0) add(S, i, A[i]), tot += A[i];
        else if (A[i] < 0) add(i, T, -A[i]);

    add(t, s, INF); // 先以 s, t 为源点(注意到这是最后增加的两条边, 输出结果时会用到这个地方)
    
    if (dinic() < tot) puts("No Solution");
    else {
        int res = f[idx - 1];
        S = s, T = t;
        f[idx - 1] = f[idx - 2] = 0;
        printf("%d\n", res + dinic());
        
        /* 有源汇上下界最小流 只需要修改这里即可
        int res = f[idx - 1];
        S = t, T = s;
        f[idx - 1] = f[idx - 2] = 0;
        printf("%d\n", res - dinic());
        */
    }

    return 0;
}
```



### 最大权闭合子图

```markdown
最大权闭合图的解法
	新建源点S,向正权点连容量为点权的边；新建汇点T,负权点向T连容量为点权的相反数的边。图中原有的边容量改为正无穷。正权点点权和减去最小割即为答案。
正确性证明
	选择用户i ⇒ ai,bi 必须选择，即必须割掉ai,bi连向T的边；放弃用户i ⇒ 割掉S连向i的边。
```

```markdown
问题描述
    在前期市场调查和站址勘测之后，公司得到了一共 N 个可以作为通讯信号中转站的地址，而由于这些地址的地理位置差异，在不同的地方建造通讯中转站需 要投入的成本也是不一样的，所幸在前期调查之后这些都是已知数据：
    建立第 i 个通讯中转站需要的成本为 Pi(1≤i≤N)。
    另外公司调查得出了所有期望中的用户群，一共 M 个。
    关于第 i 个用户群的信息概括为 Ai,Bi 和 Ci：这些用户会使用中转站 Ai 和中转站 Bi 进行通讯，公司可以获益 Ci。（1≤i≤M,1≤Ai,Bi≤N）
    THU 集团的 CS&T 公司可以有选择的建立一些中转站（投入成本），为一些用户提供服务并获得收益（获益之和）。
    那么如何选择最终建立的中转站才能让公司的净获利最大呢？（净获利 = 获益之和 – 投入成本之和）

输入格式
	第一行有两个正整数 N 和 M。第二行中有 N 个整数描述每一个通讯中转站的建立成本，依次为 P1,P2,…,PN 。以下 M 行，第 (i+2) 行的三个数 Ai,Bi 和 Ci 描述第 i 个用户群的信息。所有变量的含义可以参见题目描述。

输出格式
	输出一个整数，表示公司可以得到的最大净获利。
```

```c++
// 模板省略，同上
int main() {
    scanf("%d%d", &n, &m);
    S = 0, T = n + m + 1;
    memset(h, -1, sizeof h);
    for (int i = 1; i <= n; i ++ ) {
        int p;
        scanf("%d", &p);
        add(m + i, T, p);
    }

    int tot = 0;
    for (int i = 1; i <= m; i ++ ) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(S, i, c);
        add(i, m + a, INF);
        add(i, m + b, INF);
        tot += c;
    }

    printf("%d\n", tot - dinic());

    return 0;
}
```



### 费用流

```markdown
问题描述
    给定一个包含 n 个点 m 条边的有向图，并给定每条边的容量和费用，边的容量非负。
    图中可能存在重边和自环，保证费用不会存在负环。
    求从 S 到 T 的最大流，以及在流量最大时的最小费用。

输入格式
    第一行包含四个整数 n,m,S,T。接下来 m 行，每行三个整数 u,v,c,w，表示从点 u 到点 v 存在一条有向边，容量为 c，费用为 w(这里费用指的是单位流量的费用)。点的编号从 1 到 n。

输出格式
    输出点 S 到点 T 的最大流和流量最大时的最小费用。
    如果从点 S 无法到达点 T 则输出 0 0。
```

```c++
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 5010, M = 100010, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d) {
    e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx ++ ;
    e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx ++ ;  // w[idx] = -d : 退流
}

bool spfa() { // 找最短增广路
    int hh = 0, tt = 1;
    memset(d, 0x3f, sizeof d);
    memset(incf, 0, sizeof incf);
    q[0] = S, d[S] = 0, incf[S] = INF;
    while (hh != tt) {
        int t = q[hh ++ ];
        if (hh == N) hh = 0;
        st[t] = false;

        for (int i = h[t]; ~i; i = ne[i]) {
            int ver = e[i];
            if (f[i] && d[ver] > d[t] + w[i]) {
                d[ver] = d[t] + w[i];
                pre[ver] = i;
                incf[ver] = min(f[i], incf[t]);
                if (!st[ver]) {
                    q[tt ++ ] = ver;
                    if (tt == N) tt = 0;
                    st[ver] = true;
                }
            }
        }
    }

    return incf[T] > 0;
}

void EK(int& flow, int& cost) {
    flow = cost = 0;
    while (spfa()) {
        int t = incf[T];
        flow += t, cost += t * d[T];
        for (int i = T; i != S; i = e[pre[i] ^ 1]) {
            f[pre[i]] -= t;
            f[pre[i] ^ 1] += t;
        }
    }
}

int main() {
    scanf("%d%d%d%d", &n, &m, &S, &T);
    memset(h, -1, sizeof h);
    while (m -- ) {
        int a, b, c, d;
        scanf("%d%d%d%d", &a, &b, &c, &d);
        add(a, b, c, d);
    }

    int flow, cost;
    EK(flow, cost); // flow 为 最大流   cost 为 最小费用流
    printf("%d %d\n", flow, cost);
    
    /* 若还要求最大费用流, 则先恢复图, 然后将费用取反再跑 EK, -cost 则为最大费用流
    for (int i = 0; i < idx; i += 2) {
        f[i] += f[i ^ 1], f[i ^ 1] = 0;
        w[i] = -w[i], w[i ^ 1] = -w[i ^ 1];
    }
    
    EK(flow, cost);
    printf("%d\n", -cost);
	*/
    
    return 0;
}
```



