# A*算法

## 思路

A\*搜索算法（英文：A\*search algorithm，A\*读作 A-star），简称 A\*算法，是一种在图形平面上，对于有多个节点的路径求出最低通过成本的算法。它属于图遍历（英文：Graph traversal）和最佳优先搜索算法（英文：Best-first search），亦是 `BFS` 的改进。

定义起点 $s$，终点 $t$，从起点（初始状态）开始的距离函数 $g(x)$，到终点（最终状态）的距离函数 $h(x)$，$h^{\ast}(x)$，以及每个点的估价函数 $f(x)=g(x)+h(x)$。

A\*算法每次从优先队列中取出一个 $f$ 最小的元素，然后更新相邻的状态。

如果 $h\leq h*$，则 A\*算法能找到最优解。

上述条件下，如果 $h$ 满足三角形不等式，则 A\*算法不会将重复结点加入队列。

当 $h=0$ 时，A\*算法变为`DFS`；当 $h=0$ 并且边权为 $1$ 时变为 BFS。

[查看原文](https://oi-wiki.org/search/astar/)

---



在 $A*$ 算法中，我们需要使用四个距离函数 $F(x), G(x), H(x), H^*(x)$ ，其中 $F(x), G(x), H(x)$是可以求出的，而 $H^*(x)$ 是无法求出的，我们需要用 $H(x)$近似 $H^*(x)$。设起点为 $s$，终点为 $t$，这些距离函数的意义如下：

+ $G(x)$ 表示从起点 $s$ 到节点 $x$ 的「实际」路径长度，注意 $G(x)$ 并不一定是最短的；

+ $H(x)$ 表示从节点 $x$ 到终点 $t$ 的「估计」最短路径长度，称为启发函数；

+ $H^*(x)$ 表示从节点 xx 到终点 tt 的「实际」最短路径长度，这是我们在广度优先搜索的过程中无法求出的，我们需要用 $H(x)$ 近似 $H^*(x)$；

+ $F(x)$ 满足 $F(x) = G(x) + H(x)$，即为从起点 $s$ 到终点 $t$ 的「估计」路径长度。我们总是挑选出最小的 $F(x)$ 对应的 $x$ 进行搜索，因此 $A*$ 算法需要借助优先队列来实现。

如果读者熟悉求解最短路的 $Dijkstra$ 算法，就可以发现 $Dijkstra$ 算法是 $A*$ 算法在 $H(x) \equiv 0$ 时的特殊情况。

$A*$ 算法具有两个性质：

+ 如果对于任意的节点 $x$，$H(x) \leq H^*(x)$恒成立，即我们「估计」出的从节点 $x$ 到终点 $t$ 的最短路径长度总是不超过「实际」的最短路径长度，那么称启发函数 $H(x)$ 是可接纳的（admissible heuristic）。在这种情况下，$A*$ 算法一定能找到最短路，但同一节点可能需要加入优先队列并搜索多次，即当我们从优先队列中取出节点 $x$ 时，$G(x)$ 并不一定等于从起点到节点 $x$ 的「实际」最短路径的长度；

+ 如果对于任意的两个节点 $x$ 和 $y$，并且 $x$ 到 $y$ 有一条长度为 $D(x, y)$ 的有向边，$H(x) - H(y) \leq D(x, y)$ 恒成立，并且 $H(t)=0H(t)=0$，那么称启发函数 $H(x)$ 是一致的（consistent heuristic）。可以证明，一致的启发函数一定也是可接纳的。在这种情况下，同一节点只会被加入优先队列一次，并搜索不超过一次，即当我们从优先队列中取出节点 $x$ 时，$G(x)$ 一定等于从起点到节点 $x$ 的「实际」最短路径的长度。

[查看原文](https://leetcode-cn.com/problems/open-the-lock/solution/da-kai-zhuan-pan-suo-by-leetcode-solutio-l0xo/)

## 例题

[LeetCode 752](https://leetcode-cn.com/problems/open-the-lock/)

### 示例代码

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Set;

class Solution {

    public static final String BEGIN = "0000";

    private static int h(String str, String target) {
        int res = 0;
        for (int i = 0; i < 4; i++) {
            int diff = Math.abs(str.charAt(i) - target.charAt(i));
            // 正转 反转
            int min = Math.min(diff, 10 - diff);
            res += min;
        }
        return res;
    }

    public int openLock(String[] deadends, String target) {
        Set<String> visited = new HashSet<>(Arrays.asList(deadends));
        if (visited.contains(BEGIN)) {
            return -1;
        }
        if (BEGIN.equals(target)) {
            return 0;
        }

        PriorityQueue<Node> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a.val));
        Map<String, Node> map = new HashMap<>();
        Node root = new Node(BEGIN, h(BEGIN, target), 0);
        pq.offer(root);
        map.put(BEGIN, root);

        while (!pq.isEmpty()) {
            Node poll = pq.poll();
            char[] chars = poll.str.toCharArray();
            int step = poll.step;
            if (poll.str.equals(target)) {
                return step;
            }
            for (int i = 0; i < 4; i++) {
                for (int j = -1; j <= 1; j += 2) {
                    int cur = chars[i] - '0';
                    int next = (cur + j) % 10;
                    if (next == -1) {
                        next = 9;
                    }

                    char tmp = chars[i];
                    chars[i] = (char) (next + '0');
                    String str = String.valueOf(chars);
                    chars[i] = tmp;
                    if (visited.contains(str)) {
                        continue;
                    }
                    if (!map.containsKey(str) || map.get(str).step > step + 1) {
                        Node node = new Node(str, step + 1 + h(str, target), step + 1);
                        map.put(str, node);
                        pq.offer(node);
                    }
                }
            }
        }

        return -1;
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

