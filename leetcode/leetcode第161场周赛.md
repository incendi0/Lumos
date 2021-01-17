# leetcode第161场周赛

### 1247. 交换字符使得字符串相同

分别记录x对应y和y对应x的数目，分别记为count1和count2：

1. 如果count1和count2奇偶性不一致，则返回-1；
2. 如果count1和count2都是偶数，x对应y变换一次，计数-2，同理y对应x变换一次，计数02，结果为(count1 + count2) / 2；
3. 如果count1和count2都是奇数，根据2，先处理(count1 - 1)和(count2 - 1)的情况，然后处理xy对应yx，或者yx对应xy的情况，此时变换数为2，结果为(count1 + count2 - 2) / 2 + 2。

```java
public int minimumSwap(String s1, String s2) {
    int count1 = 0, count2 = 0;
    for (int i = 0; i < s1.length(); i++) {
        if (s1.charAt(i) != s2.charAt(i)) {
            if (s1.charAt(i) == 'x') {
                count1++;
            } else {
                count2++;
            }
        }
    }
    if (count1 % 2 != count2 % 2) {
        return -1;
    }
    if (count1 % 2 == 0 && count2 % 2 == 0) {
        return (count1 + count2) /2;
    } else {
        return (count1 + count2 - 2) / 2 + 2;
    }
}
```

### 1248. 统计「优美子数组」

使用滑动窗口来解决：

1. 需要找到窗口最左侧，即数组起点或者前一位是奇数a；
2. 最右侧是数组终点或者后一位是奇数b；
3. 包含k个奇数的左右区间内（包括端点）需要找到第一个奇数i和最后一个奇数j；
4. 四个位置即能确定这个窗口「优美子数组」的个数，同时满足a <= i <= j <= b，则连续的「优美子数组」个数为(i - a + 1) * (b - j + 1)。

我们可以只记录奇数的位置，然后迭代求和。为了方便处理边界值，首位位置加上-1和数组长度n。

```java
public int numberOfSubarrays(int[] nums, int k) {
    int n = nums.length;
    List<Integer> oddIndexList = new ArrayList<>();
    oddIndexList.add(-1);
    for (int i = 0; i < n; i++) {
        if (nums[i] % 2 == 1) {
            oddIndexList.add(i);
        }
    }
    oddIndexList.add(n);
    int ret = 0;
    for (int i = 1; i + k < oddIndexList.size(); i++) {
        ret += (oddIndexList.get(i) - oddIndexList.get(i - 1)) * (oddIndexList.get(i + k) - oddIndexList.get(i + k - 1));
    }
    return ret;
}
```

### 1249. 移除无效的括号

栈的使用。

```java
public String minRemoveToMakeValid(String s) {
    int n = s.length();
    List<Integer> invalidCharIndex = new ArrayList<>();
    List<Integer> stack = new ArrayList<>();
    for (int i = 0; i < n; i++) {
        if (s.charAt(i) == '(') {
            stack.add(i);
        } else if (s.charAt(i) == ')') {
            if (stack.isEmpty()) {
                invalidCharIndex.add(i);
            } else {
                stack.remove(stack.size() - 1);
            }
        }
    }
    Set<Integer> invalid = new HashSet<>(invalidCharIndex);
    invalid.addAll(stack);
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < n; i++) {
        if (!invalid.contains(i)) {
            sb.append(s.charAt(i));
        }
    }
    return sb.toString();
}
```

### 1250. 检查「好数组」

裴蜀定理。

```java
public boolean isGoodArray(int[] nums) {
    int n = nums[0];
    for (int i = 1; i < nums.length; i++) {
        n = gcd(n, nums[i]);
    }
    return n == 1;
}

private int gcd(int m, int n) {
    if (n == 0) {
        return m;
    }
    return gcd(n, m % n);
}
```