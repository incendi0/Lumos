# leetcode weekly contest 160

### 5238. 找出给定方程的正整数解

题目给出了单调性，应该是二分搜索的应用，二分搜索是一直以来的头痛的问题，无论是单调数组寻找插入位置，单调数组旋转后寻找目标值，寻找可重复不降序数组中目标值出现的左右端点，我一直以来都是按照Java的Arrays.binarySearch方法来套模板，但总是不得要点，直到我看了[闫总的视频](https://www.bilibili.com/video/av59202632)，使用了[闫总的模板](https://www.acwing.com/blog/content/31/)，总算是有了整体的认识和根据具体细节调整的窍门。

题目保证 `f(x, y) == z` 的解处于 `1 <= x, y <= 1000` 的范围内，这就给了我们二分的左右端点。固定x，对y进行二分查找，复杂段O(nlogn)。

```java
public class P5238 {
    interface CustomFunction {
        int f(int x, int y);
    }

    public List<List<Integer>> findSolution(CustomFunction customfunction, int z) {
        List<List<Integer>> ret = new ArrayList<>();
        for (int x = 1; x <= 1000; x++) {
            int lhs = 1, rhs = 1000;
            while (lhs < rhs) {
                int mid = (lhs + rhs + 1) / 2;
                if (customfunction.f(x, mid) <= z) {
                    lhs = mid;
                } else {
                    rhs = mid - 1;
                }
            }
            if (customfunction.f(x, lhs) == z) {
                ret.add(Arrays.asList(x, lhs));
            }
        }
        return ret;
    }
}
```

### 5239. 循环码排列

循环码就是格雷码旋转后的一种情况，格雷码序列可以通过二进制数字通过数学公式直接转换，或者迭代求取，因为格雷码首位本就是只差一位，所以这道题只需要旋转一下格雷码就OK了。

```
public class P5239 {

    public List<Integer> circularPermutation(int n, int start) {
        List<Integer> ret = grayCode(n);
        for (int i = 0; i < ret.size(); i++) {
            if (ret.get(i) == start) {
                List<Integer> result = ret.subList(i, ret.size());
                result.addAll(ret.subList(0, i));
                return result;
            }
        }
        return new ArrayList<>();
    }

    public List<Integer> grayCode(int n) {
        List<Integer> ret = new ArrayList<>();
//        使用公式，二进制转格雷码，保留最高位，然后错位异或
//        for (int i = 0; i < (1 << n); i++) {
//            ret.add(i ^ (i >> 1));
//        }
//        迭代构造
//        正序前置0，逆序前置1
				ret.add(0);
        for (int i = 0; i < n; i++) {
            int h = 1 << i;
            for (int j = ret.size() - 1; j >= 0; j--) {
                ret.add(h + ret.get(j));
            }
        }
        return ret;
    }
}
```

### 5240. 串联字符串的最大长度

回溯剪枝，dfs的应用，需要注意的是剪枝的条件，不仅需要和之前的字符没有重复过，每个字符串里也不应该有重复的字符串。

```java
public class P5240 {

    int ret = 0;
    public int maxLength(List<String> arr) {
        int n = arr.size();
        dfs(0, n, new ArrayList<>(), new HashSet<>(), arr);
        return ret;
    }

    private void dfs(int start, int n, List<String> path, Set<Character> chars, List<String> arr) {
        if (start == n) {
            int sum = chars.size();
            ret = Math.max(ret, sum);
            return;
        }
        for (int i = start; i < n; i++) {
            String s = arr.get(i);
            if (canAdd(chars, s)) {
                path.add(s);
                addToset(s, chars);
                dfs(i + 1, n, path, chars, arr);
                removeFromSets(s, chars);
                path.remove(path.size() - 1);
            } else {
                dfs(i + 1, n, path, chars, arr);
            }
        }
    }
    private void removeFromSets(String s, Set<Character> chars) {
        for (int i = 0; i < s.length(); i++) {
            chars.remove(s.charAt(i));
        }
    }
    private void addToset(String s, Set<Character> chars) {
        for (int i = 0; i < s.length(); i++) {
            chars.add(s.charAt(i));
        }
    }
    private boolean canAdd(Set<Character> chars, String s) {
        int[] count = new int[26];
        for (int i = 0; i < s.length(); i++) {
            char ch = s.charAt(i);
            if (chars.contains(ch)) {
                return false;
            }
            count[ch - 'a']++;
            if (count[ch - 'a'] > 1) {
                return false;
            }
        }
        return true;
    }
}
```

### 5241. 铺瓷砖

本题看起来像个DP问题，但是最后一个例子像是个反例，谷歌了一下，发现问题并不简单，不过本题的数据集较小，后来看很多人直接打表了。

想继续研究的可以参考：

http://int-e.eu/~bf3/squares/

http://www.squaring.net/sq/tws.html