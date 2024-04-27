## 最小生成树
由最小生成树的定义不难发现,其充要条件即为
对于图上一条边$(x,y)$,生成树上的路径x->y上的所有边权都小于其边权,
否则显然可以将其替换
由此产生了2中求法
### prim
将迪杰斯特拉的距离换成边权即可,加边前判断是否在2个联通块
不在就加边
### kruskal
将所有边按照边权排序,从小到大加边,加边前判断是否在2个连通块
## kruskal重构树
跑最小生成树时将并不连边,而是将边权作为节点的权值建立节点,
这样在新的生成树上仍保留了 *边的权值大小关系* 
即深度越浅(越后加入)的点其所代表的边权值就越大
如可用于找出从x出发,边权值$\leq w$所构成的连通块

或者也可理解成,
让本来各元素间没有等级区分的并查集变成了有等级区分

eg : [Stamp Rally](https://vjudge.net/problem/AtCoder-agc002_d)

## 建图技巧
### 虚点
在遇到点集A要想B中两两连边,$n^2$肯定会超时,
所以可以建一个虚点t,A和B中每个点分别向t连边,
将时间复杂度降至$n$
eg : [团](https://www.luogu.com.cn/problem/P7100)

### 决策
有一类题目在最短路的同时会有一些限制导致边权发生变化,
这个时候可以给一些点建立虚点,
使得走过这些虚点是必然满足限制条件
eg : [robot](https://www.luogu.com.cn/problem/P7407)

## bfs
在类似合并子树时是dfs(便利下一棵子树时已经完整便利了上一棵子树)
与深度(距离)有关时bfs,
bfs的遍历过程带有深度约束 eg : [[YsOI2023] 广度优先遍历](https://www.luogu.com.cn/problem/P9534)
### 0/1bfs
更改了之前错误的认知
0/1bfs需要用双端队列来维护,将边权为0的放到前面,边权为1的放到后面
因为需要将所有的0处理完之后才能处理1
不要与1/2bfs混淆了
eg:[Nastya and Unexpected Guest](https://www.luogu.com.cn/problem/CF1340C)
### 1/2bfs
用单个队列维护即可
对于边权1让其出队两次,第一次出队时不处理,第二次再处理
>注意 :一定不要与0/1bfs混淆,0/1bfs不能这么写,
因为0/1背包中的0加2遍还是0,这样写就错了s
### Dijkstra
节点的遍历顺序(出队列的顺序),就是其离出发点的距离
当dp的更新顺序与距离(值的大小)有关时可以考虑dj/01bfs
## dp
### 图上
当走过的 边数/距离 在状态里时,
图上的dp一个经典优化,
每次拓展一条边就已经便利了所有的情况,不用便利所有的组合
如走5条边不用便利走2条边和3条边,只要便利走4条边与新加1条边就够了
eg:
[Nastya and Unexpected Guest](https://www.luogu.com.cn/problem/CF1340C)
[Autobus](https://www.luogu.com.cn/problem/P8312)

### 前缀和优化
打开思路,前缀和在dp数组中并不必须是连续的,
当需要将一个$O(n)/O(n^k)$*遍历*求得的东西转换为$O(1)$时,
就可以想到前缀和,
而前缀和可以根据需要决定在dp数组中的取得顺序,
eg:[Smaller Averages G](https://www.luogu.com.cn/problem/P10282)

## 单调性
在题目要求维护严格单调递增时,比较难时
一个经典转换:
将每个数都减去它的下标,即可转换为维护单调不降
eg:[Sonya and Problem Wihtout a Legend](https://codeforces.com/problemset/problem/713/C)

同样,让每个数加它的下标,即可把单调不降变为严格单增

## gcd
### 区间
#### 时间复杂度
区间gcd有一个经典结论:
在一个序列上,序列的长度越长,显然区间gcd越小
同时从一个点出发,其变化次数最多为log(V)次
因为gcd每次变化最大变为原来的一半
如当前区间gcd为24,加入一个数使其减小最大为12
#### 求得
显然gcd拥有结合律
所以可有用线段树和st表求得

eg:[Be Geeks!](https://www.luogu.com.cn/problem/P9607)

## 坐标系
在坐标系中,有一些经典结论,
若b在a,c之间,那么有
$dis(a \rightarrow c) = dis(a \rightarrow b ) + dis(b \rightarrow c)$
以及$ dis(a \rightarrow c) > dis(a \rightarrow b )和 dis(b \rightarrow c) $
这启示我们一些题目只要保留相邻的坐标,
即可在后续如dp/最短路/生成树等操作获得全部需要的信息
eg:[SVEMIR](https://www.luogu.com.cn/problem/P8074)

## 约分
对于需要输出分数时需要约分,这时先除再乘可以多算一点
$$
{p1\over q1 } + {p2 \over q2} = {p1*q2 \over q1 * q2} + {p2 * q1 \over q1 * q2}
$$
这个时候可以先让上下都除以$gcd(q1,q2)$再把两者加起来,再约分