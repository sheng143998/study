# bfs(广度优先搜索)

## 1.什么是bfs

[广度优先搜索](https://so.csdn.net/so/search?q=广度优先搜索&spm=1001.2101.3001.7020)（也称宽度优先搜索，缩写BFS，以下采用广度来描述）是**连通图**的一种遍历策略。因为它的思想是从一个顶点开始，辐射状地优先遍历其周围较广的区域，因此得名。

## 2.bfs用来干什么

bfs最经典的就是求起点到终点的距离

我们以一个题来引入bfs

[1162. 地图分析 - 力扣（LeetCode）](https://leetcode.cn/problems/as-far-from-land-as-possible/description/?envType=problem-list-v2&envId=breadth-first-search)

我们的思路就是从陆地开始，把每个陆地当作源头，开始向4周的海洋扩散（只能上下左右4个方向走），我们把源头当作第0层，随着不断扩散，层数会依次增加，直到图中所有的点都用完之后，所求的层数就是相应的距离（曼哈顿距离为水平的距离加上竖直的距离）

那么如何实现呢，就是下面这个code，我们有几个点需要注意

1. 怎么让扩散停止  

   我们选择队列这个数据结构，将源头先放入队列（第0层），然后扩散，将扩散的每一个点放入队列（新的一层），同时弹出上一层的全部元素，当队列为空时，整个图全部遍历完毕，扩散停止

2. 已经放入队列中的元素，不会在重新放入队列

   我们在扩散的过程中，会给图中每一个点打上标记，被标记过的点不在使用

```c++
class Solution {
public: 
        static const int N=101*101;
        static const int M=101;
        int q[N][2];//队列
        int l=0,r=0;
        bool visited[M][M];//查询这个点有没有用过（标记这个点）
        int move[5]={-1,0,1,0,-1};//后面上下左右4个方向移动会用到
public:
    int maxDistance(vector<vector<int>>& grid) {
        int n=grid.size();
        int m=grid[0].size();
        int sea=0;
        for(int i=0;i<n;i++)
        {
            for(int j=0;j<m;j++)
            {
                if(grid[i][j]==1)//多个源头
                {
                    visited[i][j]=true;//
                    q[r][0]=i;
                    q[r++][1]=j;
                }
                else
                {
                    visited[i][j]=false;
                    sea++;
                }
            }
        }
        if(sea==0||sea==n*m)//全是陆地或全是海洋
        return -1;
        int level=0;//bfs的层数，同时就是本题要求的东西
        while(l<r)
        {
            level++;
            int sz=r-l;//整层弹出，sz为当前层里的全部元素
            for(int i=0;i<sz;i++)
            {
                int x=q[l][0];
                int y=q[l++][1];
                for(int j=0;j<4;j++)//上下左右4个方向移动，直到图中所有元素都被使用过
                {
                   int  nx=x+move[j];
                   int ny=y+move[j+1];
                    if(nx>=0&&nx<n&&ny>=0&&ny<m&&grid[nx][ny]==0&&!visited[nx][ny])//不越界并且没有被走过并且为海洋
                    {
                        visited[nx][ny]=true;//标记已经走过
                        q[r][0]=nx;//同时添加到队列里去
                        q[r++][1]=ny;
                    }
                }
            }
        }
        return level-1;
    }
};
```

## 3.拓展

### 1.01bfs

01bfs主要适用于在一个无相图中，边的权值只有0和1两种，这时候通过01bfs求最短路等等很方便，我们以一道例题引入

[2290. 到达角落需要移除障碍物的最小数目 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-obstacle-removal-to-reach-corner/description/)

和上述传统的bfs不同，我们引入了一个distance数组，这个数组代表从起始点到这个点的最小花费（最短距离），同时在数据结构我们换成了双端队列，在每个点进入时有所区别，当遇到权值为0的点时，我们让这个点从头部进入，其他的情况都从尾巴进入，弹出的时候都从队列头部弹出，我们下面来详细看一下distance数组的作用

```c++
class Solution {
public: 
         int move[5]={-1,0,1,0,-1};//上下左右4个方向移动
         deque<pair<int,int>> a;//存放每个点的坐标
public:
    int minimumObstacles(vector<vector<int>>& grid) {
        int n=grid.size();
        int m=grid[0].size();
        vector<vector<int>> d(n,vector<int>(m,0));//distance数组，我们先让d数组中的每个值都为int最大值
        for(int i=0;i<n;i++)
        {
            for(int j=0;j<m;j++)
            d[i][j]=1e9;
        }
        a.push_front({0,0});//这个题起始点为(0,0)
        d[0][0]=0;//将从起始点到起始点距离肯定为0
        while(!a.empty())//
        {
            pair<int,int> record=a.front();
            a.pop_front();
            int x=record.first;
            int y=record.second;
            if(x==n-1&&y==m-1)//如果此时弹出的坐标刚好为角落的坐标，直接返回d[n-1][m-1]
            {
                return d[x][y];
            }
            for(int i=0;i<4;i++)//上下左右4个方向移动
            {
                int nx=x+move[i];
                int ny=y+move[i+1];
                if(nx>=0&&nx<n&&ny>=0&&ny<m&&d[nx][ny]>d[x][y]+grid[nx][ny])//如果距离更短，修改此时这个点d数组的值
                {
                    d[nx][ny]=d[x][y]+grid[nx][ny];
                    if(grid[nx][ny]==0)//如果此时边权为0
                    {
                        a.push_front({nx,ny});//从头进入
                    }
                    else
                    {
                        a.push_back({nx,ny});//从尾巴进入
                    }
                }
            }
        }
        return d[n-1][m-1];
    }
};
```

类似的题

[1368. 使网格图至少有一条有效路径的最小代价 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/submissions/587864827/)

### 2.双向广搜

![image-20241220000901044](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20241220000901044.png)

双向广搜和分治不同，分治是把大问题分解为若干个子问题，但是双向广搜没有，它只是把数据分为了两个部分，然后将两个部分整合，这样可以大大降低时间复杂度

例题[P4799 [CEOI2015 Day2\] 世界冰球锦标赛 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/problem/P4799)

以此题为题，我们排列组合，如果最大数据量为40，每场比赛在不超过最大花费的前提下，都分为观看和不观看两种，我们暴力展开，要执行2^40这样条的指令，肯定会超时，但是如果我们将数据拆为两半，每边分别暴力展开，最后在合并，这样的可以降到2*2^20这种也就是10^6的二倍,这样就不会超时了

```c++
#include<bits/stdc++.h>
using namespace std;
const int N = 50;
const long long M = (1 << 20) + 10;
long long a[N];
long long l[M];//存放的是由0展开到n/2这样在不超过财产的前提下，每个方案的花费
long long r[M];//存放的是由n/2展开到n这样在不超过财产的前提下，每个方案的花费
long long m;
long long fil;//用来记录l和r数组中元素的数量
void f(int i, int e, long long s, long long ans[])//i是起始位置，e是展开的终止位置，s为当前这个方案的花费
{
    if (s > m)//如果花费大于财产，就退出
    {
        return;
    }
    if (i == e)
    {
        ans[fil++] = s;//如果展开到终止位置记录答案
    }
    else
    {
        f(i + 1, e, s, ans);//如果没到终止，来到下一个位置，分为观看和不观看两种
        f(i + 1, e, s + a[i], ans);
    }
}
int main()
{
    int n;
    cin >> n >> m;
    for (int i = 0; i < n; i++)
    {
        cin >> a[i];
    }
    f(0, n >> 1, 0, l);
    long long ls = fil;
    fil = 0;
    f(n >> 1, n, 0, r);
    long long rs = fil;
    sort(l, l + ls);//排序，方便后续双指针
    sort(r, r + rs);
    long long ans = 0;
    for (long long i = ls-1, j = 0; i>=0; i--)//双指针合并答案(类似归并找逆序对)
    {
        while (j <rs && l[i] + r[j] <= m)
        {
            j++;
        }
        ans += j;
    }
    cout << ans;
    return 0;
}
```

