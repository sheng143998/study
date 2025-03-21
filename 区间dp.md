# 区间dp

## 1.什么是区间dp

## ![image-20250123112007819](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250123112007819.png)

## 2.两侧端点展开的情况

[1312. 让字符串成为回文串的最少插入次数 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

这个题我们首先要考虑最基本的情况，就是只有1个字符时是不用插入额外字符的，还有2个字符时，如果这两个字符相同，本身就是回文串，不用插入，如果2个字符不一样我们只需要随便插入两个字符的任意一种即可变成回文串

下面看code实现

```c++
//记忆化搜索版本
class Solution {
public:
    int dp[501][501];
    int f(int l,int r,string &s)//在l和r区间上让这个区间上的子串成为回文串的最小插入次数
    {
        if(l==r)//两种基本情况
        return 0;
        if(l+1==r)
        {
            return s[l]==s[r]?0:1;
        }
        if(dp[l][r]!=-1)
        return dp[l][r];//记忆化
        int ans=0;
        if(s[l]==s[r])//如果这左右侧的字符相等，那么问题可以转化为在l-1和r-1区间上需要插入最少的次数
        ans= f(l+1,r-1,s);
        else
        {
            ans=min(f(l+1,r,s),f(l,r-1,s))+1;//如果不相等，我们可以在左右两侧插左右两侧的字符使区间缩小，比如在r的右边插入s【l】或在l的左边插入s【r】,这样就可以转化为上一种情况，使区间缩小
        }
        dp[l][r]=ans;
        return ans;
    }
    int minInsertions(string s) {
        int n=s.size();
        memset(dp,-1,sizeof dp);
        return f(0,n-1,s);
    }
};
```

```c++
//严格位置依赖的版本
//我们观察在递归函数中的写法即可分析出位置依赖

```

[486. 预测赢家 - 力扣（LeetCode）](https://leetcode.cn/problems/predict-the-winner/)

博弈问题

```c++
class Solution {
public:
    int dp[21][21];
    int f(int l,int r,vector<int> &nums)//返回玩家一取得的最大得分
    {
       if(l==r)//一样使是两种基本情况
       {
         return nums[l];
       }
       if(l+1==r)
       return max(nums[l],nums[r]);
       if(dp[l][r]!=-1)
       return dp[l][r];
       int ans=0;
       //p1，p2为玩家1选左右两边的结果
       int p1=nums[l]+min(f(l+2,r,nums),f(l+1,r-1,nums));//，不管玩家1选左边，还是右边，玩家2绝顶聪明，所以肯定要选让玩家1得分最小的情况
       int p2=nums[r]+min(f(l+1,r-1,nums),f(l,r-2,nums));
       ans=max(p1,p2);//玩家1肯定要的是自己最好的情况
       dp[l][r]=ans;
       return ans;
    }
    bool predictTheWinner(vector<int>& nums) {
        memset(dp,-1,sizeof dp);
        int sum=0;
        int n=nums.size();
        for(int i=0;i<n;i++)
        {
            sum+=nums[i];
        }
        int first1=f(0,n-1,nums);
        int second1=sum-first1;
        return first1>=second1;
    }
};
```

## 3.范围划分点的情况

[1039. 多边形三角剖分的最低得分 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-score-triangulation-of-polygon/)

这个就是从范围内部划分点的情况，比如区间0-5，我们可以取0，1，5这3个点连成3角形那么剩下的就只有区间2-4，取0，2，5 3个点

剩下区间0-1和2-4构成3角形；依次枚举每个点，取最大值

```c++
class Solution {
public:
    int dp[52][52];
    int f(int l,int r,vector<int>& values)
    {
           if(dp[l][r]!=-1)
           return dp[l][r];
           int ans=0x3f3f3f3f;
           if(l==r||l==r-1)
           ans=0;
           else
           {
             for(int m=l+1;m<r;m++)
             {
                ans=min(ans,f(l,m,values)+f(m,r,values)+values[l]*values[m]*values[r]);
             }
           }
           dp[l][r]=ans;
           return ans;
    }
    int minScoreTriangulation(vector<int>& values) {
        memset(dp,-1,sizeof dp);
        int n=values.size();
        return f(0,n-1,values);
    }
};
```

时间复杂度为n^3，这个枚举过程无法优化

[312. 戳气球 - 力扣（LeetCode）](https://leetcode.cn/problems/burst-balloons/)

![image-20250123185809642](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250123185809642.png)

```c++
class Solution {
public:
    int dp[302][302];
    int f(int l,int r,vector<int> &arr)
    {
        if(dp[l][r]!=-1)
        return dp[l][r];
        int ans=0;
        if(l==r)
        ans=arr[l-1]*arr[l]*arr[r+1];
        else
        {
            ans=max(arr[l-1]*arr[l]*arr[r+1]+f(l+1,r,arr),arr[l-1]*arr[r]*arr[r+1]+f(l,r-1,arr));
            for(int i=l+1;i<r;i++)//和上一题一样，每个区间内枚举
            {
                ans=max(ans,arr[l-1]*arr[i]*arr[r+1]+f(l,i-1,arr)+f(i+1,r,arr));
            }
        }
        dp[l][r]=ans;
        return ans;
    }
    int maxCoins(vector<int>& nums) {
      int n=nums.size();
      vector<int> arr(n+3,1);
      for(int i=1;i<=n;i++)
      arr[i]=nums[i-1];
      memset(dp,-1,sizeof dp);
      return f(1,n,arr);//范围要注意
    }
};
```

