# 树形dp

## 1.树形dp相关

树形dp顾名思义就是在树上做动态规划，具体分析思路就在下面

![image-20250127110527437](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250127110527437.png)

我们来看例题

[1373. 二叉搜索子树的最大键值和 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-sum-bst-in-binary-tree/)

二叉搜索子树就是bst子树，我们先来分析需要哪些信息，假设以x作为开头的一个树，bst子树的最大键值和只有两种情况，<u>1.包含x这个节点，就是x开头的这颗树是bst树，最大键值和就是这棵树上所有的键值和。2.不包含x节点，就是x的左树的最大bst子树的最大键值和</u>

<u>和x的右树最大键值和取较大的一个(max(bstleftsum,bstrightsum));</u>我们所需要的信息就已经可以知道了，假设以x作为开头的一个树，**1.这颗树总的键值和，2.这颗树最大bst子树的键值和，3.这棵树是不是bst树，4.这颗树上的最大值，5.这棵树上的最小值。**4和5是为了判断是不是bst树的条件(==假设以x作为开头的一个树，他要是bst树的话，得保证左树和右数都是bst树，而且左数的最大值要小于x节点的值，右树的最小值要大于x节点的值)==

code如下

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public://定义子树的全集信息
   struct Info {
    int max;
    int min;
    int sum;
    bool isBst;
    int maxBstSum;

    Info(int a, int b, int c, bool d, int e)//构造函数
        : max(a), min(b), sum(c), isBst(d), maxBstSum(e) {}
};

// 递归函数
Info f(TreeNode* x) {
    if (x == nullptr) {
        return Info(INT_MIN, INT_MAX, 0, true, 0);//basecase
    }
    Info infol = f(x->left);//左树的信息
    Info infor = f(x->right);//右树的信息
    int max = max(x->val, max(infol.max, infor.max));//更新以x开头的树的最大和最小值
    int min = min(x->val, min(infol.min, infor.min));
    int sum = infol.sum + infor.sum + x->val;//整颗树的键值和
    bool isBst = infol.isBst && infor.isBst && infol.max < x->val && x->val < infor.min;//判断是否为bst
    int maxBstSum = max(infol.maxBstSum, infor.maxBstSum);//先在左树和右树中取一个最大，如果是bst再和总键值和sum比较，取最大的
    if (isBst) {
        maxBstSum = max(maxBstSum, sum);
    }
    return Info(max, min, sum, isBst, maxBstSum);//子树返回全局信息
}

// 主函数
int maxSumBST(TreeNode* root) {
    return f(root).maxBstSum;//直接返回
}

};
```

相关题目

[543. 二叉树的直径 - 力扣（LeetCode）](https://leetcode.cn/problems/diameter-of-binary-tree/description/)

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    struct info
    {
       int d;//以x开头的树的最大直径
       int h;//以x开头的最大深度
       info(int x,int y)
       :d(x),h(y){}
    };
    //本题依然分为要x节点和不要x节点，不要x节点就是左树和右树比较一下，要x就是左边最大深度+右边最大深度，然后2者取最大
    info f(TreeNode *x)
    {
        if(x==nullptr)
        return info(0,0);
        info left1=f(x->left);
        info right1=f(x->right);
        int d=max(left1.d,right1.d);
        d=max(d,left1.h+right1.h);
        int h=max(left1.h,right1.h)+1;
        return info(d,h);
    }
    int diameterOfBinaryTree(TreeNode* root) {
        return f(root).d;
    }
};
```

[979. 在二叉树中分配硬币 - 力扣（LeetCode）](https://leetcode.cn/problems/distribute-coins-in-binary-tree/)

[968. 监控二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/binary-tree-cameras/)

这个题是有一个小的贪心优化

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int ans=0;
    //递归函数的定义，返回3个状态，任何一个节点都可以被下面3个状态表示
    //假设x上方一定有父亲，以x开头的树都能被覆盖
   //状态0：x是无覆盖的状态，x下方都已经被覆盖
   //状态1：x是覆盖状态，x上无摄像头，x下方都已经被覆盖
   //状态2：x是覆盖状态，x上有摄像头，x下方都已经被覆盖
    int f(TreeNode* x)
    {
        if(x==nullptr)//空树肯定要被父亲节点监控
        return 1;
        int left1=f(x->left);
        int right1=f(x->right);
        //由子树返回的状态一定可以决定出一个最优的状态
        if(left1==0||right1==0)//如果左树和右树有一个是0，那必然需要加一个摄像头，监控下面的
        {
            ans++;
            return 2;
        }
        if(left1==1&&right1==1)//如果有两个都是1，在这个节点上放摄像头只能监控本身和父亲节点，不划算，交给他父亲解决
        return 0;
        return 1;
    }                                                                                                                                                      
    int minCameraCover(TreeNode* root) {
        if(f(root)==0)//递归函数不能解决头节点，特判
        ans++;
        return ans;
    }
};
```

[LCR 050. 路径总和 III - 力扣（LeetCode）](https://leetcode.cn/problems/6eUYwP/)

[2477. 到达首都的最少油耗 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-fuel-cost-to-report-to-the-capital/)

```c++
class Solution {
public:
    static const int N=100005;
    vector<int> a[N];
    int size[N];//每一颗树上现在的人数
    long long cost[N];//花费
    //当来到一个节点x，cost【x】等于其所有子树的cost之和加上从所有子树上运输人的代价，这时假设一颗子树上有size【u】人，
    //那么运输的代价就是size【u】/seats向上取整
    void f(int seats,int u,int p)//u为当前来到的节点，p为u的父节点
    {
        size[u]=1;
        for(int i=0;i<a[u].size();i++)
        {
            if(a[u][i]!=p)//如果不是u的父节点就可以往下走
            {
                f(seats,a[u][i],u);
                size[u]+=size[a[u][i]];
                cost[u]+=cost[a[u][i]];
                cost[u]+=(size[a[u][i]]+seats-1)/seats;
            }                             
           
        }
    }
    long long minimumFuelCost(vector<vector<int>>& roads, int seats) {
        for(int i=0;i<roads.size();i++)//这个题建图特殊，要双向建图，后面可以通过小技巧来实现从父到子的遍历
        {
            a[roads[i][0]].push_back(roads[i][1]);
            a[roads[i][1]].push_back(roads[i][0]);
        }
        f(seats,0,-1);//一开始用-1代表0无头节点，就可以递归下去
       return cost[0];
    }
};
```

## 2.dfn序解决树形dp

### 1.什么是dfn序

![image-20250127175232284](https://xiaoyao1112.oss-cn-nanjing.aliyuncs.com/image/image-20250127175232284.png)

我们来看一个例题解释dfn序的用法

[2458. 移除子树后的二叉树高度 - 力扣（LeetCode）](https://leetcode.cn/problems/height-of-binary-tree-after-subtree-removal-queries/)

删除某一个子树，剩下的部分就是==移除的子树的这个节点的dfn序号-1和这个节点的dfn序+当前子树的大小==这两个部分，我们用一个deep数组存放从头节点到某个节点的深度，在这两个部分的区间里取最大值即可

maxl和maxr数组为了方便求解设置

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    static const int N=1e5+5;
    int size[N];//size[i]的含义是以i开头的子树一共有多少个节点
    int dfn[N];
    int deep[N];//从头节点到当前dfn序节点的深度(一共有几条边)
    int maxl[N];//maxl[i]含义为从0-i上最大的深度
    int maxr[N];//maxr[i]含义为从最后一个节点的dfn序号到i上最大深度
    int cnt=1;
    void f(TreeNode* x,int k)
    {
        int i=cnt++;//构建dfn序
        dfn[x->val]=i;
        deep[i]=k;
        size[i]=1;
        if(x->left!=nullptr)
        {
            f(x->left,k+1);
            size[i]+=size[dfn[x->left->val]];
        }
        if(x->right!=nullptr)
        {
            f(x->right,k+1);
            size[i]+=size[dfn[x->right->val]];
        }
    }
    vector<int> treeQueries(TreeNode* root, vector<int>& queries) {
        f(root,0);
        for(int i=1;i<=cnt;i++)//预处理数组
        {
            maxl[i]=max(maxl[i-1],deep[i]);
        }
        maxr[cnt+1]=0;//避免边界讨论
        for(int i=cnt;i>=1;i--)//同样操作
        maxr[i]=max(maxr[i+1],deep[i]);
        int m=queries.size();
        vector<int> ans(m,0);
        for(int i=0;i<m;i++)
        {
            int p1=maxl[dfn[queries[i]]-1];//这就是两个部分上的最大值
            int p2=maxr[dfn[queries[i]]+size[dfn[queries[i]]]];
            ans[i]=max(p1,p2);
        }
        return ans;
    }
};
```

