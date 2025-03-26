# dijkstra最短路

## ==1.dijkstra最短路的适用条件，边权不能为负值==

## 2.dijkstra的实现

dijkstra主要的实现结构是通过小根堆结构实现的，而且需要两个数组distance数组和visited数组，我们先说visited数组，我们每弹出一个点的时候，在visited数组中把这个点标记成true，后续对这个点的一切操作都忽略，distance数组中存放的是源点到这个点的最短距离我们的小根堆就是按照这个距离来组织的，由于小根堆的特性，每次弹出的肯定就是这个点到源点的最小距离，我们来看例题

[743. 网络延迟时间 - 力扣（LeetCode）](https://leetcode.cn/problems/network-delay-time/solutions/2668220/liang-chong-dijkstra-xie-fa-fu-ti-dan-py-ooe8/)

 ```c++
 //这个题的意思是信号可以同时传播，但是信号要到达所有的节点的时间肯定是源点到某个点的距离最大，所以我们找出d数组中的最大值即可
 class Solution {
 public:
        struct cmp{
         bool operator()(const vector<int>&a,const vector<int>&b)//根据距离来组织小根堆
         {
             return a[1]>b[1];
         }
        };
 public:
         struct node{
             int t;//去往的节点
             int v;//边权
         };
         vector<node> a[101];//邻接表建图
         int distance[101];
         priority_queue<vector<int>,vector<vector<int>>,cmp> heap;
         bool visited[101];
 public:
     int networkDelayTime(vector<vector<int>>& times, int n, int k) {
         memset(distance,0x3f,sizeof distance);//一开始初始化d数组距离为无穷
         for(int i=0;i<times.size();i++)
         {
             a[times[i][0]].push_back({times[i][1],times[i][2]});//建图
         }
         distance[k]=0;//到自己的距离肯定为0 
         heap.push({k,0});//放入堆
         while(!heap.empty())//堆不为空不停
         {
             int u=heap.top()[0];
             heap.pop();
             if(visited[u])//如果这个点已经弹出过了，跳出本次循环
             continue;
             visited[u]=true;//标记弹出过的点
             for(int i=0;i<a[u].size();i++)
             {
                 int v=a[u][i].t;
                 int w=a[u][i].v;
                 if(!visited[v]&&distance[u]+w<distance[v])//如果去往这点的距离能变得更小就改
                 {
                     distance[v]=distance[u]+w;
                     heap.push({v,distance[u]+w});//重新入堆
                 }
             }
         }
         int ans=-1e9;
         for(int i=1;i<=n;i++)
         {
             if(distance[i]==0x3f3f3f3f)//这个点不能到达
             return -1;
             ans=max(ans,distance[i]);
         }
         return ans;
     }
 };
 ```

模板练手题

[778. 水位上升的泳池中游泳 - 力扣（LeetCode）](https://leetcode.cn/problems/swim-in-rising-water/description/)

## 2.反向索引堆优化dj算法



```c++
#include<bits/stdc++.h>
#include<cstring>
using namespace std;
struct node {
    int t;
    int v;
};
const int M = 2e5 + 10;
const int N = 1e5 + 10;
vector<node> a[N];
long long d[N];
int heapsize;
int heap[N];
int w[N];
void my_swap(int i, int j)
{
    int tmp = heap[i];
    heap[i] = heap[j];
    heap[j] = tmp;
    w[heap[i]] = i;
    w[heap[j]] = j;
}
void heapinsert(int i)
{
    while (d[heap[i]] < d[heap[(i - 1) / 2]])
    {
        my_swap(i, (i - 1) / 2);
        i = (i - 1) / 2;
    }
}
void heapify(int i)
{
    int l = 1;
    while (l < heapsize)
    {
        int best = l + 1 < heapsize && d[heap[l + 1]] < d[heap[l]] ? l + 1 : l;
        best = d[heap[best]] < d[heap[i]] ? best : i;
            if (best == i)
                break;
        my_swap(i, best);
        i = best;
        l = i * 2 + 1;
    }
}

void f(int v, long long c)
{
    if (w[v] == -1)
    {
        heap[heapsize] = v;
        w[v] = heapsize++;
        d[v] = c;
        heapinsert(w[v]);
    }
    else if (w[v] >= 0)
    {
        d[v] = min(d[v],c);
        heapinsert(w[v]);
    }
}

int  pop()
{
    int ans = heap[0];
    my_swap(0, --heapsize);
    heapify(0);
    w[ans] = -2;
    return ans;
}
bool isempty()
{
    return heapsize == 0;
}
int main()
{
    memset(d, 0x3f, sizeof d);
    memset(w, -1, sizeof w);
    int n, m, s;
    cin >> n >> m >> s;
    for (int i = 0; i < m; i++)
    {
        int from, t, v;
        cin >> from >> t >> v;
        a[from].push_back({ t,v });
    }
    f(s, 0);
    while (!isempty())
    {
        int u = pop();
        for (int i = 0; i < a[u].size(); i++)
        {
            f(a[u][i].t, a[u][i].v + d[u]);
        }
    }
    for (int i = 1; i <= n; i++)
        cout << d[i] << " ";
    return 0;
}
```

## 3.A星优化dj最短路

a*优化dj是通过优化常数时间来实现的，而且a星适用条件更为苛刻，a星主要求的是源点到某单个目标点的最短距离，a星和dj唯一的区别就是堆组织的方式，dj算法中堆是根据源点到某个点的距离来组织的，而a星算法中堆是根据源点到某个点的距离加上这个点到目标点的预估距离来实现的。所以如何求预估距离是我们主要关心的

### 1.3种预估距离函数

在某个二维矩阵中a*的预估距离函数很好实现，在普通的图中预估函数几乎没办法实现,所以==一般只能在2维矩阵中使用a星算法==

#### 1.曼哈顿距离

```c++
int f(int x,int y,int n,int m)
{
    return abs(x-n)+abs(y-m);//适用于在二维矩阵中上下左右移动
}
```

#### 2.对角线距离

```c++
int f(int x,int y,int n,int m)
{
    return max(abs(x-n),abs(y-m));//适用于在二维矩阵中上下左右还有斜着移动
}
```

#### 3.欧式距离

```c++
int f(int x,int y,int n,int m)
{
    return sqrt(pow(x-n,2)+pow(y-m,2));//勾股定理
}
```

[844. 走迷宫 - AcWing题库](https://www.acwing.com/problem/content/846/)

```c++
#include<iostream>
#include<algorithm>
#include<vector>
#include<queue>
using namespace std;
int f(int a, int b,int c,int d)//预估函数
{
    return abs(a-c)+abs(b-d);
}
struct cmp{
    bool operator()(const vector<int> &a,const  vector<int> &b)
    {
        return a[2]>b[2];
    }
};
int d[105][105];
bool visit[105][105];
int a[105][105];
int mov[5]={-1,0,1,0,-1};
priority_queue<vector<int>,vector<vector<int>>,cmp> heap;
int main(){
    int n,m;
    cin>>n>>m;
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            cin>>a[i][j];
            d[i][j]=1e9;
        }
    }
    d[1][1]=0;
    heap.push({1,1,0+f(1,1,n,m)});//就这个不一样
    while(!heap.empty())
    {
        vector<int> record=heap.top();
        heap.pop();
        int x=record[0];
        int y=record[1];
        int c=record[2];
        if(x==n&&y==m)
        {
            cout<<c;
            return 0;
        }
        if(visit[x][y])
        continue;
        visit[x][y]=true;
        for(int i=0;i<4;i++)
        {
            int nx=x+mov[i];
            int ny=y+mov[i+1];
            if(nx>=1&&nx<=n&&ny>=1&&ny<=m&&!visit[nx][ny]&&a[nx][ny]!=1)
            {
                if(d[x][y]+1<d[nx][ny])
               {
                   d[nx][ny]=d[x][y]+1;
                   heap.push({nx,ny,d[nx][ny]+f(nx,ny,n,m)});
                }
            }
        }
    }
    cout<<d[n][m];
    return 0;
}
```

