```
def binarySearch(nums):
	left, right = 0, len(nums) - 1
	while left <= right:
		mid = (left + right) // 2
		if nums[mid] == target:
			return mid
		elif nums[mid] < target:
			left += 1
		else:
			right -= 1
	return -1 # Not Found
```
