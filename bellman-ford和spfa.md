# bellman-ford和spfa

## 1.bellman-ford也是求最短路的一种算法，这种算法在使用时允许边权为负值，可以解决dj算法不能解决的问题

## 2.bellman-ford的思想及复杂度

使用这算法我们依然需要一个距离数组d[]，这个数组代表从源点到这个点的距离，如果存在一条从源点到目标点的最短路径，我们可以遍历每一条边，每次考察这条边，尝试进行松弛操作，每次操作若干点的距离都会变小，当某一轮不再有松弛操作时，d[]数组的距离必然是最短的。

![image-20241227161623698](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20241227161623698.png)

我们来想最多的轮数，也就是最坏的情况，假设总点数为n，从源点到目标点经过的最多点必然为n个点，所以松弛的轮数肯定小于n-1

假设边的数量为m，时间复杂度为O(m*n)

## 3.bellman-ford的一个重要应用，求有限制的最短路

这个算法可以解决这样一类问题，从某个点到目标点是否存在一条经过k条边的最短路径

[853. 有边数限制的最短路 - AcWing题库](https://www.acwing.com/problem/content/855/)

```c++
#include<bits/stdc++.h>
using namespace std;
const int N=505;
vector<vector<int>> a;//由于我们这个算法只需遍历所有的边，所以存图方式有所差异
int main()
{
    int n,m,k;
    scanf("%d %d %d",&n,&m,&k);
    vector<int> cur(n+1,0x3f3f3f3f);
    a.resize(m);
    int c=0;
    while(m--)
    {
        int x,y,z;
        scanf("%d %d %d",&x,&y,&z);
        a[c++]={x,y,z};//我们直接存边就行了
    }
    cur[1]=0;
    for(int i=0;i<k;i++)
    {
        vector<int> ne(cur);//这个数组是使松弛的轮数和题目的限制k相对应
        for(int i=0;i<a.size();i++)//一定要使用这种遍历方式，不要使用迭代器，不然时间会多很多导致超时
       { 
         if(cur[a[i][0]]!=0x3f3f3f3f)
         ne[a[i][1]]=min(ne[a[i][1]],cur[a[i][0]]+a[i][2]);//我们每次只更改ne数组中的值，但是通过cur数组考察是否要更改
      }
      cur=ne;
   }
    if(cur[n]>0x3f3f3f3f/2)//题目可能存在负边使得cur[n]不一定为0x3f3f3f3f，它大于0x3f3f3f3f的一半，说明肯定也没有通路
    cout<<"impossible";
    else
    cout<<cur[n];
    return 0;
}
```

# spfa优化bellman-ford

![image-20241227163309966](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20241227163309966.png)

spfa主要通过队列来优化bellman-ford，我们主要通过spfa来判断负环是否存在

[852. spfa判断负环 - AcWing题库](https://www.acwing.com/problem/content/854/)

```c++
#include<bits/stdc++.h>
using namespace std;
const int N=2e3+10;
const int M=2e7;
bool en[N];//判断这个点在不在队列
int d[N];
int upc[N];//这个点被松弛的次数
int q[M];
int l=0,r=0;
struct node
{
    int t;
    int w;
};
vector<node> a[N];
bool spfa(int n)
{
    d[0]=0;//从虚拟源点开始
    upc[0]++;
    q[r++]=0;
    en[0]=true;
    while(l<r)
    {
        int u=q[l++];
        en[u]=false;
        for(int i=0;i<a[u].size();i++)
        {
            int v=a[u][i].t;
            int we=a[u][i].w;
            if(d[u]+we<d[v])
            {
                d[v]=we+d[u];
                if(!en[v])
                {
                    if(upc[v]++==n)//被松弛n次了肯定存在负环
                    return true;
                    q[r++]=v;
                    en[v]=true;
                }
            }
        }
    }
    return false;
}
int main()
{
    int n,m;
    cin>>n>>m;
    memset(d,0x3f,sizeof d);
    while(m--)
    {
        int x,y,z;
        cin>>x>>y>>z;
        a[x].push_back({y,z});
    }
    for(int i=1;i<=n;i++)
    {
        a[0].push_back({i,0});//题中没有说是连通图，所以我们构建虚拟源点，到所有点的距离都为0
    }
    if(spfa(n))
    cout<<"Yes";
    else
    cout<<"No";
    return 0;
}
```

