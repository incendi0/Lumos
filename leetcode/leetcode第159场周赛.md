# leetcode第159场周赛

这个题解拖了一周，自己记性不好，做过的题如果不记下来可能以后碰到就不怎么记得了，所以还是花了时间做个笔记。

### 1232. 缀点成线

题目比较简单，数据集比较小也对边界条件进行了规避，写起来就很快。判断多个点是否在一条线上，考虑斜率正常和无穷大的两种情况。

```java
public class P5230 {
    public boolean checkStraightLine(int[][] coordinates) {
        int x1 = coordinates[0][0];
        int y1 = coordinates[0][1];
        int x2 = coordinates[1][0];
        int y2 = coordinates[1][1];

        if (y1 == y2) {
            for (int i = 2; i < coordinates.length; i++) {
                if (coordinates[i][1] != y1) {
                    return false;
                }
            }
            return true;
        }

        for (int i = 2; i < coordinates.length; i++) {
            int x3 = coordinates[i][0];
            int y3 = coordinates[i][1];
            int dxi1 = x3 - x1;
            int dyi2 = y3 - y1;
            int dx21 = x2 - x1;
            int dy22 = y2 - y1;
            if (dy22 * dxi1 != dx21 * dyi2) {
                return false;
            }
        }
        return true;
    }
}
```

### 1233. 删除子文件夹

字典树的应用，先构造然后collect最小集。

```java
public class P5231 {
    class TrieNode {
        boolean isEnd;
        Map<String, TrieNode> subNodes;

        public TrieNode() {
            this.isEnd = false;
            this.subNodes = new HashMap<>();
        }
    }

    public List<String> removeSubfolders(String[] folder) {
        TrieNode root = constructTrie(folder);
        List<String> ret = new ArrayList<>();
        traverse(root, "", ret);
        return ret;
    }

    private void traverse(TrieNode root, String path, List<String> ret) {
        root.subNodes.forEach((k, v) -> {
            if (v.isEnd) {
                ret.add(path + "/" + k);
            } else {
                traverse(v, path + "/" + k, ret);
            }
        });
    }
  
    private TrieNode constructTrie(String[] folders) {
        TrieNode root = new TrieNode();
        for (String folder : folders) {
            TrieNode curr = root;
            for (String path : folder.split("/")) {
                if ("".equals(path)) {
                    continue;
                }
                curr.subNodes.putIfAbsent(path, new TrieNode());
                curr = curr.subNodes.get(path);
            }
            curr.isEnd = true;
        }
        return root;
    }
}
```

### 1234. 替换子串得到平衡字符串

滑动窗口。计算出四个字符多出平均数的个数的(qcnt, wcnt, ecnt, rcnt)，如果少于或等于则为0。通过两个指针标识可以替换的子字符串的左右端点，且子字符串的四个字符的数目均多于前面计算出的cnt，这样替换这个子字符串后就可以达到平衡。

```java
public class P5232 {
    public int balancedString(String s) {
        int[] count = new int[256];
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i)]++;
        }
        int average = s.length() / 4;
        int qcnt = count['Q'] > average ? count['Q'] - average : 0;
        int wcnt = count['W'] > average ? count['W'] - average : 0;
        int ecnt = count['E'] > average ? count['E'] - average : 0;
        int rcnt = count['R'] > average ? count['R'] - average : 0;
        if (qcnt == 0 && wcnt == 0 && ecnt == 0 && rcnt == 0) {
            return 0;
        }
        Arrays.fill(count, 0);
        int j = 0, ret = Integer.MAX_VALUE;
        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i)]++;
            while (i >= j && count['Q'] >= qcnt && count['W'] >= wcnt && count['E'] >= ecnt && count['R'] >= rcnt) {
                ret = Math.min(ret, i - j + 1);
                count[s.charAt(j++)]--;
            }
        }
        return ret;
    }
}
```

### 1235. 规划兼职工作

动态规划，二分搜索。

```java
public class P5233 {

//    先按照每段的结束时间排序
//    dp[i] 代表排序后第选择第i个schedule能获得最大profit
//    初始值dp[0]为结束最早的第一份job的profit，考虑边界排序的时候让profit高的排前面  
//    dp[i] = max(dp[i - 1], dp[j] + profit[i])
//    j代表schedule[j].end <= schedule[i].start的最大j
    public int jobScheduling(int[] startTime, int[] endTime, int[] profit) {
        int n  = startTime.length;
        Schedule[] schedules = new Schedule[n];
        for (int i = 0; i < n; i++) {
            schedules[i] = new Schedule(startTime[i], endTime[i], profit[i] );
        }
        Arrays.sort(schedules);
        int[] dp = new int[n];
        dp[0] = schedules[0].profit;
        for (int i = 1; i < n; i++) {
            int j = getLastAvailPoint(schedules, schedules[i].start);
            dp[i] = Math.max(dp[i - 1], (j >= 0 ? dp[j] : 0) + schedules[i].profit);
        }

        return dp[n - 1];
    }

    private int getLastAvailPoint(Schedule[] schedules, int target) {
        int n = schedules.length;
        if (schedules.length == 0 || schedules[0].end > target) {
            return -1;
        }
        int lhs = 0, rhs = n - 1;
        while (lhs < rhs) {
            int mid = (lhs + rhs + 1) / 2;
            if (schedules[mid].end <= target) {
                lhs = mid;
            } else {
                rhs = mid - 1;
            }
        }
        return rhs;
    }

    class Schedule implements Comparable<Schedule> {
        int start, end;
        int profit;

        public Schedule(int start, int end, int profit) {
            this.start = start;
            this.end = end;
            this.profit = profit;
        }

        @Override
        public int compareTo(Schedule o) {
            if (end == o.end) {
                return profit - o.profit;
            }
            return end - o.end;
        }
    }
}
```

