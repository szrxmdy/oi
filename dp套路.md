# dp的套路
## 计数类dp
### 排列
在排列时经常会遇到转移与前面构造的排列中的数有关的情况,
#### 需要维护新加入的数与前数的大小关系
这时可以用这种构造排列方式:

当构造到排列$a$的前$i$位时,保证所有$a_j\leq i ,j\in[1,i]$,
当从尾部插入一个数$p,p\in [1,i+1]$时,对所有$a_j\geq p,a_j \leftarrow a_j+1$
这样构造的好处是加入数时不会破坏前排列中每相邻2数的大小关系,同时也能得到全部排列
例题:[abc282_g](https://www.luogu.com.cn/problem/AT_abc282_g)
## 前缀和
当我们需要维护一段区间中a出现的次数>=b出现的次数时,并不需要单独维护a,b前缀和
可以将a记为1,b记为-1,前缀和>=0即为a出现次数>=b
eg:[p3105](https://www.luogu.com.cn/problem/P3105)

## 斜率优化
1.参变分离:
$i$为不变量(当前更新的,决策点改变不会改变的量),$j$为变量
2.**写成 $b = y - kx$ 的形式**:
$b$为要最大/小化的值($f[i]$),与j有关的是$y,x$,与$i$有关的是$k$
即将每个决策j变成了二维坐标上的点
3.由此即最大/小化截距,最终只需要维护凸包即可,
4.如果k有单调性,用单调队列可以做到 O(n),否则二分最优决策点,为O(n*lgn)
eg:[玩具装箱](https://www.luogu.com.cn/problem/P3195)

## 背包类
- 交换状态与答案 
eg : [スタンプラリー 3 (Collecting Stamps 3)](https://www.xinyoudui.com/ac/contest/74700232B00040A022519C6/problem/8650)
- dp退出
类似统计方案这种加入顺序没有影响的题,且没有取max这种阴间转移的题,可以考虑将dp数组回溯到之前的状态
eg : [消失之物](https://www.luogu.com.cn/problem/P4141)
- 枚举顺序

## 区间dp
区间dp常常需要记录上次转移来的位置,从左或右

## 装压dp
### 小技巧
预处理集合大小
```cpp
fq(i,1,S) {
    c[i] = c[i ^ (i & (-i))] + 1;
    或
    c[i] = c[i & (i - 1)] + 1
}
```
自带函数
```cpp
__builtin_popcount(x); //返回int类型数的1的个数
__builtin_popcountll(x); //返回long long类型
```