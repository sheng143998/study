# 数位dp

## 1.什么是数位dp

数位dp很像线性dp，线性dp一般是在数组上做选择，而数位dp是在一个数的每一位上进行选择，数位dp需要一点排列组合的知识

## 2.例题

[902. 最大为 N 的数字组合 - 力扣（LeetCode）](https://leetcode.cn/problems/numbers-at-most-n-given-digit-set/description/)

```c++
class Solution {
public:
    int a[10];
    long cnt[10];//cnt为预处理数组，作用下面会介绍
    //递归函数的定义就是还剩下len位没有决定，之前的位和n一样
	// 返回最终<=num的可能性有多少种
    int f(int num,int offset,int len,int n)
    {
        if(len==0)//本身
        return 1;
        int cur=(num/offset)%10;//取出当前的位，可以自行举例
        int ans=0;
        for(int i=0;i<n;i++)//遍历数组的每一个数
        {
            if(a[i]<cur)//如果小于，之后的位都可以任意选
            ans+=cnt[len-1];
            else if(a[i]==cur)//如果等于继续递归
            {
                ans+=f(num,offset/10,len-1,n);
            }
            else//一旦大于，后面的数都是大于的(数组为非降序排列)，直接break
            break;
        }
        return ans;
    }
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        //大体思路，就是位数小于n的数字一定符合，位数等于n的是我们要计算的，两者相加即可
        int m=digits.size();
        for(int i=0;i<digits.size();i++)//转换成数字
        {
            a[i]=digits[i][0]-'0';
        }
        int len=1,offset=1;//len代表有多少位，offest是取出n的每一位用的
        int tmp=n/10;
        while(tmp>0)
        {
            tmp/=10;
            len++;
            offset*=10;
        }
         // cnt[i] : 已知前缀比num小，剩下i位没有确定，请问前缀确定的情况下，一共有多少种数字排列
		// cnt[0] = 1，表示后续已经没有了，前缀的状况都已确定，那么就是1种
		// cnt[1] = m
		// cnt[2] = m * m
        cnt[0]=1
        int ans=0;
        for(int i=m,k=1;k<len;i*=m,k++)//先计算位数小于n的(从1位到len-1位)，每一位上数组digits都可以选
        {
            cnt[k]=i;
            ans+=i;
        }
        return ans+f(n,offset,len,m);//位数等于n的
    }
};
```

[2719. 统计整数数目 - 力扣（LeetCode）](https://leetcode.cn/problems/count-of-integers/)

```c++
class Solution {
public:
    int len;
    int maxa;
    int mina;
    int mod=1e9+7;
    string num;
    int dp[23][401][2];//记忆化
    bool check()//特判
    {
        int sum=0;
        for(int i=0;i<len;i++)
        sum+=num[i]-'0';
        return sum<=maxa&&sum>=mina;
    }
    int f(int i,int sum,int free)//i代表来到第i位(从左到右，最左边是第0位)，sum代表累加和，free=1代表后面可以自由选择，free=0代表不能自由选择
        
    {
        if(sum>maxa)//剪枝
        return 0;
        if(sum+(len-i)*9<mina)//如果这个累加和非常小，他的后面每一位都选9加起来还是小于mina，肯定也不行
        return 0;
        if(i==len)//来到最后一位，如果前两个没有返回，这个数一定满足要求
        return 1;
        if(dp[i][sum][free]!=-1)//记忆化
        return dp[i][sum][free];
        int cur=num[i]-'0';//拿出第i位的数字
        int ans=0;
        if(free==0)
        {
            for(int pick=0;pick<cur;pick++)//可以随便选比cur小的数
            {
                ans=(ans+f(i+1,sum+pick,1))%mod;
            }
            ans=(ans+f(i+1,sum+cur,0))%mod;//当前数选等于cur的数，下一位不能任意选，只能选小于等于下一位的数
        }
        else//从0到9任意选(任意选是因为前面肯定有一位选了小于num那个位置的数)
        {
            for(int pick=0;pick<=9;pick++)
            {
                ans=(ans+f(i+1,sum+pick,1))%mod;
            }
        }
        dp[i][sum][free]=ans;
        return ans;

    }
    int count(string num1, string num2, int min_sum, int max_sum) {
     //大体思路，先计算0-num2上这样数的个数，在计算0-num1这样数的个数，两者相减就是答案，但是要特判num1这个数
     maxa=max_sum;//全局变量，方便写递归函数
     mina=min_sum;
     num=num2;
     len=num2.size();
     memset(dp,-1,sizeof dp);

     int ans=f(0,0,0);//0-num2上这样数的个数
     num=num1;
     len=num1.size();
     memset(dp,-1,sizeof dp);

     ans=(ans-f(0,0,0)+mod)%mod;//计算0-num1-1这样数的个数，两者相减
     if(check())
     ans=(ans+1)%mod;//特判num1这个数
     return ans;   
    }
};
```

[2376. 统计特殊整数 - 力扣（LeetCode）](https://leetcode.cn/problems/count-special-integers/submissions/596356226/)

```c++
class Solution {
public:
    int cnt[10];
    // 之前已经确定了和num一样的前缀，且确定的部分一定不为空
	// 还有len位没有确定
	// 哪些数字已经选了，哪些数字没有选，用s表示
	// 返回<=num且每一位数字都不一样的正整数有多少个
    int f(int n,int len,int offset,int s)
    {
        if(len==0)
        return 1;
        int ans=0;
        int first=(n/offset)%10;
        for(int i=0;i<first;i++)
        {
            if((s&(1<<i))==0)
            {
                ans+=cnt[len-1];
            }
        }
        if((s&(1<<first))==0)
        ans+=f(n,len-1,offset/10,s^(1<<first));
        return ans;
    }
    int countSpecialNumbers(int n) {
        int len=1;
        int offset=1;
        int tmp=n/10;
        while(tmp>0)
        {
            tmp/=10;
            len++;
            offset*=10;
        }
        // 一共长度为len，还剩i位没有确定，确定的前缀为len-i位，且确定的前缀不为空
		// 0~9一共10个数字，没有选择的数字剩下10-(len-i)个
		// 那么在后续的i位上，有多少种排列
		// 比如：len = 4
		// cnt[4]不计算
		// cnt[3] = 9 * 8 * 7
		// cnt[2] = 8 * 7
		// cnt[1] = 7
		// cnt[0] = 1，表示前缀已确定，后续也没有了，那么就是1种排列，就是前缀的状况
		// 再比如：len = 6
		// cnt[6]不计算
		// cnt[5] = 9 * 8 * 7 * 6 * 5
		// cnt[4] = 8 * 7 * 6 * 5
		// cnt[3] = 7 * 6 * 5
		// cnt[2] = 6 * 5
		// cnt[1] = 5
		// cnt[0] = 1，表示前缀已确定，后续也没有了，那么就是1种排列，就是前缀的状况
		// 下面for循环就是求解cnt的代码
       cnt[0]=1;
       for(int i=1,k=10-len+1;i<len;i++,k++)
       {
         cnt[i]=cnt[i-1]*k;
       } 
       int ans=0;
        // 如果n的位数是len位，先计算位数少于len的数中，每一位都互不相同的正整数个数，并累加
			// 所有1位数中，每一位都互不相同的正整数个数 = 9
			// 所有2位数中，每一位都互不相同的正整数个数 = 9 * 9
			// 所有3位数中，每一位都互不相同的正整数个数 = 9 * 9 * 8
			// 所有4位数中，每一位都互不相同的正整数个数 = 9 * 9 * 8 * 7
			// ...比len少的位数都累加...
       if(len>=2)
       {
          ans=9;
          for(int i=2,a=9,b=9;i<len;i++,b--)
          {
             a*=b;
             ans+=a;
          }
       }
        // 如果n的位数是len位，已经计算了位数少于len个的情况
		// 下面计算一定有len位的数字中，<=n且每一位都互不相同的正整数个数
       int first=n/offset;
       ans+=(first-1)*cnt[len-1];// 小于num最高位数字的情况
       ans+=f(n,len-1,offset/10,1<<first);//等于num最高位数字的情况
       return ans;
    }
};
```

[P2657 [SCOI2009\] windy 数 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/problem/P2657)

```c++
#include<bits/stdc++.h>
using namespace std;
int dp[20][20][2];
// offset 完全由 len 决定，为了方便提取 n中某一位数字
// 从 n 的高位开始，还剩下 len 位没有决定
// 前一位的数字是 pre，如果 pre == 10，表示从来没有选择过数字
// 如果之前的位已经确定比 n 小，那么 free == 1，表示接下来的数字可以自由选择
// 如果之前的位和 n一样，那么 free == 0，表示接下来的数字不能大于 n 当前位的数字
// 返回 <= n 的 windy 数有多少个
int f(int n, int offset, int len, int pre, int free1)
{
    if (len == 0)//前面的位都已经确定
        return 1;
    if (dp[len][pre][free1] != -1)//记忆化
        return dp[len][pre][free1];
    int ans = 0;
    int cur = n / offset % 10;
    if (free1 == 0)//不能任意选
    {
        if (pre == 10)//符合这种情况一定只有来到n的第一位(从左到右)
        {
            ans += f(n, offset / 10, len - 1, 10, 1);//依旧可以不选，此时位数比n小，剩下可以任意选
            for (int i = 1; i < cur; i++)//(注意i从1开始)
            {
                ans += f(n, offset / 10, len - 1, i, 1);//这个位上选比n这个位小的数，剩下的依然可以任意选
            }
            ans += f(n, offset / 10, len - 1, cur, 0);//选和n这一位相同的数，剩下的数不能任意选
        }
        else
        {
            for (int i = 0; i < cur; i++)
            {
                if (abs(pre - i) >= 2)//满足题意
                {
                    ans += f(n, offset / 10, len - 1, i, 1);
                }
            }
            if (abs(pre - cur) >= 2)//如果选和n这一位相同的数，要判断是否满足要求
            {
                ans += f(n, offset / 10, len - 1, cur, 0);
            }
        }
    }
    else//可以任意选，只要满足要求
    {
        if (pre == 10)
        {
            ans += f(n, offset / 10, len - 1, 10, 1);//和上面一样
            for (int i = 1; i <= 9; i++)//(注意i从1开始)
            {
                ans += f(n, offset / 10, len - 1, i, 1);//任意选(前一位没有选)
            }
        }
        else
        {
            for (int i = 0; i <= 9; i++)
            {
                if (abs(pre - i) >= 2)//满足要求
                    ans += f(n, offset / 10, len - 1, i, 1);
            }
        }
    }
    dp[len][pre][free1] = ans;
    return ans;
}
int cnt(int a)//0-a范围上的windy数有多少个
{
    if(a==0)//特判
    return 1;
    int len = 1, offset = 1;
    int tmp = a / 10;
    while (tmp > 0)
    {
        tmp /= 10;
        len++;
        offset *= 10;
    }
    return f(a, offset, len, 10, 0);
}
int main()
{
    int a,b;
    cin >> a >> b;
    memset(dp, -1, sizeof dp);
    int ans = cnt(b);
    memset(dp, -1, sizeof dp);
    ans -= cnt(a - 1);
    cout << ans;//0-b范围上的windy数减0-a范围上的windy数就是a-b范围上的windy数
    return 0;
}
```

[P3413 SAC#1 - 萌数 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/problem/P3413)

```c++
//本题萌数比较难求，但非萌数很容易求，所以我们先求出某个区间上非萌数的个数，再拿这个区间上所有的数减去非萌数的个数就是萌数的个数
#include<bits/stdc++.h>
using namespace std;
int mod = 1000000007;
int dp[1005][12][12][2];
//f函数求非萌数的个数，我们注意到一个数是萌数，必然在某一位上满足和它的前一位相等，或者和它的前两位相等，就是一个数为萌数，考察它的每一位，必然有第i位==第i-1位或第i-2位，举例11，101，1234321等等，我们只要不满足这个条件，就可以找出所有的非萌数
int f(string& num, int i, int pp, int p, int free1)//i代表来到第i位，pp代表i-2位，p代表i-1位，free1代表能否自由选择（要保证比num小）
{
    if (i == num.size())//当来到最终位置时，代表前缀都已经确定，这是一种有效的方案
    {
        return 1;
    }
    if (dp[i][pp][p][free1] != -1)//记忆化
        return dp[i][pp][p][free1];
    int ans = 0;
    if (free1 == 0)//当前不能自由选择
    {
        if (p == 10)//当前不能自由选择，并且没有选择过数字，只能是一开始来到第0位的情况
        {
            ans = (ans + f(num, i + 1, 10, 10, 1)) % mod;//我们可以继续不选，这样一定比num小，pp和p为10代表之前没选过
            for (int cur = 1;cur< num[i] - '0'; cur++)//可以选比num第0位小的数字
            {
                ans = (ans + f(num, i + 1, p, cur, 1)) % mod;//当前p变成pp，cur变成p继续递归
            }
            ans = (ans + f(num, i + 1, p, num[i] - '0', 0)) % mod;//还可以选和num第0位相同的shu
        }
        else
        {
            for (int cur = 0; cur < num[i] - '0'; cur++)
            {
                if (pp != cur && p != cur)
                {
                    ans = (ans + f(num, i + 1, p, cur, 1)) % mod;
                }
            }
            if (pp != num[i] - '0' && p != num[i] - '0')
                ans = (ans + f(num, i + 1, p, num[i] - '0', 0)) % mod;
        }
    }
    else
    {
        if (p == 10)
        {
            ans = (ans + f(num, i + 1, 10, 10, 1)) % mod;
            for (int cur=1; cur <= 9; cur++)
            {
                ans = (ans + f(num, i + 1, p, cur, 1)) % mod;
            }
        }
        else
        {
            for (int cur = 0; cur <= 9; cur++)
            {
                if (pp != cur && p != cur)
                    ans = (ans + f(num, i + 1, p, cur, 1)) % mod;
            }
        }
    }
    dp[i][pp][p][free1] = ans;
    return ans;
}
int count(string &a)//计算萌数的个数
{
    if (a[0] == '0')
        return 0;
    int n = a.size();
    long long base = 1;
    long long sum = 0;
    for (int i = n - 1; i >= 0; i--)
    {
        sum = (sum + base * (a[i] - '0')) % mod;
        base = (base * 10) % mod;
    }
    memset(dp,-1,sizeof dp);
    return (sum - f(a, 0, 10, 10, 0) + mod) % mod;
}
bool check(string &a)
{
    int n = a.size();
    for (int pp = -2, p = -1, i = 0; i < n; i++, p++, pp++)
    {
        if (pp >= 0 && a[pp] == a[i])
            return true;
        if (p >= 0 && a[p] == a[i])
            return true;
    }
    return false;
}
int main()
{
    string l, r;
    cin >> l >> r;
    int ans = 0;
    ans = (count(r) - count(l) + mod) % mod;
    if (check(l))
        ans = (ans + 1) % mod;
    cout << ans;
    return 0;
}
```



[P2602 [ZJOI2010\] 数字计数 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/problem/P2602)
