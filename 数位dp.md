```cpp
#include<bits/stdc++.h>
#define fq(i,d,u) for(int i(d); i<=u; ++i)
#define fr(i,u,d) for(int i(u); i>=d; --i)
#define sf scanf
#define pf printf
#define int long long
using namespace std;
int a,b;
int ans1[10],ans2[10];
int f[20][10][10];  //长i,开头是k,任意取,数码var出现的次数
//如果 k = 0 代表开头可以有前导0
void init(){
    fq(k,0,9) f[1][k][k] = 1;
    fq(i,2,19) fq(nk,0,9) {
        fq(lk,0,9)  fq(var,0,9) f[i][nk][var] += f[i - 1][lk][var];
        f[i][nk][nk] += pow(10,i - 1);
    }
}
void solve(int x,int ans[]){
    int dig[19],l(0); memset(dig,0,sizeof(dig));
    while(x) {dig[++l] = x % 10; x /= 10;}
    
    /*位数不同,没有前导0*/
    fq(i,1,l - 1) fq(k,1,9) fq(var,0,9) ans[var] += f[i][k][var];
    /*位数相同,第一位更小*/
    fq(k,1,dig[l] - 1) fq(var,0,9) ans[var] += f[l][k][var];
    /*位数相同,第i位没有贴上届,前面的贴住了上届*/
    fr(i,l - 1,1) {
        fq(k,0,dig[i] - 1) fq(var,0,9) ans[var] += f[i][k][var];
        fr(nl,l,i + 1) ans[dig[nl]] += dig[i] * pow(10,i - 1);
    }
    fq(i,1,l) ++ans[dig[i]];
}
signed main(){

    init();
    cin >> a >> b;
    solve(a - 1,ans1),solve(b,ans2);
    fq(i,0,9) cout << ans2[i] - ans1[i] << " ";

    return 0;
}
```