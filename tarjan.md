## 强联通分量
### dfs树
tarjan是基于dfs的算法,所以先了解dfs树
有向图中,dfs树是一颗叶向树(所有边都指向叶子方向)
非树边可分为 横叉边,返租边,前向边
显然需要 **返祖边** 才可以形成强联通分量,
而剩下的边则会拓展强连通分量的大小
### tarjan
我们的目标是找到返祖边包括的点,
以及通过横叉边和前向边附着在 返租边包括的点 的点
$dfn[x]$为x的dfs序,
$low[x]$为该子树中通过非树边能够到达的dfs序最小的点
如果$low[x] < dfn[x]$,说明该子树中有返租边,
当$low[x] == dfn[x]$,有2中情况:
1.该节点的子树中没有返租边,那么其自己为一个强联通分量
2.该子树中最远的返祖边到该节点,其所处强连通分量深度最小的点即为x
无论是那种情况,都意味着该子树的强连通分量已经求得了
```cpp
void tarjan(int x)
{
    dfn[x] = ++cnt;
    low[x] = dfn[x], inv[x] = 1, st.push(x);
    for (int to : mp[x])
    {
        /*如果已经搜过了且不在栈内，那么说明该条边连向一个已经缩点后的点,即这是一个横叉边,不能用来缩点*/
        if (!dfn[to]) {tarjan(to); low[x] = min(low[to], low[x]);}
        else if (inv[to]) {low[x] = min(low[x], dfn[to]);}
    }
    if (low[x] == dfn[x])
    {
        ++point;
        while (1)
        {
            color[st.top()] = point;
            inv[st.top()] = 0;
            if (st.top() == x)
            {
                st.pop();
                break;
            }
            st.pop();
        }
    }
}
```

## 割点和割边
**无向图中的dfs树不可能存在横叉边**
因此边双缩完后时一颗树,点双是园方树,而有向图则是DAG

**常以没有公共点/边的路径为标志**
**不经过重复的点/边为标志**

### 边双代码:
显然,割边一定是树上的点且$low[v] > dfn[u]$
```cpp
namespace TJ {
    vector<int> mp[N];
    int dfn[N],ctd,col[N],ctc,low[N];
    bool inst[N];
    stack<int> st;

    void dfs(int x,int fa) {
        dfn[x] = low[x] = ++ctd;
        st.push(x); inst[x] = 1;
        for(auto &to : mp[x]) {
            if(to == fa) continue;
            if(inst[to]) {low[x] = min(low[x],dfn[to]);}
            if(!dfn[to]) {
                dfs(to,x);
                low[x] = min(low[to],low[x]);
            }
        }
        // cout << x << " " << dfn[x] << " " << low[x] << "#\n";
        if(dfn[x] == low[x]) {
            ++ctc;
            while(st.top() != x) {
                col[st.top()] = ctc;
                inst[st.top()] = 0;
                st.pop();
            } col[st.top()] = ctc; 
            inst[st.top()] = 0; st.pop();
        }
    }

    void build() {
        dfs(1,1);
        fq(u,1,n)
            for(int &v : mp[u]) 
                if(low[v] > dfn[u]) {
                    TR::tr[col[u]].push_back(col[v]);
                    TR::tr[col[v]].push_back(col[u]);
                }
    }

}
```

### 点双
**一个点可以属于多个点双而一条边只能属于一个点双**
因此点双相对边双的关键在于如何处理公用的点,
我们用圆方树!!!!

#### 圆方树
定义 :
**将原图上的点双看作一个方点,然后将原图上双连通分量中的点向这个方点连边**
性质 :
- 圆点代表了了原图上的点,一个方点连的所有原点代表了一个点双
- 方点只连圆点,圆点只连方点
- 度数 > 1的圆点代表了原图上的一个割点


**注意开2倍空间**
**注意点双是不经过父亲节点而不是不经过这条边,所以dfs中不用忽略父亲**

代码:
```cpp
namespace TJ {
    vector<int> mp[N];
    int dfn[N],ctd,low[N];
    stack<int> st; bool inst[N];

    void dfs(int x) {
        // cerr << x << " ";
        dfn[x] = low[x] = ++ctd;
        st.push(x); inst[x] = 1;
        for(auto &to : mp[x]) {
            if(!dfn[to]) {
                dfs(to);
                low[x] = min(low[x],low[to]);
                if(low[to] == dfn[x]) {
                    ++TR::cnt;
                    while(1) {
                        int p = st.top();
                        st.pop(); inst[p] = 0;
                        TR::tr[p].push_back(TR::cnt);
                        TR::tr[TR::cnt].push_back(p);
                        // cout << p << " " << TR::cnt << "#\n";
                        if(p == to) {break;}
                    }
                    TR::tr[x].push_back(TR::cnt);
                    TR::tr[TR::cnt].push_back(x);
                    // cout << x << " " << TR::cnt << "#\n";
                }
            }
            if(inst[to]) low[x] = min(low[x],dfn[to]);
        }
    }

    void build() {

        fq(i,1,m) {
            int u,v; cin >> u >> v;
            mp[u].push_back(v);
            mp[v].push_back(u);
        }
        TR::cnt = n;
        dfs(1);

    }
}
```