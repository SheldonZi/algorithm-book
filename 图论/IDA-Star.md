# IDA*

学习 IDA\*之前，请确保您已经学完了 [A\*](./A-star.md) 算法和 [迭代加深搜索](./迭代加深.md)。

## IDA\*简介

IDA\*，即采用迭代加深的 A\*算法。相对于 A\*算法，由于 IDA\*改成了深度优先的方式，所以 IDA\*更实用：

1. 不需要判重，不需要排序；
2. 空间需求减少。

### 伪代码

```pseudocode
Procedure IDA_STAR(StartState)
Begin
  PathLimit := H(StartState) - 1;
  Succes := False;
  Repeat
    inc(PathLimit);
    StartState.g = 0;
    Push(OpenStack, StartState);
    Repeat
      CurrentState := Pop(OpenStack);
      If Solution(CurrentState) then
        Success = True
      Elseif PathLimit >= CurrentState.g + H(CurrentState) then
        For each Child(CurrentState) do
          Push(OpenStack, Child(CurrentState));
    until Success or empty(OpenStack);
  until Success or ResourceLimtsReached;
end;
```

### 优点

1. 空间开销小，每个深度下实际上是一个深度优先搜索，不过深度有限制，而 DFS 的空间消耗小是众所周知的；
2. 利于深度剪枝。

### 缺点

重复搜索：回溯过程中每次 depth 变大都要再次从头搜索。

> 其实，前一次搜索跟后一次相差是微不足道的。

## 例题

[LeetCode 752](https://leetcode-cn.com/problems/open-the-lock/)

### 示例代码

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

class Solution {

    public static final String BEGIN = "0000";
    private String cur;
    private String target;
    private Map<String, Integer> map = new HashMap<>();
    private Set<String> visited = new HashSet<>();

    public int openLock(String[] deadends, String target) {
        visited.addAll(Arrays.asList(deadends));
        if (visited.contains(BEGIN)) {
            return -1;
        }
        if (BEGIN.equals(target)) {
            return 0;
        }
        this.target = target;

        // 当前迭代最大层数
        int depth = 0;
        // 最大层数
        int max = getMax();
        cur = BEGIN;
        map.put(cur, 0);

        while (depth <= max && !dfs(0, depth)) {
            map.clear();
            cur = BEGIN;
            map.put(cur, 0);
            depth++;
        }
        // 超过最大层数则返回找不到
        return depth > max ? -1 : depth;
    }

    int getMax() {
        int ans = 0;
        for (int i = 0; i < 4; i++) {
            int origin = BEGIN.charAt(i) - '0', next = target.charAt(i) - '0';
            int a = Math.min(origin, next), b = Math.max(origin, next);
            // 注意这里是最多需要多少步
            int max = Math.max(b - a, a + 10 - b);
            ans += max;
        }
        return ans;
    }

    private int h(String cur, String target) {
        int res = 0;
        for (int i = 0; i < 4; i++) {
            int diff = Math.abs(cur.charAt(i) - target.charAt(i));
            // 正转 反转
            int min = Math.min(diff, 10 - diff);
            res += min;
        }
        return res;
    }

    private boolean dfs(int step, int max) {
        if (step + h(cur, target) > max) {
            return false;
        }
        if (cur.equals(target)) {
            return true;
        }
        String backup = cur;
        char[] chars = cur.toCharArray();

        for (int i = 0; i < 4; i++) {
            for (int j = -1; j <= 1; j += 2) {
                int origin = chars[i] - '0';
                int next = (origin + j) % 10;
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
                if (!map.containsKey(str) || map.get(str) > step + 1) {
                    cur = str;
                    map.put(str, step + 1);
                    if (dfs(step + 1, max)) {
                        return true;
                    }
                    cur = backup;
                }
            }
        }
        return false;
    }

}

```

