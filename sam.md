## sam自动机
sam是一个DAG,边权是字母,
从起点出发能够跑出字符串s的所有字串
### 朴素思想
要想跑出字符串的所有字串,
可以直接建一颗字典树,然后把所有的后缀都插进去,
但这样是时空都是$n^2$的,思考如何进行优化

先从简单的考虑,字符串abcd,
显然,插入完abcd后就不用插入bcd了,
直接从原点连向代表ab的点即可,边权为b
剩下的后缀同理优化,
具体而言,我们在插入节点时额外从原点连一条边即可

这样就结束啦~~ 了 吗 ?
如果字符串abcb,那就直接寄了,
因为原点没法连出2条边权相同的边,
否则就失去了字典树快速便利的意义,

我们在原思想上进行修改,
原来之所以能快速插入,
是因为到节点abc,还能跑出bc,c,
这样插入d时只要连一条边就可以跑出除d外的所有后缀,
此时再从原点连一条d即可,
我们希望保留这个优美的性质,
进一步发现如果节点是如abcb时,我们肯定无法跑出其所有后缀,
但我们能够跑出其一部分后缀,
此时结合自动机思想,那一个link指针指向其不能跑出的后缀,
这样加入新节点时一直跳link,连边即可

### 实现
记起点为1号,代表跑出了后缀""
对于每个点记录len,link,
表示其能跑出长度为$tr[link].len - len$的后缀,
当我们加入一个节点时只要一直跳link,连边即可
如果跳到一个点已经连过该边了,连向q,那么此时分为2种情况
1.$tr[q].len = tr[p].len + 1$,那么直接将np的link连向q即可
2.$tr[q].len > tr[p].len + 1$,
那么此时就要新建一个点nq,其len为$tr[p].len + 1$
此时nq与q的位置冲突了,但是没关系,
发现nq是q的一个后缀,所以我们然nq来替代q的位置,然后将q的link连向nq,
同时要记得把前面连向q的边改为连向nq以符合定义,
注意q的孩子也要复制

```cpp
class Sam {
public:
    class nd{public: int ch[26],len,link;} tr[N << 1];
    int cnt,last;

    Sam() {last = cnt = 1;}
    void add(char c) {
        c -= 'a';
        int np = ++cnt,p = last; tr[np].len = tr[last].len + 1;
        last = np;
        while(p && !tr[p].ch[c]) {tr[p].ch[c] = np; p = tr[p].link;}
        if(!p) {tr[np].link = 1; return ;}

        /*此时走到节点np,能够跑出长度为tr[p].len + 1 ~ tr[np].len的后缀*/
        int q = tr[p].ch[c];
        if(tr[q].len == tr[p].len + 1) {tr[np].link = q; return ;}

        /*此时nq和q的位置冲突了,注意到nq是q的后缀,所以用nq代替q的位置*/
        int nq = ++cnt;
        tr[nq] = tr[q]; tr[nq].len = tr[p].len + 1;
        tr[q].link = tr[np].link = nq;
        while(p && tr[p].ch[c] == q) {tr[p].ch[c] = nq; p = tr[p].link;}
    }
    void build(const string &s) {
        int n = s.size();
        fq(i,0,n - 1) {add(s[i]);}
    }
    bool find(const string &s) {
        int n = s.size() , p(1);
        fq(i,0,n - 1) {p = tr[p].ch[s[i] - 'a']; if(!p) return 0;}
        return 1;
    }
} sam;
```