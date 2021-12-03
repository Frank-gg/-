## [K 次取反后最大化的数组和](https://leetcode-cn.com/problems/maximize-sum-of-array-after-k-negations/)

给你一个整数数组 nums 和一个整数 k ，按以下方法修改该数组：

- 选择某个下标 i 并将 nums[i] 替换为 -nums[i] 。

重复这个过程恰好 k 次。可以多次选择同一个下标 i 。

以这种方式修改数组后，返回数组 可能的最大和 。

> 示例 1：
>
> 输入：nums = [4,2,3], k = 1
> 输出：5
> 解释：选择下标 1 ，nums 变为 [4,-2,3] 。
>
> 示例 2：
>
> 输入：nums = [3,-1,0,2], k = 3
> 输出：6
> 解释：选择下标 (1, 2, 2) ，nums 变为 [3,1,0,2] 。
>
> 示例 3：
>
> 输入：nums = [2,-3,-1,5,-4], k = 2
> 输出：13
> 解释：选择下标 (1, 4) ，nums 变为 [2,3,-1,5,4] 。
>
>
> 提示：
>
> - 1 <= nums.length <= 10 ^ 4
> - -100 <= nums[i] <= 100
> - 1 <= k <= 10 ^ 4

题解

- 第一次排序：处理负数
- 第二次排序：k还有剩余，找到最小的值，适当取反。

```java
class Solution {
    public int largestSumAfterKNegations(int[] nums, int k) {
        Arrays.sort(nums);
        int i = 0;
        int sum = 0;
        while (k > 0 && i < nums.length && nums[i] < 0) {
            nums[i] = - nums[i];
            i++;
            k--;
        }
        Arrays.sort(nums);
        if (k % 2 == 1) {
            nums[0] = - nums[0];
        }
        for (int j = 0; j < nums.length; j++) {
            sum += nums[j];
        }
        return sum;
    }
}

class Solution {
    public int largestSumAfterKNegations(int[] nums, int k) {
        // 排序，把可能有的负数排到前面
        Arrays.sort(nums);
        int sum = 0;
        for (int i = 0; i < nums.length; i++) {
            // 贪心：如果是负数，而k还有盈余，就把负数反过来
            if (nums[i] < 0 && k > 0) {
                nums[i] = -1 * nums[i];
                k--;
            }
            sum += nums[i];
        }
        Arrays.sort(nums);
        // 如果k没剩，那说明能转的负数都转正了，已经是最大和，返回sum
        // 如果k有剩，说明负数已经全部转正，所以如果k还剩偶数个就自己抵消掉，不用删减，如果k还剩奇数个就减掉2倍最小正数。
        return sum - (k % 2 == 0 ? 0 : 2 * nums[0]); 
    }
}
```

