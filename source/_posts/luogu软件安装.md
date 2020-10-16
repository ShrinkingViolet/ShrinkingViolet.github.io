---
title: HAOI2010软件安装|题解
date: 2020-09-12
tags:
 - 题解
 - tarjan
 - 背包
categories:
 - 算法
mathjax: true

---

[原题链接](https://www.luogu.com.cn/problem/P2515)

***
我：“这题居然还是个紫题诶。”
同机房的lw同学：“这也配称为紫题？看我分分钟A了它。”
一个晚上过去了...
lw同学：“我*，这题我居然想用kruskal。算了等明天早上再来A了它。”

***
嗯，说了点题外话，现在言归正传。  
这是一道好想但不好写的树形dp。  

# 预处理  
从题目可以看出，由原先的依赖关系建成的图可能会包含环，比如d\[1\]=2,d\[2\]=3,d\[3\]=1，这时1,2,3便形成一个独立的联通块并且构成环，而环里面的软件是相互依赖的，想要装上1就必须把1,2,3都装上，因此我们可以在dp前先缩点，新的点的$w=\sum_e w[e]$，$v=\sum_e v[e]$。  
之后，选定一个点向每一个入度为0的点连一条边，此时的图就转化成了一棵以该点为根节点的树了。

# 动态规划
预处理就到这里，接下来就是dp函数了。  
我们设***ans\[u\]\[w\]***为以u点为根节点的树w体积下所能取得的最大的价值。  
遍历***u***的每一个儿子***v***并算出***ans\[v\]\[i\](i$\in$\[0,m\])***。由于每个软件只能够安装一次，因此这里我们选择对它进行01背包。  

# 一些细节
值得注意的是，为了确保每个体积下所然装得软件都包含根节点，也就是所有子软件包所依赖的软件包，dp之后我们从后往前令***ans\[u\]\[i\]=ans\[u\]\[i-W\[u\]\]+V\[u\]***。从后往前时，前面的体积尚未加上根节点，可以确保根节点只被添加一次，而如果从前往后添加则会导致后面的体积下安装的软件安装两次根节点。
# 完整代码
```
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <stack>
const int MAX = 2000;
int n,m,cnt,depth,Bcnt,ww[MAX],vv[MAX],dd[MAX],Bnum[MAX],dfn[MAX],low[MAX],head[MAX],W[MAX],V[MAX],ans[MAX][MAX];
bool vis[MAX];
std::stack<int> s;
struct edge
{
    int u, v, next;
} e[MAX];
void add(int u, int v)
{
    ++cnt;
    e[cnt].u = u;
    e[cnt].v = v;
    e[cnt].next = head[u];
    head[u] = cnt;
}
void input()
{
    memset(head, -1, sizeof(head));
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", ww + i);
    }
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", vv + i);
    }
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", dd + i);
        if (!dd[i])
            continue;
        add(dd[i], i);
    }
}
void tarjan(int u)
{
    dfn[u] = low[u] = ++depth;
    vis[u] = 1;
    s.push(u);
    for (int i = head[u]; i != -1; i = e[i].next)
    {
        int v = e[i].v;
        if (!dfn[v])
        {
            tarjan(v);
            low[u] = std::min(low[u], low[v]);
        }
        else
        {
            if (vis[v])
            {
                low[u] = std::min(low[u], dfn[v]);
            }
        }
    }
    if (low[u] == dfn[u])
    {
        int j;
        ++Bcnt;
        do
        {
            j = s.top();
            s.pop();
            vis[j] = 0;
            Bnum[j] = Bcnt;
        } while (j != u);
    }
    return;
}
void dfs(int u)
{
    for (int i = head[u]; i != -1; i = e[i].next)
    {
        int v;
        dfs(v = e[i].v);
        for (int f_w = m; f_w; f_w--)
            for (int s_w = f_w; s_w; s_w--)
                ans[u][f_w] = std::max(ans[u][f_w], ans[u][f_w - s_w] + ans[v][s_w]);
    }
    for (int i = m; i; i--)
        if (i >= W[u])
            ans[u][i] = ans[u][i - W[u]] + V[u];
        else
            ans[u][i] = 0;
}
int main()
{
    input();
    for (int i = 1; i <= n; i++)
    {
        if (!dfn[i])
            tarjan(i);
    }
    memset(head, -1, sizeof(head));
    cnt = 0;
    for (int i = 1; i <= n; i++)
    {
        W[Bnum[i]] += ww[i];
        V[Bnum[i]] += vv[i];
        if (Bnum[i] == Bnum[dd[i]])
            continue;
        if (!dd[i])
            continue;
        add(Bnum[dd[i]], Bnum[i]);
        vis[Bnum[i]] = 1;
    }
    for (int i = 1; i <= Bcnt; i++)
    {
        if (vis[i])
            continue;
        add(Bcnt + 1, i);
    }
    dfs(Bcnt + 1);
    printf("%d", ans[Bcnt + 1][m]);
    return 0;
}
```
代码有点丑将就着看看吧。