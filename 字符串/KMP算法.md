# KMP算法

### 1.kmp算法主要解决的问题

kmp算法主要解决字符串的匹配问题，比如一个经典的问题就是:在给定目标串s1中能否找到s2的这个子串，并返回第一次匹配上的下标

## 2.两个核心要点

### 1.next数组的生成

什么是next数组呢 假设s1为目标串，s2为需要在目标串中找到的子串，我们定义next[i]为之前子串的前后缀最大匹配长度(不含这个串整体)比如"aabsab"  （下标从0开始） 定义==next[0]=-1,next[1]=0==,当来到2下标时，之前子串为"aa"，前缀取一个a，后缀取一个a，最大匹配长度就为1，所以next[2]=1,来到3下标时，之前子串为"aab",不管前后缀怎么取，都不相等，所以匹配长度为0，next[3]=0,next[4]=0,来到5下标时，之前子串为"aabsa",前缀取一个a，后缀取一个1，所以next[5]=1 。这就是next数组的定义，那么如何快速生成next数组呢，就是我们需要解决的

我们生成next数组是通过dp的思想解决的，我们知道next[0]=-1,next[1]=0这两个是永远固定的，我们从2位置开始推，如果第一个字符==第二个字符，next[2]=1,否则为0。之后就是每当我们来到一个位置时，看前一个位置的next值，跳转到那个位置的下标，如果那个位置的字符=前一个位置的字符，那么当前mext的值就是前一个next的值加1，我们举个例子**"abasabasf"**,我们来到f这个字符的位置，要求此时的next值，我们前一个字符s对应next值为3，我们跳转到3下标，这个位置也是s，就等于f的前一个字符s，所以f位置next值就是3+1=4，如果不相等，就继续跳转，直到相等，如果跳到0位置，此时next[0]=-1,那么这个位置的next值肯定为0，不能往前跳了

code实现

```c++
vector<int> next1(string s)
{
    int m=s.size();
    vector<int> next(m,0);
    next[0]=-1;
    nxet[1]=0;
    int i=2,cn=0;
    while(i<m)
    {
        if(s[i-1]==s[cn])
            next[i++]=++cn;
        else if(cn>0)
        {
            cn=next[cn];
        }
        else
            next[i++]=0;
    }
    retrun next;
}
```



2.匹配过程

```c++
int kmp(string &s1,string &s2)
{
    int n=s1.size();
    int m=s2.size();
    int x=0,y=0;
    while(x<n&&y<m)
    {
        if(s1[x]==s2[y])
        {
            x++;
            y++;
        }
        else if(y==0)
        {
            x++;
        }
        else
            y=next[y];
    }
    return y==m?x-y:-1;
}
```

## 3.例题

[P4391 [BalticOI 2009\] Radio Transmission 无线传输 - 洛谷](https://www.luogu.com.cn/problem/P4391)

```c++
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+10;
int next1[N];
void next(string &s)
{
    int m=s.size();
    next1[0]=-1;
    next1[1]=0;
    int i=2,cn=0;
    while(i<=m)
        {
            if(s[i-1]==s[cn])
            {
                next1[i++]=++cn;
            }
            else if(cn>0)
            {
                cn=next1[cn];
            }
            else
                next1[i++]=0;
        }
}
int main()
{
    int n;
    string s;
    cin>>n>>s;
    next(s);
    cout<<n-next1[n];//结论
}
```

[P4824 [USACO15FEB\] Censoring S - 洛谷](https://www.luogu.com.cn/problem/P4824)

```c++
#include<bits/stdc++.h>
using namespace std;
const int N=1e6+10;
int sta1[N];
int sta2[N];
int next1[N];
void next(string &s)
{
    int m=s.size();
    next1[0]=-1;
    next1[1]=0;
    int i=2,cn=0;
    while(i<m)
        {
            if(s[i-1]==s[cn])
                next1[i++]=++cn;
            else if(cn>0)
            {
                cn=next1[cn];
            }
            else
                next1[i++]=0;
        }
}
int main()
{
    string s,t;
    cin>>s;
    cin>>t;
    int n=s.size();
    int m=t.size();
    next(t);
    int x=0,y=0;
    int size=0;
    while(x<n)
        {
            if(s[x]==t[y])
            {
                sta1[size]=x;
                sta2[size]=y;
                size++;
                x++;
                y++;
            }
            else if(y==0)
            {
                sta1[size]=x;
                sta2[size]=-1;
                size++;
                x++;
            }
            else
                y=next1[y];
            if(y==m)
            {
                size-=m;
                y=size>0?(sta2[size-1]+1):0;
            }
        }
    for(int i=0;i<size;i++)
        {
            cout<<s[sta1[i]];
        }
    return 0;
}
```

