# leetcode第164场周赛

### [5271. 访问所有点的最小时间](https://leetcode-cn.com/problems/minimum-time-visiting-all-points/)

```java
public int minTimeToVisitAllPoints(int[][] points) {
    int ret = 0;
    for (int i = 0; i < points.length - 1; i++) {
        ret += minTimeBetweenPoints(points[i + 1], points[i]);
    }
    return ret;
}

private int minTimeBetweenPoints(int[] p1, int[] p2) {
    if (p1[0] == p2[0]) {
        return Math.abs(p1[1] - p2[1]);
    }
    if (p1[1] == p2[1]) {
        return Math.abs(p1[0] - p2[0]);
    }

    int xabs = Math.abs(p1[0] - p2[0]);
    int yabs = Math.abs(p1[1] - p2[1]);

    return Math.max(xabs, yabs);
}
```

### [5272. 统计参与通信的服务器](https://leetcode-cn.com/problems/count-servers-that-communicate/)

```java
public int countServers(int[][] grid) {
    int n = grid.length, m = grid[0].length;
    int[] rowCount = new int[n];
    int[] colCount = new int[m];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (grid[i][j] == 1) {
                rowCount[i]++;
                colCount[j]++;
            }
        }
    }
    int ret = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (grid[i][j] == 1 && (rowCount[i] > 1 || colCount[j] > 1)) {
                ret++;
            }
        }
    }
    return ret;
}
```

### [5273. 搜索推荐系统](https://leetcode-cn.com/problems/search-suggestions-system/)

字典树，代码可以优化，每个节点放一个大小为3的大根堆。

```java
public class P5273_搜索推荐系统 {

    class TrieNode {
        boolean isEnd;
        TrieNode[] subs;
        Set<Integer> idexSet;

        public TrieNode() {
            this.isEnd = false;
            this.subs = new TrieNode[26];
            this.idexSet = new HashSet<>();
        }
    }

    public List<List<String>> suggestedProducts(String[] products, String searchWord) {
        Map<Integer, String> map = new HashMap<>();
        TrieNode root = constructTrieNode(products, map);
        List<List<String>> ret = new ArrayList<>();
        for (int i = 0; i < searchWord.length(); i++) {
            List<String> level = new ArrayList<>();
            TrieNode curr = root;
            traverse(searchWord.substring(0, i + 1), level, curr, map, products);
            if (level.size() > 3) {
                level = level.subList(0, 3);
            }
            ret.add(level);
            if (level.isEmpty()) {
                break;
            }
        }
        int padding = searchWord.length() - ret.size();
        for (int i = 0; i < padding; i++) {
            ret.add(new ArrayList<>());
        }
        return ret;
    }

    private void traverse(String path, List<String> level, TrieNode curr, Map<Integer, String> map, String[] products) {
        for (int i = 0; i < path.length(); i++) {
            int idx = path.charAt(i) - 'a';
            if (curr.subs[idx] == null) {
                return;
            }
            curr = curr.subs[idx];
        }
        if (curr == null) {
            return;
        }
        curr.idexSet.forEach(k -> {
            level.add(map.get(k));
        });
        level.sort(String::compareTo);
    }

    private TrieNode constructTrieNode(String[] products, Map<Integer, String> map) {
        TrieNode root = new TrieNode();
        for (int j = 0; j < products.length; j++) {
            String product = products[j];
            map.put(j, product);
            TrieNode curr = root;
            for (int i = 0; i < product.length(); i++) {
                int idx = product.charAt(i) - 'a';
                if (curr.subs[idx] == null) {
                    curr.subs[idx] = new TrieNode();
                }
                curr = curr.subs[idx];
                curr.idexSet.add(j);
            }
            curr.isEnd = true;
        }
        return root;
    }

    public static void main(String[] args) {
        P5273_搜索推荐系统 solution = new P5273_搜索推荐系统();
        System.out.println(solution.suggestedProducts(new String[] {"mobile","mouse","moneypot","monitor","mousepad"}, "mouse"));
    }
}
```

### [5274. 停在原地的方案数](https://leetcode-cn.com/problems/number-of-ways-to-stay-in-the-same-place-after-some-steps/)

开始使用回溯，感觉就会超时:<。

动态规划，每次有三种转移，根据操作step递推。

```java
/**
 * f[i][j]表示走了i步，停留在下标j的方案数
 * 初始值 f[0][0] = 1;
 * 转移方程f[i][j] = f[i - 1][j] + f[i - 1][j - 1] + f[i - 1][j + 1]
 * j == 0 || j == arrLen - 1只有两种转移方案
 */
public class P5274_停在原地的方案数 {

    private static final int mod = 1000000007;

    public int numWays(int steps, int arrLen) {
        if (arrLen == 1) {
            return 1;
        }
//        空间优化，不然超出内存限制
        arrLen = Math.min(steps + 1, arrLen);
        int[][] f = new int[steps + 1][arrLen];
        f[0][0] = 1;
        for (int step = 1; step <= steps; step++) {
            f[step][0] = (f[step - 1][0] + f[step - 1][1]) % mod;
            f[step][arrLen - 1] = (f[step - 1][arrLen - 1] + f[step - 1][arrLen - 2]) % mod;
            for (int idx = 1; idx < arrLen - 1; idx++) {
                f[step][idx] = ((f[step - 1][idx] + f[step - 1][idx - 1]) % mod + f[step - 1][idx + 1]) % mod;
            }
        }
        return f[steps][0];
    }
//    private int ret;
//    public int numWays(int steps, int arrLen) {
//        ret = 0;
//        dfs(0, steps, 0, arrLen);
//        return ret;
//    }

//    private void dfs(int start, int end, int idx, int len) {
//        if (start == end && idx == 0) {
//            ret++;
//            return;
//        }
//        if (start > end || idx < 0 || idx >= len) {
//            return;
//        }
//
//        dfs(start + 1, end, idx + 1, len);
//        dfs(start + 1, end, idx, len);
//        dfs(start + 1, end, idx - 1, len);
//    }

    public static void main(String[] args) {
        P5274_停在原地的方案数 solution = new P5274_停在原地的方案数();
        System.out.println(solution.numWays(3, 2));
        System.out.println(solution.numWays(2, 4));
        System.out.println(solution.numWays(4, 2));
    }
}
```