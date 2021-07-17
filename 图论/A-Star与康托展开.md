# A*+康托展开

## 例题

[leetcode 773](https://leetcode-cn.com/problems/sliding-puzzle/)

### 未使用康托展开优化

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Set;

class Solution {
    private static final String TARGET = "123450";
    private static final int[][] DIRECTIONS = new int[][]{{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    private static final int[][] MANHATTAN_DIST = {
            {0, 1, 2, 1, 2, 3},
            {1, 0, 1, 2, 1, 2},
            {2, 1, 0, 3, 2, 1},
            {1, 2, 3, 0, 1, 2},
            {2, 1, 2, 1, 0, 1},
            {3, 2, 1, 2, 1, 0}
    };

    private static void swap(char[] c, int a, int b) {
        char t = c[a];
        c[a] = c[b];
        c[b] = t;
    }

    private static int h(String str) {
        int ret = 0;
        for (int i = 0; i < 6; ++i) {
            if (str.charAt(i) != '0') {
                ret += MANHATTAN_DIST[i][str.charAt(i) - '1'];
            }
        }
        return ret;
    }

    public int slidingPuzzle(int[][] board) {
        char[] boardChars = new char[6];
        for (int i = 0; i < 2; i++) {
            for (int j = 0; j < 3; j++) {
                boardChars[i * 3 + j] = (char) (board[i][j] + '0');
            }
        }
        String begin = String.valueOf(boardChars);
        if (TARGET.equals(begin)) {
            return 0;
        }

        if (!check(begin)) {
            return -1;
        }

        Set<String> visited = new HashSet<>();
        PriorityQueue<Node> pq = new PriorityQueue<>(Comparator.comparingInt(x -> x.val));
        Map<String, Node> map = new HashMap<>();
        Node root = new Node(begin, h(begin), 0);
        pq.offer(root);
        map.put(begin, root);

        while (!pq.isEmpty()) {
            Node poll = pq.poll();
            char[] c = poll.str.toCharArray();
            int step = poll.step;
            if (poll.str.equals(TARGET)) {
                return step;
            }

            for (int[] dir : DIRECTIONS) {
                int idx = poll.str.indexOf('0');
                int x = idx / 3;
                int y = idx % 3;
                int nx = x + dir[0];
                int ny = y + dir[1];
                if (nx < 0 || nx >= 2 || ny < 0 || ny >= 3) {
                    continue;
                }
                int newIdx = nx * 3 + ny;

                swap(c, idx, newIdx);
                String next = String.valueOf(c);
                swap(c, idx, newIdx);

                if (visited.contains(next)) {
                    continue;
                }
                if (!map.containsKey(next) || map.get(next).step > step + 1) {
                    Node node = new Node(next, step + 1 + h(next), step + 1);
                    map.put(next, node);
                    pq.offer(node);
                }
            }
        }

        return -1;
    }

    boolean check(String str) {
        char[] cs = str.toCharArray();
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 6; i++) {
            if (cs[i] != '0') {
                list.add(cs[i] - '0');
            }
        }
        int cnt = 0;
        for (int i = 0; i < list.size(); i++) {
            for (int j = i + 1; j < list.size(); j++) {
                if (list.get(i) < list.get(j)) {
                    cnt++;
                }
            }
        }
        return cnt % 2 == 0;
    }

    class Node {
        private String str;
        private int val;
        private int step;

        /**
         * str : 对应字符串
         * val : 估值（与目标字符串 target 的最小转换成本）
         * step: 对应字符串是经过多少步转换而来
         */
        public Node(String str, int val, int step) {
            this.str = str;
            this.val = val;
            this.step = step;
        }
    }
}

```

### 康托展开优化

