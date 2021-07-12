# 双向BFS

## 思路

我们知道，递归树的展开形式是一棵多阶树。

使用朴素 BFS 进行求解时，队列中最多会存在“两层”的搜索节点。

因此搜索空间的上界取决于**目标节点所在的搜索层次的深度所对应的宽度**。

下图展示了朴素 BFS 可能面临的搜索空间爆炸问题：

![image.png](https://raw.githubusercontent.com/FSYP/image-host/master/img/20210706150537.png)

**在朴素的 BFS 实现中，空间的瓶颈主要取决于搜索空间中的最大宽度。**

那么有没有办法让我们不使用这么宽的搜索空间，同时又能保证搜索到目标结果呢？

「双向 BFS」 可以很好的解决这个问题：

**同时从两个方向开始搜索，一旦搜索到相同的值，意味着找到了一条联通起点和终点的最短路径。**

对于「有解」、「有一定数据范围」同时「层级节点数量以倍数或者指数级别增长」的情况，「双向 BFS」的搜索空间通常只有「朴素 BFS」的空间消耗的几百分之一，甚至几千分之一。

![image.png](https://raw.githubusercontent.com/FSYP/image-host/master/img/20210706150651.png)

「双向 BFS」的基本实现思路如下：

1. 创建「两个队列」分别用于两个方向的搜索；

2. 创建「两个哈希表」用于「解决相同节点重复搜索」和「记录转换次数」；

3. 为了尽可能让两个搜索方向“平均”，每次从队列中取值进行扩展时，先判断哪个队列容量较少；

4. 如果在搜索过程中「搜索到对方搜索过的节点」，说明找到了最短路径。

「双向 BFS」基本思路对应的伪代码大致如下：

```java
d1、d2 为两个方向的队列
m1、m2 为两个方向的哈希表，记录每个节点距离起点的
    
// 只有两个队列都不空，才有必要继续往下搜索
// 如果其中一个队列空了，说明从某个方向搜到底都搜不到该方向的目标节点
while(!d1.isEmpty() && !d2.isEmpty()) {
    if (d1.size() < d2.size()) {
        update(d1, m1, m2);
    } else {
        update(d2, m2, m1);
    }
}

// update 为从队列 d 中取出一个元素进行「一次完整扩展」的逻辑
void update(Deque d, Map cur, Map other) {}
```

---

[查看原文](https://leetcode-cn.com/problems/open-the-lock/solution/gong-shui-san-xie-yi-ti-shuang-jie-shuan-wyr9/)

## 例题

[LeetCode 752](https://leetcode-cn.com/problems/open-the-lock/)

### 示例代码

```java
import java.util.ArrayDeque;
import java.util.Arrays;
import java.util.Collections;
import java.util.Deque;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

class Solution {

    public static final String BEGIN = "0000";

    public int openLock(String[] deadends, String target) {
        Set<String> deadSet = new HashSet<>(Arrays.asList(deadends));
        if (deadSet.contains(BEGIN)) {
            return -1;
        }
        if (BEGIN.equals(target)) {
            return 0;
        }

        // 起点队列 终点队列
        Deque<String> d1 = new ArrayDeque<>();
        Deque<String> d2 = new ArrayDeque<>();
        // 结点 步数
        Map<String, Integer> m1 = new HashMap<>();
        Map<String, Integer> m2 = new HashMap<>();
        d1.offer(BEGIN);
        m1.put(BEGIN, 0);
        d2.offer(target);
        m2.put(target, 0);

        Set<String> visited = new HashSet<>(Collections.singletonList(BEGIN));
        visited.addAll(deadSet);

        while (!d1.isEmpty() && !d2.isEmpty()) {
            int t;
            if (d1.size() <= d2.size()) {
                t = update(d1, m1, m2, visited);
            } else {
                t = update(d2, m2, m1, visited);
            }
            if (t != -1) {
                return t;
            }
        }
        return -1;
    }

    private int update(Deque<String> deque, Map<String, Integer> cur, Map<String, Integer> other, Set<String> visited) {
        String poll = deque.poll();
        char[] chars = poll.toCharArray();
        int step = cur.get(poll);
        for (int j = 0; j < 4; j++) {
            char tmp = chars[j];

            chars[j] = tmp == '9' ? '0' : (char) (tmp + 1);
            String s = new String(chars);
            if (!visited.contains(s) && !cur.containsKey(s)) {
                if (other.containsKey(s)) {
                    return step + 1 + other.get(s);
                } else {
                    deque.offer(s);
                    cur.put(s, step + 1);
                }
            }

            chars[j] = tmp == '0' ? '9' : (char) (tmp - 1);
            s = new String(chars);
            if (!visited.contains(s) && !cur.containsKey(s)) {
                if (other.containsKey(s)) {
                    return step + 1 + other.get(s);
                } else {
                    deque.offer(s);
                    cur.put(s, step + 1);
                }
            }

            chars[j] = tmp;
        }
        return -1;
    }
}

```

