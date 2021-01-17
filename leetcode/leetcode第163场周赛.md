# leetcode第163场周赛

### [1260. 二维网格迁移](https://leetcode-cn.com/problems/shift-2d-grid/)

```java
public List<List<Integer>> shiftGrid(int[][] grid, int k) {
    int n = grid.length, m = grid[0].length, len = n * m;
    k = k % (m * n);
    List<List<Integer>> ret = new ArrayList<>();
    int count = 0;
    for (int i = 0; i < n; i++) {
        ret.add(new ArrayList<>(m));
        for (int j = 0; j < m; j++) {
            int fromIdx = count < k ? len + count - k : count - k;
            ret.get(i).add(grid[fromIdx / m][fromIdx % m]);
            count++;
        }
    }
    return ret;
}
```

### [1261. 在受污染的二叉树中查找元素](https://leetcode-cn.com/problems/find-elements-in-a-contaminated-binary-tree/)

遍历构造二叉树，同时更新为正确的元素。如果是一棵完美的二叉树，则可发现，根据构造规律可以反算出target值的路径，沿着路径就可以寻找到target元素的节点。

```java
public class FindElements {

    private TreeNode root;

    public FindElements(TreeNode root) {
        this.root = root;
        root.val = 0;
        traverse(root);
    }

    private void traverse(TreeNode root) {
        if (root != null) {
            if (root.left != null) {
                root.left.val = root.val * 2 + 1;
                traverse(root.left);
            }
            if (root.right != null) {
                root.right.val = root.val * 2 + 2;
                traverse(root.right);
            }
        }
    }

    public boolean find(int target) {
        List<Integer> ret = new ArrayList<>();
        while (target > 0) {
            int parity = target % 2;
            if (parity == 0) {
                target = target / 2 - 1;
            } else {
                target = (target - 1) / 2;
            }
            ret.add(parity);
        }
        TreeNode curr = root;
        for (int i = ret.size() - 1; i >= 0; i--) {
            if (curr == null) {
                return false;
            }
            if (ret.get(i) == 0) {
                curr = curr.right;
            } else {
                curr = curr.left;
            }
        }
        return curr != null;
    }
}
```

### [1262. 可被三整除的最大和](https://leetcode-cn.com/problems/greatest-sum-divisible-by-three/)

动态规划

1. 最大和可以分解成三种状态，可以被3整除，模3余1和模3余2，分别用f[i]，g[i]和h[i]表示[0..i]区间三种状态的最大和；
2. 初始值：手动计算nums[0]模3的结果，并写入相应的状态数组里。然而这种方法最多有两种状态有初始值（f[i]初始化为0是满足状态要求的，可以被3整除），剩下的不满足状态特点，用0当作标记（因为nums[i]是在[0..10^4]内），如果数组的值是0，则表示[0..i]区间还没有满足状态的最大和；
3. 状态转移：如果当前值能被3整除，f[i]直接相加转移，g[i]和h[i]前一个状态如果不是0，直接转移，否则无法转移，值还是0，同理处理当前值模3余1和余2的情况；
4. 结果为f[n-1]。

```java
public int maxSumDivThree(int[] nums) {
    int n = nums.length;
    int[] f = new int[n], g = new int[n], h = new int[n];
    if (nums[0] % 3 == 0) {
        f[0] = nums[0];
    } else if (nums[0] % 3 == 1) {
        g[0] = nums[0];
    } else {
        h[0] = nums[0];
    }

    for (int i = 1; i < n; i++) {
        if (nums[i] % 3 == 0) {
            f[i] = f[i - 1] + nums[i];
            g[i] = g[i - 1] == 0 ? 0 : g[i - 1] + nums[i];
            h[i] = h[i - 1] == 0 ? 0 : h[i - 1] + nums[i];
        } else if (nums[i] % 3 == 1) {
            f[i] = Math.max(f[i - 1], h[i - 1] == 0 ? f[i - 1] : h[i - 1] + nums[i]);
            g[i] = Math.max(g[i - 1], f[i - 1] + nums[i]);
            h[i] = Math.max(h[i - 1], g[i - 1] == 0 ? h[i - 1] : g[i - 1] + nums[i]);
        } else {
            f[i] = Math.max(f[i - 1], g[i - 1] == 0 ? f[i - 1] : g[i - 1] + nums[i]);
            g[i] = Math.max(g[i - 1], h[i - 1] == 0 ? g[i - 1] : h[i - 1] + nums[i]);
            h[i] = Math.max(h[i - 1], f[i - 1] + nums[i]);
        }
    }
    return f[n - 1];
}
```

### [1263. 推箱子](https://leetcode-cn.com/problems/minimum-moves-to-move-a-box-to-their-target-location/)

[题解来源](https://leetcode-cn.com/problems/minimum-moves-to-move-a-box-to-their-target-location/solution/tie-liao-xia-cuiaoxiangda-lao-de-jie-fa-by-mike-me/)，bfs + 双端队列运用确实很巧妙。

```java
public class P1263_推箱子 {

    private static final int[] dx = {1, 0, -1, 0};
    private static final int[] dy = {0, 1, 0, -1};
    private static final int N = 20;
    private int[][][][] distance = new int[N][N][N][N];

    static class State {
        int bx, by;
        int hx, hy;

        public State(int bx, int by, int hx, int hy) {
            this.bx = bx;
            this.by = by;
            this.hx = hx;
            this.hy = hy;
        }
    }

    public int minPushBox(char[][] grid) {
        int n = grid.length, m = grid[0].length;
        int humanX = 0, humanY = 0;
        int boxX = 0, boxY = 0;
        int destX = 0, destY = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 'S') {
                    humanX = i; humanY = j;
                    grid[i][j] = '.';
                }
                if (grid[i][j] == 'B') {
                    boxX = i; boxY = j;
                    grid[i][j] = '.';
                }
                if (grid[i][j] == 'T') {
                    destX = i; destY = j;
                    grid[i][j] = '.';
                }
            }
        }

        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                for (int k = 0; k < N; k++) {
                    for (int h = 0; h < N; h++) {
                        distance[i][j][k][h] = Integer.MIN_VALUE;
                    }
                }
            }
        }
        distance[boxX][boxY][humanX][humanY] = 0;
        Deque<State> deque = new ArrayDeque<>();
        deque.addLast(new State(boxX, boxY, humanX, humanY));
        while (!deque.isEmpty()) {
            State s = deque.pollFirst();
            boxX = s.bx; boxY = s.by; humanX = s.hx; humanY = s.hy;
            if (boxX == destX && boxY == destY) {
                return distance[boxX][boxY][humanX][humanY];
            }
            for (int k = 0; k < 4; k++) {
                int newHumanX = humanX + dx[k];
                int newHumanY = humanY + dy[k];
                if (!isValid(newHumanX, newHumanY, n, m, grid)) {
                    continue;
                }
                if (newHumanX == boxX && newHumanY == boxY) {
                    continue;
                }
                if (distance[boxX][boxY][newHumanX][newHumanY] >= 0) {
                    continue;
                }
                distance[boxX][boxY][newHumanX][newHumanY] = distance[boxX][boxY][humanX][humanY];
                deque.addFirst(new State(boxX, boxY, newHumanX, newHumanY));
            }

            if (Math.abs(boxX - humanX) + Math.abs(boxY - humanY) == 1) {
                for (int k = 0; k < 4; k++) {
                    int newHumanX = humanX + dx[k];
                    int newHumanY = humanY + dy[k];
                    if (newHumanX == boxX && newHumanY == boxY) {
                        int newBoxX = boxX + dx[k];
                        int newBoxY = boxY + dy[k];
                        if (isValid(newBoxX, newBoxY, n, m, grid)
                                && distance[newBoxX][newBoxY][newHumanX][newHumanY] < 0) {
                            distance[newBoxX][newBoxY][newHumanX][newHumanY] = distance[boxX][boxY][humanX][humanY] + 1;
                            deque.addLast(new State(newBoxX, newBoxY, newHumanX, newHumanY));
                        }
                    }
                }
            }
        }
        return -1;
    }

    private boolean isValid(int x, int y, int rs, int cs, char[][] grid) {
        return x >= 0 && x < rs && y >= 0 && y < cs && grid[x][y] == '.';
    }

    public static void main(String[] args) {
        P1263_推箱子 solution = new P1263_推箱子();
        System.out.println(solution.minPushBox(new char[][] {
                {'#','#','#','#','#','#'},
                {'#','T','#','#','#','#'},
                {'#','.','.','B','.','#'},
                {'#','.','#','#','.','#'},
                {'#','.','.','.','S','#'},
                {'#','#','#','#','#','#'},
        }));

        System.out.println(solution.minPushBox(new char[][] {
                {'#','#','#','#','#','#'},
                {'#','T','.','.','#','#'},
                {'#','.','#','B','.','#'},
                {'#','.','.','.','.','#'},
                {'#','.','.','.','S','#'},
                {'#','#','#','#','#','#'},
        }));

        System.out.println(solution.minPushBox(new char[][] {
                {'#','#','#','#','#','#'},
                {'#','T','#','#','#','#'},
                {'#','.','.','B','.','#'},
                {'#','#','#','#','.','#'},
                {'#','.','.','.','S','#'},
                {'#','#','#','#','#','#'},
        }));

        System.out.println(solution.minPushBox(new char[][] {
                {'#','.','.','#','#','#','#','#'},
                {'#','.','.','T','#','.','.','#'},
                {'#','.','.','.','#','B','.','#'},
                {'#','.','.','.','.','.','.','#'},
                {'#','.','.','.','#','.','S','#'},
                {'#','.','.','#','#','#','#','#'}
        }));

    }
}
```