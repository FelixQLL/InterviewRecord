# 滑动窗口
场景：找最长/最短
### 最长：input - string s
```
T res;
unordered_map<T, T> map;

while(right < size) {
  while (res not match) {
    left = new left; // update left
  }
  map[s[right]] = right; // update map[s[right]]
  res = max res;         // update res
  right++;
}
```

最短：
```
T res;
unordered_map<T, T> map;

while (right < size) {
  while (res match) {
    res = min res;
    left = new left;
    map[s[left]] = null;
  }
  right ++;
}
```
