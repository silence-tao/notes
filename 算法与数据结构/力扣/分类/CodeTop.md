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

## 54. 螺旋矩阵

原题链接：[54. 螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix/)

> 给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。
>
> 提示：
>
> m == matrix.length
> n == matrix[i].length
> 1 <= m, n <= 10
> -100 <= matrix[i][j] <= 100
>

### 1.迭代

``` java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;

        // 上下左右边界
        int u = 0, d = m - 1, l = 0, r = n - 1;
        // move代表左下右上移动 0->1->2->3
        int move = 0;
        // step代表移动了多少步
        int step = m * n, i = 0, j = 0;

        List<Integer> res = new ArrayList<>();
        while (step-- > 0) {
            res.add(matrix[i][j]);
            if (move == 0) { // 相右移动
                if (j == r) {
                    // 转到下一层
                    i++;
                    // 移动方向转为向下
                    move = 1;
                    // 上边界缩小1
                    u++;

                    continue ;
                }

                // 向右移动一格
                j++;
            } else if (move == 1) {
                if (i == d) {
                    j--;
                    move = 2;
                    r--;

                    continue ;
                }

                i++;
            } else if (move == 2) {
                if (j == l) {
                    i--;
                    move = 3;
                    d--;

                    continue ;
                }

                j--;
            } else {
                if (i == u) {
                    j++;
                    move = 0;
                    l++;

                    continue ;
                }

                i--;
            }
        }

        return res;
    }
}
```

## 300. 最长递增子序列

原题链接：[300. 最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

> 给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
>
> 子序列 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。
>
> **提示：**
>
> - `1 <= nums.length <= 2500`
> - `-104 <= nums[i] <= 104`

### 1.动态规划

``` java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int length, max = 1;
        if ((length = nums.length) <= 1) {
            return length;
        }

        // 利用dp数组来保存每个元素前面面最长严格递增子序列的长度
        int[] dp = new int[length];
        for (int i = 0; i < length; i++) {
            // 初始长度为1
            dp[i] = 1;
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    // 更新dp[i]的值
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }

            // 更新返回结果
            max = Math.max(max, dp[i]);
        }

        return max;
    }
}
```

## 704. 二分查找

原题链接：[704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

> 给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。
>
> **提示：**
>
> 1. 你可以假设 `nums` 中的所有元素是不重复的。
> 2. `n` 将在 `[1, 10000]`之间。
> 3. `nums` 的每个元素都将在 `[-9999, 9999]`之间。

### 1.二分法

``` java
class Solution {
    public int search(int[] nums, int target) {
        int length = nums.length;

        // 将区间[left, right]分为两段
        // 其中左边为[left, mid]
        // 右边为[mid + 1, right]
        int left = 0, right = length - 1;
        while (left < right) {
            int mid = (left + right) >>> 1;
            if (nums[mid] < target) {
                // target大于中间值
                // 则缩小左边的范围
                left = mid + 1;
            } else {
                // 否则缩小右边的范围
                right = mid;
            }
        }

        // 最后退出while循环时，left会等于right
        // 此时如果nums[left] == target则返回left
        // 否则返回-1
        return nums[left] == target ? left : -1;
    }
}
```

## 42. 接雨水

原题链接：[42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

> 给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。
>
> **提示：**
>
> - `n == height.length`
> - `1 <= n <= 2 * 104`
> - `0 <= height[i] <= 105`

### 1.双指针

``` java
class Solution {
    public int trap(int[] height) {
        int length;
        if ((length = height.length) == 0) {
            return 0;
        }

        // 每个柱子能接多少雨水等于当前柱子两边最大高度的较小值减去当前高度的值
        int sum = 0, left, right;
        for (int i = 1; i < length; i++) {
            left = right = height[i];

            // 找到左边最大最大高度的柱子
            for (int j = i - 1; j >= 0; j--) {
                left = Math.max(left, height[j]);
            }

            // 找到右边最大最大高度的柱子
            for (int j = i + 1; j < length; j++) {
                right = Math.max(right, height[j]);
            }

            // 当前柱子两边最大高度的较小值减去当前高度的值
            // 就是当前柱子能接的雨水
            sum += Math.min(left, right) - height[i];
        }

        return sum;
    }
}
```

## 143. 重排链表

原题链接：[143. 重排链表](https://leetcode-cn.com/problems/reorder-list/)

> 给定一个单链表 L 的头节点 head ，单链表 L 表示为：
>
> ```
> L0 → L1 → … → Ln - 1 → Ln
> ```
>
>
> 请将其重新排列后变为：
>
> ```
> L0 → Ln → L1 → Ln - 1 → L2 → Ln - 2 → …
> ```
>
> 不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。
>
> **提示：**
>
> - 链表的长度范围为 `[1, 5 * 104]`
> - `1 <= node.val <= 1000`

### 1.快慢指针+栈

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
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) {
            return ;
        }

        // 利用快慢指针将链表一分为二
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // slow就是链表点中点或者中点左边的节点
        // right就是右半部分链表
        ListNode right = slow.next;
        // 将左右两部分链表断开
        slow.next = null;

        // 利用栈保存右半部分链表的所有节点
        LinkedList<ListNode> stack = new LinkedList<>();
        // 遍历右半部分链表并将每个节点压入栈中
        while (right != null) {
            ListNode cur = right;
            right = right.next;

            // 断开后面的链接
            cur.next = null;
            // 压入栈中
            stack.push(cur);
        }

        ListNode node = head;
        while (!stack.isEmpty()) {
            // 当前节点原next节点
            ListNode next = node.next;
            // 将栈顶节点弹出接入node的next中
            node.next = stack.pop();

            // 跳到栈弹出的栈顶节点
            node = node.next;
            // 将前节点原next节点接入到栈顶节点next节点
            node.next = next;
            // 然后再跳到下一个节点
            node = node.next;
        }
    }
}
```

## 94. 二叉树的中序遍历

原题链接：[94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

> 给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。
>
> **提示：**
>
> - 树中节点数目在范围 `[0, 100]` 内
> - `-100 <= Node.val <= 100`

### 1.中序遍历

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
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> list = new ArrayList<>();

        // 中序遍历
        inorderTraversal(root, list);

        return list;
    }

    /**
     * 中序遍历
     */
    private void inorderTraversal(TreeNode root, List<Integer> list) {
        if (root == null) {
            return ;
        }

        // 先遍历左节点
        inorderTraversal(root.left, list);

        // 再遍历当前节点
        list.add(root.val);

        // 最后遍历右节点
        inorderTraversal(root.right, list);
    }
}
```

## 124. 二叉树中的最大路径和

原题链接：[124. 二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

> 路径 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点。
>
> 路径和 是路径中各节点值的总和。
>
> 给你一个二叉树的根节点 root ，返回其 最大路径和 。
>
> **提示：**
>
> - 树中节点数目范围是 `[1, 3 * 104]`
> - `-1000 <= Node.val <= 1000`

### 1.后序遍历

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

    int res = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        maxPathSumHelper(root);

        return res;
    }

    /**
     * 后序遍历
     */
    private int maxPathSumHelper(TreeNode root) {
        if (root == null) {
            // 空节点的贡献值为0
            return 0;
        }

        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int left = Math.max(maxPathSumHelper(root.left), 0);
        int right = Math.max(maxPathSumHelper(root.right), 0);

        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int nodeMax = left + right + root.val;

        // 更新答案
        res = Math.max(nodeMax, res);

        // 返回节点的最大贡献值
        return Math.max(left, right) + root.val;
    }
}
```

## 232. 用栈实现队列

原题链接：[232. 用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

> 请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（push、pop、peek、empty）：
>
> 实现 MyQueue 类：
>
> void push(int x) 将元素 x 推到队列的末尾
> int pop() 从队列的开头移除并返回元素
> int peek() 返回队列开头的元素
> boolean empty() 如果队列为空，返回 true ；否则，返回 false
> 说明：
>
> 你 只能 使用标准的栈操作 —— 也就是只有 push to top, peek/pop from top, size, 和 is empty 操作是合法的。
> 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。
>
> 提示：
>
> 1 <= x <= 9
> 最多调用 100 次 push、pop、peek 和 empty
> 假设所有操作都是有效的 （例如，一个空的队列不会调用 pop 或者 peek 操作）
>

### 1.栈

``` java
class MyQueue {
    // 用两个栈实现队列
    // 一个栈用于入队
    private LinkedList<Integer> stack1;
    // 一个栈用于出队
    private LinkedList<Integer> stack2;

    public MyQueue() {
        // 初始化队列
        stack1 = new LinkedList<>();    
        stack2 = new LinkedList<>();
    }
    
    public void push(int x) {
        // 入队放入stack1
        stack1.push(x);
    }
    
    public int pop() {
        // 出队用stack2
        // 如果stack2为空
        if (stack2.isEmpty()) {
            // 则将stack1中的所有元素弹出放入stack2中
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        // 然后从stack2弹出
        return stack2.pop();
    }
    
    public int peek() {
        // 出队用stack2
        // 如果stack2为空
        if (stack2.isEmpty()) {
            // 则将stack1中的所有元素弹出放入stack2中
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        // 然后从stack2弹出
        return stack2.peek();
    }
    
    public boolean empty() {
        // 只有两个栈都为空时，队列才为空
        return stack1.isEmpty() && stack2.isEmpty();
    }
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = new MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * boolean param_4 = obj.empty();
 */
```

## 199. 二叉树的右视图

原题链接：[199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

> 给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。
>
> **提示:**
>
> - 二叉树的节点个数的范围是 `[0,100]`
> - `-100 <= Node.val <= 100` 

### 1.层序遍历

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
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        // 利用队列实现层序遍历
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.add(root);

        while (!queue.isEmpty()) {
            // 将同一层的节点都放入list集合中
            List<TreeNode> list = new ArrayList<>();
            while (!queue.isEmpty()) {
                list.add(queue.poll());
            }

            // 然后再遍历这一层的节点
            for (int i = 0; i < list.size(); i++) {
                TreeNode node = list.get(i);
                if (i == list.size() - 1) {
                    // 如果是这一层的最后一个节点
                    // 则将节点值加入res结果集合中
                    res.add(node.val);
                }

                // 将当前节点不为空的左子节点加入队列
                if (node.left != null) {
                    queue.add(node.left);
                }

                // 将当前节点不为空的右子节点加入队列
                if (node.right != null) {
                    queue.add(node.right);
                }
            }
        }

        return res;
    }
}
```

## 70. 爬楼梯

原题链接：[70. 爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

> 假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。
>
> 每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> **提示：**
>
> - `1 <= n <= 45`

### 1.动态规划

``` java
class Solution {
    public int climbStairs(int n) {
        if (n <= 2) {
            return n;
        }

        // f[n] = f[n - 2] + f[n - 1]
        // first = f[n - 2]
        // second = f[n - 1]
        int first = 1, second = 2, res = 0;
        for (int i = 3; i <= n; i++) {
            // 更新res
            res = first + second;
            // 下一轮的fisrt等于现在的second
            first = second;
            // 下一轮的second等于现在的res
            second = res;
        }

        return res;
    }
}
```

## 19. 删除链表的倒数第 N 个结点

原题链接：[19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

> 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
>
> **提示：**
>
> - 链表中结点的数目为 `sz`
> - `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`

### 1.倒数指针

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
  
    /**
     * 如果想得到倒数第n个节点
     * 可以先让node1遍历到第n个节点
     * 然后定义node2从头和node1一起向后遍历
     * 直到node1遍历到最后一个节点时
     * node2就到了倒数第n个节点
     */
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode node = head;
        // 先遍历到第n + 1个节点
        while (n-- > 0 && node != null) {
            node = node.next;
        }

        // 如果第n + 1个节点是null
        // 说明要删除的是倒数第n个节点
        if (node == null) {
            // 直接返回头节点的next节点
            return head.next;
        }
        
        // pre为待删除节点的前一个节点
        // pre从头开始遍历
        ListNode pre = head;
        // node继续向后遍历，直到倒数第1个节点为止
        // 此时pre就是待删除节点的前一个节点
        while (node != null && node.next != null) {
            node = node.next;
            pre = pre.next;
        }

        // 将pre.next指向pre.next.next
        // 删除待删除的节点
        pre.next = pre.next.next;

        // 返回头节点即可
        return head;
    }
}
```

## 69. x 的平方根

原题链接：[69. x 的平方根 ](https://leetcode-cn.com/problems/sqrtx/)

> 给你一个非负整数 x ，计算并返回 x 的 算术平方根 。
>
> 由于返回类型是整数，结果只保留 整数部分 ，小数部分将被 舍去 。
>
> 注意：不允许使用任何内置指数函数和算符，例如 pow(x, 0.5) 或者 x ** 0.5 。
>
> **提示：**
>
> - `0 <= x <= 231 - 1`

### 1.二分法

``` java
class Solution {
    public int mySqrt(int x) {
        if (x <= 1) {
            return x;
        }

        // 二分法，因为x的算术平方根肯定小于等于x / 2
        // 所以二分的区间为[1, x / 2]
        // 将[1, x / 2]分为[1, mid - 1]、[mid, x / 2]两个区间
        int left = 1, right = x / 2;
        while (left < right) {
            // mid取右边界
            int mid = (left + right + 1) >>> 1;
            // 需要找到第一个mid * mid > x的整数
            // 那么mid - 1就是x的算术平方根
            if (mid > x / mid) {
                right = mid - 1;
            } else {
                left = mid;
            }
        }

        // 退出循环时，left == right
        // 返回任一都是x的算术平方根
        return left;
    }
}
```

## 56. 合并区间

原题链接：[56. 合并区间](https://leetcode-cn.com/problems/merge-intervals/)

> 以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间 。
>
> 提示：
>
> 1 <= intervals.length <= 104
> intervals[i].length == 2
> 0 <= starti <= endi <= 104
>

### 1.排序

``` java
class Solution {
    public int[][] merge(int[][] intervals) {
        int length = intervals.length;
        
        // 先对集合intervals按区间左边的值排序
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

        int pos = -1;
        // res保存结果
        int[][] res = new int[length][2];
        // 遍历intervals合并区间
        for (int i = 0; i < length; i++) {
            // pos == -1是初始情况
            // res[pos][1] < intervals[i][0]表示区间不重叠
            if (pos == -1 || res[pos][1] < intervals[i][0]) {
                // 直接将intervals[i]赋给res
                res[++pos] = intervals[i];
            } else {
                // 否则有重叠
                // 那么合并这两个区间
                // 且区间右边的值为res[pos][1],intervals[i][1]中的最大值
                res[pos][1] = Math.max(res[pos][1], intervals[i][1]);
            }
        }
        
        // 将结果复制返回
        return Arrays.copyOf(res, pos + 1);
    }
}
```

## 4. 寻找两个正序数组的中位数

原题链接：[4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

> 给定两个大小分别为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。请你找出并返回这两个正序数组的 中位数 。
>
> 算法的时间复杂度应该为 O(log (m+n)) 。
>
> 提示：
>
> nums1.length == m
> nums2.length == n
> 0 <= m <= 1000
> 0 <= n <= 1000
> 1 <= m + n <= 2000
> -106 <= nums1[i], nums2[i] <= 106
>

### 1.合并有序数组

``` java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int len1 = nums1.length, len2 = nums2.length, len = len1 + len2;
        int[] num = new int[len];

        // 合并两个有序数组
        int i = 0, j = 0, k = 0;
        while (i < len1 || j < len2) {
            if (i >= len1) {
                num[k] = nums2[j++];
            } else if (j >= len2) {
                num[k] = nums1[i++];
            } else if (nums1[i] < nums2[j]) {
                num[k] = nums1[i++];
            } else {
                num[k] = nums2[j++];
            }

            k++;
        }

        // 然后取合并后数组的中位数
        int mid = len / 2;
        if (len % 2 == 0) {
            // 如果合并后数组的长度是偶数
            // 则取中间的两个元素相加再除以2
            return (num[mid - 1] + num[mid]) / 2.0D;
        } else {
            // 否则取中间的元素
            return num[mid];
        }
    }
}
```

## 82. 删除排序链表中的重复元素 II

原题链接：[82. 删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

> 给定一个已排序的链表的头 `head` ， *删除原始链表中所有重复数字的节点，只留下不同的数字* 。返回 *已排序的链表* 。
>
> **提示：**
>
> - 链表中节点数目在范围 `[0, 300]` 内
> - `-100 <= Node.val <= 100`
> - 题目数据保证链表已经按升序 **排列**

### 1.迭代

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
    public ListNode deleteDuplicates(ListNode head) {
        if (head == null) {
            return head;
        }

        // 给head创建一个新的头节点
        ListNode newHead = new ListNode(0, head);
        ListNode cur = newHead;
        
        // 然后遍历链表，遍历到倒数第二个节点
        while (cur.next != null && cur.next.next != null) {
            // 如果当前节点后面两个节点值相等
            if (cur.next.val == cur.next.next.val) {
                int val = cur.next.val;
                // 然后将相等的节点删除
                while (cur.next != null && cur.next.val == val) {
                    // 将当前节点的next指向next的next
                    // 就把和val相等的节点删除了
                    cur.next = cur.next.next;
                }
            } else {
                // 没有相等的节点，直接跳到下一个节点
                cur = cur.next;
            }
        }

        // 返回newHead的next就是删除重复值后的链表
        return newHead.next;
    }
}
```

## 72. 编辑距离

原题链接：[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

> 给你两个单词 word1 和 word2， 请返回将 word1 转换成 word2 所使用的最少操作数  。
>
> 你可以对一个单词进行如下三种操作：
>
> - 插入一个字符
> - 删除一个字符
> - 替换一个字符
>
> **提示：**
>
> - `0 <= word1.length, word2.length <= 500`
> - `word1` 和 `word2` 由小写英文字母组成

### 1.动态规划

``` java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();

        if (m * n == 0) {
            return m + n;
        }

        // dp[i][j]表示word1的前i个字母到word2的前j个字母之间的编辑距离
        int[][] dp = new int[m + 1][n + 1];

        // word1的前i个字母到空的word2的编辑距离为i
        for (int i = 0; i <= m; i++) {
            dp[i][0] = i;
        }

        // 空的word1到word2的前j个字母的编辑距离为j
        for (int j = 0; j <= n; j++) {
            dp[0][j] = j;
        }

        // 如果word1和word2最后一个字母相同
        // dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - ][j - 1])
        // 如果word1和word2最后一个字母不同
        // dp[i][j] = min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - ][j - 1] + 1)
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                int left = dp[i - 1][j] + 1;
                int down = dp[i][j - 1] + 1;
                int leftDown = dp[i - 1][j - 1];

                if (word1.charAt(i - 1) != word2.charAt(j - 1)) {
                    leftDown += 1;
                }

                dp[i][j] = Math.min(left, Math.min(down, leftDown));
            }
        }

        // dp[m][n]就是word1转换成word2所使用的最少操作数
        return dp[m][n];
    }
}
```

## 2. 两数相加

原题链接：[2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

> 给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。
>
> 请你将两个数相加，并以相同形式返回一个表示和的链表。
>
> 你可以假设除了数字 0 之外，这两个数都不会以 0 开头。
>
> **提示：**
>
> - 每个链表中的节点数在范围 `[1, 100]` 内
> - `0 <= Node.val <= 9`
> - 题目数据保证列表表示的数字不含前导零

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
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        // 创建一个head节点作为和的链表的头节点
        ListNode head = new ListNode();
        ListNode node = head;

        // 用t来保存进位
        int t = 0;
        // 遍历链表l1 l2，将相同位数的节点相加
        while (l1 != null || l2 != null) {
            // sum保存当前位数的节点的和
            int sum = t;

            // l1节点不为空才做加法
            if (l1 != null) {
                sum += l1.val;

                l1 = l1.next;
            }

            // l2节点不为空才做加法
            if (l2 != null) {
                sum += l2.val;

                l2 = l2.next;
            }

            // sum对10取余作为新节点的值
            // 并放到node的next节点上
            node.next = new ListNode(sum % 10);
            // node跳到下一个节点
            node = node.next;

            // sum / 10计算进位
            t = sum / 10;
        }

        // 如果进位大于0
        if (t > 0) {
            // 则还需要再新建一个节点
            node.next = new ListNode(t);
            node = node.next;
        }

        // head.next就是结果链表
        return head.next;
    }
}
```

## 148. 排序链表

原题链接：[148. 排序链表](https://leetcode-cn.com/problems/sort-list/)

> 给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。
>
> **提示：**
>
> - 链表中节点的数目在范围 `[0, 5 * 104]` 内
> - `-105 <= Node.val <= 105`

### 1.归并排序

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

    /**
     * 归并排序
     * 用递归的方式不断的将链表一分为二
     * 直到分到每段只剩下一个节点时
     * 再做递归
     */
    public ListNode sortList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        // 利用快慢指针将链表一分为二
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // 将链表分为first、second
        ListNode first = head;
        ListNode second = slow.next;
        // 断开first、second的连接
        slow.next = null;

        // 继续对链表进行递归切分
        // 此时返回的left、right就是有序的链表
        ListNode left = sortList(first);
        ListNode right = sortList(second);

        // 合并这两个有序的链表
        return merge(left, right);
    }

    /**
     * 合并两个有序链表
     */
    private ListNode merge(ListNode first, ListNode second) {
        if (first == null && second == null) {
            return null;
        }

        if (first == null) {
            return second;
        }

        if (second == null) {
            return first;
        }

        ListNode newHead = new ListNode();
        ListNode node = newHead;
        while (first != null || second != null) {
            if (first == null) {
                node.next = second;
                
                break ;
            } else if (second == null) {
                node.next = first;

                break ;
            } else if (first.val < second.val) {
                node.next = first;

                first = first.next;
            } else {
                node.next = second;

                second = second.next;
            }

            node = node.next;
        }

        return newHead.next;
    }
}
```

## 8. 字符串转换整数 (atoi)

原题链接：[8. 字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

> 请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。
>
> 函数 myAtoi(string s) 的算法如下：
>
> 读入字符串并丢弃无用的前导空格
> 检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
> 读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
> 将前面步骤读入的这些数字转换为整数（即，"123" -> 123， "0032" -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
> 如果整数数超过 32 位有符号整数范围 [−231,  231 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −231 的整数应该被固定为 −231 ，大于 231 − 1 的整数应该被固定为 231 − 1 。
> 返回整数作为最终结果。
> 注意：
>
> 本题中的空白字符只包括空格字符 ' ' 。
> 除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符。
>
> **提示：**
>
> - `0 <= s.length <= 200`
> - `s` 由英文字母（大写和小写）、数字（`0-9`）、`' '`、`'+'`、`'-'` 和 `'.'` 组成

### 1.迭代法

``` java
class Solution {
    public int myAtoi(String s) {
        int length = s.length();

        // true：表示正数；false：表示负数
        boolean flag = true;
        // 记录数字出现的数量包括+、-符号
        int t = 0;

        // 将所有数字放入str中
        StringBuilder str = new StringBuilder();
        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);
            if (c == ' ') {
                // 如果在数字后面还出现空格
                if (t > 0) {
                    // 就直接跳出循环
                    break ;
                }
            } else if (c == '+') {
                // 如果在数字后面还出现+号
                if (t > 0) {
                    // 就直接跳出循环
                    break ;
                }

                // 记数
                t++;
            } else if (c == '-') {
                // 如果在数字后面还出现-号
                if (t > 0) {
                    // 就直接跳出循环
                    break ;
                }

                // 标记为负数
                flag = false;

                // 记数
                t++;
            } else if (c >= '0' && c <= '9') {
                // 将数字放入str中
                str.append(c);

                // 记数
                t++;
            } else {
                // 如果出现其它字符，就直接跳出循环
                break ;
            }
        }

        if (str.length() == 0) {
            return 0;
        }

        try {
            // 将字符串转为整型
            int result = Integer.parseInt(str.toString());

            // 根据flag返回正负结果
            return flag ? result : -result;
        } catch (Exception ignore) {
            // 如果抛了异常表示字符串转为整型溢出了
        }

        // 到了这里就返回int整型的极值
        return flag ? Integer.MAX_VALUE : Integer.MIN_VALUE;
    }
}
```

## 剑指 Offer 22. 链表中倒数第k个节点

原题链接：[剑指 Offer 22. 链表中倒数第k个节点](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)

> 输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。
>
> 例如，一个链表有 6 个节点，从头节点开始，它们的值依次是 1、2、3、4、5、6。这个链表的倒数第 3 个节点是值为 4 的节点。

### 1.双指针

``` java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode getKthFromEnd(ListNode head, int k) {
      	// 先定义firstNode向右遍历k个节点
        ListNode firstNode = head;
        while (k-- > 0 && firstNode != null) {
            firstNode = firstNode.next;
        }

        // 然后定义secondNode从头开始遍历
      	// firstNode继续向右遍历，直到firstNode遍历到空节点为止
        ListNode secondNode = head;
        while (firstNode != null) {
          	// firstNode和secondNode一起向右遍历
            firstNode = firstNode.next;
            secondNode = secondNode.next;
        }

      	// secondNode节点就是倒数第k个节点
        return secondNode;
    }
}
```

## 41. 缺失的第一个正数

原题链接：[41. 缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)

> 给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。
>
> 请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。
>
> **提示：**
>
> - `1 <= nums.length <= 5 * 105`
> - `-231 <= nums[i] <= 231 - 1`

### 1.HashSet

``` java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int length = nums.length;

        // 将所有整数放入set集合中
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            set.add(num);
        }

        // 整数数组长度为length
        // 那么先看看nums中的整数在区间[1, length]内有没有缺失的正整数
        // 从1开始只要缺失那就是最小的正整数
        for (int i = 1; i <= length; i++) {
            // set中不存在i
            if (!set.contains(i)) {
                // 那么i就是没有出现的最小的正整数
                return i;
            }
        }

        // 区间[1, length]的正整数在nums都存在
        // 那么length + 1就是没有出现的最小的正整数
        return length + 1;
    }
}
```

## 31. 下一个排列

原题链接：[31. 下一个排列](https://leetcode.cn/problems/next-permutation/)

> 整数数组的一个 排列  就是将其所有成员以序列或线性顺序排列。
>
> 例如，arr = [1,2,3] ，以下这些都可以视作 arr 的排列：[1,2,3]、[1,3,2]、[3,1,2]、[2,3,1] 。
> 整数数组的 下一个排列 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 下一个排列 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。
>
> 例如，arr = [1,2,3] 的下一个排列是 [1,3,2] 。
> 类似地，arr = [2,3,1] 的下一个排列是 [3,1,2] 。
> 而 arr = [3,2,1] 的下一个排列是 [1,2,3] ，因为 [3,2,1] 不存在一个字典序更大的排列。
> 给你一个整数数组 nums ，找出 nums 的下一个排列。
>
> 必须 原地 修改，只允许使用额外常数空间。
>
>  **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 100`

### 1.两遍扫描

``` java
class Solution {
    public void nextPermutation(int[] nums) {
        int length = nums.length, i = length - 2;
        // 先从后往前找逆序对
        while (i >= 0 && nums[i] >= nums[i + 1]) {
            i--;
        }

        if (i >= 0) { // i >= 0 表示区间[i + 1, length)都是逆序的
            // 首先需要找到区间[i + 1, length)第一个大于num[i]的数
            int j = length - 1;
            while (j > i && nums[i] >= nums[j]) {
                j--;
            }

            // 然后再交换i、j两个位置的值
            swap(nums, i, j);
        }

        // 最后将区间[i + 1, length)之间的元素位置反转
        // 就可以得到整数数组的下一个排列 
        reverse(nums, i + 1, length - 1);
    }

    /**
     * 交换数组nums中的i、j两个位置的元素
     */
    private void swap(int[] nums, int i, int j) {
        if (i == j) {
            return ;
        }

        int t = nums[i];
        nums[i] = nums[j];
        nums[j] = t;
    }

    /**
     * 将数组nums中区间[i, j]之间的元素位置反转
     */
    private void reverse(int[] nums, int left, int right) {
        if (left == right) {
            return ;
        }

        while (left < right) {
            swap(nums, left++, right--);
        }
    }
}
```

## 22. 括号生成

原题链接：[22. 括号生成](https://leetcode.cn/problems/generate-parentheses/)

> 数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。
>
>  **提示：**
>
> - `1 <= n <= 8`

### 1.递归

``` java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> res = new ArrayList<>();
        StringBuilder builder = new StringBuilder();

        // 递归
        helper(n, 0, 0, res, builder);

        return res;
    }

    /**
     * 递归：每次都可能放入“(”或者在 right < left 时放入“)”
     * left：左括号使用的次数
     * right：有括号使用的次数
     * res：保存结果的集合
     * builder：保存当前轮次结果
     */
    private void helper(int n, int left, int right, List<String> res, StringBuilder builder) {
        if (left == n && right == n) {
            // 当左右括号使用次数等于n时
            // 将结果加入集合中
            res.add(builder.toString());

            return ;
        }

        if (left < n) {
            // left < n 还有左括号没用
            // 可以将左括号加入builder
            builder.append("(");
            // 左括号使用次数加1，继续递归
            helper(n, left + 1, right, res, builder);
            // 然后将刚刚加入的左括号删除
            builder.delete(builder.length() - 1, builder.length());
        }

        if (right < left) {
            // right < left 还有右括号没用
            // 可以将右括号加入builder
            builder.append(")");
            // 右括号使用次数加1，继续递归
            helper(n, left, right + 1, res, builder);
            // 然后将刚刚加入的右括号删除
            builder.delete(builder.length() - 1, builder.length());
        }
    }
}
```

## 1143. 最长公共子序列

原题链接：[1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)

> 给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。
>
> 一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。
>
> 例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。
> 两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。
>
> **提示：**
>
> - `1 <= text1.length, text2.length <= 1000`
> - `text1` 和 `text2` 仅由小写英文字符组成。

### 1.动态规划

``` java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        // 利用二维数组dp保存两个字符串公共子序列的长度
        // 其中dp[i][j]表示text1[0:i]和text2[0:j]的最长公共子序列的长度
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            char c1 = text1.charAt(i - 1);
            for (int j = 1; j <= n; j++) {
                char c2 = text2.charAt(j - 1);
                if (c1 == c2) {
                    // 当前c1 == c2时
                    // 要考虑text1[0:i-1]和text2[0:j-1]的公共子序列
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    // 否则，需要考虑text1[0:i-1]和text2[0:j]的公共子序列
                    // 以及text1[0:i]和text2[0:j-1]的公共子序列
                    // 并取两者的最大值
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }
}
```

## 93. 复原 IP 地址

原题链接：[93. 复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)

> 有效 IP 地址 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。
>
> 例如："0.1.2.201" 和 "192.168.1.1" 是 有效 IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 无效 IP 地址。
> 给定一个只包含数字的字符串 s ，用以表示一个 IP 地址，返回所有可能的有效 IP 地址，这些地址可以通过在 s 中插入 '.' 来形成。你 不能 重新排序或删除 s 中的任何数字。你可以按 任何 顺序返回答案。
>
> **提示：**
>
> - `1 <= s.length <= 20`
> - `s` 仅由数字组成

### 1.递归

``` java
class Solution {
    // 整型数组来保存当前生成的ip地址
    int[] ip;
    // res集合保存结果
    List<String> res;
    public List<String> restoreIpAddresses(String s) {
        ip = new int[4];
        res = new ArrayList<>();

        helper(s, 0, 0);

        return res;
    }

    /**
     * 递归将字符串s转成ip地址
     * start：表示字符串遍历的位置
     * pos：表示ip地址生成到第几位了
     */
    private void helper(String s, int start, int pos) {
        int length = s.length();
        if (pos == 4) {
            // pos == 4表示当前ip地址已经生成了4位
            if (start == length) {
                // start == length表示当前生成的ip地址使用了所有的字符串
                // 是有效的ip地址，将结果加入结果集合当中
                StringBuilder builder = new StringBuilder();
                for (int i = 0; i < 4; i++) {
                    builder.append(ip[i]);

                    if (i < 3) {
                        builder.append(".");
                    }
                }

                res.add(builder.toString());
            }

            return ;
        }

        if (start == length) {
            return ;
        }

        // 遇到0则直接将0作为ip地址中的一位
        if (s.charAt(start) == '0') {
            ip[pos] = 0;

            helper(s, start + 1, pos + 1);

            return ;
        }

        // num用来控制当前位的ip地址有效
        int num = 0;
        // 从start开始往后遍历字符串
        // 生成当前有效的ip地址位
        for (int i  = start; i < length; i++) {
            num = num * 10 + (s.charAt(i) - '0');
            // 控制num在区间(0, 255]内，保证有效
            if (num > 0 && num <= 255) {
                ip[pos] = num;

                helper(s, i + 1, pos + 1);
            } else {
                // 否则不需要忘后取了
                break ;
            }
        }
    }
}
```

