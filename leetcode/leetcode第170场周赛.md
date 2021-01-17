# leetcode第170场周赛

### [5303. 解码字母到整数映射](https://leetcode-cn.com/problems/decrypt-string-from-alphabet-to-integer-mapping/)

给你一个字符串 s，它由数字（'0' - '9'）和 '#' 组成。我们希望按下述规则将 s 映射为一些小写英文字符：

- 字符（'a' - 'i'）分别用（'1' - '9'）表示。
- 字符（'j' - 'z'）分别用（'10#' - '26#'）表示。 
  返回映射之后形成的新字符串。

题目数据保证映射始终唯一。

示例 1：

> 输入：s = "10#11#12"
> 输出："jkab"
> 解释："j" -> "10#" , "k" -> "11#" , "a" -> "1" , "b" -> "2".

示例 2：

> 输入：s = "1326#"
> 输出："acz"

示例 3：

> 输入：s = "25#"
> 输出："y"

示例 4：

> 输入：s = "12345678910#11#12#13#14#15#16#17#18#19#20#21#22#23#24#25#26#"
> 输出："abcdefghijklmnopqrstuvwxyz"


提示：

- 1 <= s.length <= 1000
- s[i] 只包含数字（'0'-'9'）和 '#' 字符。
- s 是映射始终存在的有效字符串。

算法：

```Java
public String freqAlphabets(String s) {
    String[] xs = s.split("#");
    StringBuilder sb = new StringBuilder();
    boolean endsWithNo = s.endsWith("#");
    for (int i = 0; i < xs.length; i++) {
        String x = xs[i];
        if (i == xs.length - 1 && !endsWithNo) {
            append(sb, false, x);
        } else {
            append(sb, true, x);
        }
    }
    return sb.toString();
}

private void append(StringBuilder sb, boolean useLastTwo, String x) {
    if (!useLastTwo) {
        for (int i = 0; i < x.length(); i++) {
            char ch = (char) (x.charAt(i) - '1' + 'a');
            sb.append(ch);
        }
    } else {
        for (int i = 0; i < x.length() - 2; i++) {
            char ch = (char) (x.charAt(i) - '1' + 'a');
            sb.append(ch);
        }
        int n = Integer.parseInt(x.substring(x.length() - 2));
        char ch = (char) (n - 1 + 'a');
        sb.append(ch);
    }
}
```

### [5304. 子数组异或查询](https://leetcode-cn.com/problems/xor-queries-of-a-subarray/)

有一个正整数数组 arr，现给你一个对应的查询数组 queries，其中 queries[i] = [Li, Ri]。

对于每个查询 i，请你计算从 Li 到 Ri 的 XOR 值（即 arr[Li] xor arr[Li+1] xor ... xor arr[Ri]）作为本次查询的结果。

并返回一个包含给定查询 queries 所有结果的数组。

示例 1：

> 输入：arr = [1,3,4,8], queries = [[0,1],[1,2],[0,3],[3,3]]
> 输出：[2,7,14,8] 
> 解释：
> 数组中元素的二进制表示形式是：
> 1 = 0001 
> 3 = 0011 
> 4 = 0100 
> 8 = 1000 
> 查询的 XOR 值为：
> [0,1] = 1 xor 3 = 2 
> [1,2] = 3 xor 4 = 7 
> [0,3] = 1 xor 3 xor 4 xor 8 = 14 
> [3,3] = 8

示例 2：

> 输入：arr = [4,8,2,10], queries = [[2,3],[1,3],[0,0],[0,3]]
> 输出：[8,0,4,4]


提示：

- 1 <= arr.length <= 3 * 10^4
- 1 <= arr[i] <= 10^9
- 1 <= queries.length <= 3 * 10^4
- queries[i].length == 2
- 0 <= queries[i][0] <= queries[i][1] < arr.length

算法：

(前缀和)O(N)

1. 对于区间[L, R]，XorRange(L, R) = XorRange(0, R) ^ XorRange(0, L - 1);
2. 前提先计算XorRange(0, i)

```Java
public int[] xorQueries(int[] arr, int[][] queries) {
    int n = arr.length;
    int[] rangeXor = new int[n];
    rangeXor[0] = arr[0];
    for (int i = 1; i < n; i++) {
        rangeXor[i] = rangeXor[i - 1] ^ arr[i];
    }
    int m = queries.length;
    int[] ret = new int[m];
    for (int i = 0; i < m; i++) {
        int l = queries[i][0], r = queries[i][1];
        if (l == 0) {
            ret[i] = rangeXor[r];
        } else {
            ret[i] = rangeXor[r] ^ rangeXor[l - 1];
        }
    }
    return ret;
}
```

### [5305. 获取你好友已观看的视频](https://leetcode-cn.com/problems/get-watched-videos-by-your-friends/)

有 n 个人，每个人都有一个  0 到 n-1 的唯一 id 。

给你数组 watchedVideos  和 friends ，其中 watchedVideos[i]  和 friends[i] 分别表示 id = i 的人观看过的视频列表和他的好友列表。

Level 1 的视频包含所有你好友观看过的视频，level 2 的视频包含所有你好友的好友观看过的视频，以此类推。一般的，Level 为 k 的视频包含所有从你出发，最短距离为 k 的好友观看过的视频。

给定你的 id  和一个 level 值，请你找出所有指定 level 的视频，并将它们按观看频率升序返回。如果有频率相同的视频，请将它们按名字字典序从小到大排列。

示例 1：

> 输入：watchedVideos = [["A","B"],["C"],["B","C"],["D"]], friends = [[1,2],[0,3],[0,3],[1,2]], id = 0, level = 1
> 输出：["B","C"] 
>
> ![level1](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/03/leetcode_friends_1.png)
>
> 解释：
> 你的 id 为 0 ，你的朋友包括：
> id 为 1 -> watchedVideos = ["C"] 
> id 为 2 -> watchedVideos = ["B","C"] 
> 你朋友观看过视频的频率为：
> B -> 1 
> C -> 2

示例 2：

> 输入：watchedVideos = [["A","B"],["C"],["B","C"],["D"]], friends = [[1,2],[0,3],[0,3],[1,2]], id = 0, level = 2
>
> ![leve2](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/03/leetcode_friends_2.png)
>
> 输出：["D"]
> 解释：
> 你的 id 为 0 ，你朋友的朋友只有一个人，他的 id 为 3 。


提示：

- n == watchedVideos.length == friends.length
- 2 <= n <= 100
- 1 <= watchedVideos[i].length <= 100
- 1 <= watchedVideos[i][j].length <= 8
- 0 <= friends[i].length < n
- 0 <= friends[i][j] < n
- 0 <= id < n
- 1 <= level < n
- 如果 friends[i] 包含 j ，那么 friends[j] 包含 i

算法：

(BFS)

1. 好友无向图连表构造，从id出发，距离为level，或者说是进行level次搜索；
2. 使用哈希表构造结果，注意，Java中int[]的equals是==，不是值相等，比较可参考Arrays.equals(int[], int[])

```Java
public List<String> watchedVideosByFriends(List<List<String>> watchedVideos, int[][] friends, int id, int level) {
    int n = friends.length;
    List<List<Integer>> graph = new ArrayList<>(n);
    for (int i = 0; i < n; i++) {
        graph.add(new ArrayList<>());
    }
    for (int i = 0; i < n; i++) {
        int[] friend = friends[i];
        for (int j = 0; j < friend.length; j++) {
            graph.get(i).add(friend[j]);
        }
    }
    Queue<Integer> q = new LinkedList<>();
    q.add(id);
    boolean[] visited = new boolean[n];
    visited[id] = true;
    for (int i = 0; i < level; i++) {
        int zs = q.size();
        for (int j = 0; j < zs; j++) {
            int t = q.poll();
            List<Integer> fs = graph.get(t);
            for (int f : fs) {
                if (!visited[f]) {
                    q.add(f);
                    visited[f] = true;
                }
            }
        }
    }
    List<String> ret = new ArrayList<>();
    if (q.isEmpty()) {
        return ret;
    }
    Map<String, Integer> map  = new HashMap<>();
    for (int f : q) {
        List<String> ss = watchedVideos.get(f);
        for (String s : ss) {
            map.put(s, map.getOrDefault(s, 0) + 1);
        }
    }
    List<Map.Entry<String, Integer>> list = new ArrayList<>(map.entrySet());
    list.sort((o1, o2) -> {
        if (o1.getValue().equals(o2.getValue())) {
            return o1.getKey().compareTo(o2.getKey());
        } else {
            return o1.getValue() - o2.getValue();
        }
    });
    list.forEach(e -> ret.add(e.getKey()));
    return ret;
}
```

### [5306. 让字符串成为回文串的最少插入次数](https://leetcode-cn.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)

给你一个字符串 s ，每一次操作你都可以在字符串的任意位置插入任意字符。

请你返回让 s 成为回文串的 最少操作次数 。

「回文串」是正读和反读都相同的字符串。

示例 1：

> 输入：s = "zzazz"
> 输出：0
> 解释：字符串 "zzazz" 已经是回文串了，所以不需要做任何插入操作。

示例 2：

> 输入：s = "mbadm"
> 输出：2
> 解释：字符串可变为 "mbdadbm" 或者 "mdbabdm" 。

示例 3：

> 输入：s = "leetcode"
> 输出：5
> 解释：插入 5 个字符后字符串变为 "leetcodocteel" 。

示例 4：

> 输入：s = "g"
> 输出：0

示例 5：

> 输入：s = "no"
> 输出：1


提示：

- 1 <= s.length <= 500
- s 中所有字符都是小写字母。

算法：

(动态规划)(O(N^2))

1. f(i, j)表示将区间[i, j]变为回文串的最小操作数；
2. 状态转移，f(i, j) = min(f(i + 1, j), f(i, j - 1)) + 1表示区间[i, j]可以通过将区间[i + 1, j]变为回文串之后在j后插入一个s[i]，或者将区间[i, j - 1]变为回文串之后在i前插入一个s[j]；当s[i] == s[j]时，f(i, j) = min(f(i, j), f(i + 1, j - 1));
3. 最终答案为f(0, n - 1)

```Java
public int minInsertions(String s) {
    int n = s.length();
    int[][] f = new int[n][n];
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i < n - len + 1; i++) {
            int j = i + len - 1;
            f[i][j] = Math.min(f[i + 1][j], f[i][j - 1]) + 1;
            if (s.charAt(i) == s.charAt(j)) {
                f[i][j] = Math.min(f[i][j], f[i + 1][j - 1]);
            }
        }
    }
    return f[0][n - 1];
}
```