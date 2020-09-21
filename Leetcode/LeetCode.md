# LeetCode

## 3.无重复字符的最长子串

```python
st = set()
slow, fast = 0, 0
res = 0
while fast <= len(nums) - 1:
  if nums[fast] not in st:
    st.add(nums[fast])
    fast += 1
  else:
    while nums[fast] in st:
      st.remove(nums[slow])
      slow += 1
    res = max(res, fast - slow)
return res
```



## 5. 最长回文子串

```python
tmp_s = "#"
for ch in s:
  tmp_s += ch + "#"
res = ""
for i in range(len(tmp_s)):
  left, right = i - 1, i + 1
  while left >= 0 and right <= len(s) - 1:
    if s[left] != s[right]:
      break
    if right - left > len(res):
      res = s[left: right + 1]
    left -= 1
    right += 1
return s.replace("#","")
```



## 11. 盛最多水的容器

```python
left, right = 0, len(height) - 1
res = 0
while left < right:
	res = max(res, (right - left) * min(height[left], height[right]))
  if height[left] < height[right]:
		left += 1
	else:
		right -= 1
return res
```



## 13. 罗马数字转整数

```python
d = {"I" : 1, "V" : 5, "X" : 10, "L" : 50, "C" : 100, "D" : 500, "M" : 1000}
res = 0
        
for i in range(len(s)):
	res += d[s[i]]
	if i > 0:
		if (s[i] == "V" or s[i] == "X") and s[i - 1] == "I":
			res -= 2
		if (s[i] == "L" or s[i] == "C") and s[i - 1] == "X":
			res -= 20
		if (s[i] == "D" or s[i] == "M") and s[i - 1] =="C":
			res -= 200

return res
```



## 14. 最长公共前缀

```python
if not strs:
	return ""
        
res = strs[0]
for i in range(1, len(strs)):
	j = 0
	while j < len(res) and j < len(strs[i]):
		if res[j] != strs[i][j]:
			break
		j += 1
	res = res[:j]
return res
```

