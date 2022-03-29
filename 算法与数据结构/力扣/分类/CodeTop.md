# CodeTop

题目来源于CodeTop中互联网大厂面试的高频考题。

## 206. 反转链表

原题链接：[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list)

> 给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。
>
> **提示：**
>
> - 链表中节点的数目范围是 `[0, 5000]`
> - `-5000 <= Node.val <= 5000`

### 1.迭代法

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        // 创建newHead作为新链表的头节点
        ListNode newHead = new ListNode();

        // 临时节点
        ListNode node = head;
        // 开始迭代
        while (node != null) {
            // 当前节点
            ListNode cur = node;
            // 临时节点指向下一个节点
            node = node.next;
	
            // 让当前节点指向newHead的next节点
            cur.next = newHead.next;
            // newHead的next节点指向当前节点
            // 这样就把当前节点拼到了前继节点的前面
            newHead.next = cur;
        }

        // 返回新链表
        return newHead.next;
    }
}
```

### 2.递归法

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        // 先要递归到底
        if (head == null || head.next == null) {
            return head;
        }

        ListNode next = head.next;
        // 递归到底返回的节点就是新链表的头节点
        ListNode node = reverseList(next);

        // 当前节点指向null
        head.next = null;
        // next节点指向当前节点
        next.next = head;

        // 返回新链表的头节点node节点
        return node;
    }
}
```

## 3. 无重复字符的最长子串

原题链接：[3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

> 给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。
>
> **提示：**
>
> - `0 <= s.length <= 5 * 104`
> - `s` 由英文字母、数字、符号和空格组成

### 1.滑动窗口

``` java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int length;
        if (s == null || (length = s.length()) == 0) {
            return 0;
        }

        // left、right作为滑动窗口的左右指针
        int left = 0, right = 0, res = 0;
        // windows保存left和right之间滑动窗口每个字符出现的次数
        Map<Character, Integer> windows = new HashMap<>();
        while (right < length) {
            // 遍历字符串s
            // 每次遍历结束都保证滑动窗口的字符是不重复的
            char c = s.charAt(right++);
            // 记录字符c出现一次
            windows.put(c, windows.getOrDefault(c, 0) + 1);

            // 如果字符出现次数大于1
            // 说明字符c重复了
            while (windows.get(c) > 1) {
                // 这个时候缩小滑动窗口的范围
                char t = s.charAt(left++);
                // t已经不在滑动窗口里面了
                // 所以要减去
                windows.put(t, windows.get(t) - 1);
            }

            // 再重新计算滑动窗口内的长度
            // 和res取最大值更新结果res
            res = Math.max(res, right - left);
        }

        return res;
    }
}
```

## 146. LRU 缓存

原题链接：[146. LRU 缓存](https://leetcode-cn.com/problems/lru-cache/)

> 请你设计并实现一个满足  LRU (最近最少使用) 缓存 约束的数据结构。
> 实现 LRUCache 类：
> LRUCache(int capacity) 以 正整数 作为容量 capacity 初始化 LRU 缓存
> int get(int key) 如果关键字 key 存在于缓存中，则返回关键字的值，否则返回 -1 。
> void put(int key, int value) 如果关键字 key 已经存在，则变更其数据值 value ；如果不存在，则向缓存中插入该组 key-value 。如果插入操作导致关键字数量超过 capacity ，则应该 逐出 最久未使用的关键字。
> 函数 get 和 put 必须以 O(1) 的平均时间复杂度运行。
>
> 提示：
>
> 1 <= capacity <= 3000
> 0 <= key <= 10000
> 0 <= value <= 105
> 最多调用 2 * 105 次 get 和 put

### 1.链表+哈希

``` java
class LRUCache {
    private int capacity;
    // memory维护key和Node的关系
    private Map<Integer, Node> memory;
    // 利用链表维护最近最久未使用的Node
    private LinkedList<Node> linkedList;

    /**
     * Node用来保存key和value的关系
     */
    private class Node {
        int key;
        int value;

        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }

    /**
     * 初始化
     */
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.memory = new HashMap<>();
        this.linkedList = new LinkedList<>();
    }
    
    public int get(int key) {
        // key不存在就返回-1
        if (!memory.containsKey(key)) {
            return -1;
        }

        // key存在获取对应Node
        Node node = memory.get(key);

        // 将Node调整到链表的最前面
        linkedList.remove(node);
        linkedList.addFirst(node);

        return node.value;
    }
    
    public void put(int key, int value) {
        // key已经存在
        if (memory.containsKey(key)) {
            // 获取对应Node
            Node node = memory.get(key);
            // 更新value
            node.value = value;

            // 将Node调整到链表的最前面
            linkedList.remove(node);
            linkedList.addFirst(node);

            return ;
        }

        // 如果已经满了
        if (memory.size() == capacity) {
            // 就删除最后一个Node
            Node node = linkedList.removeLast();
            memory.remove(node.key);
        }

        // 将新的key和value放到链表的最前面
        Node node = new Node(key, value);
        linkedList.addFirst(node);
        memory.put(key, node);
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

## 215. 数组中的第K个最大元素

原题链接：[215. 数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

> 给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。
>
> 请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。
>
> **提示：**
>
> - `1 <= k <= nums.length <= 104`
> - `-104 <= nums[i] <= 104`

### 1.API排序

``` java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        // 先对数组排序
        Arrays.sort(nums);

        // 取倒数第k个数就是第 k 个最大的元素
        return nums[nums.length - k];
    }
}
```

## 25. K 个一组翻转链表

原题链接：[25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

> 给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。
>
> k 是一个正整数，它的值小于或等于链表的长度。
>
> 如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序。
>
> 进阶：
>
> 你可以设计一个只使用常数额外空间的算法来解决此问题吗？
> 你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。
>
> **提示：**
>
> - 列表中节点的数量在范围 `sz` 内
> - `1 <= sz <= 5000`
> - `0 <= Node.val <= 1000`
> - `1 <= k <= sz`

### 1.分割法

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null) {
            return head;
        }

        ListNode node = head;
        int i = 0;
        // 先将链表按k个节点进行分割
        while (node != null && ++i < k) {
            node = node.next;
        }

        // 如果剩余链表长度小于k
        if (i < k) {
            // 就直接返回头节点head
            return head;
        }

        // 分割链表
        ListNode next = node.next;
        node.next = null;

        // 反转当前段的链表
        ListNode newHead = reverseListNode(head);
        // 然后继续递归反转剩余的链表
        head.next = reverseKGroup(next, k);

        return newHead;
    }

    /**
     * 反转链表
     */
    private ListNode reverseListNode(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode next = head.next;
        ListNode node = reverseListNode(next);

        head.next = null;
        next.next = head;

        return node;
    }
}
```

## 912. 排序数组

原题链接：[912. 排序数组](https://leetcode-cn.com/problems/sort-an-array/)

> 给你一个整数数组 `nums`，请你将该数组升序排列。
>
> **提示：**
>
> - `1 <= nums.length <= 5 * 104`
> - `-5 * 104 <= nums[i] <= 5 * 104`

### 1.双路快排

``` java
class Solution {
    public int[] sortArray(int[] nums) {
        int length = nums.length;

        helper(nums, 0, length - 1);

        return nums;
    }

    /**
     * 双路快排
     */
    private void helper(int[] nums, int l, int r) {
        if (l >= r) {
            return ;
        }

        int e = nums[l];
        int lt = l, gt = r + 1, i = l + 1;
        while (i < gt) {
            if (nums[i] < e) {
                swap(nums, ++lt, i++);
            } else if (nums[i] > e) {
                swap(nums, --gt, i);
            } else {
                i++;
            }
        }

        swap(nums, l, lt);

        helper(nums, l, lt - 1);
        helper(nums, gt, r);
    }

    /**
     * 交换数组中两个位置的值
     */
    private void swap(int[] nums, int i, int j) {
        if (i == j) {
            return ;
        }

        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

## 15. 三数之和

原题链接：[15. 三数之和](https://leetcode-cn.com/problems/3sum/)

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。
>
> 注意：答案中不可以包含重复的三元组。
>
> **提示：**
>
> - `0 <= nums.length <= 3000`
> - `-105 <= nums[i] <= 105`

### 1.排序+左右指针

``` java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        int length;
        if ((length = nums.length) == 0) {
            return res;
        }

        // 先对数组nums进行排序
        Arrays.sort(nums);

        // 从第一个元素开始
        // 将当前元素和后面元素一头一尾相加
        for (int i = 0; i < length - 2; i++) {
            // 跳过重复的元素
            if (i > 0 && nums[i - 1] == nums[i]) {
                continue ;
            }

            // l和r之间形成一个区间
            int l = i + 1, r = length - 1;
            while (l < r) {
                int num = nums[i] + nums[l] + nums[r];
                if (num == 0) {
                    // 如果结果等于0
                    // 则记录结果
                    List<Integer> list = new ArrayList<>();
                    list.add(nums[i]);
                    list.add(nums[l]);
                    list.add(nums[r]);

                    // 并将结果加入res
                    res.add(list);

                    l++;
                    r--;

                    // 从左边跳过重复的元素
                    while (l < r && nums[l - 1] == nums[l]) {
                        l++;
                    }

                    // 从右边跳过重复的元素
                    while (l < r && nums[r] == nums[r + 1]) {
                        r--;
                    }
                } else if (num < 0) {
                    // num小于0说明左边太小
                    // l向右移动
                    l++;
                } else {
                    // num大于0说明左边太大
                    // r向左移动
                    r--;
                }
            }
        }

        return res;
    }
}
```

## 53. 最大子数组和

原题链接：[53. 最大子数组和](https://leetcode-cn.com/problems/maximum-subarray/)

> 给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> **子数组** 是数组中的一个连续部分。
>
> **提示：**
>
> - `1 <= nums.length <= 105`
> - `-104 <= nums[i] <= 104`

### 1.动态规划

``` java
class Solution {
    public int maxSubArray(int[] nums) {
        // max记录最大值
        // pre记录连续子数组的和
        int max = nums[0], pre = 0;
        for (int num : nums) {
            // 更新连续子数组的和
            pre = Math.max(pre + num, num);
            // 再更新max
            max = Math.max(pre, max);
        }

        return max;
    }
}
```

## 1. 两数之和

原题链接：[1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

> 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
>
> 你可以按任意顺序返回答案。
>
> 提示：
>
> 2 <= nums.length <= 104
> -109 <= nums[i] <= 109
> -109 <= target <= 109
> 只会存在一个有效答案

### 1.哈希

``` java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int length = nums.length;

        // 利用HashMap保存元素与下标的关系
        Map<Integer, Integer> memory = new HashMap<>();
        for (int i = 0; i < length; i++) {
            int num = nums[i];
            int x = target - num;
            if (memory.containsKey(x)) {
                // 如果x存在于memory中
                // 直接返回i和x对应的数组下标
                return new int[] {i, memory.get(x)};
            }

            // 保存元素和下标的关系
            memory.put(num, i);
        }

        return null;
    }
}
```

## 21. 合并两个有序链表

原题链接：[21. 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

> 将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 
>
> **提示：**
>
> - 两个链表的节点数目范围是 `[0, 50]`
> - `-100 <= Node.val <= 100`
> - `l1` 和 `l2` 均按 **非递减顺序** 排列

### 1.迭代法

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null && list2 == null) {
            return null;
        }

        // 定义一个head节点作为合并后链表到头节点
        ListNode head = new ListNode();
        // node作为活动节点，用来合并节点
        ListNode node = head;
        while (list1 != null || list2 != null) {
            if (list1 == null) {
                // 如果list1为空，那么直接将list2合入node.next
                node.next = list2;
                // 然后直接break
                break ;
            } else if (list2 == null) {
                // 如果list2为空，那么直接将list1合入node.next
                node.next = list1;
                // 然后直接break
                break ;
            } else if (list1.val < list2.val) {
                // list1.val更小，就将list1合入node.next
                node.next = list1;
                // list1跳到下一个节点
                list1 = list1.next;
            } else {
                // list2.val更小，就将list2合入node.next
                node.next = list2;
                // list2跳到下一个节点
                list2 = list2.next;
            }

            // node跳到下一个节点
            node = node.next;
        }

        // 返回头节点到next即新链表到头节点
        return head.next;
    }
}
```

## 141. 环形链表

原题链接：[141. 环形链表](https://leetcode-cn.com/problems/linked-list-cycle/)

> 给你一个链表的头节点 head ，判断链表中是否有环。
>
> 如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。注意：pos 不作为参数进行传递 。仅仅是为了标识链表的实际情况。
>
> 如果链表中存在环 ，则返回 true 。 否则，返回 false 。
>
> **提示：**
>
> - 链表中节点的数目范围是 `[0, 104]`
> - `-105 <= Node.val <= 105`
> - `pos` 为 `-1` 或者链表中的一个 **有效索引** 。

### 1.快慢指针

``` java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }

        // 利用快慢指针来判断链表中是否有环
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            // 慢指针跳一个节点
            slow = slow.next;
            // 快指针跳两个节点
            fast = fast.next.next;

            // 如果有环，快慢指针最终会相遇
            if (slow == fast) {
                // 直接返回true
                return true;
            }
        }
        
        // 否则返回false
        return false;
    }
}
```

## 102. 二叉树的层序遍历

原题链接：[102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

> 给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。
>
> **提示：**
>
> - 树中节点数目在范围 `[0, 2000]` 内
> - `-1000 <= Node.val <= 1000`

### 1.队列

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        // 利用队列实现层序遍历
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.addFirst(root);

        while (!queue.isEmpty()) {
            // 因为每一层都需要放在一个集合中
            // 所以这里先把同一层的节点放到list中
            List<TreeNode> list = new ArrayList<>();
            // 遍历队列
            while (!queue.isEmpty()) {
                // 将队列中的元素放入list中
                // 这里就是将同一层到节点放入到list中
                list.add(queue.removeLast());
            }

            List<Integer> nums = new ArrayList<>();
            // 然后再遍历这一层的节点
            for (TreeNode node : list) {
                // 当前节点值放入nums集合中
                nums.add(node.val);

                // 如果当前节点左节点不为空
                if (node.left != null) {
                    // 就将左节点放入队列中
                    queue.addFirst(node.left);
                }

                // 如果当前节点右节点不为空
                if (node.right != null) {
                    // 就将右节点放入队列中
                    queue.addFirst(node.right);
                }
            }

            // 将当前层的结果放入res中
            res.add(nums);
        }

        // 返回结果
        return res;
    }
}
```

## 121. 买卖股票的最佳时机

原题链接：[121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

> 给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。
>
> 你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。
>
> 返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。
>
> **提示：**
>
> - `1 <= prices.length <= 105`
> - `0 <= prices[i] <= 104`

### 1.贪心算法

``` java
class Solution {
    public int maxProfit(int[] prices) {
        // 贪心算法，min为数组中最小的元素
        int length = prices.length, res = 0, min = prices[0];

        // 通过遍历数组找到最小的元素
        // 然后和后面的元素相减得出的差中取最大值
        for (int i = 1; i < length; i++) {
            // 更新最小值
            if (min > prices[i]) {
                min = prices[i];
            }

            // 和后面的元素相减并与res取最大值
            res = Math.max(res, prices[i] - min);
        }

        return res;
    }
}
```

## 103. 二叉树的锯齿形层序遍历

原题链接：[103. 二叉树的锯齿形层序遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

> 给你二叉树的根节点 `root` ，返回其节点值的 **锯齿形层序遍历** 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。
>
> **提示：**
>
> - 树中节点数目在范围 `[0, 2000]` 内
> - `-100 <= Node.val <= 100`

### 1.栈

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        // 利用栈实现层序遍历
        LinkedList<TreeNode> stack = new LinkedList<>();
        stack.push(root);

        // true：表示从左往右
        // false：表示从右往左
        boolean flag = true;
        while (!stack.isEmpty()) {
            // 因为每一层都需要放在一个集合中
            // 所以这里先把同一层的节点放到list中
            List<TreeNode> list = new ArrayList<>();
            // 遍历队列
            while (!stack.isEmpty()) {
                // 将栈中的元素放入list中
                // 这里就是将同一层到节点放入到list中
                list.add(stack.pop());
            }

            List<Integer> nums = new ArrayList<>();
            // 然后再遍历这一层的节点
            for (TreeNode node : list) {
                // 当前节点值放入nums集合中
                nums.add(node.val);

                if (flag) { // 如果当前是从左往右
                    // 则先将left入栈
                    if (node.left != null) {
                        stack.push(node.left);
                    }
                    // 再将right入栈
                    if (node.right != null) {
                        stack.push(node.right);
                    }
                } else { // 如果当前是从右往左
                    // 则先将right入栈
                    if (node.right != null) {
                        stack.push(node.right);
                    }
                    // 再将left入栈
                    if (node.left != null) {
                        stack.push(node.left);
                    }
                }
            }

            // 变换方向
            flag = !flag;
            // 将当前层的结果放入res中
            res.add(nums);
        }

        return res;
    }
}
```

## 160. 相交链表

原题链接：[160. 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

> 给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 null 。
>
> 图示两个链表在节点 c1 开始相交：
>
> ![img](../../../img/160_statement.png)
>
> 题目数据 保证 整个链式结构中不存在环。
>
> 注意，函数返回结果后，链表必须 保持其原始结构 。
>
> 自定义评测：
>
> 评测系统 的输入如下（你设计的程序 不适用 此输入）：
>
> intersectVal - 相交的起始节点的值。如果不存在相交节点，这一值为 0
> listA - 第一个链表
> listB - 第二个链表
> skipA - 在 listA 中（从头节点开始）跳到交叉节点的节点数
> skipB - 在 listB 中（从头节点开始）跳到交叉节点的节点数
> 评测系统将根据这些输入创建链式数据结构，并将两个头节点 headA 和 headB 传递给你的程序。如果程序能够正确返回相交节点，那么你的解决方案将被 视作正确答案 。
>
> 提示：
>
> listA 中节点数目为 m
> listB 中节点数目为 n
> 1 <= m, n <= 3 * 104
> 1 <= Node.val <= 105
> 0 <= skipA <= m
> 0 <= skipB <= n
> 如果 listA 和 listB 没有交点，intersectVal 为 0
> 如果 listA 和 listB 有交点，intersectVal == listA[skipA] == listB[skipB]
>

### 1.集合

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        // 如果headA或者headB其中一个为null
        // 那么它们必然不会相交，所以直接返回null
        if (headA == null || headB == null) {
            return null;
        }

        // 利用Set集合保存headA链表的所有节点
        Set<ListNode> set = new HashSet<>();
        ListNode node = headA;
        // 遍历headA将节点放入set集合中
        while (node != null) {
            set.add(node);

            node = node.next;
        }

        node = headB;
        // 再遍历headB
        // 并判断headB中是否有和set中相同的节点
        while (node != null) {
            // 如果有相同的节点
            if (set.contains(node)) {
                // 那么第一个相同的节点就是相交节点
                // 直接返回
                return node;
            }

            // 跳到下一个节点
            node = node.next;
        }

        return null;
    }
}
```

## 20. 有效的括号

原题链接：[20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

> 给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。
>
> 有效字符串需满足：
>
> 左括号必须用相同类型的右括号闭合。
> 左括号必须以正确的顺序闭合。
>
> **提示：**
>
> - `1 <= s.length <= 104`
> - `s` 仅由括号 `'()[]{}'` 组成

### 1.栈

``` java
class Solution {
    public boolean isValid(String s) {
        int length = s.length();
        if (length % 2 == 1) {
            return false;
        }

        // 利用栈来完成括号匹配
        LinkedList<Character> stack = new LinkedList<>();
        // 遍历字符串的每一个字符
        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);
            if (c == '(' || c == '{' || c == '[') {
                // 如果当前字符是左括号
                // 则则直接入栈
                stack.push(c);

                continue ;
            }

            // 如果遇到右括号，但是栈是空
            // 说明当前字符前面没有与之匹配的左括号
            if (stack.isEmpty()) {
                // 直接返回false
                return false;
            }

            // 否则弹出栈顶元素
            char l = stack.pop();
            if (l == '(' && c != ')' || l == '{' && c != '}' || l == '[' && c != ']') {
                // 如果栈顶元素不能和当前字符匹配成功
                // 则直接返回false
                return false;
            }
        }

        // 栈不为空表示还有括号没有匹配成功
        if (!stack.isEmpty()) {
            // 这里直接返回false
            return false;
        }

        // 到了这里表示所有括号都匹配成功，返回true
        return true;
    }
}
```

## 236. 二叉树的最近公共祖先

原题链接：[236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

> 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
>
> 百度百科中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”
>
> 提示：
>
> 树中节点数目在范围 [2, 105] 内。
> -109 <= Node.val <= 109
> 所有 Node.val 互不相同 。
> p != q
> p 和 q 均存在于给定的二叉树中。
>

### 1.先序遍历

``` java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        // 先序遍历
        if (root == null) {
            return null;
        }

        // 只要遇到了p或者q
        if (root == p || root == q) {
            // 则直接返回一个不为空的节点
            // 这里返回root节点
            return root;
        }

        // 然后先遍历左节点
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        // 再遍历右节点
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        // 如果遍历左右节点返回的结果都不为空
        // 则表示当前root节点就是p和q的最近公共祖先
        if (left != null && right != null) {
            // 直接返回root
            return root;
        }

        // 否则返回遍历左右节点不为空的结果
        return left != null ? left : right;
    }
}
```

## 88. 合并两个有序数组

原题链接：[88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

> 给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。
>
> 请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。
>
> 注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。
>
> 提示：
>
> nums1.length == m + n
> nums2.length == n
> 0 <= m, n <= 200
> 1 <= m + n <= 200
> -109 <= nums1[i], nums2[j] <= 109
>

### 1.迭代法

``` java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        // 先用临时数组temp将nums1数组中的元素保存起来
        int[] temp = new int[m];
        for (int i = 0; i < m; i++) {
            temp[i] = nums1[i];
        }

        int i = 0, j = 0, k = 0;
        // 合并数组temp和nums2
        while (i < m || j < n) {
            if (i == m) {
                // 如果temp遍历完了，那么直接将nums2[j]合入nums1
                nums1[k] = nums2[j++];
            } else if (j == n) {
                // 如果nums2遍历完了，那么直接将temp[i]合入nums1
                nums1[k] = temp[i++];
            } else if (temp[i] < nums2[j]) {
                // 如果temp[i]更小就将temp[i]合入nums1
                nums1[k] = temp[i++];
            } else {
                // 否则nums2[j]更小就将nums2[j]合入nums1
                nums1[k] = nums2[j++];
            }

            k++;
        }
    }
}
```

## 33. 搜索旋转排序数组

原题链接：[33. 搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

> 整数数组 nums 按升序排列，数组中的值 互不相同 。
>
> 在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为 [4,5,6,7,0,1,2] 。
>
> 给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回 -1 。
>
> 提示：
>
> 1 <= nums.length <= 5000
> -10^4 <= nums[i] <= 10^4
> nums 中的每个值都 独一无二
> 题目数据保证 nums 在预先未知的某个下标上进行了旋转
> -10^4 <= target <= 10^4
>

### 1.二分法变种

``` java
class Solution {
    public int search(int[] nums, int target) {
        int length = nums.length, left = 0, right = length - 1;

        // 二分法变种
        while (left <= right) {
            int mid = (left + right) >> 1;
            // 如果nums[mid] == target
            // 找到到了target
            if (nums[mid] == target) {
                // 直接返回mid
                return mid;
            }

            // 如果nums[left] <= nums[mid]
            if (nums[left] <= nums[mid]) {
                // 表示区间[left, mid]是递增的
                if (nums[left] <= target && target < nums[mid]) {
                    // 表示targer在区间[left, mid]内
                    // 所以收缩right，mid - 1
                    right = mid - 1;
                } else {
                    // 否则表示targer不在区间[left, mid]内
                    // 收缩left，mid + 1
                    left = mid + 1;
                }
            } else {// 否则表示区间[mid, right]是递增的
                if (nums[mid] < target && target <= nums[right]) {
                    // 表示target在区间[mid, right]内
                    // 所以收缩left，mid + 1
                    left = mid + 1;
                } else {
                    // 否则表示targer不在区间[mid, right]内
                    // 收缩right，mid - 1
                    right = mid - 1;
                }
            }
        }

        // 到了这里表示没有找到target
        // 返回-1
        return -1;
    }
}
```

## 5. 最长回文子串

原题链接：[5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。
>
> **提示：**
>
> - `1 <= s.length <= 1000`
> - `s` 仅由数字和英文字母组成

### 1.左右指针

``` java
class Solution {
    public String longestPalindrome(String s) {
        int length = s.length(), st = 0, end = 0;

        // 遍历字符串找出最长的回文子串
        for (int i = 0; i < length; i++) {
            // 先以位置i为中心向两边统计回文子串的长度
            int s1 = helper(s, i, i);
            // 再以位置i和下一个位置i + 1为中心向两边统计回文子串的长度
            int s2 = helper(s, i, i + 1);
            
            // 取两者中的最大值
            int max = Math.max(s1, s2);
            // 如果最大值大于现有的回文子串长度
            if (max > end - st + 1) {
                // 则更新st和end
                // 如果max为奇数，则i为中心位置
                // 如果max为偶数，则i为中心位置中左边的位置
                // 所以计算st时要减去(max - 1) / 2
                st = i - (max - 1) / 2;
                // 计算end时直接加上max的一半即max / 2
                end = i + max / 2;
            }
        }

        return s.substring(st, end + 1);
    }

    /**
     * 统计字符串在left和right位置两边回文子串的长度
     */
    private int helper(String s, int left, int right) {
        int length = s.length();
        while (left >= 0 && right < length && s.charAt(left) == s.charAt(right)) {
            left--;
            right++;
        }

        return right - left - 1;
    }
}
```

## 200. 岛屿数量

原题链接：[200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

> 给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。
>
> 提示：
>
> m == grid.length
> n == grid[i].length
> 1 <= m, n <= 300
> grid\[i][j] 的值为 '0' 或 '1'

### 1.深度优先搜索

``` java
class Solution {
    public int numIslands(char[][] grid) {
        int m = grid.length, n = grid[0].length, sum = 0;
        
        // visited用来标记访问过的陆地
        boolean[][] visited = new boolean[m][n];
        // 遍历二维数组
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (visited[i][j]) {
                    continue ;
                }

                // 当遇到陆地时
                if (grid[i][j] == '1') {
                    // 陆地加1
                    sum++;
                    // 再进行深度优先搜索标记相连的陆地
                    helper(grid, visited, i, j);
                }
            }
        }

        return sum;
    }

    /**
     * 深度优先搜索标记相连的陆地
     */
    private void helper(char[][] grid, boolean[][] visited, int i, int j) {
        int m = grid.length, n = grid[0].length;
        // 遇到边界或者访问过的陆地或者非陆地时
        // 直接返回
        if (i < 0 || i >= m || j < 0 || j >= n || visited[i][j] || grid[i][j] == '0') {
            return ;
        }

        visited[i][j] = true;

        // 上右下左访问旁边的陆地
        helper(grid, visited, i - 1, j);
        helper(grid, visited, i, j + 1);
        helper(grid, visited, i + 1, j);
        helper(grid, visited, i, j - 1);
    }
}
```

## 46. 全排列

原题链接：[46. 全排列](https://leetcode-cn.com/problems/permutations/)

> 给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。
>
> **提示：**
>
> - `1 <= nums.length <= 6`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有整数 **互不相同**

### 1.递归

``` java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        int length = nums.length;

        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();

        
        helper(nums, 0, res, list);

        return res;
    }

    /**
     * 递归全排列
     */
    private void helper(int[] nums,
                        int pos,
                        List<List<Integer>> res,
                        List<Integer> list) {
        int length = nums.length;
        // 如果已经递归到了最后一个元素
        if (length == pos) {
            // 那就将结果放入res
            res.add(new ArrayList<>(list));

            return ;
        }

        // 遍历数组
        for (int i = 0; i < length; i++) {
            // 如果当前元素已经在list当中
            if (list.contains(nums[i])) {
                // 那就直接跳过
                continue ;
            }

            // 将当前元素加入到list中
            list.add(nums[i]);
            // 递归，pos加1
            helper(nums, pos + 1, res, list);
            // 将当前元素移出list
            list.remove(list.size() - 1);
        }
    }
}
```

## 415. 字符串相加

原题链接：[415. 字符串相加](https://leetcode-cn.com/problems/add-strings/)

> 给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和并同样以字符串形式返回。
>
> 你不能使用任何內建的用于处理大整数的库（比如 BigInteger）， 也不能直接将输入的字符串转换为整数形式。
>
> 提示：
>
> 1 <= num1.length, num2.length <= 104
> num1 和num2 都只包含数字 0-9
> num1 和num2 都不包含任何前导零
>

### 1.倒序迭代

``` java
class Solution {
    public String addStrings(String num1, String num2) {
        int len1 = num1.length(), len2 = num2.length();

        // sum保存临时的和以及上次相加结果的进位
        int sum = 0, i = len1 - 1, j = len2 - 1;
        StringBuilder res = new StringBuilder();
        // 倒叙遍历两个字符串
        while (i >= 0 || j >= 0) {
            int a = 0;
            if (i >= 0) {
                // 将字符转为数字
                a = num1.charAt(i--) - '0';
            }

            int b = 0;
            if (j >= 0) {
                // 将字符转为数字
                b = num2.charAt(j--) - '0';
            }

            // 相加
            sum += a + b;
            // 对10取余放到结果中
            res.append(sum % 10);
            // 除以10获取进位
            sum /= 10;
        }

        // 如果进位不为0
        if (sum > 0) {
            // 则将进位放入结果中
            res.append(sum);
        }

        // 反转结果返回
        return res.reverse().toString();
    }
}
```

## 92. 反转链表 II

原题链接：[92. 反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

> 给你单链表的头指针 head 和两个整数 left 和 right ，其中 left <= right 。请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。
>
> **提示：**
>
> - 链表中节点数目为 `n`
> - `1 <= n <= 500`
> - `-500 <= Node.val <= 500`
> - `1 <= left <= right <= n`

### 1.迭代法

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        // 创建h节点作为反转链表的头节点
        ListNode h = new ListNode();
        // pre作为反转链表的前继节点
        // last作为反转后链表的最后一个节点
        ListNode node = head, pre = null, last = null;
        // pos记录遍历到了第几个节点
        int pos = 0;

        // 遍历到链表的第right个节点
        while (node != null && pos++ < right) {
            // 如果遍历到了第left个节点及以后的节点
            if (left <= pos) {
                // left == pos时表示便利到了反转链表的第一个节点
                // 即反转后链表的最后一个节点
                if (left == pos) {
                    // 用last记录反转后链表的最后一个节点
                    last = node;
                }

                // 临时存放当前节点
                ListNode cur = node;
                // node跳到下一个节点
                node = node.next;

                // 实现反转
                // 当前节点的next指向头节点的下一个节点
                cur.next = h.next;
                // 头节点的next指向当前节点
                h.next = cur;
            } else {
                // 在遍历到第left个节点前
                // 一直更新pre
                pre = node;
                // node跳到下一个节点
                node = node.next;
            }
        }

        // 如果pre不为空表示left > 1
        if (pre != null) {
            // 这里需要把反转链表前面的节点连接上反转后的链表
            pre.next = h.next;
        }

        if (last != null) {
            // 将反转后的链表连接上反转链表后面的节点
            last.next = node;
        }

        // 如果left == 1直接返回h.next
        // 否则返回head
        return left == 1 ? h.next : head;
    }
}
```

## 23. 合并K个升序链表

原题链接：[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

> 给你一个链表数组，每个链表都已经按升序排列。
>
> 请你将所有链表合并到一个升序链表中，返回合并后的链表。
>
> 提示：
>
> k == lists.length
> 0 <= k <= 10^4
> 0 <= lists[i].length <= 500
> -10^4 <= lists[i][j] <= 10^4
> lists[i] 按 升序 排列
> lists[i].length 的总和不超过 10^4
>

### 1.归并

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) {
            return null;
        }

        // 利用栈实现归并合并链表
        LinkedList<ListNode> stack = new LinkedList<>();
        // 先遍历lists将所有链表放入stack中
        for (ListNode list : lists) {
            stack.push(list);
        }

        // 然后将栈中的每两个链表弹出进行合并
        // 合并结果再放入栈中
        // 当栈中只剩下一个链表时停止合并
        while (stack.size() > 1) {
            stack.push(merge(stack.pop(), stack.pop()));
        }

        // 栈中剩余的最后一个链表就是合并后的结果
        return stack.pop();
    }

    /**
     * 合并两个升序链表
     */
    private ListNode merge(ListNode node1, ListNode node2) {
        if (node1 == null) {
            return node2;
        }

        if (node2 == null) {
            return node1;
        }

        ListNode head = new ListNode();
        ListNode node = head;

        while (node1 != null || node2 != null) {
            if (node1 == null) {
                node.next = node2;

                break ;
            } else if (node2 == null) {
                node.next = node1;

                break ;
            } else if (node1.val < node2.val) {
                node.next = node1;

                node1 = node1.next;
            } else {
                node.next = node2;

                node2 = node2.next;
            }

            node = node.next;
        }

        return head.next;
    }
}
```

## 142. 环形链表 II

原题链接：[142. 环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

> 给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。
>
> 如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。
>
> 不允许修改 链表。
>
> **提示：**
>
> - 链表中节点的数目范围在范围 `[0, 104]` 内
> - `-105 <= Node.val <= 105`
> - `pos` 的值为 `-1` 或者链表中的一个有效索引

### 1.快慢指针

``` java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode detectCycle(ListNode head) {
        if (head == null || head.next == null) {
            return null;
        }

        // 快慢指针
        // 快指针每次跳一个节点
        // 慢指针每次跳两个节点
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            // 如果链表有环，则快慢指针相遇，即slow == fast
            if (slow == fast) {
                // 这时直接跳出循环
                break ;
            }
        }

        // 如果快慢指针不相等，则链表无环
        if (slow != fast) {
            // 直接返回null
            return null;
        }

        // 既然判定链表有环
        // 那让慢指针从头节点开始
        // 和快指针以相同的速度前进
        // 当两个快慢指针再次相遇时的节点
        // 就是环的起点
        slow = head;
        while (slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }

        // 返回快慢指针都是环的起点
        return slow;
    }
}
```



