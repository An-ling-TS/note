# LC移动零

## 移动零

> 给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

## 示例

```
输入: [0,1,0,3,12]
输出: [1,3,12,0,0]
```

## 说明

> 1. 必须在原数组上操作，不能拷贝额外的数组。
> 2. 尽量减少操作次数。

## 思路

> 冒泡，当某个不为0的数字的前一位是0时，交换顺序

## 源码

```tsx
function moveZeroes(nums: number[]): void {
    //冒泡，当某个不为0的数字的前一位是0时，交换顺序
    const len = nums.length
    for (let i = 0; i < len; i++) {
        for (let r = 0; r < len; r++) {
            if (nums[r] === 0 && r + 1 < len && nums[r + 1] !== 0) {
                let temp = nums[r]
                nums[r] = nums[r + 1]
                nums[r + 1] = temp;
            }
        }
    }
};
```

## 结果

![image-20210817174423587](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210817174423.png)

## 进阶思路

> 作者：wang_ni_ma
> 链接：https://leetcode-cn.com/problems/move-zeroes/solution/dong-hua-yan-shi-283yi-dong-ling-by-wang_ni_ma/
> 来源：力扣（LeetCode）

### 两次遍历

> 我们创建两个指针i和j，第一次遍历的时候指针j用来记录当前有多少非0元素。即遍历的时候每遇到一个非0元素就将其往数组左边挪，第一次遍历完后，j指针的下标就指向了最后一个非0元素下标。
> 第二次遍历的时候，起始位置就从j开始到结束，将剩下的这段区域内的元素全部置为0。

![283_1.gif](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210817175601.gif)

```python
class Solution(object):
	def moveZeroes(self, nums):
		"""
		:type nums: List[int]
		:rtype: None Do not return anything, modify nums in-place instead.
		"""
		if not nums:
			return 0
		# 第一次遍历的时候，j指针记录非0的个数，只要是非0的统统都赋给nums[j]	
		j = 0
		for i in xrange(len(nums)):
			if nums[i]:
				nums[j] = nums[i]
				j += 1
		# 非0元素统计完了，剩下的都是0了
		# 所以第二次遍历把末尾的元素都赋为0即可
		for i in xrange(j,len(nums)):
			nums[i] = 0

```

### 一次遍历

> 这里参考了快速排序的思想，快速排序首先要确定一个待分割的元素做中间点x，然后把所有小于等于x的元素放到x的左边，大于x的元素放到其右边。
> 这里我们可以用0当做这个中间点，把不等于0(注意题目没说不能有负数)的放到中间点的左边，等于0的放到其右边。
> 这的中间点就是0本身，所以实现起来比快速排序简单很多，我们使用两个指针i和j，只要nums[i]!=0，我们就交换nums[i]和nums[j]

![283_2.gif](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20210817175905.gif)

```py
class Solution(object):
	def moveZeroes(self, nums):
		"""
		:type nums: List[int]
		:rtype: None Do not return anything, modify nums in-place instead.
		"""
		if not nums:
			return 0
		# 两个指针i和j
		j = 0
		for i in xrange(len(nums)):
			# 当前元素!=0，就把其交换到左边，等于0的交换到右边
			if nums[i]:
				nums[j],nums[i] = nums[i],nums[j]
				j += 1

```

