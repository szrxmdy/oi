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
        /*如果已经搜过了且不在栈内，那么说明该条边连向一个已经缩点后的点,
        即不在返租边的覆盖范围内,不能用来缩点*/
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