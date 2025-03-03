## 最小生成树
由最小生成树的定义不难发现,其充要条件即为
对于图上一条边$(x,y)$,生成树上的路径x->y上的所有边权都小于其边权,
否则显然可以将其替换
由此产生了2种求法
### prim
将迪杰斯特拉的距离换成边权即可,加边前判断是否在2个联通块
不在就加边
### kruskal
将所有边按照边权排序,从小到大加边,加边前判断是否在2个连通块
## kruskal重构树
跑最小生成树时将并不连边,而是将边权作为节点的权值建立节点,
这样在新的生成树上仍保留了 **边的权值大小关系**
即深度越浅(越后加入)的点其所代表的边权值就越大
如可用于找出从x出发,边权值$\leq w$所构成的连通块

或者也可理解成,
让本来各元素间没有等级区分的并查集变成了有等级区分

**常常以找到权值 $\leq q$的连通块为标志**

eg : [Stamp Rally](https://vjudge.net/problem/AtCoder-agc002_d)
[Qpwoeirut and Vertices](https://www.luogu.com.cn/problem/CF1706E)

性质 :
- 边权集合都相同,边权互不相同时MST唯一
- 最小生成树上的路径最大值最小,
  因为如果有最大值更小的路径,肯定会形成环,然后把最大值更大的那个值替换掉

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

## floyd
floyd是基于dp的算法,其状态含义为:
f[k][i][j]表示从i -> j 除i,j外只经过前k个点时的最短路
所以当转移点k时,即在所有点对的路径中尝试加入点k
```cpp
for (k = 1; k <= n; k++) {
  for (x = 1; x <= n; x++) {
    for (y = 1; y <= n; y++) {
      f[k][x][y] = min(f[k - 1][x][y], f[k - 1][x][k] + f[k - 1][k][y]);
    }
  }
}
```
不难发现第一维k可以滚动数组滚掉
因为Floyd的状态设计来自路径,
凡是与 **路径** 有关的都要想到Floyd

## 树状数组后缀和
后缀 = 总和 - 前缀
## dp方程设计
在设计dp方程时,先确定最后的目标,
然后根据目标来确定需求,再根据需求来确定状态

状态考虑因素:
1.每次转移改变的状态较少的. eg: 最长上升子序列
2.递推时尽量使用如 1.$f[i][j] = .... $
而非如 2.$f[i + x][j + y] = ...f[i][j]$
因为许多优化都要 1 才能看出来
eg : [基站选址](https://www.luogu.com.cn/problem/P2605)
3.找出的限制尽量有满足了这个限制不会满足其他限制,即有无后效性
eg : [删数](https://www.luogu.com.cn/problem/P5324)
## 线段树
### 写法
线段树开4倍空间是因为最坏情况下将叶子节点补全到2的幂次个
最后在加上非叶子节点为4倍
la的定义: **该节点进行了什么操作没有下传**

注意: **线段树的叶子节点是不能进行down和up的**,
否则会导致RE,并且叶子节点值更新出错,
因此在 add 的最后一步不能加 la 然后 down,
要直接更新 val 和 la
因此当到了查询区间时就不能down了,
down也要修改为更新左右儿子的值
```cpp
void down(int x,int s,int t) {
    tr[L].val += tr[x].la * (M - s + 1);
    tr[R].val += tr[x].la * (t - M);
    tr[L].la += tr[x].la; tr[R].la += tr[x].la;
    tr[x].la = 0;
}
void add(int x,int l,int r,int s,int t,int val) {
    if(l <= s && t <= r) {
        tr[x].la += val;
        tr[x].val += val * (t - s + 1);
        return ;
    }
    down(x,s,t);
    if(l <= M) add(L,l,r,s,M,val);
    if(M + 1 <= r) add(R,l,r,M + 1,t,val);
    up(x);
}
int query(int x,int l,int r,int s,int t) {
    if(l <= s && t <= r) {
        return tr[x].val;
    }
    down(x,s,t);
    int ans(0);
    if(l <= M) ans += query(L,l,r,s,M);
    if(M + 1 <= r) ans += query(R,l,r,M + 1,t);
    return ans;
}
```

### 多个标记
**线段树上能打多个标记的前提是这些标记能够合并**
具体而言,我们可以把标记看作一个已经操作完尚未下传的序列,
而每次下传后想到与在儿子的标记序列后加入了新的标记,
然后我们需要合并这些标记才能保证时间复杂度
注意合并标记时,进行操作时要规定顺序

### 区间历史和
- 法一 : 我们可以多打两个标记,代表$sum += val$的次数和$add$标记对$sum$的贡献
- 法二 : 考虑区间加会对历史版本和如何变化,设一个时间$t$,
  如果没有区间加,版本和$h_i = t * a_i$,
  再记录一个$c_i$表示$h_i = t*a_i + c_i$,即由于区间加造成的变化,
  那么区间$+x$,$c_i = t * x$,也用区间加维护即可

## 启发式合并
对于两个数组以一定要求合并(一般是取较大值)时,
先找出较小的,只遍历较小的那个,在较大的基础上合并,
并返回较大的地址,以获取更低的复杂度
eg : [春节十二响](https://www.luogu.com.cn/problem/P5290)

**更一般的,将2个集合合并时,将siz小的集合暴力插入大的中**
即每次合并用$min(siz[a],siz[b])$的时间
### 时间复杂度
$nlog(n) $,
当一个数做贡献时,其所在集合大小至少变为2倍

## 倍增
想到倍增是难的,但如果知道了这道题是倍增,很多题都会变得显然

倍增常常以 **第一个/最小值达成某个条件** 为标志,
倍增可以看作是一种特殊的二分,所以许多与二分的标志相似.
理解 : 倍增可以看为已经预处理好信息 (check的值) 的二分,
因为不用check的时间,所以能够获得更优的时间复杂度
eg : [国旗计划](https://www.luogu.com.cn/problem/P4155)

倍增还有一种标志,即**凑出某个数/某个区间**
理解: 可以看成是一种二进制拆分
eg : st表
### 树状数组上倍增

## 括号配对
### 判断
以 （ 为 +1, ） 为 -1 ,
从一个 **（** 开始,第一次前缀和为0时意味着找到了他的配对括号
注 : 不是从头开始,防止出现类似  ）（
eg : [钥匙](https://www.luogu.com.cn/problem/P8339)

## 树状数组
### 维护前缀max...
运算只要有结合律就可以用树状数组维护前缀,
如max,|,&,gcd等等... ,但这些没法维护区间,只能维护前缀
### 区间修改 + 区间查询
普通的树状数组只能实现 
区间修改 + 单点查询 或 单点修改 + 区间查询
但我们想要二者兼得,
$$
d[i] = a[i] - a[i - 1] \\
sum[n] = \sum _{i} ^{i \leq n} a[i]\\
=\sum _{i} ^ {i \leq n} d[i] * (n - i + 1)\\
=(n + 1) * \sum _{i} ^{i \leq n} d[i] - \sum _{i} ^{i \leq n} {d[i] * i}
$$
所以只需要维护2颗树状数组,维护 $d[i]$ 和 $d[i] * i$
**本质是交换求和顺序来将重复的加法变为乘法**
### 高维树状数组
通过维度理解高维树状数组很难且难以拓展的 ?
因此此处用树套树的方式来理解
即树状数组的每个点还是一颗树状数组,每一层树状数组都代表了一维
当$qy(x,y)$时即在第一层树状数组qy(x),然后第二层qy(y)
eg : [上帝造题的七分钟](https://www.luogu.com.cn/problem/P4514)

## hash
### 自然溢出和大质数取模
当只有 加法 / 异或 等时可以用自然溢出,
但如果带有乘法,如字符串hash,自然溢出可以被轻松卡掉
### 映射函数
在hash中常将一个小数映射成另一个复杂的数来降低碰撞概率
此处给出一种方法
```cpp
ull g(ull x) {
    x ^= mask;
    x ^= (x << 13); x ^= (x >> 7); x ^= (x << 17);
    x ^= mask; return x;
}
```
其中mask随便选,要大!!!,加ull
通过更改mask合并并不能提供更强的g(),要改移动位数

其本质是 xorshift 取随机数的片段,其有全域形(所有值都可以搞出来),平均性(每个值出现概率相等)的好性质,
其中 mask 在正确性上并没有什么用处,只是将数映射大一点,在类似集合 hash 中加减时更不容易碰撞
### 集合hash
**hash除了比较单个数是否相同,也可以比较数集是否相同**
即给数集中每个数分配一个数,求全部数的异或和为整个数集的hash值
多重集可以考虑相乘 / 加
### 树hash
判断2棵树是否同构
1.将无根树转换为有根树,以树的重心为根,注意树可能有2个重心!!
2.每个点$hash(u) = c + \sum {g(hash(v))} $,
即对每个节点的子树做集合hash

#### 树hash的字符串实现
将子树按照hash值/长度排序后,拼接字符串为 $dep_u + hash_{v1} + hash_{v2} + \dots $
 
## 启发式搜索
估算一下当前情况往后至少要多少代价,如果已经劣于最优解就退出
因为其依赖于最优解,所以要考虑搜索顺序,或者**迭代加深**

## 笛卡尔树
笛卡尔树在维护最大最小值中具有天然优势,
即将序列中的大小关系用树形来维护,因此
**当最大/小值 + 连续子序列时**,要想到笛卡尔树上dp
eg : [Comfortably Numb](https://www.luogu.com.cn/problem/CF1777F)
[Array Collapse](https://www.luogu.com.cn/problem/CF1913D)

## 堆拓展
当多个单调递增的序列求第k大时,
可以将每个序列的第一个数放入堆中,
每次取出堆头,然后在放入取出数所在序列中的下一个数,
重复k次即可

## 连续异或
$a_l\oplus a_{l + 1} \dots\oplus a_n = s_n \oplus s_{l - 1}$
即前缀异或和
**异或可以做前缀!!!**
**异或可以做前缀!!!**
**异或可以做前缀!!!**

## 树的重心
### 定义
删除该节点后分成子树大小的最大值最小(将树平均分为几个子树)
### 性质
1. **重心等价于其子树大小不超过原树的一半**
  证明:若u的子树$siz[v] > n / 2 $,向v移动一定更优
  为换为v后u所在$siz < n / 2$,肯定更优

- 树的重心最多只有2个,且必然相邻
  由1易得

1. 树上所有点到重心的距离和最小
  证明:类似换根dp的思路,u向v走,距离和变化量为$n - 2siz[v]$
  而重心又一直向$siz > n/2$的方向走,距离和一定最小

- 将两树连接新的重心在两树重心路径上
  
5. 树上添加/删除1个节点,其重心最多移动1格

**因为重心的优秀性质,其在各种问题中常被用做根**

## 树的直径
求法
- 两次dfs(只能边权非负)
- 树形dp,记录当前节点的最深深度,
  每次加入一颗子树时$d = max(d,maxn + dfs(v) + w(u,v))$


性质:
- 如果边权为正,树上所有直径有一个公共中点
- 将两棵树合并时,新的直径端点属于原树的直径端点
  具体而言,枚举6种直径可能的情况
  **注意:合并时就算是点交叉者并起来也可以,并不需要原来在两棵树中,**
  **一棵树中的点集合并也满足该性质**
  证明可以画出两点集的直径,枚举四个点所有连接方式,运用反证发现都满足该性质
- 一个点离树上最远的点是直径的一个端点

## 欧拉序
树上遍历除了dfs序还有欧拉序,
也就是在进入和退出时都加入序列
**这样相较于dfs序的优点是其出现之后有他们父亲节点的信息,**
### O(1) 的 lca
记录每个点第一次出现位置,
两点出现位置深度最浅的点就是他们的lca,用st表维护就是O(1)

## 分治妙用
### 向上合并答案
### 向下统计答案
分治能够到达每个叶子节点,
在向下过程中另一半区间不会再遍历到了,
所以在递归一个区间时加入另一个区间的答案
也可以看作是另一种倍增 ? 对求和顺序的优化 ?
**能够处理加入容易,删除困难的情况**
eg : [求和42](https://www.xinyoudui.com/ac/contest/74700232800040A022526E6/problem/7258)

#### 进一步
**加入很容易,删除很难,如操作类题目,**
**可以对时间建立线段树,**
得到某个一个操作在哪些时间会存在,
遍历叶子节点

**注意 : 要求满足操作具有交换性**

## 换根dp
换根dp的转移很复杂,给出一般转移方式
- 在dfs开始时重新更新u的节点信息
- 在遍历边过程中从u中删去v的贡献
- dfs后重新加上v的贡献
- 在结束时从u中删去fa贡献

大部分时候不用4步全写,简单dp大都可以合并几步

### 常见转移
1.f来自max,换根时记录最大值和次大值
2.f来自加减乘除,注意乘除时不一定有逆元,用前后缀积

## 除以数
**转移有加法时候除以数不一定逆元(刚好是p的倍数),**
所以要灵活的利用前后缀积避免除法
eg : [Road Improvement](https://www.luogu.com.cn/problem/CF543D)

## 贪心
### 邻项交换
很多贪心题只要排序一下就结束了,但是很难看出来,这里给出满足排序即可的条件

考虑原序列的相邻两项,分别算出交换前后的价值$a,b$,
**通过化简使不等式的一边只与一项有关,**
**这样其就满足了传递性**
证明其满足传递形后就可以排序做了

eg : [国王游戏](https://www.luogu.com.cn/problem/P1080)

#### 依赖类
这类贪心往往是在树形结构上,选一个点必须要选前面一些点,
这个时候可以先整体考虑,考虑全局最优,
该点在选完其前一个点后一定会选他,然后把这两个点合并为一个点
用邻项交换分析合为整体后的权值
eg : [给树染色](https://cdn.acwing.com/problem/content/description/117/)

### 反悔贪心
贪心时每次都选择,知道不能选时去掉前面一个更劣的换成这个
eg : [Work Scheduling G](https://www.luogu.com.cn/problem/P2949)

## 欧拉路径
Hierholzer算法:
去除s的一条出边和t的一条入边,使图有欧拉回路
先找到一个环,去除图中环上所有边,然后再从环上的点出发出发寻找新的环,递归进行,可得到欧拉回路
小奇技淫巧:
在一张所有入度出度都相等的有向图上找环,
一直走第一条边一定能找到环,即不会对边进行反悔不选
证明: 假设出发点为u,经过了点v
当到达v时,如果进入了v i次，那么最多从v出去i-1次,此时v必然还有出边
因而当无路可走时一定停在了u的位置,即找到了一个环

优化: 上述算法时间复杂度为O(n+m)但编码难度大,
既然我们知道无脑深搜一定可以找到环(找到环的顺序对正确性没有影响),
那么递归找环没有必要,直接dfs爆搜就可以了

### 倒序输出
在找欧拉回路/路径的时候按照出栈顺序倒着输出 ,
可以考虑为在退栈过程中为遍历的点给拼上去

## 区间内数出现次数为偶数
异或拥有同一个数异或两次为0的优秀性质,
**有一类题询问区间内是否数的出现次数为2次**
**为了防止冲突,可以给每个数随机赋一个新数,然后看区间异或值是否为0**

## 随机化
当想出一些暴力做法后,如果没有什么思路想正解,可以考虑一些减枝,然后对遍历顺序随机化来骗更多的分数

## 龟速乘
为了防止乘法越界,可以用龟速乘,变乘变加
```cpp
int mul(int x,int k) {
    int ans(0);
    while(k) {
        if(k & 1) ans += x;
        k >>= 1; x += x;
    } return ans;
}
```

## johnson全源最短路(负边使用dijkstra)
普通的迪杰正确性依靠加点的顺序和dis的顺序相同来保证正确性,
但负权边导致后加入的点可能dis更小,所以就错掉了

朴素的思想是给所有边都加一个定值谁,使其变为正的,最后再减掉,
但这样就错了,因为这样会导致较短的路径明明不优却被选为最短路

我们需要一种增加方法使得$s\rightarrow t$间所有路径无论长短都加了相同的长度
注意到这和物理学中势能的定义完全相同,
所以可以给每个点规定一个势能,将边权变为$w + h_u - h_v$
问题来到如何规定势能使得所有边权非负呢 ?
$w \ge dis_v - dis_u$ ,
所以选一个点做spfa,用每个点的dis做势能即可
为了防止图不联通,可以用超级源点向每个点连一条权为0的边
这可以用spfa中直接入队实现

### 原始对偶算法
费用流用spfa可以说是非常慢了,如何用dj呢 ?
我们可以用上述的思路也规定一个势能来进行,
问题在于每次会增加一些负权边怎么办 ?
注意到增加的负权边全部来自最短路径上,所以可以考虑用新的dis来更新h
具体而言 $h_u += dis_u$,
对于原来的边$$w + h_u - h_v \ge dis_v - dis_u \rightarrow
w + dis_u + h_u - h_v - dis_v \ge 0$$
对于新的边$$dis_v = dis_u + d(u,v) + h_u - h_v\\\rightarrow h_v + dis_v - (h_u + dis_u) + d(v,u) = 0 $$

eg : [Bakery](https://www.luogu.com.cn/problem/AT_arc137_e)

## 线性基
有数集$S$,求一个最小的数集,使这两个集合能异或出的数集相同
考虑一个个加入数,如果 x 可以被前面的数表示出来,那么x就没必要加入,
如何知道x是否能被前面的数表示出来 ? 

拆位进行考虑,即x的每位1要与奇数个该位为1的数异或,
奇数很难维护,我们希望每位1的数都只有1个,有因为$a\oplus b,a,b$只要保$a\oplus b,a$即可,所以这是可以做到的

具体而言,第 i 位放一个最高位为 i 的 $d[i]$ ,当插入x时,从高到低试着消去其最高位,直达其有一位消不掉时放入集合
```cpp
    fq(i,1,n) {
        ll x; cin >> x;
        fr(i,49,0) {
            if(!((x >> i) & 1)) continue;
            if(d[i]) x ^= d[i];
            else {d[i] = x; break;}
        }
    }
```

### 第k大
如果每一个有数值的都是2的幂次,我们可会求了,直接对k二进制拆分即可,
问题在于异或上某些值可能变小,导致我们无法直接二进制拆分
为了使异或值一定使答案变大,我们可以预处理,把高位的其他位位1的全变成0
```cpp
fr(i,50,0) fr(j,i - 1,0) if((d[i] >> j) & 1ll) d[i] ^= d[j];
```
### 求交

## 类欧几里得
### 特例引入
log(n)求$$f(p,q,r,n) = \sum_{x = 0}^n \lfloor \frac{px + r}{q} \rfloor$$
取整想到除法分块,但是变得是被除数,且带了个常数,所以每个x的值可能都不一样,直接错了

在坐标轴中画出这条直线, 
一个显然的想法是:
每q个方格形成了一个循环,内部东西一模一样,
所以可以轻易的转换为等差数列求和
但如果q非常大还是做不了~~~

**所以问题关键是如何把p,q变小 ?**
类似gcd的做法
- $$f(p,q,r,n) = f(p\%q,q,r\%q,n) + \lfloor \frac{p}{q} \rfloor \frac{n(n + 1)}{2} + (n + 1)\lfloor \frac{r}{q}\rfloor$$
- $p < q,r < q$ ,我们需要调换p,q的位置,
  $$\begin{aligned}
  f(p,q,r,n) & = \sum_{x = 0}^n \sum_{j = 0}^{\lfloor \frac{px + r}{q} \rfloor - 1} 1\\
  & = \sum_{j = 0}^{\lfloor \frac{pn + r}{q} \rfloor - 1}\sum_{x = 0}^n [j + 1\le \frac{px + r}{q}]\\
  & = \sum_{j = 0}^{\lfloor \frac{pn + r}{q} \rfloor - 1} \sum_{x = 0}^{n} [x > \frac{jq + q -r - 1}{p}]\\
  & = \sum_{j = 0}^{\lfloor \frac{pn + r}{q} \rfloor - 1}n - \lfloor\frac{jq + q - r - 1}{p}\rfloor\\
  & = \lfloor \frac{pn + r}{q} \rfloor n - f(q,p,q - r - 1,\lfloor \frac{pn + r}{q} \rfloor - 1)
\end{aligned} $$

为什么r 也要 mod q呢 ? 因为在转换为限制移项时要求 $q-r-1\ge 0$
**推式子时,特别是限制类时要时刻关注每个变量的可行范围**

### 整理思路
考虑上一个特例我们实际进行了什么

1. 我们发现当p,q很小时答案可以很快求得,因此考虑将p,q减小
2. p 可以拆位 $\lfloor p / q\rfloor q + p \mod q$,其中前一位可以和下面的q消掉,
   因此可以变为$p\% q,q $
3. 类似 gcd 的思路,交换p,q位置,这可以通过把贡献全拆为1后限制转移做到
   **因为转换为限制后就可以进行移项等操做,所以类似更换函数内位置的操做都通过转限制做到**

**当式子中出现类似$\frac{a}{b}$时就可以考虑类欧算法了,**
因为$\lfloor \frac{a}{b} \rfloor b$可以被直接消掉,就很递归

## 树形背包的复杂度证明
**一定要写成这样形式**,
**即复杂度为siz已经加入的 * 新的siz**
```cpp
fr(i,siz[u],0) fq(j,1,siz[v]) f[i + j] = f[i] .. f[j]. ..
```
进行节点u时复杂度
$siz_{v_1} + siz_{v_1}siz_{v_2} + (siz_{v_1} + siz_{v_2})siz_{v_3} + \dots $,
即
$$\begin{aligned}
  \sum_i siz_{v_i}\sum_{j > i}siz_{v_j} & \le (\sum siz_{v_i})^2 - \sum siz_{v_i}^2
\end{aligned}$$
其加上子节点$siz_v^2$的复杂度,复杂度恰为$siz_u^2$

## ac自动机
ac自动机的重要思想是通过修改 ch 来达到快速跳fail,
具体而言,正常时,为了获得一个fail,我们需要一直跳fail直到节点1或遇到fail + c,
但如果一个点没有儿子 c ,我们将儿子 c 直接修改为 fail + c即可,
这样只要跳一次 fail 即可,因为该 fail 中已经包含了所有往前跳 fail 的信息
```cpp
void build() {
    tr[0][0] = tr[0][1] = 1;
    queue<int> q; q.push(1);
    while(q.size()) {
        int u(q.front()); q.pop();
        for(int c : {0,1}) {
            int v(tr[u][c]);
            if(v) {q.push(v); fail[v] = tr[fail[u]][c]; ed[v] |= ed[fail[v]];}
            else {tr[u][c] = tr[fail[u]][c];}
        }
    }
}
```


## ac自动机上dp
**有一类问题,要求统计字串中(不)出现一些串的字符串个数**

考虑简单情况,如果不能出现的字符串 t 只有一个怎么办,
$f[i][j]$ 表示长度为i,构造出的字符串后缀和 t 前 j 个字符相同的情况,
每次加入字符 c ,就是在匹配t[1 - j] 的后缀 + c 与 t 的匹配,这时kmp模板

如果不能出现的有多个,放到ac自动机上就可以啦
用$f[i][j] $ 表示长度为 i ,后缀在字典树上所在节点为 j 即可
eg : [[JSOI2007] 文本生成器](https://www.luogu.com.cn/problem/P4052)

## 珂朵莉树
珂朵莉树是用来处理颜色覆盖的好工具,有split 和 assign 两个函数
```cpp
auto split(int x) { // 代表分裂出一个以 x 开头的区间
    auto it = st.lower_bound({x,0,0});
    if(it != st.end() && it->l == x) return it;
    --it; if(it->r < x) return st.end();
    auto [l,r,v] = *it; st.erase(it);
    st.insert({l,x - 1,v}); 
    return st.insert({x,r,v}).first;
}
void assign(int l,int r,int v) {
    auto itr = split(r + 1),itl = split(l);
    st.erase(itl,itr); st.insert({l,r,v});
}
```
set 在插入/删除其他迭代器时该迭代器不失效
**注意 : 一定要先split(r + 1),否则可能会把分裂出的区间删掉**

## 容斥反演
容斥定理过于抽象,因此为了更方便的运用,我们抽象出常用的反演形式来快速证明其正确性

### 核心等式
$$\begin{aligned}
\sum_{T\in S} (-1)^{|S| - |T|} & = \sum_{T \in S} [(|S| - |T|) \mod 2 = 0] - \sum_{T\in S} [(|S| - |T|) \mod 2 = 1]\\
& = \sum_{i = 0}^{|S|} {|S| \choose i}[i \mod 2 = 0] - {|S| \choose i}[i \mod 2 = 1]\\
& = [|S| = 0] = [S = \emptyset]
\end{aligned} $$
推论 :
固定 H , S $\sum_{H \in T \in S}(-1)^{|S| - |T|} = \sum_{T \in (S \oplus H)} = [H = S] $

### 原始形式
令全集为 $U$
- 枚举子集
$$f_S = \sum_{T \in S} g_T \Leftrightarrow g_S = \sum_{T\in S} (-1)^{|S| - |T|} f_T $$
证明 : 
$$\begin{aligned}
g_S & = \sum_{T\in S} (-1)^{|S| - |T|} \sum_{H\in T} g_H\\
& = \sum_{H\in S} g_H (\sum_{H\in T\in S} (-1)^{|S| - |T|})\\
& = \sum_{H\in S} g_H [H = S]
\end{aligned}$$
- 枚举超集
$$f_S = \sum_{S \in T \in U} g_T \Leftrightarrow g_S = \sum_{S \in T \in U} (-1)^{|T| - |S|} f_T $$
证明同上

### 二项式反演
**本质是如果集合的每个元素等价,我们可以由枚举子集变为枚举子集大小**
其常常用于将恰好的限制减少为 至多/少 来简化

- 枚举子集 :
$$f_n = \sum_{i = 0}^n {n\choose i} g_i \Leftrightarrow g_n = \sum_{i = 0}^n (-1)^{n - i} {n\choose i} f_i $$
- 枚举超集 :
$$f_k = \sum_{i = k}^n {i \choose k} g_i \Leftrightarrow g_k = \sum_{i = k}^n (-1)^{i - k} {i\choose k} f_i$$

### min - max 反演
不妨假设集合中每个数都不同(相同可以强制钦定大小),下式中 $kthmax(S)$ 只有 $|S| \ge k$时才有意义,否则自动被排除枚举序列
我们用一个集合子集的最小值来求出该集合中的第 k 大值,不妨设 $f_T$ 为容斥系数
$$kthmax (S) = \sum_{T \in S} f_T * min(T) \Leftrightarrow f_S * min(S) = \sum_{T\in S}(-1)^{|S| - |T|} kthmax (T) $$
如果 $kth_max(T) $ 与 S 中最小值 a 不同时,$kth_max(T)\Delta \{a\} = kth_max(T)$,
所以所有 $kth_max \neq min(S)$ 的都会被消掉,不会作出贡献,而有 ${|S| - 1 \choose k - 1} $ 个子集最小值为 $min(S)$,
所以有 $f_S = (-1)^{|S| - k} {|S| - 1 \choose k - 1} $ ,带入$f_S$
$$kthmax(S) = \sum_{T \in S} (-1)^{|T| - k}{|T| - 1\choose k - 1}min(T) $$
特别的常用的 ,
$$max(S) = \sum_{T \in S}(-1)^{|T| - 1}min(T) $$

使用方式 : 
我们将第 k 大的值用最小值表示出来,普通数集好像没什么用,
但在类似计算期望时常常使用,
由于概率不独立,我们无法计算出集合中某个元素的期望,但我们能得到

### 子集反演

### 斯特林反演

### 单位根反演

## 李超线段树
先考虑简单的线段树,每个区间维护了区间中点的最大值,
但注意到标记并不好进行合并,两条线段可能一部分为 a 大,另一部分 b 大
**所以考虑标记永久化,记录使当前区间中点最大的线段,**
每次加入一条线段时,只要去更新新线段较大的那一端即可,
因为另一端仍然使用该标记对答案没有影响
```cpp
void insert(int x,int l,int r,int s,int t,line li) {
    int mid((s + t) >> 1);
    if(l <= s && t <= r) {
        if(calc(tr[x],mid) < calc(li,mid)) swap(tr[x],li);
        if(s == t) return ;
        if(calc(li,t) > calc(tr[x],t)) insert(R,l,r,mid + 1,t,li);
        if(calc(li,s) > calc(tr[x],s)) insert(L,l,r,s,mid,li);
        return ;
    } 
    if(l <= mid) insert(L,l,r,s,mid,li);
    if(mid + 1 <= r) insert(R,l,r,mid + 1,t,li);
}
double qy(int x,int p,int s,int t) {
    if(s == t) return calc(tr[x],s);
    int m((s + t) >> 1);
    double ans = calc(tr[x],p);
    if(p <= m) ans = max(ans,qy(L,p,s,m));
    else ans = max(ans,qy(R,p,m + 1,t));
    return ans;
}
```

### 合并
李超树合并 x,y 时先把 y 节点的线插入到树 x 中,然后再合并左右两颗子树,
由于时间复杂度来自于线段下移,而每条线段最多移动 log 次,因此时间复杂度为 $nlog n$

## 平衡树
**写平衡树时要先确定 rk ,不能merge 时再取随机数**
如果要现场生成随机数需要给随机数的概率改为$\frac{siz_u}{siz_u + siz_v}$

## wqs 二分
**wqs二分常常以恰好选 m 个物品为标致,我们能够快速求出没有选 m 个限制时的答案**

记 f(x) 为选 x 个物品时的方案数,如果 f(x) 构成凸函数,就可以使用 wqs 二分

对于一个上凸来说,一条斜率为 k 的的直线和其切点存在单调性,
同时其切点是使这条直线截距最大的点,
所以当我们知道斜率时,我们相当于要求最大的$f(x) - kx$,
所以我们将每个物品的价值 - k ,求没有物品限制时的最大值即可.
我们求出切点后如果切点 x 过大就增大斜率,反之亦然,
而斜率可以通过二分来确定,最终得到切点恰为 x 时的 f(x) 即为答案

注意 : 要处理多点共线的问题,可以钦定共线点的较小那个作为切点
eg : [忘情](https://www.luogu.com.cn/problem/P4983)

## cdq 分治
考虑一个操做序列,有删除有增加,我们只会处理增加,不会处理删除

可以先处理完左边的,处理出左边整体插入的,再处理右边

## 线段树分治
对操做序列构建线段树,在某个插入存在的节点内插入,
询问相当于从根到叶子节点的一条路径

## 凸包
一次函数可以将 k 和 x 互换,转换维护点或线
可以用平衡树来维护线的形式,具体而言,按照斜率 k 插入
点一般用单调栈维护

## 带修主席树
注意到主席树其实是一种前缀和的思想,所以要修改可以树状数组套权值线段树

## 决策单调性
处理类似 $f_i = \min f_j + w(j,i),j < i $ 的做法
我们成第一个最小的 j 是决策点,如果决策点随 i 增大而增大,我们有一些套路性的优化
- $f_i = \min g_j + w(j,i),j < i$,即转移与 f 无关,我们可以递归来做
 即每次求出区间终点 $f_{mid}$ 的值,然后得出决策点 k ,
 区间左半之后只要扫$[s,k]$,右半扫$[k,t]$,这样每层递归只要扫 O(n) 的点,共 log层,
 总复杂度 n \log n
 eg : [Ciel and Gondolas](https://codeforces.com/problemset/problem/321/E)
- $f_i = \min f_j + w(j,i),j < i$,即转移从自己过来,
  没法直接优化,因为我们无法快速得到下一个决策点是哪个,需要更多的性质

### 四边形不等式
$\forall a \le b \le c \le d,\\w(a,c) + w(b,d) \le w(a,d) + w(b,c) \\\Leftrightarrow w(a,d) - w(a,c) \ge w(b,d) - w(b,c) $
即交叉优于覆盖 或 较早的增加同样长度其增加值 $\ge$ 较晚的(越来越劣)

性质 :  如果满足四边形不等式,一定满足决策单调性
证明 : 感性理解,随着 i 的增加,靠后的本来不优,但会逐渐优于前面的

优化自转移 dp :
对于两个决策点 j1,j2,由于越来越劣,
我们知道一定有一个 x 使得,$\le x$ 时,j1较优,否则j2较优,
而 x 可以通过二分来求得,
我们需要维护一个相邻 x 单增的 j 序列,
因为如果有 j1,j2,j3 使得 j1 和 j2 间 x $\ge$ j2 和 j3 间,j2一定不会被取到,
这用一个单调栈维护即可
而 i 从那个点转移过来用二分 或者 双端队列维护一下即可

### 优化区间dp
如果$\forall l1 \le l2 \le l3,r1 \le r2 \le r3$,满足决策点$k1 \le k2 \le k3$,则该区间满足决策单调性
这样转移 $f[i][j]$ 时从 $k[i + 1][j]$ 枚举到 $k[i][j - 1]$ 即可,时间$n^2$,

如果$f[i][j] = min(f[i][k] + f[k][j]) + w(i,j) $ ,
且 w(i,j) 满足四边形不等式,
且 $\forall a \le b \le c\le d,w(b,c) \le w(a,d)$,
则f[i][j]决策单调

## FWT
规定$\sum_i$ 表示 $\sum_{i = 0} ^ {n - 1}$
FWT 指使用类似 fft 的方法来实现位运算卷积,
要求$C_n = \sum_{n = i \bigodot j} A_i * B_j $
我们需要构造一种方式 F ,使 $FC_i = FA_i * FB_i$ , 然后再$F^{-1}$ 回去
$$\begin{aligned}
    &FA_i = \sum_j w(i,j) A_j &\\
    &FC_i = \sum_j w(i,j) C_j &\\
    & = \sum_j\sum_{j = u \bigodot v} w(i,j) A_uB_v\\
    & = \sum_{u}\sum_{v}w(i,u\bigodot v) A_uB_v \\
    & = \sum_u\sum_vw(i,u)w(i,v)A_uB_v
\end{aligned} $$
即构造一个 w ,满足$w(i,u)w(i,v) = w(i,u \bigodot v) $ 即可
$\bigodot$ 是某种位运算
因为位运算各位数之间互不影响,
要让$w(i,\overline{nu})w(i,\overline{mv}) = w(i,\overline{nu} \bigodot \overline{mv}) $,
只需定义$w(i,\overline{uv}) = w(i,u)w(i,v) $即可,
对于 i 要增加位数,也可以如此定义
### 正变换
$$\begin{aligned}
    FA_i & = \sum_j w(i,j)A_j \\
     & = \sum_{u = 0}^{n / 2 - 1} w(i,u)A_j + \sum_{v = n / 2}^{n - 1}w(i,v)A_j
\end{aligned}$$
发现 u,v 只有最高位不同,假设 i 最高位为 i0,i,u,v去除最高位后为 i',u',v'
$FA_i = w(i0,0) \sum_{u = 0}^{n / 2 - 1}w(i',u')A_{u'} + w(i0,1)\sum_{v = n / 2}^{n - 1}w(i',v')A_{v'} $
$FA_i = w(i0,0)FA_{0i'} + w(i0,1)FA_{1i'} $
我们直接提取最高位出来,源于上述对 w 拓展位数时的定义,因此可直接分配律提取

### 逆变换
类似 fft,在分析逆变换时,使用矩阵工具,要求
$$\begin{bmatrix}
    w_{00} & w_{01} & w_{02}  & \dots \\
    w_{10} & w_{11} & w_{12} & \dots \\
    w_{20} & w_{21} & w_{22} & \dots\\ \dots
\end{bmatrix}^{-1}$$
注意到该矩阵的其他值都是由左上4个值相乘得到,
如果我们求出左上4个值得逆矩阵,
手模发现其得到拓展矩阵和该矩阵相乘也应该是单位矩阵
所以我们只要构造出 2 * 2 得矩阵即可
注意不能构造一个全1矩阵或一列一行全为0,没有逆元
比赛时忘记了某种构造把 1,-1,0 都带一下即可
### 异或
正变换矩阵
$$\begin{bmatrix}
    1 & 1\\1 & -1
\end{bmatrix}$$
注意到该 w 等价于 $FA_i = \sum_j (-1)^{popcount(i \& j)} A_j $
逆变换矩阵
$$\begin{bmatrix}
    0.5 & 0.5\\0.5 & -0.5
\end{bmatrix}$$
```cpp
void FWT(ll a[],int lim,int fg) {
    if(lim == 1) return ;
    ll a0[lim >> 1],a1[lim >> 1];
    for(int i(0); i < (lim >> 1); ++i) a0[i] = a[i];
    for(int i(0); i < (lim >> 1); ++i) a1[i] = a[i + (lim >> 1)];
    FWT(a0,lim >> 1,fg); FWT(a1,lim >> 1,fg);
    for(int i(0); i < (lim >> 1); ++i) {
        a[i] = (w[fg][0][0] * a0[i] + w[fg][0][1] * a1[i]) % P;
        a[i + (lim >> 1)] = (w[fg][1][0] * a0[i] + w[fg][1][1] * a1[i]) % P;
    }
}
```

### 对位相加与FWT变换
**如果将两个数组相同位置相加,其 FWT 数组相同位置也相加**
证明 : 
令 $C_i = A_i + B_i$ , $FA$ 表示 $A$ 经过变换后的数组
$$\begin{aligned} 
FC_i & = \sum_j w(i,j) C_j \\
& = \sum_j w(i,j) (A_j + B_j) \\
& = (\sum_j w(i,j) A_j) + (\sum_j w(i,j) B_j) \\
& = FA_i + FB_i
\end{aligned}$$

注意 fft 也满足该性质

## 失配树
字符串上每个点向其最长公共前后缀连边,得到一颗失配树
性质 : 
- 从点 i 到根的路径代表了 s[1 : i] 的所有border
- 点 i 的子树大小为 s[1 : i] 的出现次数

## lct

## 多重背包的单调队列优化
$f_i = max{g_{i - k w_i} + kv_i},k \in [0,c_i]$
我们希望不用枚举 k ,考虑对于固定的 j,取不同个数的影响
$$\begin{aligned}
f_{j} & = g_j\\
f_{j + w_i} & = max(g_{j + w_i},g_j + v_i)\\
f_{j + 2w_i} &= max(g_{j + 2w_i},g_{j + w_i} + v_i,g_{j} + 2v_i)\\
\dots &
\end{aligned} $$
同时 $+ v_i$ 并不会改变相邻两个数的大小关系,所以可以单调队列优化,对每个 j 都建立单调队列
注意自己的坐标也要取 max

## 点分治
处理树上**路径问题**时常常使用点分治
其本质是通过**合理的容斥快速合并答案**

可以将路径分为经过点 u 和不经过的,每次选一个点处理经过 u 的,删除 u,
希望删除 u 后每个子树尽量小,每次选重心即可

考虑如何统计答案,如果直接再 u 处合并每个子树,和直接dfs没有区别,每次合并都和路径长度有关,会爆炸
我们进行高妙容斥,先不管路径合法的限制,钦定每个路径都是 s - u - t 的,不管路径是否合法
这样统计路径时我们只需要求解一次 u 到子树中所有点的距离再做处理即可,
然后去除不合法的路径,只需要对 u 的所有子树在做一次上述操做,贡献为 - 即可,注意坐标上加上 u - v 的距离

eg : [【模板】点分治 1](https://www.luogu.com.cn/problem/P3806)

## 边分治
每次选边将树分成尽量平均的两部分,
但遇到菊花图就直接爆炸了,所以需要优化建图.
常用的方法是对于度数超标的点用类似线段树的方法重建图,把其儿子作为线段树的叶子,线段树间建 0 边

## Slope trick
该 trick 常常用于维护斜率为 O(n) 级别的凸包进行各种操作
**其常常与有绝对值的式子一起出现**

**考虑两个凸包相加时斜率的变化 : 某点所在线段的斜率即为原来两凸包该点线段的斜率**
因此如果我们需要维护斜率的变化,可以改为维护拐点,每个拐点代表斜率变化了 1 ,
所以相加两个凸包只要简单的合并拐点即可

对于取 min / max 操作则使用人类智慧考虑其对拐点的影响,这
些往往与斜率为 0 的两个拐点 L / R 有关

eg : [[APIO2016] 烟火表演](https://www.luogu.com.cn/problem/P3642)
[序列 sequence](https://www.luogu.com.cn/problem/P4597)

## 竞赛图
竞赛图指不管方向为完全图的有向图,记有方向的完全图
性质 :
- 将其缩点后其形成的类似一条链的 DAG , 链上每个点向后面的每个点有边(由两两点之间都有边,显然成立)
- 其每个强连通块都存在一条哈密顿回路
证明 : 考虑连通块有三个点时显然成立,每次加入一个点时,该性质依然成立
- 其存在一条哈密顿路径
- 在 > 1 的连通块中长度为 [3,n] 的简单环均存在
证明 : 把新加入的点放在原来的环中间,必然有出边和入边交界的地方,从地方出去,走一圈回来即可构造出来
- 竞赛图的充要条件为,将出度排序后,出度第 i 大记为 $s_i$,有 $\sum_{i \le k} \ge {k \choose 2}$,且$\sum s = {n \choose 2} $
证明 : 
必要性 : 对前 k 个点来说,出度最少的情况是无指向为遍历到的点的边,即为 ${k \choose 2} $ 条边贡献了出度
充分性 : 考虑根据出度构造竞赛图
... 不会 不知道 完蛋啦

## 线段树分裂 / 合并
对于**值域相交的树合并,常常使用线段树分裂/合并**
```cpp
void split(int x,int &y,int val,int s = 1,int t = n) { //val on origin
    if(val == t) {y = 0; return ;} y = ++cnt;
    if(val <= M) {tr[y].rs = tr[x].rs; tr[x].rs = 0;}
    if(val < M) split(tr[x].ls,tr[y].ls,val,s,M);
    if(val > M) split(tr[x].rs,tr[y].rs,val,M + 1,t);
    up(x); up(y);
}
```
注意需要特判 val = 0 的情况

对于线段树合并而言,其复杂度为重复的节点个数,最多有 n log n 个重复节点,而分裂不会制造重复节点,
因此线段树分裂 / 合并的均摊复杂度为 n log n 的
eg : [【模板】线段树分裂](https://www.luogu.com.cn/problem/P5494)

### 值域相交平衡树的合并 / 分裂
平衡树值域相交合并复杂度为 $n log^2 $
具体而言,合并时将**不作为根的那棵树按照根的大小分裂再向两个子树递归合并**
大佬的证明 : [题解 & 平衡树合并](https://www.luogu.com.cn/article/p4ejw9j6)

## dp套dp
在一些约束条件非常复杂的计数类dp中,
我们常常设计出判定形dp,然后以该dp的状态和值作为状态跑计数dp
eg : [小 N 的独立集](https://www.luogu.com.cn/problem/P8352)
eg : [[TJOI2018] 游园会](https://www.luogu.com.cn/problem/P4590)

## 前缀线性异或基
该技巧由于快速得到区间 $[l,r]$ 的线性基,其基本思路是贪心的使构成线性基的数尽可能靠后
具体而言,插入数字时,如果能够当前插入的数坐标比该位置上的数坐标更大,就交换出来,

正确性证明 : 
考虑新数 x 可以由线性基中 $d_1 \oplus d_2 \oplus d_3 \dots$ 得到,
那么 x 可以替换 $d_i$ 中的任意一个数,且只能替换一个,
在做的过程中我们会且只会遍历到每个$d_i$ , 那么我们就将他们中坐标最小的换出来了

处理能异或出 0 的小细节,如何记录最大的能够异或出 0 的坐标呢 ? 
每次插入新数时如果最后能消完,就用当前换出来的最小的那个坐标和原来能异或出 0 的坐标取大的即可
这依然从  $x = d_1 \oplus d_2 \oplus d_3 \dots$ 考虑
```cpp
void insert(ll val,int pos) {
    fr(i,62,0) if(val >> i & 1) {
        if(d[i]) {
            if(pos > id[i]) swap(id[i],pos),swap(d[i],val);
            val ^= d[i];
        } else {d[i] = val; id[i] = pos; return ;}
    } if(!p_0 || pos > p_0) p_0 = pos;
}
```

模板 : [Ivan and Burgers](https://codeforces.com/contest/1100/problem/F)

## 树上 ddp
树上修改一个节点时只会修改一条链上的 dp 值,所以先树剖一下,
令 s 表示其重儿子,
将所有轻儿子对 u 的贡献记为 $g_u$ ,将 $f_u$ 表示为 一个$g_u$ 的矩阵 * $f_s$ 的矩阵得到,
当我们需要某点 f 值时,只需要求点到链底的矩阵相乘即可,用线段树可维护,
同时由树剖只换 log 次链,所以我们只会修改 log 个点的 g 值
写的时候注意 f 和 g 的维护,维护增量时要注意其是否是已被修改,
注意 : **加法 & min/max 矩阵的矩阵尽量避免涉及到无穷的加减,非常容易爆炸,能不用就不用,线段树只建议小分讨**
模板 : [【模板】"动态 DP"&动态树分治](https://www.luogu.com.cn/problem/P4719)

## 杜教筛
杜教筛用于在线性时间一下求解积性函数的前缀和,其用一个便于求前缀和的辅助积性函数 g
令 $s_n = \sum_{i = 1}^n f_i $
$$\begin{aligned}
\sum_{i = 1}^n (f * g)(i) & = \sum_{i = 1}^n \sum_{d | i} g(d)f(\frac{i}{d}) \\
& = \sum_{d = 1}^n g(d) \sum_{i = 1}^{\lfloor \frac{n}{d} \rfloor} f(i) \\
& = \sum_{d = 1}^n g(d) s(\lfloor \frac{n}{d} \rfloor)
\end{aligned} $$
要求 $s(n)$ 有 $g(1)s(n) = \sum_{i = 1}^n (f * g)(i) - \sum_{d = 2}^n g(d)s(\lfloor \frac{n}{d} \rfloor)$
用除法分块枚举 d ,提前处理 $\le 10^6$ 的前缀和并记忆化搜索

- 求 $\mu$ 前缀和
$\mu * \Iota = \epsilon \Rightarrow s(n) = 1 - \sum_{d = 2}^n s(\lfloor \frac{n}{d} \rfloor) $
其中$\Iota(n) = 1$ , $\epsilon(n) = [n = 1] $
- 求$\phi$前缀和
$\phi * \Iota = id \Rightarrow s(n) = \frac{n(n + 1)}{2} - \sum_{d = 2}^n s(\lfloor \frac{n}{d} \rfloor)$
其中$id(n) = n$

## 全局平衡二叉树
其本质是一个静态的 lct , 比lct好写 , 并且比树剖少一个 log
先将树进行重链剖分,然后将每条重链建成一颗平衡树,像 lct 一样轻边认父不认子

但直接建成平衡树时间还是 2 个log,
朴素的想法是一个重链上某个点如果练了更多的轻点,要将他向上提一点位置,
所以每次取轻点 siz 和 + 1 的带权中点,
此时每向上一个重边,轻点 + 链上点(即整棵树) 的 siz 和翻倍,所以最多走 log 个条重链,
因此总时间复杂度是 log 的

树上修改记录一下每棵树的根节点,然后从根节点开始类似 fhq 那样修改,但不用分裂
eg : [[LNOI2014] LCA](https://www.luogu.com.cn/problem/P4211)

## 矩阵 - 树定理
该定理用于求解一个图的生成树个数

### 一些基础约定
#### 图的关联矩阵(用矩阵来描述一个图的边集和点集的关系)
一张图由点集和边集构成,用矩阵描述图可以将点放在行上,将边放在列上,$M_{i,j}$ 为点 i 个边 j 的关系
$$M_{i,j} = \begin{cases} 1 & 边j 是 点i 的入边 \\ -1 & 边j 是 点i 的出边 \\ 0 & 边 j 与点 i 没关系 \end{cases} $$

##### 关联矩阵与秩与连通性
我们发现如果图是一个 n 个点的弱连通块,那么其关联矩阵的秩是$n - 1$ , 
证明 : 
秩 $\le n - 1$ 是显然的,直接把所有点加起来就会发现显然是一个全 0 ,那么我们减取任意一个点就能表示出这个点
秩 $\ge n - 1$ 用类似思路证明,问题就是选出只用 $\le n - 1$ 个只通过加减能否得到全 0 的一行,
显然是不行的,因为是连通块,一定有一条边连接了未被选择的点集和选择的点集,那这一列永远无法消为 0

#### 拉普拉斯矩阵(其实就是更改了一点的邻接矩阵😒)
$L_{i,j} = M * M^{T} $,其中 $M^T$ 为 M 的转置矩阵,则有
$$L_{i,j} = \begin{cases} i 的度数 & i = j \\ - (i 连向 j 的边数) & i \neq j \end{cases} $$

证明 : 
- $i = j$ , 则 $L_{i,i} = \sum_k M_{i,k} M^T_{k,i} = \sum_k M_{i,k}^2 $
- $i \neq j$ , 则 $L_{i,j} = \sum_k M_{i,k}M_{j,k} $ , 就是枚举边,看他是不是 i 的出边且是 j 的入边

### 前置理论
我们找到一棵生成树,就是要在关联矩阵的列中选出 n - 1 个列作为边集,
(我们记 $M[S]$ 为 $M$ 保留边集合为 S 的关联矩阵. 为了方便,记$M[S]^T$ 为 M^T[S])

那么问题就是判断 $M[S]$ 的连通性,
那么我们只要 $M[S]$ 的秩为 n - 1 即可,
因此就是将 $M[S]$ 随便扔掉一行,然后其 $\det M[S] \neq 0$ 即可 

同时 $\det M[S] = 1 或 -1$ , 

所以我们就是要求出所有 $|S| = n - 1$ 的 $\sum (\det M[S])^2$ 即可

### Cauchy-Binet 定理
A 为 $n * m $ 的矩阵 , B 为 $m * n$ 的矩阵
$\det(AB) = \sum_{|S| = n} (\det A[S])(\det B[S]) = \sum_{|S| = n} \det (A[S]B[S]) $

### 矩阵 - 树定理本体 !!!
令 $L_0$ 为 L 将 $L_{i,i}$ 全部置为 0 的矩阵,
$$\begin{aligned} \det L_0 & = \det (M * M^T) \\
& = 
\end{aligned}$$

## 可撤销结构的指针写法
对于可撤销结构如 并查集 , 线段树分治 等,
记录具体的操作在逆过来是一种方式,但非常麻烦,
**我们可以直接记录所有数的改变情况,这样就比较好写且不容易出错**
```cpp
stack<pair<int*,int>> stk;
void upd(int &x,int y) { stk.emplace(&x,x); x = y; }
void reset(int n) { while(stk.size() ^ n) {*stk.top().first = stk.top().second; stk.pop();} }
```
然后直接再改变数时用 upd 就可以做到可撤销
```cpp
void merge(int u,int v) {
    u = find(u) , v = find(v);
    if(u == v) return ;
    if(siz[u] < siz[v]) swap(u,v);
    upd(siz[u],siz[u] + siz[v]); upd(fa[v],u);
}
```
其他类型数据结构的也可以用相同方式做到

## 常见分块技巧
- 一个块中最多只有块长个不同的值
如我们求解区间增减颜色,查询区间内每个颜色有多少个时可以使用

## tarjan = 逆拓扑序
强连通分量 tarjan 求出来的颜色就是逆着的拓扑序,
所以要求拓扑序就可以直接把颜色倒过来即可

## 范德蒙特卷积
$$\sum_{i = 0}^m {a\choose i}{b\choose m - i} = {a + b\choose m} $$
考虑组合意义就是分成两组数来选择
同时可以拓展到拆分为更多个数.

## Boruvka 算法
每次对每个点找到离他最近的点连起来,
然后变成最多 n / 2个联通快,把每个联通快看作一个点重复上述过程

## 回滚莫队
对于便于添加但不便于删除的询问询问,
先和普通莫队一样,最短点根据块排序,一个块内的左端点右端点坐标递增,
然后每次如果不换块,就把左端点放到该块的最右侧然后向左移动,
加入右端点时我们需要维护的就是当前块右侧到 r 的答案

## 有趣小事实
$2^31 - 1$ 是质数

## 线段树上二分
实现线段树上二分时考虑叶子节点的特殊情况(返回 -1, +1, s ...) , 以及普通节点往 左/右

## prufer 序列
prufer 序列用于将有 n 个点的**有标号无根树**和一个长 **n - 2 的每个位置值域为 $[1,n]$ 任意的序列**进行映射
其是树形态计数中的常用方式

### 从树到序列
每次去除一个编号最小的叶子节点,将其父亲的编号放到当前序列的末尾,然后把该节点删去

### 从序列到树
我们先通过序列中每个数的出现次数得到每个编号的度数,
然后每次选择一个编号最小的叶子节点,将其和序列首位连边,然后把序列首位删除,并将其度数 -1

由于每个序列中的点会贡献 1 的度数,最终总度数是 2n - 2 , 而每次确定一个点,总度数 -2 ,因此每个序列总能构造出一棵树

[实现](https://www.luogu.com.cn/problem/P6086)

显然一棵树对应一个序列,一个序列对应一棵树,同时树 a 映射的序列反映射回去还是 a , 所以构成了双射

### Cayley 公式
对于完全图来说 , 方案数即为 $n^{n - 2} $

进一步拓展能得到联通块的方案数 : [Clues](https://codeforces.com/contest/156/problem/D)
**其标准方式是先分配度数,然后根据度数得到每个编号在序列中的出现次数,得到方案数**

## [矩阵树定理](https://www.luogu.com.cn/problem/P6178)
### 内容
对于有向图来说,设其邻接矩阵为 A , 我们将 $A_{i,i}$ 减去 i 的出度再取反得到矩阵 K ,
则以 u 为根的内向生成树的个数即为 K 去除 u 行 u 列后的行列式

对于无向图来说显然把每个无向边拆为两个有向边是一样的

### 证明
考虑如果边集 E 构成了以 u 为根的内向生成树的充要条件
- 除了 u 外每个点至少有一个出度
- 将有向图变为无向图后图中没有环

如何用矩阵的行列式判断是否满足条件 ?
我们构造 边集 - 点集 矩阵来代表一个可能的生成树,
矩阵的第 i 行表示第 i 个点,第 j 列表示第 j 条边,矩阵去除了根的行

对于第一个条件,构造矩阵 S,如果边 i 为 $u \rightarrow v$ , 则 $S_{u,i} = 1 $ , 
则 S 满足其行列式为 1 当且仅当矩阵中每个点至少有一个出度

对于第二个条件,构造矩阵 T,如果边 i 为 $u \rightarrow v$ , 则 $T_{u,i} = 1,T_{v,i} = -1 $
显然,如果 T 代表的图中有环,则 T 的行列式一定为 0 , 
因为绕着环上的边加一圈,一定能得到一个全是 0 的列,那么矩阵就不满秩,
如果没有环,其行列式一定为 1 , 考虑归纳证明,
只有一条边显然正确,
加一个点时我们直接把该点放在矩阵的最后一行,
1 刚好在最后一行最后一列,而 -1 在上三角,并不会影响行列式的值

因此我们大小为 n - 1 的边集 $\det S \det T = \det (S\times T) = 1$ 则该边集合构成合法内向生成树

由 柯西 - 比内 定理 , 如果对整张图的边集构建 S 和 T,
答案即为 $\det S \times T = \det K $

## hall 定理
二分图存在完美匹配 $\Leftrightarrow$ 左侧任意取点集 |S| , 其连的右侧点集 |T| 满足 $|T| \ge |S|$
证明 : 
必要性显然,充分性考虑归纳法
假设左侧有 n 个时满足该定理,尝试递推到 n + 1,
即证明 n + 1 个点满足该定理时,让 n + 1和右侧某个点配对后剩下的 n 个点仍满足该定理

记 $f(S)$ 为左点集 S 能导出的右点集,
令 S 为 $\{1,2,\dots,n\}$ 的子集,可分为两种情况讨论
- $|f(S)| \ge |S| + 1$
则 n + 1 与任意右侧点配对任满足 $|f(S)| \ge |S|$
- $|f(S)| = |S|$
则 n + 1 必须与点 $x \notin f(S)$ 配对才能满足 $|f(S)| \ge |S|$ , 
即要证明 $\exists x$ , 满足 $\forall S,|f(S)| = |S|$ , 有 $x\notin f(S)$ , 
考虑反证,
如果该定理不成立,则必然可以有多个相交为空集的 S 满足 $f(S_1) \cup f(S_2) \dots = $ 全集 U,
但我们知道 $|f(S)| = |S|$ , 所以 $|f(S_1) \cup f(S_2) \dots | \le n $ , 而 $|U| \ge n + 1 $,矛盾

证毕

## 质因数分解与线性代数
数论题中将数映射到一个每个维度为质数的坐标系中是常见的方法,

eg : [【模板】Dirichlet 前缀和](https://www.luogu.com.cn/problem/P5495)
做高维前缀和即可,即枚举每个维度然后累加

## LGV 引理
对于一张有向无环图,有大小为 n 的不相交点集 A 和 B , 设计矩阵,
$M_{i,j} = \sum_{S : A_i \rightarrow B_j} \prod w(S_j) $,
即 $A_i \rightarrow B_j$ 所有路径的边权的乘积,
则 $\det M$ 为使 $A_i \rightarrow B_i$ 的路径不相交的所有方案的边权积的和

证明 :
感性理解一下 : 
$\det M = \sum_{p} (-1)^{\sigema (p)} \prod M_{i,p_i} $

若有路径 $A_1\rightarrow B_1$ 和 $A_2\rightarrow B_2$ 交点为 C,
则最终统计时 $(A_1B_1)(A_2B_2) - (A_1B_2)(A_2B_1)$ 刚好抵消,贡献为 0

若在一个方案中有交点,我们交换这个交点的两条路径对应关系其贡献值不变,因此会在 $(-1)^{\sigema (p)}$ 中使贡献为 0