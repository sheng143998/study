# 倍增和st表

## 1.引入

我们先要知道如何求一个数最接近2的几次幂

```c++
int log2(int n)
{
    int ans=0;
    while((1<<ans)<=(n/2))
    ans++;
    return ans;
}
ans就是2的次方数，我们可以举几个例子假设我们求15最接近2的几次幂
15/2=7
ans一开始为0，1<<0=1<=7,ans=1,1<<1=2<=7,ans=2,1<<2=4<=7,ans=3,1<<3=8>=7,ans=4,此时跳出循环
答案就是2^4最接近15
```

  ![image-20250415172011016](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250415172011016.png)

我们构建出来的这张表就叫ST表 ST[i] [p]代表当前在i位置，向右跳$2^p$步，最远能到达哪个点

那我们如何构建呢

对于一个普遍的情况，ST[i] [p]，我们可以先来到**==i+2^(p-1)^==**位置，此时我们已经走了2^(p-1)^,我们再在此基础上走2^(p-1)^步

就相当于走了2^p^步，举个例子，假设我们求ST[2] [2],一共要走4步，我们可以先走2步来到4这个位置，已经走了两步，然后就是求ST[4] [1]，就是在4位置再走两步能到多远的位置，所以可以推出公式，ST[i] [p]=ST[i+1<<(p-1)] [p-1]

对于每个ST值都依赖它上一列的某个ST值，所以我们可以按照列数从左往右去填

那么如何计算任意两点，最少跳几步能到达呢

我们举例说明

假设我们要从17到37，两个点距离为20，最接近20的数为16，16可以分解为5个2进制位，我们从最高位开始

看第4二进制个位，ST[17] [4]=38的话，这个就不取，来到第3个二进制位 ST[17] [3] =27话， 我们ans+=2^3^,来到27位置，继续操作 假设 ST[27] [2] =28,  可以取 ans+=2^2^ .......... 最后假设来到33位置，ST[33] [0] =34, ans+=1,最后我们ans再++，因为我们每一步都取的是比目标小的位置，所以最后再多加一步一定能到达

[P4155 [SCOI2015\] 国旗计划 - 洛谷](https://www.luogu.com.cn/problem/P4155)

```c#
简化题意
给定一个个点，它们构成一个环，同时有一段段区间，求在某个区间一定参与的情况下，最少要多少个区间才能形成环
我们可以化环为线，如果一个区间左端点>右端点，那它肯定在环两端，我们只需要在区间右端点加上最大的编号即可
然后我们把每个区间拷贝一份，这个区间左端点和右端点都加上最大的编号，我们只需要求，某个区间到它的复制区间，需要多少个区间才能到达
 
#include<iostream>
#include<algorithm>
#include<vector>
using namespace std;
const int N = 2e5 + 10;
vector<vector<int>> line(N << 1, vector<int>(3, 0));//line[i][0]代表区间编号，line[i][0]代表区间左端点，line[i][1]代表区间右端点
int n, m;
int st[N << 1][18];//st表,st[i][p]代表，从i位置走2^p个区间，能来到哪个区间
int answer[N];//答案数组
bool cmp(const vector<int> &a, const vector<int> &b)
{
    return a[1] < b[1];
}
int lo2(int n)//求n最接近2的几次幂
{
    int ans = 0;
    while ((1 << ans) <= (n >> 1))
        ans++;
    return ans;
}
void build()
{
    for (int i = 1; i <= n; i++)
    {
        if (line[i][1] > line[i][2])
        {
            line[i][2] += m;//化环为线
        }
    }
    sort(line.begin()+1, line.begin()+n+1, cmp);//按照区间左端点排序
    for (int i = 1; i <= n; i++)
    {
        line[i + n][0] = line[i][0];//复制一份
        line[i + n][1] = line[i][1] + m;
        line[i + n][2] = line[i][2] + m;
    }
    int e = n << 1;//复制后现在有2^n个区间  
    for (int i = 1, a = 1; i <=e; i++)
    {
        while (a + 1 <= e && line[a + 1][1] <= line[i][2])
            a++;
        st[i][0] = a;//在第i个区间内，最远能到达那个区间，就是选最靠近第i个区间的右端点的区间的左端点
    }
    int power = lo2(n);
    for (int p = 1; p <= power; p++)
    {
        for (int i = 1; i <= e; i++)
        {
            st[i][p] = st[st[i][p - 1]][p - 1];//构建st表
        }
    }
}
int jump(int i)//求答案
{
    int aim = line[i][1] + m;
    int cur = i;
    int ans = 1,next;
    int power = lo2(n);
    for (int p = power; p >= 0; p--)
    {
        next = st[cur][p];
        if (next != 0 && line[next][2] < aim)
        {
            cur = next;
            ans += 1 << p;
        }
    }
    return ans+1;
}
int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i++)
    {
        line[i][0] = i;
        cin >> line[i][1] >> line[i][2];
    }
    build();
    for (int i = 1; i <= n; i++)
        answer[line[i][0]] = jump(i);
    for (int i = 1; i <= n; i++)
        cout << answer[i] << " ";
    return 0;
}
```

## 2.st表维护更多信息

![image-20250416223258567](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250416223258567.png)

1.最大值，最小值

[P2880 [USACO07JAN\] Balanced Lineup G - 洛谷](https://www.luogu.com.cn/problem/P2880)

```c++
#include<iostream>
using namespace std;
const int N = 5e4 + 10;
int a[N];
int stmin[N][16];
int stmax[N][16];
int n, t;
int lo(int n)
{
    int ans = 0;
    while ((1 << ans) <= (n >> 1))
        ans++;
    return ans;
}
void build()
{
    
    int power = lo(n);
    for (int p = 1; p <= power; p++)
    {
        for (int i = 1; i + (1 << (p - 1)) <= n; i++)
        {
            stmin[i][p] = min(stmin[i][p - 1], stmin[i + (1 << (p - 1))][p - 1]);//
            //i到i+(1 << (p - 1))区间和i+(1 << (p - 1))到i+2*(1 << (p - 1))的最值，就是
            //i到i+(1 << p)这个区间的最值
            stmax[i][p] = max(stmax[i][p - 1], stmax[i + (1 << (p - 1))][p - 1]);
        }
    }
}
int query(int l, int r)
{
    int maxa = 0;
    int mina = 0;
    int cha = r - l+1;
    int tmp = lo(cha);
    //比如求17——37这个区间的最值，它们的差值是21，最近的2的幂是16，先求17——32区间的最值，在求37-16+1--37这个区间的最值，两个取最大值就行了
    maxa = max(stmax[l][tmp], stmax[r - (1<<tmp)+1][tmp]);
    mina = min(stmin[l][tmp], stmin[r - (1<<tmp)+1][tmp]);
    return maxa - mina;
}
int main()
{
    cin >> n >> t;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        stmin[i][0] = a[i];
        stmax[i][0] = a[i];
    }
    build();
    while (t--)
    {
        int l, r;
        cin >> l >> r;
        cout << query(l, r) << "\n";
    }
    return 0;
}
```

[P1890 gcd区间 - 洛谷](https://www.luogu.com.cn/problem/P1890)

```c++
#include<iostream>
using namespace std;
const int N=1005;
int a[N];
int st[N][11];
int n,t;
int gcd(int a,int b)
{
   return b == 0 ? a : gcd(b, a % b);
}
int lo(int n)
{
    int ans=0;
    while((1<<ans)<=(n>>1))
    ans++;
    return ans;
}
int query(int l,int r)
{
    int tmp=r-l+1;
    int power=lo(tmp);
    int ans=gcd(st[l][power],st[r-(1<<tmp)+1][power]);
    return ans;
}
int main()
{
   cin>>n>>t;
   for(int i=1;i<=n;i++)
       {
           cin>>a[i];
           st[i][0]=a[i];
       }
    int power=lo(n);
    for(int p=1;p<=power;p++)
        {
            for(int i=1;i+(1<<p)<=n;i++)
            st[i][p]=gcd(st[i][p-1],st[i+(1<<(p-1))][p-1]);
        }
    while(t--)
        {
            int l,r;
            cin>>l>>r;
            cout<<query(l,r)<<"\n";
        }
    return 0;
}
```

