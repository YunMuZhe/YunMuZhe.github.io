#算法总结
## 常用API
```java
Arrays.sort(intervals, Comparator.comparingInt(o -> o[0]));
```
##回溯算法

[回溯法总结篇](https://programmercarl.com/%E5%9B%9E%E6%BA%AF%E6%80%BB%E7%BB%93.html#%E8%A7%A3%E6%95%B0%E7%8B%AC%E9%97%AE%E9%A2%98)

回溯算法能解决如下问题：

* 组合问题：N个数里面按一定规则找出k个数的集合
* 排列问题：N个数按一定规则全排列，有几种排列方式
* 切割问题：一个字符串按一定规则有几种切割方式
* 子集问题：一个N个数的集合里有多少符合条件的子集
* 棋盘问题：N皇后，解数独等等

###回溯法的模板：

```java
void backtracking(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        处理节点;
        backtracking(路径，选择列表); // 递归
        回溯，撤销处理结果
    }
}
```

# 贪心算法

## 难题
* [跳跃游戏II](https://programmercarl.com/0045.%E8%B7%B3%E8%B7%83%E6%B8%B8%E6%88%8FII.html#%E6%96%B9%E6%B3%95%E4%BA%8C)

* [加油站](https://leetcode.cn/problems/gas-station/)
    [代码随想录](https://programmercarl.com/0134.%E5%8A%A0%E6%B2%B9%E7%AB%99.html)

* [根据身高重建队列](https://leetcode.cn/problems/queue-reconstruction-by-height/)
    [代码随想录](https://programmercarl.com/0406.%E6%A0%B9%E6%8D%AE%E8%BA%AB%E9%AB%98%E9%87%8D%E5%BB%BA%E9%98%9F%E5%88%97.html)
    
## 贪心算法总结篇
[贪心算法总结篇](https://programmercarl.com/%E8%B4%AA%E5%BF%83%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93%E7%AF%87.html#%E6%80%BB%E7%BB%93)


# 动态规划

* [总结篇](https://programmercarl.com/%E8%83%8C%E5%8C%85%E6%80%BB%E7%BB%93%E7%AF%87.html#%E8%83%8C%E5%8C%85%E9%80%92%E6%8E%A8%E5%85%AC%E5%BC%8F)

背包递推公式
问能否能装满背包（或者最多装多少）：`dp[j] = max(dp[j], dp[j - nums[i]] + nums[i]);` ，对应题目如下：

* 动态规划：416.分割等和子集
* 动态规划：1049.最后一块石头的重量 II

问装满背包有几种方法：`dp[j] += dp[j - nums[i]]` ，对应题目如下：

* 动态规划：494.目标和
* 动态规划：518. 零钱兑换 II
* 动态规划：377.组合总和Ⅳ
* 动态规划：70. 爬楼梯进阶版（完全背包）

问背包装满最大价值：`dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);` ，对应题目如下：

* 动态规划：474.一和零

问装满背包所有物品的最小个数：`dp[j] = min(dp[j - coins[i]] + 1, dp[j]);` ，对应题目如下：

* 动态规划：322.零钱兑换
* 动态规划：279.完全平方数

# 单调栈

## 用处

通常是一维数组，要寻找任一个元素的右边或者左边第一个比自己大或者小的元素的位置，此时我们就要想到可以用单调栈了。



