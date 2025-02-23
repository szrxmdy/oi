## FFT
从初三就听过了,但一直没学,每次学着学着就开始发呆,浪费了大量时间,此处给出整理

对于一个多项式而言,点值表示法易于加减乘除,
而系数表示法虽然有很大应用,但直接乘是$n^2$的,
所以要将系数表示法转换为点值表示法来乘法,然后转换回去

#### 复数根的性质
用$w_n^k表示x^n的第k个解,log_2 n \in \Z$,令$m = n/2$
- $(w_n^k)^2 = w_m^k$
- $w_n^{m + k} = -w_n^k$
- $(w_n^k)^i = w_n^{ki}$

#### 转换为点值
以下公式i都从0开始
$$A(x) = \sum_i^{n-1}a_ix^i = \sum_i^{m-1}a_{2i}x^{2i} + x\sum_i^{m-1}a_{2i + 1}x^{2i}\\
=A_0(x^2) + xA_1(x^2)$$
$$k < m,A(w_n^k) = A_0(w_m^k) + w_n^kA_1(w_m^k)\\
k > m,A(w_n^{m+k}) = A_0(w_m^k) - w_n^kA_1(w_m^k)
$$

这样可以把$T(n) = 4T(n/2) + n \rightarrow 2T(n/2) + n$
#### 从点值转换为系数
写成矩阵显然可以做到
若 $b_k = \sum_{i = 0}^{n - 1} a_iw_n^{ki} $ , 则
$a_k = \frac{1}{n} \sum_{i = 0}^{n - 1} a_iw_n^{-ki}$

## ntt
注意到 fft 关键是构造出一些 $w_n^k$ 满足性质 :
- $(w_n^k)^i = w_n^{ki}$
- $(w_{ni}^{ki}) = w_n^k $
- $w_{n}^m = -w_n^0 = -1$

我们可以使用原根来做,记$w_n^k = g^{\frac{P - 1}{n}k} $
前两个显然满足,考虑最后一个 
原根为 $g^i$ 互不相同的 g, $w_n^m = g^{\frac{P - 1}{2}}$,因为$g^{P - 1} = 1$,而两者不同,故$g^{\frac{P - 1}{2}} = -1$

将原式中的 w 换成原根即可,注意模数要选取一个 2 的幂次 + 1,
eg : $998244353 = 2^{23} \times 7 \times 17 + 1 $

原根求解 : 

**注意 : fft做做乘法是如果要保留前 n 位,不能只对前 n 位做乘法,要先求出所有位的答案,然后再舍去后几位**

code : 
```
void ntt(int lim,int a[],bool fg) {
    if(lim == 1) return ;
    int a0[lim >> 1],a1[lim >> 1];
    for(int i(0); i < lim; ++i) if(i & 1) a1[i >> 1] = a[i]; else a0[i >> 1] = a[i];
    ntt(lim >> 1,a0,fg); ntt(lim >> 1,a1,fg);

    ll w(1),w1 = qpow(3,(P - 1) / lim); if(fg) w1 = qpow(w1,lim - 1);
    for(int i(0); i < (lim >> 1); ++i,w = w * w1 % P) {
        a[i] = (a0[i] + 1ll * w * a1[i]) % P;
        a[i + (lim >> 1)] = (a0[i] - 1ll * w * a1[i]) % P;
    }
}
```

### 迭代写法
考虑最底层位置 i 的编号,由于每次递归把当前最低位为 0 的放左边(最高位置为0),为 1 的放右边(最高位置为 1),
因此在 i 位置上的即为 i 的二进制位反过来的数,最底层放好后一层层递归上去是简单的
```cpp
    static int r[S]; r[0] = 0; 
    for(int i(1); i < lim; ++i) r[i] = r[i >> 1] >> 1 | (i & 1) << (lg - 1);
    
    void ntt(int lim,int *a,bool mode,int *r) { //mode == 1 ? intt else ntt
        for(int i(0); i < lim; ++i) if(i < r[i]) std::swap(a[i],a[r[i]]);
        for(int len(2); len <= lim; len <<= 1) {
            int w1 = (mode ? qpow(g,P - 1 - (P - 1) / len) : qpow(g,(P - 1) / len));
            for(int i(0),w(1); i < (len >> 1); ++i , w = 1ll * w * w1 % P) 
                for(int bg(0); bg < lim; bg += len) {
                    const int x = a[i | bg] , y = a[bg | i | (len >> 1)];
                    const int z = 1ll * y * w % P;
                    a[i | bg] = add(x,z);
                    a[bg | i | (len >> 1)] = add(x,P - z);
                }
        }
    }
```

## 多项式求逆
已知 f , 求解 $f * g = 1 (\mod x^n)$
显然多项式存在逆的前提是常数项不为0,

考虑牛顿迭代求解,即求 $h(g) = f - \frac{1}{g} = 0$ 的解,
$g_{i + 1} = g_{i} - \frac{h(g_i)}{h'(g_i)} $
对 h 求 g 的导, $\frac{dy}{dg} h = g^{-2} $,
$$\begin{aligned}
g_{i + 1} & = g_i - \frac{f - g_i^{-1}}{g^{-2}} \\
& = 2g_i - fg_i^2
\end{aligned} $$
牛顿迭代每次精度翻倍,因此每次算完后保留上次精度 * 2 的项数,
初始时精度为一项,即 $ g_0 = [0]f^{-1} $

如果这次迭代完后精度为 n ,那么我们取前 n 项的 f 和 $g^2$ 卷积得到 $2n$ 项后舍去后面的项,只保留前 n 项

### 精度问题 
多项式求逆时并不能保证到无穷项都为 0 , 
只能够保证前 n 项满足为 100...0,n 就是多项式的精度,
而牛顿迭代保证了每次精度翻倍

## 多项式求ln
求解 ln f 显然不能直接将 f 带入 ln ,
考虑到多项式求导和积分是非常容易的,而 $\frac{dy}{dx} ln = \frac{1}{x}$,
所以我们可以先对 ln 求导,带入 f 是好做的,然后再将 ln f 积分回去

## 多项式求 exp
同理进行牛顿迭代

## 多项式中牛顿迭代的正确性

## 多项式整除和取余
对于多项式 f , g , 存在唯一的 Q 和 R 满足 $f = g * Q + R$ , 其中 $deg R < deg G$ , deg 为次数
如果我们写成长除法的形式,不难发现 Q 和 R 的分解是唯一的

> 值得一提,多项式的最大公因式可以用欧几里得来求,即 $gcd (f,g) = gcd(g,f \mod g)$

应用长除法的方法,我们可以推广很多初等数论的结论
- $(f\mod p) * (g\mod p) = (f * g) \mod p $
- $(f + g) \mod p = (f \mod p) + (g \mod p) $

### 循环卷积
对于多项式 f , 如果要将指数为 km + i 的项数贡献到 i 上,直接 $f \mod x^m - 1 \rightarrow f $ ,
证明可以考虑模拟长除法的过程,在满足 km + i 项的值时,会通过 -1 贡献到 (k - 1)m + i 项,对于负指数是类似的.
由于取模对乘法的分配律,因此常用于对环上问题转化为类似序列的多项式变形

### 求解方法