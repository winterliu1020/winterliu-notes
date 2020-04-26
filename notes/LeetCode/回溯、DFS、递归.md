## 回溯、DFS、递归

今天写[322题](https://leetcode.com/problems/coin-change/)，一看题目想到用回溯法，尽管题目提示用 DP，但是自己在写回溯法的时候脑子里总是感觉和 DFS 有点像，而且回溯和递归是什么关系呢？查看了一下别人写的一些博客，总结一下为了让自己写代码的时候思路更清晰。

回溯和 DFS 都是一种算法思想，而递归则是一种编程的技巧，一般使用递归去实现回溯，但是 DFS 可以用递归去实现，也可以用栈去实现。

我看到别人的博客说 DFS 是一种特殊的回溯，因为回溯可能在任何节点上回溯，这取决于你的回溯条件了，但是 DFS 就是在叶子节点上的回溯。

回溯和 DFS 还有就是：回溯不会保留完整的搜索树结构，而 DFS 则会保留完整的搜索树结构。DFS 有点类似于暴力，而回溯则会进行剪枝、会回头。

下面是我看到的一个比较[好的总结](https://www.cnblogs.com/ganganloveu/p/4188131.html)：

### 1. 回溯和 DFS 的相同点：

回溯也是遵循深度优先的，一步步往前，而不是像广度那样往周围。

### 2. 不同点：

（1）访问序

深度优先遍历：目的是遍历，所以本质是“无序”的，也就是访问次序不重要，重要的是都被访问过了，所以在实现上只需要对每个位置记录是否被 visited 就够了。

回溯法：目的是求解过程，本质是有序的，也就是说每一步都必须是要求的次序，所以在实现上不能只是记录是否 visited 就够了，因为同样的内容不同的序访问会造成不同的结果。要使用访问状态来记录，也就是对于每个点记录已经访问过的邻居方向，回溯之后从新的未访问过的方向去访问邻居。至于这点之前有没有被访问过并不重要，重要的是没有以当前的序进行访问。

（2）访问的次数

深度优先遍历：访问过的点不再访问，所有点仅访问一次。

回溯法：已经访问过的点可能再次访问，也可能存在没有访问过的点。



322 题利用 DFS 来做：

```java
class Solution {
    int resultCoin = Integer.MAX_VALUE;
    public int coinChange(int[] coins, int amount) {
        // 方法一：
        // 题意：coins数组是你有的硬币种类，你可以假设每种硬币都有无限个，问你怎么拿硬币 硬币面值和等于amount，要你返回硬币数最少的是几个？如果拼凑不成功就返回-1
        // 看上去像找零钱问题   
        Arrays.sort(coins);
        // int i = coins.length - 1; // 硬币指针
        // 首先你得明白这个回溯函数的作用是什么，它的作用就是返回硬币个数或者-1
        // int result = backTracking(coins, amount, i, 0);
        // return result;
        
        for (int i = coins.length - 1; i >=0 ; i--) {
            DFS(coins, amount, i, 0);
        }
        if (resultCoin != Integer.MAX_VALUE) {
            return resultCoin;
        }
        return -1;
    }

		// 方法一：应该叫做深度优先搜索 其实也就是回溯法 这里会超时，题解说这种就相当于暴力解法了。。。  
    public void DFS(int[] coins, int amount, int i, int currCoin) {    
        if (amount == 0) {
            resultCoin = Math.min(resultCoin, currCoin);
            return;
        }
        for (int j = i; j >= 0; j--) {
            if (coins[j] <= amount) {
              DFS(coins, amount - coins[j], j, currCoin + 1);
            }
        }
    }
} 
```





