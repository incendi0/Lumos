



# leetcode第162场周赛

#### [1252. 奇数值单元格的数目](https://leetcode-cn.com/problems/cells-with-odd-values-in-a-matrix/)

直接模拟，空间优化一些，记录增量操作的行和列，然后枚举每个单元格，找出奇数值。

```python3
class Solution:
    def oddCells(self, n: int, m: int, indices: List[List[int]]) -> int:
        rows = [0] * n
        cols = [0] * m
        for x, y in indices:
            rows[x] += 1
            cols[y] += 1
        return sum((rows[x] + cols[y]) % 2 == 1 for x in range(n) for y in range(m))
```

#### [1253. 重构 2 行二进制矩阵](https://leetcode-cn.com/problems/reconstruct-a-2-row-binary-matrix/)

开始以为是dfs问题，如果获得所有符合要求的集合会超时，不停的debug，甚至加了一个count计数。后来发现，只有当colsum[i] == 1时，需要判断1，0怎么放置的问题。有点类似生成匹配的括号，如果upper > lower，则upper放1，如果lower > 0则lower放1。

```java
public class P5256 {

//    private int count = 0;
//
//    public List<List<Integer>> reconstructMatrix(int upper, int lower, int[] colsum) {
//        List<List<Integer>> ret = new ArrayList<>();
//        int sum = 0;
//        for (int col : colsum) {
//            sum += col;
//        }
//        if (sum != upper + lower) {
//            return ret;
//        }
//
//        int n = colsum.length, remaining = n;
//        int[][] matrix = new int[2][n];
//        for (int i = 0; i < n; i++) {
//            if (colsum[i] == 2) {
//                matrix[0][i] = 1;
//                matrix[1][i] = 1;
//                upper -= 1;
//                lower -= 1;
//                remaining--;
//            } else if (colsum[i] == 0) {
//                remaining--;
//            }
//        }
//        dfs(n, remaining, matrix, ret, upper, lower, colsum);
//        return ret;
//    }
//
//    private void dfs(int n, int remaining, int[][] matrix, List<List<Integer>> ret, int upper, int lower, int[] colsum) {
//        if (remaining == 0 && upper == 0 && lower == 0) {
//            for (int i = 0; i < matrix.length; i++) {
//                ret.add(new ArrayList<>());
//                for (int j = 0; j < matrix[0].length; j++) {
//                    ret.get(i).add(matrix[i][j]);
//                }
//            }
//            count++;
//        }
//        if (count == 1 || upper < 0 || lower < 0 || remaining <= 0 || n <= 0) {
//            return;
//        }
//        int c = n - 1;
//        if (colsum[c] == 1) {
//            for (int i = 0; i < 2; i++) {
//                matrix[0][c] = i;
//                matrix[1][c] = 1 - i;
//                dfs(n - 1, remaining - 1, matrix, ret, upper - i, lower + i - 1, colsum);
//            }
//        } else {
//            dfs(n - 1, remaining, matrix, ret, upper, lower, colsum);
//
//        }
//    }


    public List<List<Integer>> reconstructMatrix(int upper, int lower, int[] colsum) {
        List<List<Integer>> ret = new ArrayList<>();
        int sum = 0;
        for (int col : colsum) {
            sum += col;
        }
        if (sum != upper + lower) {
            return ret;
        }
        ret.add(new ArrayList<>());
        ret.add(new ArrayList<>());
        for (int i = 0; i < colsum.length; i++) {
            if (colsum[i] == 0 || colsum[i] == 2) {
                ret.get(0).add(colsum[i] / 2);
                ret.get(1).add(colsum[i] / 2);
                upper -= colsum[i] / 2;
                lower -= colsum[i] / 2;
            } else if (colsum[i] == 1) {
                if (upper > lower) {
                    ret.get(0).add(1);
                    upper--;
                    ret.get(1).add(0);
                } else if (lower > 0) {
                    ret.get(1).add(1);
                    lower--;
                    ret.get(0).add(0);
                }
            }
        }
        return upper == 0 && lower == 0 ? ret : new ArrayList<>();
    }

    public static void main(String[] args) {
        P5256 solution = new P5256();
        int[] colsum = {0,1,2,0,0,0,0,0,2,1,2,1,2};
        int upper = 9, lower = 2;
        List<List<Integer>> ret = solution.reconstructMatrix(upper, lower, colsum);
        for (int i = 0; i < ret.size(); i++) {
            for (int j = 0; j < ret.get(i).size(); j++) {
                System.out.print(ret.get(i).get(j));
            }
            System.out.println();
        }
    }

}
```

#### [1254. 统计封闭岛屿的数目](https://leetcode-cn.com/problems/number-of-closed-islands/)

dfs或者bfs，flood fill

```java
public class P5257 {

    private static final int[] dx = {0, 1, 0, -1};
    private static final int[] dy = {1, 0, -1, 0};
//
//    public int closedIsland(int[][] grid) {
//        int m = grid.length, n = grid[0].length;
//        boolean[][] visit = new boolean[m][n];
//        int ret = 0;
//        for (int i = 0; i < m; i++) {
//            for (int j = 0; j < n; j++) {
//                if (grid[i][j] == 0 && !visit[i][j]) {
//                    if (dfs(grid, visit, i, j, m, n)) {
//                        ret += 1;
//                    }
//                }
//            }
//        }
//        return ret;
//    }
//
//    private boolean dfs(int[][] grid, boolean[][] visit, int r, int c, int rs, int cs) {
//        if (r < 0 || r >= rs || c < 0 || c >= cs) {
//            return false;
//        }
//        if (grid[r][c] == 1 || visit[r][c]) {
//            return true;
//        }
//        visit[r][c] = true;
//        boolean ret = true;
//        for (int i = 0; i < 4; i++) {
//            int nr = r + dx[i];
//            int nc = c + dy[i];
////            此处非短路与
//            ret = ret & dfs(grid, visit, nr, nc, rs, cs);
//        }
//        return ret;
//    }

    public int closedIsland(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int ret = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 0) {
                    ret += bfs(grid, i, j, m, n) ? 1 : 0;
                }
            }
        }
        return ret;
    }

    private boolean bfs(int[][] grid, int r, int c, int rs, int cs) {
        Queue<int[]> q = new LinkedList<>();
        q.add(new int[] {r, c});
        boolean ret = true;
        while (!q.isEmpty()) {
            int[] curr = q.poll();
            for (int i = 0; i < 4; i++) {
                int nr = curr[0] + dx[i];
                int nc = curr[1] + dy[i];
                if (nr < 0 || nr >= rs || nc < 0 || nc >= cs) {
                    ret = false;
                    continue;
                }
                if (grid[nr][nc] == 0) {
                    grid[nr][nc] = 1;
                    q.add(new int[] {nr, nc});
                }
            }
        }
        return ret;
    }

    public static void main(String[] args) {
        int[][] grid = {
                {0, 0, 1, 1, 0, 1, 0, 0, 1, 0},
                {1, 1, 0, 1, 1, 0, 1, 1, 1, 0},
                {1, 0, 1, 1, 1, 0, 0, 1, 1, 0},
                {0, 1, 1, 0, 0, 0, 0, 1, 0, 1},
                {0, 0, 0, 0, 0, 0, 1, 1, 1, 0},
                {0, 1, 0, 1, 0, 1, 0, 1, 1, 1},
                {1, 0, 1, 0, 1, 1, 0, 0, 0, 1},
                {1, 1, 1, 1, 1, 1, 0, 0, 0, 0},
                {1, 1, 1, 0, 0, 1, 0, 1, 0, 1},
                {1, 1, 1, 0, 1, 1, 0, 1, 1, 0}
        };

        int[][] grid1 = {
                {1,1,1,1,1,1,1,0},
                {1,0,0,0,0,1,1,0},
                {1,0,1,0,1,1,1,0},
                {1,0,0,0,0,1,0,1},
                {1,1,1,1,1,1,1,0}
        };
        P5257 solutoin = new P5257();
        System.out.println(solutoin.closedIsland(grid));
        System.out.println(solutoin.closedIsland(grid1));
        System.out.println();
    }
}
```

#### [1255. 得分最高的单词集合](https://leetcode-cn.com/problems/maximum-score-words-formed-by-letters/)

枚举words的subset，看是否满足letters的字符出现次数，计算score，获取最大值。

```java
public class P5258 {

    public int maxScoreWords(String[] words, char[] letters, int[] score) {
        int n = words.length;
        int[] chcnt = new int[26];
        for (char ch : letters) {
            chcnt[ch - 'a']++;
        }
        int ret = 0;
        for (int i = 0; i < (1 << n); i++) {
            int totalScore = 0;
            int[] usedChar = new int[26];
            for (int j = 0; j < n; j++) {
                if (((i >> j) & 1) == 1) {
                    for (int k = 0; k < words[j].length(); k++) {
                        usedChar[words[j].charAt(k) - 'a']++;
                        totalScore += score[words[j].charAt(k) - 'a'];
                    }
                }
            }
            boolean flag = true;
            for (int k = 0; k < 26; k++) {
                if (usedChar[k] > chcnt[k]) {
                    flag = false;
                    break;
                }
            }
            if (flag) {
                ret = Math.max(ret, totalScore);
            }
        }
        return ret;
    }

    public static void main(String[] args) {
        P5258 solution = new P5258();
        System.out.println(solution.maxScoreWords(
                new String[] {"dog","cat","dad","good"},
                new char[] {'a','a','c','d','d','d','g','o','o'},
                new int[] {1,0,9,5,0,0,3,0,0,0,0,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0}));
    }
}
```