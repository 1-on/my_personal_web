---
title: 算法板子
abbrlink: 3
categories: 学习笔记
tags: 算法
updated: 2023-4-1
data: 2023-04-01
---


### 求素数（2 3 5 7 11……）

```c++
bool isPrime(int n) {
    if (n <= 1) {
        return false;
    }
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            return false;
        }
    }
    return true;
}
```
<!-- more -->
### 欧拉筛

```c++
const int N = 1000010;
int primes[N], cnt; //记录质数
bool st[N];  //划掉合数

void get_primes(int n)  //线性筛法
{
    for (int i = 2; i <= n; i++) {
        if (!st[i]) primes[cnt++] = i;
        for (int j = 0; 1ll * i * primes[j] <= n; j++) {
            st[primes[j] * i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}
```

### 并查集

```C++
const int N = 1000010;
int fa[N];  //存放节点x的父节点
初始化 for (int i = 1; i <= n; ++i) fa[i] = i;

//查找
int find(int x) {
    if (fa[x] == x) return x;
    return fa[x] = find(fa[x]);
}
//合并
void unionset(int x, int y) {
    fa[find(x)] = find(y);
}

//按秩合并（启发式合并）
//把小集合的根指向大集合的根
void unionset(int x, int y) {
    x = find(x);
    y = find(y);
    if (x == y) return;
    if (siz[x] > siz[y]) swap(x, y);
    fa[x] = y;
    siz[y] += siz[x];
}
```



### Dijkstra算法

```c++
scanf("%d%d%d",&n,&m,&s);
for (int i = 0; i < m; ++i) {
    scanf("%d%d%d",&a,&b,&c);
    e[a].push_back({b,c});
}


struct edge {
    int v, w;;
};
vector<edge> e[N];
int d[N], vis[N];

void dijkstra(int s) {
    for (int i = 0; i < n; ++i) d[i] = INF;
    d[s] = 0;
    for (int i = 1; i < n; ++i) {  //枚举次数
        int u = 0;
        for (int j = 1; j <= n; ++j) {  //枚举点
            if (!vis[j] && d[j] < d[u]) u = j;
        }
        vis[u] = 1;  //标记u移出集合
        for (auto ed: e[u]) {  //枚举邻边
            int v = ed.v, w = ed.w;
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
            }
        }
    }
}

//队列优化
struct edge {
    int v, w;;
};
vector<edge> e[N];
int d[N], vis[N];
priority_queue<pair<int, int>> q;

void dijkstra(int s) {
    for (int i = 0; i <= n; ++i) d[i] = INF;
    d[s] = 0;
    q.push({0, s});
    while (q.size()) {
        auto t = q.top();
        q.pop();
        int u = t.second;
        if (vis[u]) continue;  //再出队跳过
        vis[u] = 1;  //标记u第一次出队
        for (auto ed: e[u]) {
            int v = ed.v, w = ed.w;
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
                q.push({-d[v], v}); //大根堆
            }
        }
    }
}
```

### Bellman-Ford算法

```c++
struct edge {
    int v, w;;
};
vector<edge> e[N];
int d[N], vis[N];

bool bellmanford(int s) {
    memset(d, INF, sizeof(d));
    d[s] = 0;
    bool flag;  //是否松弛
    for (int i = 1; i <= n; ++i) {  //n轮
        flag = false;
        for (int u = 1; u <= n; ++u) {   //n点
            if (d[u] == INF) continue;
            for (auto ed: e[u]) {  //u的出边
                int v = ed.v, w = ed.w;
                if (d[v] > d[u] + w) {
                    d[v] = d[u] + w;
                    flag = true;
                }
            }
        }
        if (!flag) break;
    }
    return flag;   //第n轮=true则有环
}

//SPFA算法
struct edge {
    int v, w;;
};
vector<edge> e[N];
int d[N], vis[N], cnt[N];
queue<int> q; //队列

bool spfa(int s) {
    ::memset(d, INF, sizeof(d));
    d[s] = 0;
    vis[s] = 1;
    q.push(s);
    while (q.size()) {
        int u = q.front();
        q.pop();
        vis[u] = 0;
        for (auto ed: e[u]) {
            int v = ed.v, w = ed.w;
            if (d[v] > d[u] + w) {
                d[v] = d[u] + w;
                cnt[v] = cnt[u] + 1; //记录边数
                if (cnt[v] >= n) return true;
                // v点被更新且不在对内，则入队
                if (!vis[v]) {
                    q.push(v);
                    vis[v] = 1;
                }
            }
        }
    }
    return false;
}
```


|            |              暴力Dijkstra               |              Heap-Dijkstra               |              Bellman-Ford               |                  SPFA                   |
| :--------: | :-------------------------------------: | :--------------------------------------: | :-------------------------------------: | :-------------------------------------: |
| 最短路类型 |                  单源                   |                   单源                   |                  单源                   |                  单源                   |
|  数据维护  |            e[u],d[u],vis[u]             | e[u],d[u],vis[u]  <br>优先队列：距离优先 |            e[u],d[u],vis[u]             |   e[u],d[u],vis[u] <br>队列：时间优先   |
|    算法    |            贪心、松弛、出圈             |          贪心、松弛、入队、出队          |               所有边松弛                |            出队点的出边松弛             |
|   负边权   |                  不能                   |                   不能                   |                   能                    |                   能                    |
|   判负环   |                  不能                   |                   不能                   |                   能                    |                   能                    |
| 时间复杂度 |            O(n<sup>2</sup>)             |                 O(mlogm)                 |                  O(nm)                  |                O(km~nm)                 |
|  图的大小  | 中/小 n=10<sup>3</sup>,m=10<sup>6</sup> |  大/中n=10<sup>5</sup>,m=10<sup>6</sup>  | 中/小 n=10<sup>3</sup>,m=10<sup>4</sup> | 中/小 n=10<sup>3</sup>,m=10<sup>4</sup> |

### 01背包

```c++
//二维
for(int i = 1; i <= n; i++) {    //遍历物品
    for(int j = 1; j <= m; j++)   //遍历背包容量
    {
        //  当前背包容量装不进第i个物品，则价值等于前i-1个物品
        if(j < v[i])	d[i][j] = d[i - 1][j];
            // 能装，需进行决策是否选择第i个物品
        else	d[i][j] = max(d[i - 1][j], d[i - 1][j - v[i]] + w[i]);
    }
}
//一维
for(int i = 1; i <= n; i++) {
    for(int j = m; j >= v[i]; j--)
        d[j] = max(d[j], d[j - v[i]] + w[i]);
}

```

### 完全背包

```c++
//二维
for(int i = 1 ; i <=n ;i++) {
    for (int j = 0; j <= m; j++) {
        d[i][j] = d[i - 1][j];
        if (j - v[i] >= 0)
            d[i][j] = max(d[i][j], d[i][j - v[i]] + w[i]);
    }
}

//一维
for(int i = 1 ; i<=n ;i++) {
    for (int j = v[i]; j <= m; j++) {
        d[j] = max(d[j], d[j - v[i]] + w[i]);
    }
}
```

### 树的构造

```c++
int posorder[N], inorder[N]; //后序，中序
struct node {
    int val;
    node *left, *right;
};
//根据后序和中序构建二叉树
node *build(int h1, int t1, int h2, int t2) {
    if (h1 > t1) return nullptr;
    node *root = new node;
    root->val = posorder[t1];
    int index;
    for (; inorder[index] != posorder[t1]; index++);
    root->left = build(h1, index - 1 - h2 + h1, h2, index - 1);
    root->right = build(index - h2 + h1, t1 - 1, index + 1, t2);
    return root;
}
node *root = new node;
root = build(0, n - 1, 0, n - 1);
//2 3 1 5 7 6 4 后序
//1 2 3 4 5 6 7 中序
//4 1 6 3 5 7 2 层序

//根据前序和中序构建二叉树
node *build(int h1, int t1, int h2, int t2) {
    if (h1 > t1) return nullptr;
    node *root = new node;
    root->val = preorder[h1];
    int index;
    for (; inorder[index] != preorder[h1]; index++);
    root->left = build(h1 + 1, index - h2 + h1, h2, index - 1);
    root->right = build(index - h2 + h1 + 1, t1, index + 1, t2);
    return root;
}
//4 1 3 2 6 5 7 前序
//1 2 3 4 5 6 7 中序
//4 1 6 3 5 7 2 层序
```

### LCA（倍增算法）

```C++
const int N = 5e5 + 10;
int n, m, s, a, b;
vector<int> e[N];
int dep[N]; //dep[u]存u点的深度 
int fa[N][20]; ///fa[u][i]存从u点向上跳2^i层的祖先结点

void dfs(int u, int father) {
    dep[u] = dep[father] + 1;
    //向上跳1，2,4……步的公共祖先
    fa[u][0] = father;
    for (int i = 1; i <= 19; ++i) {
        fa[u][i] = fa[fa[u][i - 1]][i - 1];
    }
    for (int v: e[u]) {
        if (v != father) dfs(v, u);
    }
}

int lca(int u, int v) {
    if (dep[u] < dep[v]) swap(u, v);
    //先跳到同一层
    for (int i = 19; i >= 0; i--) {
        if (dep[fa[u][i]] >= dep[v])
            u = fa[u][i];
    }
    if (u == v) return v;
    //跳到LCA的下一层
    for (int i = 19; i >= 0; i--) {
        if (fa[u][i] != fa[v][i])
            u = fa[u][i], v = fa[v][i];
    }
    return fa[u][0];
}
```

### LCA（tarjan算法）

```c++
const int N = 1e6 + 10;
int n, m, s, a, b;
vector<int> e[N];
vector<pair<int, int>> query[N];
int fa[N], vis[N], ans[N];

int find(int u) {
    if (u == fa[u]) return u;
    return fa[u] = find(fa[u]);
}

void tarjan(int u) {
    vis[u] = true;  //入u时 标记u
    for (auto v: e[u]) {
        if (!vis[v]) {
            tarjan(v);
            fa[v] = u;  //回u时，v指向u
        }
    }
    //离u时，枚举LCA
    for (auto q: query[u]) {
        int v = q.first, i = q.second;
        if (vis[v]) {
            ans[i] = find(v);
        }
    }
}

int main() {
    cin >> n >> m >> s;
    for (int i = 1; i < n; ++i) {
        cin >> a >> b;
        e[a].push_back(b);
        e[b].push_back(a);
    }
    for (int i = 1; i <= m; ++i) {
        cin >> a >> b;
        query[a].push_back({b, i});
        query[b].push_back({a, i});
    }
    for (int i = 1; i <= N; ++i) {
        fa[i] = i;
    }
    tarjan(s);
    for (int i = 1; i <= m; ++i) {
        cout << ans[i] << endl;
    }
}
```

|            |         倍增算法          |            Tarjan算法            |
| :--------: | :-----------------------: | :------------------------------: |
|  数据结构  |     fa[u] [i] ,dep[u]     |   fa[u],vis[u],query[u],ans[u]   |
|    算法    | 倍增法 深搜打表，跳跃查询 | 并查集，深搜，回时指父，离时搜根 |
| 时间复杂度 |       O((n+m)logn)        |              O(n+m)              |

