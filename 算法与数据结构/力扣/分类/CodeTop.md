# CodeTop

题目来源于CodeTop中互联网大厂面试的高频考题。

## 206. 反转链表

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★☆

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

掌握程度：★★★

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

掌握程度：★☆☆

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

### 2.堆排序

``` java
class Solution {
    public int[] sortArray(int[] nums) {
        int length = nums.length;
        // heapify：构建一个最大堆
        for (int i = (length - 1) / 2; i >= 0; i--) {
            shiftDown(nums, length, i);
        }

        // 原地堆排序
        // 将最大堆堆顶元素nums[0]与堆中最后一个叶子节点元素nums[i]交换位置
        // 然后通过shiftDown维护区间[0, i)范围内最大堆的性质
        // 直到只剩下根节点
        for (int i = length - 1; i > 0; i--) {
            // 交换堆顶与最后一个叶子的位置
            swap(nums, 0, i);
            // 维护区间[0, i)范围内最大堆的性质
            shiftDown(nums, i, 0);
        }

        return nums;
    }

    /**
     * 将以nums[k]为根节点
     * 且在区间[0, length)范围内的子树构建成最大堆
     */
    private void shiftDown(int[] nums, int length, int k) {
        // nums[2 * k + 1]表示根节点num[k]的左节点
        // nums[2 * k + 2]表示根节点num[k]的右节点
        while (2 * k + 1 < length) {
            int j = 2 * k + 1;
            // 取根节点nums[i]左右节点中较大的节点
            if (j + 1 < length && nums[j] < nums[j + 1]) {
                j = j + 1;
            }

            // 如果根节点比左右节点中较大的节点还大
            // 说明当前以nums[k]为根节点的树已经是最大堆
            if (nums[k] > nums[j]) {
                // 直接break退出循环
                break ;
            }

            // 交换nums[k]与nums[j]
            // 以维护最大堆的性质
            swap(nums, k, j);

            // 然后切换根节点为nums[j]
            // 继续维护nums[k]的子树为最大堆
            k = j;
        }
    }

    /**
     * 交换数组中i和j两个元素的位置
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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：☆☆☆

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：☆☆☆

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

掌握程度：★★★

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

掌握程度：★★★

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

掌握程度：★☆☆

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

掌握程度：★★☆

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

掌握程度：★☆☆

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

掌握程度：★☆☆

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

## 151. 颠倒字符串中的单词

掌握程度：★★★

原题链接：[151. 颠倒字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

> 给你一个字符串 s ，颠倒字符串中 单词 的顺序。
>
> 单词 是由非空格字符组成的字符串。s 中使用至少一个空格将字符串中的 单词 分隔开。
>
> 返回 单词 顺序颠倒且 单词 之间用单个空格连接的结果字符串。
>
> 注意：输入字符串 s中可能会存在前导空格、尾随空格或者单词间的多个空格。返回的结果字符串中，单词间应当仅用单个空格分隔，且不包含任何额外的空格。
>
> **提示：**
>
> - `1 <= s.length <= 104`
> - `s` 包含英文大小写字母、数字和空格 `' '`
> - `s` 中 **至少存在一个** 单词

### 1.遍历+集合

``` java
class Solution {
    public String reverseWords(String s) {
        int length = s.length();

        // st和end记录单词的起止位置
        int st = 0, end = 0;
        List<String> res = new ArrayList<>();
        // 遍历字符串的每一个字符
        // 将单词分割开，然后放入集合res中
        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);
            if (c == ' ') {
                // 遇到空格时
                // 如果st < end
                if (st < end) {
                    // 表示单词结束了
                    // 将当前单词加入集合res中
                    res.add(s.substring(st, end));
                }

                // 然后更新st和end的值为下一个位置i + 1
                st = end = i + 1;
            } else {
                // 遇到的不是空格
                // 那么单词终止位置+1
                end++;
            }
        }

        // st < end表示还有最后一个单词没有放入集合res中
        if (st < end) {
            // 将最后一个单词放入集合res中
            res.add(s.substring(st, end));
        }

        // 最后将每一个单词倒序放入builder中
        StringBuilder builder = new StringBuilder();
        for (int i = res.size() - 1; i >= 0; i--) {
            builder.append(res.get(i));
            
            if (i > 0) {
                builder.append(" ");
            }
        }

        return builder.toString();
    }
}
```

### 2.倒序遍历

``` java
class Solution {
    public String reverseWords(String s) {
        int length = s.length();

        // st和end记录单词的起止位置
        int st = length, end = length;
        StringBuilder builder = new StringBuilder();
        // 倒序遍历字符串的每一个字符
        // 将单词分割开，然后放入builder中
        for (int i = length - 1; i >= 0; i--) {
            char c = s.charAt(i);
            if (c == ' ') {
                // 遇到空格时
                // 如果st < end
                if (st < end) {
                    // 表示单词结束了
                    // 将当前单词加入builder中
                    builder.append(s.substring(st, end));
                    // 并插入一个空格
                    builder.append(" ");
                }

                // 然后更新st和end的值为下一个位置i
                st = end = i;
            } else {
                // 遇到的不是空格
                // 那么更新单词起始位置，-1
                st--;
            }
        }

        // st < end表示还有最后一个单词没有放入builder中
        if (st < end) {
            // 将最后一个单词放入集合res中
            builder.append(s.substring(st, end));
        } else {
            // 如果最后没有单词要放入builder中了
            // 那么需要删除最后一个空格
            builder.delete(builder.length() - 1, builder.length());
        }

        return builder.toString();
    }
}
```

## 144. 二叉树的前序遍历

掌握程度：★★★

原题链接：[144. 二叉树的前序遍历](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

> 给你二叉树的根节点 `root` ，返回它节点值的 **前序** 遍历。
>
> **提示：**
>
> - 树中节点数目在范围 `[0, 100]` 内
> - `-100 <= Node.val <= 100`

### 1.前序遍历

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
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        // 前序遍历
        preorderTraversal(root, res);

        return res;
    }

    /**
     * 前序遍历
     */
    private void preorderTraversal(TreeNode root, List<Integer> res) {
        if (root == null) {
            return ;
        }

        // 前遍历当前节点
        res.add(root.val);

        // 再遍历左右子节点
        preorderTraversal(root.left, res);
        preorderTraversal(root.right, res);
    }
}
```

## 105. 从前序与中序遍历序列构造二叉树

掌握程度：★☆☆

原题链接：[105. 从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

> 给定两个整数数组 preorder 和 inorder ，其中 preorder 是二叉树的先序遍历， inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。
>
> 提示:
>
> - 1 <= preorder.length <= 3000
> - inorder.length == preorder.length
> - -3000 <= preorder[i], inorder[i] <= 3000
> - preorder 和 inorder 均 无重复 元素
> - inorder 均出现在 preorder
> - preorder 保证 为二叉树的前序遍历序列
> - inorder 保证 为二叉树的中序遍历序列

### 1.根据先序遍历和中序遍历递归构造二叉树

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
    
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        // 递归构造二叉树
        return buildTree(preorder, inorder, 0, 0, inorder.length - 1);
    }

    /**
     * 根据先序遍历和中序遍历递归构造二叉树
     * 在先序遍历preorder中找到根节点，也就是preorder[pos]
     * 然后将中序遍历inorder以根节点分为左右两部分
     * 左边为根节点的左子树，右边为根节点的右子树
     * 以此类推
     * @param preorder 先序遍历结果
     * @param inorder 中序遍历结果
     * @param pos 先序遍历结果访问到的位置
     * @param st 中序遍历结果的左边界
     * @param end 中序遍历结果的右边界
     * @return
     */
    private TreeNode buildTree(int[] preorder, int[] inorder, int pos, int st, int end) {
        if (pos >= preorder.length || st > end) {
            return null;
        }

        // 起始的preorder[pos]就是当前根节点的值
        int cur = preorder[pos];
        // 创建根节点
        TreeNode node = new TreeNode(cur);
        // 如果中序遍历结果的左右边界相等
        // 说明当前节点没有子节点
        if (st == end) {
            // 直接返回即可
            return node;
        }

        // 然后在中序遍历结果inorder的区间[st, end]范围内
        // 找到节点cur在inorder中的位置
        int mid = st;
        for (int i = st; i <= end; i++) {
            if (inorder[i] == cur) {
                mid = i;

                break ;
            }
        }

        // 再将inorder以item的位置一分为二
        // 其中左边区间为[st, mid - 1]
        // preorder的起始位置为pos + 1
        node.left = buildTree(preorder, inorder, pos + 1, st, mid - 1);
        // 右边区间为[mid + 1, end]
        // preorder的起始位置为pos + mid - st + 1
        node.right = buildTree(preorder, inorder, pos + mid - st + 1, mid + 1, end);

        return node;
    }

    public TreeNode buildTreeByPostorder(int[] inorder, int[] postorder) {
        return buildTreeByPostorder(inorder, postorder, postorder.length - 1, 0, inorder.length - 1);
    }

    /**
     * 根据中序遍历和后序遍历递归构造二叉树
     * 在后序遍历postorder中找到当前根节点，也就是postorder[pos]
     * 然后将中序遍历inorder以根节点分为左右两部分
     * 左边为根节点的左子树，右边为根节点的右子树
     * 以此类推
     * @param inorder 中序遍历结果
     * @param postorder 后序遍历结果
     * @param pos 后序遍历结果访问到的位置
     * @param st 中序遍历结果的左边界
     * @param end 中序遍历结果的右边界
     * @return
     */
    private TreeNode buildTreeByPostorder(int[] inorder, int[] postorder, int pos, int st, int end) {
        if (pos < 0 || st > end) {
            return null;
        }

        // postorder[pos]就是当前根节点的值
        int cur = postorder[pos];
        // 创建根节点
        TreeNode node = new TreeNode(cur);
        // 如果中序遍历结果的左右边界相等
        // 说明当前节点没有子节点
        if (st == end) {
            // 直接返回即可
            return node;
        }

        // 然后在中序遍历结果inorder的区间[st, end]范围内
        // 找到节点cur在inorder中的位置
        int mid = st;
        for (int i = st; i <= end; i++) {
            if (inorder[i] == cur) {
                mid = i;

                break ;
            }
        }

        // 再将inorder以item的位置一分为二
        // 其中左边区间为[st, mid - 1]
        // postorder的起始位置为pos - (end - mid + 1)
        node.left = buildTreeByPostorder(inorder, postorder, pos - (end - mid + 1), st, mid - 1);
        // 右边区间为[mid + 1, end]
        // postorder的起始位置为pos - 1
        node.right = buildTreeByPostorder(inorder, postorder, pos - 1, mid + 1, end);

        return node;
    }
}
```

### 2.根据中序遍历和后序遍历递归构造二叉树

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

    public TreeNode buildTreeByPostorder(int[] inorder, int[] postorder) {
        return buildTreeByPostorder(inorder, postorder, postorder.length - 1, 0, inorder.length - 1);
    }

    /**
     * 根据中序遍历和后序遍历递归构造二叉树
     * 在后序遍历postorder中找到当前根节点，也就是postorder[pos]
     * 然后将中序遍历inorder以根节点分为左右两部分
     * 左边为根节点的左子树，右边为根节点的右子树
     * 以此类推
     * @param inorder 中序遍历结果
     * @param postorder 后序遍历结果
     * @param pos 后序遍历结果访问到的位置
     * @param st 中序遍历结果的左边界
     * @param end 中序遍历结果的右边界
     * @return
     */
    private TreeNode buildTreeByPostorder(int[] inorder, int[] postorder, int pos, int st, int end) {
        if (pos < 0 || st > end) {
            return null;
        }

        // postorder[pos]就是当前根节点的值
        int cur = postorder[pos];
        // 创建根节点
        TreeNode node = new TreeNode(cur);
        // 如果中序遍历结果的左右边界相等
        // 说明当前节点没有子节点
        if (st == end) {
            // 直接返回即可
            return node;
        }

        // 然后在中序遍历结果inorder的区间[st, end]范围内
        // 找到节点cur在inorder中的位置
        int mid = st;
        for (int i = st; i <= end; i++) {
            if (inorder[i] == cur) {
                mid = i;

                break ;
            }
        }

        // 再将inorder以item的位置一分为二
        // 其中左边区间为[st, mid - 1]
        // postorder的起始位置为pos - (end - mid + 1)
        node.left = buildTreeByPostorder(inorder, postorder, pos - (end - mid + 1), st, mid - 1);
        // 右边区间为[mid + 1, end]
        // postorder的起始位置为pos - 1
        node.right = buildTreeByPostorder(inorder, postorder, pos - 1, mid + 1, end);

        return node;
    }
}
```

## 239. 滑动窗口最大值

掌握程度：★☆☆

原题链接：[239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

> 给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。
>
> 返回 滑动窗口中的最大值 。
>
> **提示：**
>
> - `1 <= nums.length <= 105`
> - `-104 <= nums[i] <= 104`
> - `1 <= k <= nums.length`

### 1.单调递减队列

``` java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int length = nums.length;

        // 结果数组的长度为length - k + 1
        int[] res = new int[length - k + 1];
        // 用LinkedList实现一个单调递减队列
        LinkedList<Integer> queue = new LinkedList<>();
        int j = 0;
        // 遍历数组中的元素
        for (int i = 0; i < length; i++) {
            if (i < k - 1) {
                // i < k - 1时将元素放入队列中
                // 保证滑动窗口内的前k - 1个元素形成一个单调递减队列
                putItem(queue, nums[i]);
            } else {
                // 然后继续往队列添加添加滑动窗口的第k个元素
                // 这样滑动窗口的k个元素就形成了一个单调递减队列
                putItem(queue, nums[i]);
                // 队列中的头部元素就是滑动窗口的最大值
                res[j++] = queue.getFirst();
                // 因为滑动窗口要向右移动
                // 所以判断一下队列的头部元素是否等于滑动窗口的第一个元素
                if (queue.getFirst() == nums[i - k + 1]) {
                    // 相等则删除掉第一个元素
                    queue.removeFirst();
                }
            }
        }

        return res;
    }

    /**
     * 把元素放入单调递减队列
     * 把队列前面小于item的元素删除
     * 然后再将item放入队列中
     * 这样就保证了队列单调递减的性质
     * 队列头部的元素就一定是滑动窗口的最大值
     */
    private void putItem(LinkedList<Integer> queue, int item) {
        while (!queue.isEmpty() && queue.getLast() < item) {
            // 把队列前面小于item的元素删除
            queue.removeLast();
        }

        // 然后再将item放入队列中
        queue.addLast(item);
    }
}
```

## 76. 最小覆盖子串

掌握程度：★☆☆

原题链接：[76. 最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)

> 给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
>
>  
>
> 注意：
>
> 对于 t 中重复字符，我们寻找的子字符串中该字符数量必须不少于 t 中该字符数量。
> 如果 s 中存在这样的子串，我们保证它是唯一的答案。
>
> **提示：**
>
> - `1 <= s.length, t.length <= 105`
> - `s` 和 `t` 由英文字母组成

### 1.滑动窗口

``` java
class Solution {
    public String minWindow(String s, String t) {
        int len1 = s.length(), len2 = t.length();

        // 用needMap记录字符串t中出现的字符及对应的次数
        Map<Character, Integer> needMap = new HashMap<>();
        // 用windowMap记录字符串s中特定区间内出现的字符及对应的次数
        Map<Character, Integer> windowMap = new HashMap<>();

        // 遍历字符串t中的每一个字符
        for (int i = 0; i < len2; i++) {
            char c = t.charAt(i);
            // 然后在needMap中记录次数
            needMap.put(c, needMap.getOrDefault(c, 0) + 1);
        }

        // 用left和right表示字符串s中的特定区间[left, right)
        // need表示区间[left, right)中涵盖字符串t中字符的个数
        // st和end表示字符串s内满足涵盖t中所有字符的子串的最小区间
        // 先让right向右移动
        // 找到区间[left, right)内满足涵盖t中所有字符的子串
        // 然后再让left向右移动，缩小区间范围
        // 找到符合条件的最小子串，然后更新st和end
        int left = 0, right = 0, need = 0, st = 0, end = 0;
        while (right < len1) {
            char c = s.charAt(right++);
            // 如果当前字符在字符串t中
            if (needMap.containsKey(c)) {
                // 先将字符c在windowMap的次数加1
                windowMap.put(c, windowMap.getOrDefault(c, 0) + 1);
                
                // 如果Objects.equals(needMap.get(c), windowMap.get(c))
                // 这里用equals是因为Ingeter类型的整型
                // 大于127就不能直接用==判断相等
                if (Objects.equals(needMap.get(c), windowMap.get(c))) {
                    // 表示区间[left, right)字符c出现的次数
                    // 已经涵盖字符串t中的字符c
                    // need加1
                    need++;
                }
            }

            // needMap.size() == need
            // 表示字符串区间[left, right)满足涵盖t中所有字符
            // while是要不断的让left向右移动
            // 缩小区间，以便找到字符串s涵盖t的最小子串
            while (needMap.size() == need) {
                // 先更新st和end
                if (end == 0 || end - st > right - left) {
                    // 没有更新过或者end - st > right - left时才更新
                    st = left;
                    end = right;
                }

                // 然后left向右移动，缩小区间范围
                c = s.charAt(left++);
                // 如果当前字符在字符串t中
                if (needMap.containsKey(c)) {
                    // 字符c在windowMap的次数减1
                    windowMap.put(c, windowMap.get(c) - 1);
                    // 如果needMap.get(c) > windowMap.get(c)
                    if (needMap.get(c) > windowMap.get(c)) {
                        // 表示区间[left, right)字符c出现的次数
                        // 无法涵盖字符串t中的字符c
                        // need减1
                        need--;
                    }
                }
            }
        }

        // 返回字符串在区间[st, end)内的字符
        // 即为s中涵盖t所有字符的最小子串
        return s.substring(st, end);
    }
}
```

## 129. 求根节点到叶节点数字之和

掌握程度：★★☆

原题链接：[129. 求根节点到叶节点数字之和](https://leetcode.cn/problems/sum-root-to-leaf-numbers/)

> 给你一个二叉树的根节点 root ，树中每个节点都存放有一个 0 到 9 之间的数字。
> 每条从根节点到叶节点的路径都代表一个数字：
>
> 例如，从根节点到叶节点的路径 1 -> 2 -> 3 表示数字 123 。
> 计算从根节点到叶节点生成的 所有数字之和 。
>
> 叶节点 是指没有子节点的节点。
>
> **提示：**
>
> - 树中节点的数目在范围 `[1, 1000]` 内
> - `0 <= Node.val <= 9`
> - 树的深度不超过 `10`

### 1.先序遍历

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

    int sum;
    public int sumNumbers(TreeNode root) {
        sum = 0;

        sumNumbers(root, 0);

        return sum;
    }

    /**
     * 先序遍历
     * root：当前遍历到的节点
     * num：当前遍历到的节点的路径和
     */
    private void sumNumbers(TreeNode root, int num) {
        // 空节点直接返回
        if (root == null) {
            return ;
        }

        // 将当前节点值加入到路径和中
        num = num * 10 + root.val;
        // 左右子节点为空
        if (root.left == null && root.right == null) {
            // 表示当前是叶子节点
            // 将路径和加入到总和中
            sum += num;

            // 并直接返回
            return ;
        }

        // 否则再继续遍历左右子节点
        sumNumbers(root.left, num);
        sumNumbers(root.right, num);
    }
}
```

## 104. 二叉树的最大深度

掌握程度：★★★

原题链接：[104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)

> 给定一个二叉树，找出其最大深度。
>
> 二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
>
> **说明:** 叶子节点是指没有子节点的节点。

### 1.原地递归

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
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        // 先遍历左右节点
        // 然后返回最大值，再深度加1
        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```

### 2.递归

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
    public int maxDepth(TreeNode root) {
        return maxDepth(root, 0);
    }

    /**
     * 递归
     */
    private int maxDepth(TreeNode root, int depth) {
        if (root == null) {
            return depth;
        }

        // 深度加1，再遍历左右节点
        // 然后返回最大值
        return Math.max(maxDepth(root.left, depth + 1), maxDepth(root.right, depth + 1));
    }
}
```

## 110. 平衡二叉树

掌握程度：★★☆

原题链接：[110. 平衡二叉树](https://leetcode.cn/problems/balanced-binary-tree/)

> 给定一个二叉树，判断它是否是高度平衡的二叉树。
>
> 本题中，一棵高度平衡二叉树定义为：
>
> 一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1 。
>
> **提示：**
>
> - 树中的节点数在范围 `[0, 5000]` 内
> - `-104 <= Node.val <= 104`

### 1.递归

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
    public boolean isBalanced(TreeNode root) {
        if (root == null) {
            return true;
        }

        // 比较左右子树的最大深度是否小于等于1
        int heightLeft = maxDepth(root.left);
        int heightRight = maxDepth(root.right);
        if (Math.abs(heightLeft - heightRight) > 1) {
            return false;
        }

        // 然后再递归判断左右子树是否为平衡二叉树
        return isBalanced(root.left) && isBalanced(root.right);
    }

    /**
     * 递归求二叉树的最大深度
     */
    private int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
    }
}
```

## 155. 最小栈

掌握程度：★★★

原题链接：[155. 最小栈](https://leetcode.cn/problems/min-stack/)

> 设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
>
> 实现 MinStack 类:
>
> - MinStack() 初始化堆栈对象。
> - void push(int val) 将元素val推入堆栈。
> - void pop() 删除堆栈顶部的元素。
> - int top() 获取堆栈顶部的元素。
> - int getMin() 获取堆栈中的最小元素。
>
> 提示：
>
> - -231 <= val <= 231 - 1
> - pop、top 和 getMin 操作总是在 非空栈 上调用
> - push, pop, top, and getMin最多被调用 3 * 104 次

### 1.最小栈

``` java
class MinStack {
    
    /**
     * 原生栈
     * 按元素入栈顺序保存元素
     */
    private LinkedList<Integer> stack;

    /**
     * 递增栈
     * 按元素从小到大的顺序入栈
     */
    private LinkedList<Integer> incrementStack;

    public MinStack() {
        stack = new LinkedList<>();
        incrementStack = new LinkedList<>();
    }
    
    public void push(int val) {
        stack.push(val);
        
        // 如果incrementStack为空或者栈顶元素小于val
        if (incrementStack.isEmpty() || incrementStack.peek() > val) {
            // 那么将val压入incrementStack
            incrementStack.push(val);
        } else {
            // 否则将栈顶元素压入incrementStack
            // 保持stack和incrementStack有相同个数的元素
            incrementStack.push(incrementStack.peek());
        }
    }
    
    public void pop() {
        // 将两个栈的栈顶元素都删除
        incrementStack.pop();
        stack.pop();
    }
    
    public int top() {
        // 返回stack的栈顶元素
        return stack.peek();
    }
    
    public int getMin() {
        // 返回incrementStack的栈顶元素
        return incrementStack.peek();
    }
}

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack obj = new MinStack();
 * obj.push(val);
 * obj.pop();
 * int param_3 = obj.top();
 * int param_4 = obj.getMin();
 */
```

## 543. 二叉树的直径

掌握程度：★★★

原题链接：[543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

> 给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。
>
> **注意：**两结点之间的路径长度是以它们之间边的数目表示。

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

    // 二叉树的直径长度
    int max = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        max = 0;

        // 后序遍历
        postOrder(root);

        return max;
    }

    /**
     * 后序遍历
     * 返回从root节点出发到左右子树叶子节点的最大路径长度
     * 并更新整棵二叉树的直径长度
     */
    private int postOrder(TreeNode root) {
        if (root == null) {
            return 0;
        }

        // root节点到左子树叶子节点的最大路径长度
        int left = postOrder(root.left);
        // root节点到右子树叶子节点的最大路径长度
        int right = postOrder(root.right);

        // 用root节点左子树的两个叶子节点的路径长度
        // 更新二叉树的直径长度
        max = Math.max(left + right, max);

        // root节点出发到左右子树叶子节点的最大路径长度
        // 为left和right中的最大值+1
        return Math.max(left, right) + 1;
    }
}
```

## 322. 零钱兑换

掌握程度：★☆☆

原题链接：[322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

> 给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。
>
> 计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回 -1 。
>
> 你可以认为每种硬币的数量是无限的。
>
> **提示：**
>
> - `1 <= coins.length <= 12`
> - `1 <= coins[i] <= 231 - 1`
> - `0 <= amount <= 104`

### 1.递归

**力扣上会超时，无法通过**

``` java
class Solution {
    int min;
    public int coinChange(int[] coins, int amount) {
        if (amount == 0) {
            return 0;
        }

        min = amount + 1;

        // 递归
        coinChange(coins, amount, 0);

        // min == amount + 1表示没有任何一种硬币组合能组成总金额
        return min == amount + 1 ? -1 : min;
    }

    /**
     * 递归
     * leftAmount：剩余需要凑的总金额
     * count：目前使用了多少个硬币
     */
    private void coinChange(int[] coins, int leftAmount, int count) {
        // 遍历coins数组
        for (int i = 0; i < coins.length; i++) {
            // 使用硬币数+1
            int temp = count + 1;
            // 减去当前硬币的金额
            int tempAmount = leftAmount - coins[i];
            // 如果剩余需要凑的总金额 <= 0
            if (tempAmount <= 0) {
                if (tempAmount == 0) {
                    // == 0表示刚好凑成了
                    // 更新一下结果值min
                    min = Math.min(temp, min);
                }

                // < 0表示当前硬币面额太大
                // 直接continue
                continue ;
            }

            // tempAmount > 0表示还需要硬币来凑成总金额
            coinChange(coins, tempAmount, temp);
        }
    }
}
```

### 2.动态规划

``` java
class Solution {
    
    public int coinChange(int[] coins, int amount) {
        int length = coins.length;

        // 用dp数组保存凑成总金额为[1, amount]需要最少的硬币数
        int[] dp = new int[amount + 1];
        // 因为要凑齐总金额amount所需要的硬币数肯定不会超过amount + 1
        // 所以给dp赋上初始值amount + 1
        Arrays.fill(dp, amount + 1);
        // 要凑齐总金额为0所需要的硬币数为0
        dp[0] = 0;

        // 从计算凑齐总金额为1需要最少硬币数开始遍历
        for (int i = 1; i <= amount; i++) {
            // 每一个硬币都判断一下
            // 看看能不能以最少的硬币数凑成总金额i
            for (int coin : coins) {
                // 当前硬币面额大于i
                if (coin > i) {
                    // 则跳过
                    continue ;
                }

                // 而凑成总金额i所需要最少的硬币数为
                // dp[i] 和 dp[i - coin] + 1 中取最小值
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }

        return dp[amount] == amount + 1 ? -1 : dp[amount];
    }
}
```

## 113. 路径总和 II

掌握程度：★★★

原题链接：[113. 路径总和 II](https://leetcode.cn/problems/path-sum-ii/)

> 给你二叉树的根节点 root 和一个整数目标和 targetSum ，找出所有 从根节点到叶子节点 路径总和等于给定目标和的路径。
>
> 叶子节点 是指没有子节点的节点。
>
> **提示：**
>
> - 树中节点总数在范围 `[0, 5000]` 内
> - `-1000 <= Node.val <= 1000`
> - `-1000 <= targetSum <= 1000`

### 1.先序遍历

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
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        // 保存从根节点到叶子节点的路径
        List<Integer> list = new ArrayList<>();

        // 先序遍历
        pathSum(root, targetSum, 0, res, list);

        return res;
    }

    /**
     * 先序遍历
     * sum：当前路径总和
     * res：路径总和等于targetSum的路径
     * list：当前从根节点到叶子节点的路径
     */
    private void pathSum(TreeNode root, int targetSum, int sum, List<List<Integer>> res, List<Integer> list) {
        // 先将当前节点值加入路径总和
        sum += root.val;
        // 并将节点值加入路径
        list.add(root.val);

        if (root.left == null && root.right == null) {
            // 如果当前是叶子节点
            if (targetSum == sum) {
                // 且当前路径总和等于targetSum
                // 那么将结果加入res集合
                res.add(new ArrayList<>(list));
            }

            // 叶子节点终止递归
            return ;
        }

        // 递归左节点
        if (root.left != null) {
            pathSum(root.left, targetSum, sum, res, list);

            // 回溯时要删除加入到list路径中的左节点
            list.remove(list.size() - 1);
        }

        // 递归右节点
        if (root.right != null) {
            pathSum(root.right, targetSum, sum, res, list);

            // 回溯时要删除加入到list路径中的右节点
            list.remove(list.size() - 1);
        }
    }
}
```

## 78. 子集

掌握程度：★★★

原题链接：[78. 子集](https://leetcode.cn/problems/subsets/)

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。
>
> **提示：**
>
> - `1 <= nums.length <= 10`
> - `-10 <= nums[i] <= 10`
> - `nums` 中的所有元素 **互不相同**

### 1.递归

``` java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();

        // 递归
        subsets(nums, 0, res, list);

        return res;
    }

    /**
     * 递归
     * st：当前递归的起始位置
     * res：结果集合
     * list：当前递归组成的子集
     */
    private void subsets(int[] nums, int st, List<List<Integer>> res, List<Integer> list) {
        int length = nums.length;

        // 将当前子集放入结果集合res中
        res.add(new ArrayList<>(list));
        
        // 起始位置大于等于nums数组的长度
        if (st >= length) {
            // 直接return，结束递归
            return ;
        }

        // 从起始位置st开始遍历数组nums
        for (int i = st; i < length; i++) {
            // 将当前元素nums[i]加入子集list中
            list.add(nums[i]);
            // 因为子集不能重复
            // 所以下一次递归的起始位置为i + 1
            subsets(nums, i + 1, res, list);

            // 每个元素当前循环中只能使用一次
            // 所以这里将当前元素nums[i]从子集list中删除
            list.remove(list.size() - 1);
        }
    }
}
```

## 32. 最长有效括号

掌握程度：☆☆☆

原题链接：[32. 最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)

> 给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号子串的长度。
>
> **提示：**
>
> - `0 <= s.length <= 3 * 104`
> - `s[i]` 为 `'('` 或 `')'`

### 1.栈

``` java
class Solution {
    public int longestValidParentheses(String s) {
        int length = s.length();

        // 始终保持栈底元素为当前已经遍历过的元素中
        // 最后一个没有被匹配的右括号的下标
        LinkedList<Integer> stack = new LinkedList<>();
        // 空栈时假设最后一个没有被匹配的右括号的下标是-1
        stack.addFirst(-1);
        int res = 0;
        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);
            if (c == '(') {
                // 对于遇到的每个(
                // 将它的下标放入栈中
                stack.addFirst(i);
            } else {
                // 对于遇到的每个)
                // 先弹出栈顶元素表示匹配了当前右括号
                stack.removeFirst();
                
                if (stack.isEmpty()) {
                    // 如果栈为空，说明当前的右括号为没有被匹配的右括号
                    // 将其下标放入栈中来更新最后一个没有被匹配的右括号的下标
                    stack.addFirst(i);
                } else {
                    // 如果栈不为空
                    // 当前右括号的下标减去栈顶元素
                    // 即为以该右括号为结尾的最长有效括号的长度
                    res = Math.max(res, i - stack.getFirst());
                }
            }
        }

        return res;
    }
}
```

## 43. 字符串相乘

掌握程度：★☆☆

原题链接：[43. 字符串相乘](https://leetcode.cn/problems/multiply-strings/)

> 给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。
>
> 注意：不能使用任何内置的 BigInteger 库或直接将输入转换为整数。
>
> 提示：
>
> - 1 <= num1.length, num2.length <= 200
> - num1 和 num2 只能由数字组成。
> - num1 和 num2 都不包含任何前导零，除了数字0本身。

### 1.迭代法

``` java
class Solution {
    public String multiply(String num1, String num2) {
        if ("0".equals(num1) || "0".equals(num2)) {
            return "0";
        }

        int len1 = num1.length(), len2 = num2.length();
        // 因为两数相乘的积位数不会超过这两个数位数的和
        // 所以可以用长度为len1 + len2的整型数组暂存两数相乘的积
        // 其中num[len1 + len2 - 1]为积的个位数
        // 其它高位数向左以此类推
        // num1[i] * num2[j]的结果正好是放在num[i + j + 1]上
        int[] num = new int[len1 + len2];
        for (int i = len1 - 1; i >= 0; i--) {
            int a = num1.charAt(i) - '0';
            for (int j = len2 - 1; j >= 0; j--) {
                int b = num2.charAt(j) - '0';

                int sum = num[i + j + 1] + a * b;
                // 取余留在原位
                num[i + j + 1] = sum % 10;
                // 进位放在左边一位
                num[i + j] += sum / 10;
            }
        }

        // 数组num中从左边开始不为0的元素就是结果
        StringBuilder res = new StringBuilder();
        // 从左遍历数组num并将结果放入res中
        for (int i = 0; i < len1 + len2; i++) {
            if (res.length() == 0 && num[i] == 0) {
                continue ;
            }

            res.append(num[i]);
        }
        
        return res.toString();
    }
}
```

## 98. 验证二叉搜索树

掌握程度：★★★

原题链接：[98. 验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)

> 给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。
>
> 有效 二叉搜索树定义如下：
>
> 节点的左子树只包含 小于 当前节点的数。
> 节点的右子树只包含 大于 当前节点的数。
> 所有左子树和右子树自身必须也是二叉搜索树。
>
> **提示：**
>
> - 树中节点数目范围在`[1, 104]` 内
> - `-231 <= Node.val <= 231 - 1`

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
    public boolean isValidBST(TreeNode root) {
        List<Integer> list = new ArrayList<>();

        // 通过中序遍历将节点值放入集合中
        helper(root, list);

        // 再遍历集合list
        // 如果二叉树root是二叉搜索树
        // 那么list集合中的元素必定是递增的
        for (int i = 1; i < list.size(); i++) {
            if (list.get(i - 1) >= list.get(i)) {
                // 不满足递增，直接返回false
                return false;
            }
        }

        // 到了这里说明list集合是递增的
        // root是二叉搜索树，返回true
        return true;
    }

    /**
     * 中序遍历
     * 通过中序遍历将节点值放入集合中
     */
    private void helper(TreeNode root, List<Integer> list) {
        if (root == null) {
            return ;
        }

        // 先访问左节点
        helper(root.left, list);
        // 将当前节点值放入集合中
        list.add(root.val);
        // 再访问右节点
        helper(root.right, list);
    }
}
```

## 101. 对称二叉树

掌握程度：★★★

原题链接：[101. 对称二叉树](https://leetcode.cn/problems/symmetric-tree/)

> 给你一个二叉树的根节点 `root` ， 检查它是否轴对称。
>
> **提示：**
>
> - 树中节点数目在范围 `[1, 1000]` 内
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
    public boolean isSymmetric(TreeNode root) {
        return isSymmetric(root.left, root.right);
    }

    /**
     * 层序遍历
     * 遍历每一层对称的两个节点left和right是否相等
     */
    private boolean isSymmetric(TreeNode left, TreeNode right) {
        if (left == null && right == null) {
            // left和right都为空
            // 肯定是对称的，直接返回true
            return true;
        }

        if (left == null || right == null) {
            // left和right只要一个不为空
            // 那么肯定不对称，直接返回false
            return false;
        }

        if (left.val != right.val) {
            // left和right值不相等
            // 那么肯定不对称，直接返回false
            return false;
        }

        // 遍历下一层节点
        // 下一层节点中left.right和right.left对称
        // left.left和right.right对称
        return isSymmetric(left.right, right.left) && isSymmetric(left.left, right.right);
    }
}
```

## 165. 比较版本号

掌握程度：★★★

原题链接：[165. 比较版本号](https://leetcode.cn/problems/compare-version-numbers/)

> 给你两个版本号 version1 和 version2 ，请你比较它们。
>
> 版本号由一个或多个修订号组成，各修订号由一个 '.' 连接。每个修订号由 多位数字 组成，可能包含 前导零 。每个版本号至少包含一个字符。修订号从左到右编号，下标从 0 开始，最左边的修订号下标为 0 ，下一个修订号下标为 1 ，以此类推。例如，2.5.33 和 0.1 都是有效的版本号。
>
> 比较版本号时，请按从左到右的顺序依次比较它们的修订号。比较修订号时，只需比较 忽略任何前导零后的整数值 。也就是说，修订号 1 和修订号 001 相等 。如果版本号没有指定某个下标处的修订号，则该修订号视为 0 。例如，版本 1.0 小于版本 1.1 ，因为它们下标为 0 的修订号相同，而下标为 1 的修订号分别为 0 和 1 ，0 < 1 。
>
> 返回规则如下：
>
> - 如果 version1 > version2 返回 1，
> - 如果 version1 < version2 返回 -1，
> - 除此之外返回 0。
>
> 提示：
>
> - 1 <= version1.length, version2.length <= 500
> - version1 和 version2 仅包含数字和 '.'
> - version1 和 version2 都是 有效版本号
> - version1 和 version2 的所有修订号都可以存储在 32 位整数 中

### 1.单指针

``` java
class Solution {
    public int compareVersion(String version1, String version2) {
        // 先将字符串version1、version2转为字符串数组
        String[] strs1 = version1.split("\\.");
        String[] strs2 = version2.split("\\.");

        // 遍历两个字符串数组
        // 按位比较每一位版本的大小
        // 如果位数不够就以0填充
        int len1 = strs1.length, len2 = strs2.length, pos = 0, ver1, ver2;
        while (pos < len1 || pos < len2) {
            if (pos >= len1) {
                // 字符串strs1位数不够用0填充
                ver1 = 0;
                ver2 = Integer.valueOf(strs2[pos]);
            } else if (pos >= len2) {
                ver1 = Integer.valueOf(strs1[pos]);
                // 字符串strs2位数不够用0填充
                ver2 = 0;
            } else {
                ver1 = Integer.valueOf(strs1[pos]);
                ver2 = Integer.valueOf(strs2[pos]);
            }

            // ver1 < ver2 表示version1 < version2
            if (ver1 < ver2) {
                // 直接返回-1
                return -1;
            } else if (ver1 > ver2) {
                // ver1 > ver2 表示version1 > version2
                // 直接返回 1
                return 1;
            }

            pos++;
        }
        
        // 如果以上都没有命中
        // 表示version1 == version2
        // 直接返回0
        return 0;
    }
}
```

### 2.迭代法

``` java
class Solution {
    public int compareVersion(String version1, String version2) {
        // 先将两个版本号分割成字符串数组
        String[] arr1 = version1.split("\\.");
        String[] arr2 = version2.split("\\.");

        int len1 = arr1.length, len2 = arr2.length;
        // 然后按位比较每一位的大小
        for (int i = 0; i < Math.max(len1, len2); i++) {
            // 将每一位修订号转成整型
            // 如果超出数组范围则转成0
            int ver1 = i < len1 ? Integer.parseInt(arr1[i]) : 0;
            int ver2 = i < len2 ? Integer.parseInt(arr2[i]) : 0;

            // 比较两个修订号是否相等
            if (ver1 != ver2) {
                // 不相等ver1 > ver2返回1
                // 否则返回-1
                return ver1 > ver2 ? 1 : -1;
            }
        }

        // 到了这里说明所有的修订号都相等
        // 即两个版本号相等，返回0
        return 0;
    }
}
```

## 64. 最小路径和

掌握程度：★★★

原题链接：[64. 最小路径和](https://leetcode.cn/problems/minimum-path-sum/)

> 给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> **说明：**每次只能向下或者向右移动一步。
>
> 提示：
>
> - m == grid.length
> - n == grid[i].length
> - 1 <= m, n <= 200
> - 0 <= grid[i][j] <= 100

### 1.动态规划

``` java
class Solution {
    public int minPathSum(int[][] grid) {
        int m = grid.length, n = grid[0].length;

        // 利用二维的dp数组保存左上角到grid任意位置的最小路径
        // 其中到grid[i][j]位置的最小路径为dp[i + 1][j + 1]
        int[][] dp = new int[m + 1][n + 1];
        // 初始化dp数组
        for (int i = 0; i <= m; i++) {
            // 因为取的是最小值，所以将dp初始化为Integer.MAX_VALUE
            Arrays.fill(dp[i], Integer.MAX_VALUE);
        }

        // 将dp[1][1]上面和左边初始化为0
        dp[0][1] = 0;
        dp[1][0] = 0;

        // 动态转移方程
        // dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1]
        // 即grid[0][0]到grid[i][j]的最小路径
        // 为grid[0][0]到grid[i - 1][j]和grid[i][j - 1]中的最小值
        // 再加上grid[i][j]
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i - 1][j - 1];
            }
        }

        // 所以dp[m][n]就是左上角到右下角的最小路径
        return dp[m][n];
    }
}
```

## 470. 用 Rand7() 实现 Rand10()

掌握程度：★★☆

原题链接：[470. 用 Rand7() 实现 Rand10()](https://leetcode.cn/problems/implement-rand10-using-rand7/)

> 给定方法 rand7 可生成 [1,7] 范围内的均匀随机整数，试写一个方法 rand10 生成 [1,10] 范围内的均匀随机整数。
>
> 你只能调用 rand7() 且不能调用其他方法。请不要使用系统的 Math.random() 方法。
>
> 每个测试用例将有一个内部参数 n，即你实现的函数 rand10() 在测试时将被调用的次数。请注意，这不是传递给 rand10() 的参数。
>
> **进阶:**
>
> - `rand7()`调用次数的 [期望值](https://en.wikipedia.org/wiki/Expected_value) 是多少 ?
> - 你能否尽量少调用 `rand7()` ?

### 1.拒绝采样

``` java
/**
 * The rand7() API is already defined in the parent class SolBase.
 * public int rand7();
 * @return a random integer in the range 1 to 7
 */
class Solution extends SolBase {
    public int rand10() {
        // 已知 rand_N() 可以等概率的生成[1, N]范围的随机数
        // 那么：
        // (rand_X() - 1) × Y + rand_Y() ==> 可以等概率的生成[1, X * Y]范围的随机数
        // 即实现了 rand_XY()

        // 又有rand_Z()，且Z是V的整数倍
        // 那么rand_V() = (rand_Z() % V) + 1

        while (true) {
            // 先实现rand49()
            int z = (rand7() - 1) * 7 + rand7();

            // 然后通过拒绝采样将生成区间在[41, 49]的数字淘汰掉
            if (z <= 40) {
                // 这样z = rand40()
                // 那么rand10() = (z % 10) + 1
                return (z % 10) + 1;
            }
        }
    }
}
```

### 2.拒绝采样优化

``` java
/**
 * The rand7() API is already defined in the parent class SolBase.
 * public int rand7();
 * @return a random integer in the range 1 to 7
 */
class Solution extends SolBase {
    public int rand10() {
        // 已知 rand_N() 可以等概率的生成[1, N]范围的随机数
        // 那么：
        // (rand_X() - 1) × Y + rand_Y() ==> 可以等概率的生成[1, X * Y]范围的随机数
        // 即实现了 rand_XY()

        // 又有rand_Z()，且Z是V的整数倍
        // 那么rand_V() = (rand_Z() % V) + 1

        while (true) {
            // 先实现rand49()
            int z = (rand7() - 1) * 7 + rand7();

            // 然后通过拒绝采样将生成区间在[41, 49]的数字淘汰掉
            if (z <= 40) {
                // 这样z = rand40()
                // 那么rand10() = (z % 10) + 1
                return (z % 10) + 1;
            }

            // rand9()
            z = z - 40;
            // rand63()
            z = (z - 1) * 7 + rand7();
            if (z <= 60) {
                // rand60()
                return (z % 10) + 1;
            }

            // rand3();
            z = z - 60;
            // rand21()
            z = (z - 1) * 7 + rand7();
            if (z <= 20) {
                // rand20()
                return (z % 10) + 1;
            }
        }
    }
}
```

## 234. 回文链表

掌握程度：★★★

原题链接：[234. 回文链表](https://leetcode.cn/problems/palindrome-linked-list/)

> 给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。
>
> **提示：**
>
> - 链表中节点数目在范围`[1, 105]` 内
> - `0 <= Node.val <= 9`

### 1.双指针

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
    public boolean isPalindrome(ListNode head) {
        if (head.next == null) {
            return true;
        }

        // 使用栈保存链表前半部分节点的值
        LinkedList<Integer> stack = new LinkedList<>();
        // 然后利用快慢指针使slow跳到链表后半部分的第一个节点上
        ListNode slow = head;
        ListNode fast = head;
        while (fast != null && fast.next != null) {
            // 节点值入栈
            stack.push(slow.val);

            // 慢指针跳一个节点
            slow = slow.next;
            // 快指针跳两个节点
            fast = fast.next.next;
        }

        // fast != null表示链表节点个数为单数
        if (fast != null) {
            // 所以slow跳到下一个节点才是链表后半部分的第一个节点上
            slow = slow.next;
        }

        // 栈里面的数据挨个出栈
        // 和链表后半部分的挨个节点值进行比较
        while (!stack.isEmpty()) {
            if (slow.val != stack.pop()) {
                // 只要有一对不相等，那就不是回文链表
                // 直接返回false
                return false;
            }

            // 跳到下一个节点
            slow = slow.next;
        }

        // 到了这里说明是回文链表
        // 返回true
        return true;
    }
}
```

## 39. 组合总和

掌握程度：★☆☆

原题链接：[39. 组合总和](https://leetcode.cn/problems/combination-sum/)

> 给你一个 无重复元素 的整数数组 candidates 和一个目标整数 target ，找出 candidates 中可以使数字和为目标数 target 的 所有 不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。
>
> candidates 中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。 
>
> 对于给定的输入，保证和为 target 的不同组合数少于 150 个。
>
> 提示：
>
> - 1 <= candidates.length <= 30
> - 1 <= candidates[i] <= 200
> - candidate 中的每个元素都 互不相同
> - 1 <= target <= 500

### 1.搜索回溯

``` java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> res = new ArrayList<>();
        List<Integer> list = new ArrayList<>();

        // 搜索回溯
        dfs(candidates, target, res, list, 0);

        return res;
    }

    /**
     * 搜索回溯
     */
    private void dfs(int[] candidates, int target, List<List<Integer>> res, List<Integer> list, int pos) {
        if (pos == candidates.length) {
            // pos遍历完，直接返回
            return ;
        }

        if (target == 0) {
            // 刚好满足组合
            // 将结果放入res集合中
            res.add(new ArrayList<>(list));

            // 然后直接返回
            return ;
        }

        // 跳过当前元素
        dfs(candidates, target, res, list, pos + 1);

        // 选择当前元素
        if (target - candidates[pos] >= 0) {
            list.add(candidates[pos]);

            // 继续选择当前元素
            dfs(candidates, target - candidates[pos], res, list, pos);

            // 当前元素使用完就要删除
            list.remove(list.size() - 1);
        }
    }
}
```

## 169. 多数元素

掌握程度：★★★

原题链接：[169. 多数元素](https://leetcode.cn/problems/majority-element/)

> 给定一个大小为 n 的数组 nums ，返回其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。
>
> 你可以假设数组是非空的，并且给定的数组总是存在多数元素。
>
> **提示：**
>
> - `n == nums.length`
> - `1 <= n <= 5 * 104`
> - `-109 <= nums[i] <= 109`

### 1.哈希

``` java
class Solution {
    public int majorityElement(int[] nums) {
        int length = nums.length;

        // 用map保存每一个元素出现的次数
        Map<Integer, Integer> memory = new HashMap<>();
        // 遍历数组
        for (int i = 0; i < length; i++) {
            // 元素个数+1
            memory.put(nums[i], memory.getOrDefault(nums[i], 0) + 1);
            // 如果元素个数大于length / 2
            if (memory.get(nums[i]) > length / 2) {
                // 就直接返回当前元素
                return nums[i];
            }
        }

        return 0;
    }
}
```

## 48. 旋转图像

掌握程度：★☆☆

原题链接：[48. 旋转图像](https://leetcode.cn/problems/rotate-image/)

> 给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。
>
> 你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。
>
> 提示：
>
> - n == matrix.length == matrix[i].length
> - 1 <= n <= 20
> - -1000 <= matrix[i][j] <= 1000

### 1.迭代法

``` java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        // 右边和下边的边界
        int border = n - 1;
        // 矩阵的层数
        int level = n / 2;

        // 每一层都要旋转，所以先遍历层数
        for (int i = 0; i < level; i++) {
            // 遍历每一层矩阵的第一行
            for (int j = i, pos = 0; j < border; j++, pos++) {
                // 取出matrix[i][j]
                int t = matrix[i][j], c;

                // 将matrix[i][j]放到matrix[j][border]
                c = matrix[j][border];
                matrix[j][border] = t;
                t = c;

                // 将matrix[j][border]放到matrix[border][border - pos]
                c = matrix[border][border - pos];
                matrix[border][border - pos] = t;
                t = c;

                // 将matrix[border][border - pos]放到matrix[border - pos][i]
                c = matrix[border - pos][i];
                matrix[border - pos][i] = t;
                t = c;

                // 最后将matrix[border - pos][i]放回matrix[i][j]
                // 就完成了一层中单个元素的旋转
                matrix[i][j] = t;
            }

            // 然后边界-1
            border--;
        }
    }
}
```

## 112. 路径总和

掌握程度：★★★

原题链接：[112. 路径总和](https://leetcode.cn/problems/path-sum/)

> 给你二叉树的根节点 root 和一个表示目标和的整数 targetSum 。判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 targetSum 。如果存在，返回 true ；否则，返回 false 。
>
> 叶子节点 是指没有子节点的节点。
>
> 提示：
>
> - 树中节点的数目在范围 [0, 5000] 内
> - -1000 <= Node.val <= 1000
> - -1000 <= targetSum <= 1000

### 1.先序遍历

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
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) {
            // 递归到空节点直接返回
            return false;
        }

        // targetSum当前节点值
        targetSum -= root.val;
        if (targetSum == 0 && root.left == null && root.right == null) {
            // 如果targetSum等于0
            // 且当前为叶子节点
            // 则返回true
            return true;
        }

        // 继续递归左右子节点
        return hasPathSum(root.left, targetSum) || hasPathSum(root.right, targetSum);
    }
}
```

## 718. 最长重复子数组

掌握程度：★☆☆

原题链接：[718. 最长重复子数组](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)

> 给两个整数数组 `nums1` 和 `nums2` ，返回 *两个数组中 **公共的** 、长度最长的子数组的长度* 。
>
>  **提示：**
>
> - `1 <= nums1.length, nums2.length <= 1000`
> - `0 <= nums1[i], nums2[i] <= 100`

### 1.动态规划

``` java
class Solution {
    public int findLength(int[] nums1, int[] nums2) {
        int len1 = nums1.length, len2 = nums2.length;

        // 用二维的dp数组保存最长公共子数组
        // 其中dp[i][j]表示nums1[i:]与nums2[j:]的最长公共子数组
        int[][] dp = new int[len1 + 1][len2 + 1];
        int max = 0;
        for (int i = len1 - 1; i >= 0; i--) {
            for (int j = len2 - 1; j >= 0; j--) {
                // 当nums1[i] == nums2[j]时，dp[i][j] = dp[i + 1][j + 1] + 1
                // 否则dp[i][j] = 0
                dp[i][j] = nums1[i] == nums2[j] ? dp[i + 1][j + 1] + 1 : 0;

                // 更新最大长度
                max = Math.max(dp[i][j], max); 
            }
        }

        return max;
    }
}
```

## 226. 翻转二叉树

掌握程度：★★★

原题链接：[226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

> 给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。
>
> **提示：**
>
> - 树中节点数目范围在 `[0, 100]` 内
> - `-100 <= Node.val <= 100`

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
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return root;
        }

        // 先暂存left节点
        TreeNode left = root.left;
        // 然后left == right，这里传right也会返回right
        root.left = invertTree(root.right);
        // right == left，这里传left也会返回left
        root.right = invertTree(left);

        // 返回当前节点即可
        return root;
    }
}
```

## 34. 在排序数组中查找元素的第一个和最后一个位置

掌握程度：★★★

原题链接：[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

> 给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。
>
> 如果数组中不存在目标值 target，返回 [-1, -1]。
>
> 你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。
>
> 提示：
>
> - 0 <= nums.length <= 105
> - -109 <= nums[i] <= 109
> - nums 是一个非递减数组
> - -109 <= target <= 109

### 1.二分查找

``` java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int[] res = new int[] {-1, -1};
        int length;
        if ((length = nums.length) == 0) {
            return res;
        }

        // 二分查找
        int left = 0, right = length - 1;
        // 将区间[0, length)分为[left, mid]和(mid, rigth]两部分
        // 然后比较nums[mid]和target，并根据结果缩小范围
        while (left < right) {
            // 计算mid，避免溢出用>>运算符
            int mid = (left + right) >> 1;
            // 如果nums[mid] < target说明target不在区间[left, mid]
            if (nums[mid] < target) {
                // 缩小left，left右移
                left = mid + 1;
            } else {
                // 否则缩小right，right左移
                right = mid;
            }
        }

        // 上面的while循环会以left == right结束
        // 如果nums[left] != target，说明nums不存在等于target的元素
        if (nums[left] != target) {
            // 直接返回res
            return res;
        }

        // 开始位置就是left
        res[0] = left;
        // 从left往后遍历nums，直到最后一个等于target的元素
        for (int i = left; i < length; i++) {
            if (nums[i] == target) {
                // 更新结束位置
                res[1] = i;
            } else {
                break ;
            }
        }

        // 返回结果
        return res;
    }
}
```

## 221. 最大正方形

掌握程度：★☆☆

原题链接：[221. 最大正方形](https://leetcode.cn/problems/maximal-square/)

> 在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。
>
> 提示：
>
> - m == matrix.length
> - n == matrix[i].length
> - 1 <= m, n <= 300
> - matrix[i][j] 为 '0' 或 '1'

### 1.迭代搜索法

``` java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int m = matrix.length, n = matrix[0].length;

        int max = 0;
        // 遍历二维矩阵，从每一个元素出发
        // 搜索只包含1的正方形
        // 然后在所有的正方形中取面积的最大值
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // 从当前的matrix[i][j]出发
                // 搜索只包含1的正方形，并更新面积最大值
                max = Math.max(max, maximalSquare(matrix, i, j));
            }
        }

        return max;
    }

    /**
     * 迭代搜索法
     * 搜索以matrix[i][j]为起点且只包含1的正方形
     * 返回正方形的面积
     */
    private int maximalSquare(char[][] matrix, int i, int j) {
        if (matrix[i][j] == '0') {
            // 当前不为1，直接返回0
            return 0;
        }

        int m = matrix.length, n = matrix[0].length;
        // 先从matrix[i][j]出发找到只包含1的长方形
        // 其中length表示长方形的长度，height表示长方形的宽度
        int length = m + 1, height = 0;

        // 以matrix[i][j]为起点，从左往右，从上往下
        // 搜索只包含1的长方形
        // 并不断缩小边界m和n
        for (int row = i; row < m; row++) {
            // l表示当前行只包含1的长度
            int l = 0;
            for (int col = j; col < n; col++) {
                if (matrix[row][col] == '1') {
                    l++;
                } else {
                    break ;
                }
            }

            // 长度为0
            if (l == 0) {
                // 则直接退出循环
                break ;
            }

            // 高度+1
            height++;

            // 更新长度，取length和当前长度l的最小值
            length = Math.min(length, l);

            // 缩小边界m和n
            // 因为最终要取的是正方形，正方形边长都是一样的
            // 所以下一层要遍历的边界m不需要超过i + length
            m = Math.min(i + length, m);
            // 并且要遍历的高度边界n也不需要超过j + length
            n = Math.min(j + length, n);
        }

        // 最后取长方形宽高的最小值
        // 就是只包含1的正方形的边长
        length = Math.min(length, height);

        // 正方形边长乘积就是面积
        return length * length;
    }
}
```

## 162. 寻找峰值

掌握程度：★★☆

原题链接：[162. 寻找峰值](https://leetcode.cn/problems/find-peak-element/)

> 峰值元素是指其值严格大于左右相邻值的元素。
>
> 给你一个整数数组 nums，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。
>
> 你可以假设 nums[-1] = nums[n] = -∞ 。
>
> 你必须实现时间复杂度为 O(log n) 的算法来解决此问题。
>
> 提示：
>
> - 1 <= nums.length <= 1000
> - -231 <= nums[i] <= 231 - 1
> - 对于所有有效的 i 都有 nums[i] != nums[i + 1]

### 1.迭代法

``` java
class Solution {
    public int findPeakElement(int[] nums) {
        int length = nums.length;

        int res = 0;
        // 遍历数组，找出那个严格大于左右相邻值的元素
        for (int i = 0; i < length; i++) {
            // 如果是第一个元素，那么它的左边就是Integer.MIN_VALUE
            int left = i == 0 ? Integer.MIN_VALUE : nums[i - 1];
            // 如果是最后一个元素，那么它的右边就是Integer.MIN_VALUE
            int right = i == length - 1 ? Integer.MIN_VALUE : nums[i + 1];

            // 比较是否为峰值元素
            if (left < nums[i] && nums[i] > right) {
                // 是的话就更新到结果res里面
                res = i;

                // 然后退出循环
                break ;
            }
        }

        // 返回结果
        return res;
    }
}
```

## 14. 最长公共前缀

掌握程度：★★★

原题链接：[14. 最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)

> 编写一个函数来查找字符串数组中的最长公共前缀。
>
> 如果不存在公共前缀，返回空字符串 `""`。
>
> **提示：**
>
> - `1 <= strs.length <= 200`
> - `0 <= strs[i].length <= 200`
> - `strs[i]` 仅由小写英文字母组成

### 1.迭代法

``` java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        int length = strs.length;

        // 把每个字符串看成是一行，每个字符看成是一列
        // 然后从第一列逐列比较，只要每一列的字符都想相同就跳到下一列
        // 直到遇到某一列有不同的字符或者某行字符遍历完了

        // 遍历到列的位置
        int pos = 0;
        // 使用StringBuilder存放结果
        StringBuilder builder = new StringBuilder();
        // 当前列比较的字符
        char c;
        // 逐列遍历字符
        // 因为是在所有行中取最小的长度作为遍历的总长度
        // 这里暂时取第一行的长度作为总长度
        // 后面遇到最短的字符，可以直接跳出循环
        while (pos < strs[0].length()) {
            // 取第一行的第pos列的字符作为比较的基准
            c = strs[0].charAt(pos);
            int i = 1;
            // 然后从第二行开始比较每一行第pos列的字符是否都相同
            for (; i < length; i++) {
                // 如果pos已经超过了当前行的长度
                if (pos >= strs[i].length()) {
                    // 那就直接跳出循环
                    break ;
                }

                // 如果当前行的第pos个字符不等于c
                if (c != strs[i].charAt(pos)) {
                    // 也直接跳出循环
                    break ;
                }
            }

            // i != length表示pos已经超过了第i行字符串的长度
            // 或者第i行字符串的第pos个字符不等于c
            if (i != length) {
                // 这里也直接跳出外层循环
                break ;
            }

            // 否则当前列的字符都相同
            // 将字符c加入到结果builder中
            builder.append(c);
            // 跳到下一列
            pos++;
        }

        // 将结果转为字符串返回
        return builder.toString();
    }
}
```

## 240. 搜索二维矩阵 II

掌握程度：★★☆

原题链接：[240. 搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)

> 编写一个高效的算法来搜索 m x n 矩阵 matrix 中的一个目标值 target 。该矩阵具有以下特性：
>
> - 每行的元素从左到右升序排列。
> - 每列的元素从上到下升序排列。
>
> 提示：
>
> - m == matrix.length
> - n == matrix[i].length
> - 1 <= n, m <= 300
> - -109 <= matrix[i][j] <= 109
> - 每行的所有元素从左到右升序排列
> - 每列的所有元素从上到下升序排列
> - -109 <= target <= 109

### 1.二分查找

``` java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length, n = matrix[0].length;

        // 逐层做二分查找
        for (int i = 0; i < m; i++) {
            // 如果target小于这一层的第一个元素
            if (target < matrix[i][0]) {
                // 那么当前层和后面层的元素肯定都大于target
                // 肯定找不到和target相等的元素了，这里直接返回false
                return false;
            }

            // 如果target大于当前层的最后一个元素
            if (target > matrix[i][n - 1]) {
                // 那么当前层肯定找不到和target相等的元素了
                // 直接continue跳到下一层继续搜索
                continue ;
            }

            // 然后在当前层进行二分查找
            int left = 0, right = n - 1;
            // 将区间[left, right]分为[left, mid]、[mid + 1, right]两部分
            while (left < right) {
                int mid = (left + right) >>> 1;
                if (matrix[i][mid] < target) {
                    // target可能在区间[mid + 1, right]
                    // 缩小left
                    left = mid + 1;
                } else {
                    // target可能在区间[left, mid]
                    // 缩小right
                    right = mid;
                }
            }

            // 当target == matrix[i][left]时
            if (target == matrix[i][left]) {
                // 搜索到target，返回true
                return true;
            }
        }

        // 否则没搜索到，返回false
        return false;
    }
}
```

## 83. 删除排序链表中的重复元素

掌握程度：★★★

原题链接：[83. 删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)

> 给定一个已排序的链表的头 `head` ， *删除所有重复的元素，使每个元素只出现一次* 。返回 *已排序的链表* 。
>
> **提示：**
>
> - 链表中节点数目在范围 `[0, 300]` 内
> - `-100 <= Node.val <= 100`
> - 题目数据保证链表已经按升序 **排列**

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
    public ListNode deleteDuplicates(ListNode head) {
        // 如果是空链表或者只有一个节点
        if (head == null || head.next == null) {
            // 直接返回head
            return head;
        }

        ListNode node = head;
        // 遍历链表直到倒数第二个节点
        while (node != null && node.next != null) {
            // 如果当前节点值与next节点值相等
            if (node.val == node.next.val) {
                // 直接让当前节点next指向next节点的next节点
                // 即从链表中删除了与当前节点值相等的next节点
                node.next = node.next.next;
            } else {
                // 否则直接跳到下一个节点
                node = node.next;
            }
        }

        return head;
    }
}
```

## 62. 不同路径

掌握程度：★★☆

原题链接：[62. 不同路径](https://leetcode.cn/problems/unique-paths/)

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
>
> 问总共有多少条不同的路径？
>
> **提示：**
>
> - `1 <= m, n <= 100`
> - 题目数据保证答案小于等于 `2 * 109`

### 1.递归

力扣上会超时

``` java
class Solution {

    int path = 0;
    public int uniquePaths(int m, int n) {
        boolean[][] visited = new boolean[m][n];
        
        uniquePaths(visited, m, n, 0, 0);

        return path;
    }

    private void uniquePaths(boolean[][] visited, int m, int n, int i, int j) {
        if (i < 0 || i >= m || j < 0 || j >= n || visited[i][j]) {
            return ;
        }

        if (i == m - 1 && j == n - 1) {
            path++;

            return ;
        }

        visited[i][j] = true;

        uniquePaths(visited, m, n, i + 1, j);
        uniquePaths(visited, m, n, i, j + 1);

        visited[i][j] = false;
    }
}
```

### 2.动态规划

``` java
class Solution {
    public int uniquePaths(int m, int n) {
        // 定义二维数组dp用来保存起始点到网格任意点的不同路径
        int[][] dp = new int[m][n];

        // 初始化：起始点到网格第一行所有点的路径只有一条
        for (int j = 0; j < n; j++) {
            dp[0][j] = 1;
        }

        // 初始化：起始点到网格第一列所有点的路径只有一条
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1;
        }

        // 动态转移方程：dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }

        // dp[m - 1][n - 1]就是起始点到网格右下角的路径
        return dp[m - 1][n - 1];
    }
}
```

## 128. 最长连续序列

掌握程度：★☆☆

原题链接：[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)

> 给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。
>
> 请你设计并实现时间复杂度为 O(n) 的算法解决此问题。
>
> **提示：**
>
> - `0 <= nums.length <= 105`
> - `-109 <= nums[i] <= 109`

### 1.哈希

``` java
class Solution {
    public int longestConsecutive(int[] nums) {
        int max = 0, length;
        if ((length = nums.length) == 0) {
            return max;
        }

        // 用HasSet保存数组nums出现的元素
        Set<Integer> memory = new HashSet<>(length);
        for (int i = 0; i < length; i++) {
            memory.add(nums[i]);
        }

        // 遍历数组，统计连续序列
        for (int i = 0; i < length; i++) {
            int num = nums[i];
            // 如果num - 1不在memory中，说明num是序列的第一位
            if (!memory.contains(num - 1)) {
                int sum = 1;
                // 对num逐次递增1，看看是否在memory中
                while (memory.contains(++num)) {
                    // 如果在sum就加1
                    sum++;
                }

                // 最后更新结果max
                max = Math.max(max, sum);
            }
        }

        return max;
    }
}
```

### 2.排序

``` java
class Solution {
    public int longestConsecutive(int[] nums) {
        int length;
        if ((length = nums.length) == 0) {
            return 0;
        }

        // 先对数组排序
        Arrays.sort(nums);
        int max = 1, sum = 1;
        // 然后遍历有序数组
        for (int i = 0; i < length - 1; i++) {
            // 如果nums[i] == nums[i + 1] - 1
            if (nums[i] == nums[i + 1] - 1) {
                // 表示连续，sum加1
                sum++;
            } else if (nums[i] != nums[i + 1]) {
                // 否则重置sum
                // 这里nums[i] != nums[i + 1]时才重置sum
                // 是为了跳过nums[i] == nums[i + 1]的情况
                sum = 1;
            }

            // 然后更新结果max
            max = Math.max(max, sum);
        }

        return max;
    }
}
```

## 394. 字符串解码

掌握程度：★★☆

原题链接：[394. 字符串解码](https://leetcode.cn/problems/decode-string/)

> 给定一个经过编码的字符串，返回它解码后的字符串。
>
> 编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
>
> 你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
> 此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。
>
> 提示：
>
> - 1 <= s.length <= 30
> - s 由小写英文字母、数字和方括号 '[]' 组成
> - s 保证是一个 有效 的输入。
> - s 中所有整数的取值范围为 [1, 300] 

### 1.递归

``` java
class Solution {

    // 定义一个全局的pos用来控制字符遍历的位置
    int pos = 0;
    public String decodeString(String s) {
        int length = s.length();

        // builder用于保存结果
        StringBuilder builder = new StringBuilder();
        // num表示当前要重复的次数
        int num = 0;
        while (pos < length) {
            char c = s.charAt(pos++);
            if (c == '[') {
                // 遇到字符[表示后面的字符需要重复
                // 通过递归的方式获取需要重复的字符串
                String str = decodeString(s);

                // 根据重复次数将字符串append到builder中
                while (num-- > 0) {
                    builder.append(str);
                }

                // 然后重置num为0
                num = 0;

                continue ;
            }

            if (c == ']') {
                // 遇到字符]表示重复的字符串结束
                // 直接跳出循环
                break ;
            }

            if (c >= '0' && c <= '9') {
                // 表示c为数字，即需要重复的次数
                // num * 10是因为重复次数可能是十位数百位数
                num = (c - '0') + num * 10;
            } else {
                // 到了这里c就是字母，直接append到builder就好了
                builder.append(c);
            }
        }

        // 返回字符串
        return builder.toString();
    }
}
```

## 24. 两两交换链表中的节点

掌握程度：★★★

原题链接：[24. 两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

> 给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。
>
> **提示：**
>
> - 链表中节点的数目在范围 `[0, 100]` 内
> - `0 <= Node.val <= 100`

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
    public ListNode swapPairs(ListNode head) {
        // 如果是空链表或者链表只有一个节点
        if (head == null || head.next == null) {
            // 直接返回head
            return head;
        }

        // 创建一个新的头节点，并让head作为newHead的next节点
        ListNode newHead = new ListNode(0, head);
        // pre就作为要交换的两个节点的前继节点
        ListNode pre = newHead;
        // 从新的头节点开始遍历链表
        while (pre.next != null && pre.next.next != null) {
            // first和second是要交换的两个节点
            ListNode first = pre.next;
            ListNode second = first.next;

            // 第一个节点的next指向第二个节点的next
            first.next = second.next;
            // 第二个节点的next指向第一个节点
            second.next = first;
            // 前继节点指向second节点，就完成了交换
            pre.next = second;

            // 再让前继节点指向first节点
            // 切换为下一次要交换节点的前继节点
            pre = first;
        }

        // newHead的next节点就是交换后的新链表
        return newHead.next;
    }
}
```

## 695. 岛屿的最大面积

掌握程度：★★★

原题链接：[695. 岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)

> 给你一个大小为 m x n 的二进制矩阵 grid 。
>
> 岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在 水平或者竖直的四个方向上 相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。
>
> 岛屿的面积是岛上值为 1 的单元格的数目。
>
> 计算并返回 grid 中最大的岛屿面积。如果没有岛屿，则返回面积为 0 。
>
> **提示：**
>
> - `m == grid.length`
> - `n == grid[i].length`
> - `1 <= m, n <= 50`
> - `grid[i][j]` 为 `0` 或 `1`

### 1.深度优先搜索

``` java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int m = grid.length, n = grid[0].length;

        int max = 0;
        // 挨个遍历矩阵
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // 从i,j位置搜索岛屿并更新结果
                max = Math.max(max, dfs(grid, m, n, i, j));
            }
        }

        return max;
    }

    /**
     * 深度优先搜索
     */
    private int dfs(int[][] grid, int m, int n, int i, int j) {
        if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] == 0) {
            return 0;
        }

        // 记录当前的1，所以sum初始值为1
        int sum = 1;
        // 将当前位置置为0，表示已经访问过了
        grid[i][j] = 0;

        // 按上右下左的顺序搜索，并把数量加到sum中
        sum += dfs(grid, m, n, i - 1, j);
        sum += dfs(grid, m, n, i, j + 1);
        sum += dfs(grid, m, n, i + 1, j);
        sum += dfs(grid, m, n, i, j - 1);

        return sum;
    }
}
```

## 122. 买卖股票的最佳时机 II

原题链接：[122. 买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)

> 给你一个整数数组 prices ，其中 prices[i] 表示某支股票第 i 天的价格。
>
> 在每一天，你可以决定是否购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以先购买，然后在 同一天 出售。
>
> 返回 你能获得的 最大 利润 。
>
> **提示：**
>
> - `1 <= prices.length <= 3 * 104`
> - `0 <= prices[i] <= 104`

### 1.贪心算法

``` java
class Solution {
    public int maxProfit(int[] prices) {
        int sum = 0, length = prices.length;

        // 遍历数组prices
        for (int i = 1; i < length; i++) {
            // 因为股票价格是预知的
            // 所以只要当天的股票价格大于前一天的
            if (prices[i - 1] < prices[i]) {
                // 就进行交易
                sum += prices[i] - prices[i - 1];
            }
        }

        return sum;
    }
}
```

## 912. 排序数组

掌握程度：★☆☆

原题链接：[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)

> 给你一个整数数组 `nums`，请你将该数组升序排列。
>
>  **提示：**
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

### 2.堆排序

``` java
class Solution {
    public int[] sortArray(int[] nums) {
        int length = nums.length;
        // heapify：构建一个最大堆
        for (int i = (length - 1) / 2; i >= 0; i--) {
            shiftDown(nums, length, i);
        }

        // 原地堆排序
        // 将最大堆堆顶元素nums[0]与堆中最后一个叶子节点元素nums[i]交换位置
        // 然后通过shiftDown维护区间[0, i)范围内最大堆的性质
        // 直到只剩下根节点
        for (int i = length - 1; i > 0; i--) {
            // 交换堆顶与最后一个叶子的位置
            swap(nums, 0, i);
            // 维护区间[0, i)范围内最大堆的性质
            shiftDown(nums, i, 0);
        }

        return nums;
    }

    /**
     * 将以nums[k]为根节点
     * 且在区间[0, length)范围内的子树构建成最大堆
     */
    private void shiftDown(int[] nums, int length, int k) {
        // nums[2 * k + 1]表示根节点num[k]的左节点
        // nums[2 * k + 2]表示根节点num[k]的右节点
        while (2 * k + 1 < length) {
            int j = 2 * k + 1;
            // 取根节点nums[i]左右节点中较大的节点
            if (j + 1 < length && nums[j] < nums[j + 1]) {
                j = j + 1;
            }

            // 如果根节点比左右节点中较大的节点还大
            // 说明当前以nums[k]为根节点的树已经是最大堆
            if (nums[k] > nums[j]) {
                // 直接break退出循环
                break ;
            }

            // 交换nums[k]与nums[j]
            // 以维护最大堆的性质
            swap(nums, k, j);

            // 然后切换根节点为nums[j]
            // 继续维护nums[k]的子树为最大堆
            k = j;
        }
    }

    /**
     * 交换数组中i和j两个元素的位置
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

## 152. 乘积最大子数组

掌握程度：☆☆☆

原题链接：[152. 乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)

> 给你一个整数数组 nums ，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。
>
> 测试用例的答案是一个 32-位 整数。
>
> 子数组 是数组的连续子序列。
>
> 提示:
>
> - 1 <= nums.length <= 2 * 104
> - -10 <= nums[i] <= 10
> - nums 的任何前缀或后缀的乘积都 保证 是一个 32-位 整数

### 1.动态规划

``` java
class Solution {
    public int maxProduct(int[] nums) {
        int length = nums.length;

        // 用min和max分别表示乘积过程中产生的最小最大值
        int min = 1, max = 1, res = Integer.MIN_VALUE;
        // 遍历数组nums，挨个做乘积
        for (int i = 0; i < length; i++) {
            if (nums[i] < 0) {
                // 当前元素小于0时，交换min和max的值
                // 这样做的目的是如果出现了偶数个负数的话
                // 根据负负得正，min和负数相乘将会得到更大的正数
                // 所以这里交换min和max
                int t = min;
                min = max;
                max = t;
            }

            // 更新最大值max：取nums[i]和max * nums[i]两者的最大值
            max = Math.max(nums[i], max * nums[i]);
            // 更新最小值min：取nums[i]和min * nums[i]两者的最小值
            min = Math.min(nums[i], min * nums[i]);

            // 再更新结果res
            res = Math.max(res, max);
        }

        // 返回结果
        return res;
    }
}
```

## 153. 寻找旋转排序数组中的最小值

掌握程度：★☆☆

原题链接：[153. 寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)

> 已知一个长度为 n 的数组，预先按照升序排列，经由 1 到 n 次 旋转 后，得到输入数组。例如，原数组 nums = [0,1,2,4,5,6,7] 在变化后可能得到：
> 若旋转 4 次，则可以得到 [4,5,6,7,0,1,2]
> 若旋转 7 次，则可以得到 [0,1,2,4,5,6,7]
> 注意，数组 [a[0], a[1], a[2], ..., a[n-1]] 旋转一次 的结果为数组 [a[n-1], a[0], a[1], a[2], ..., a[n-2]] 。
>
> 给你一个元素值 互不相同 的数组 nums ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 最小元素 。
>
> 你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。
>
> 提示：
>
> - n == nums.length
> - 1 <= n <= 5000
> - -5000 <= nums[i] <= 5000
> - nums 中的所有整数 互不相同
> - nums 原来是一个升序排序的数组，并进行了 1 至 n 次旋转

### 1.二分查找

``` java
class Solution {
    public int findMin(int[] nums) {
        int length = nums.length;

        // 二分查找
        int left = 0, right = length - 1;
        while (left < right) {
            int mid = (left + right) >>> 1;
            if (nums[mid] > nums[right]) {
                // 只要nums[mid] > nums[right]就说明最小值在mid的右边
                // 所以此时要缩小左区间
                left = mid + 1;
            } else {
                // 否则缩小右区间
                right = mid;
            }
        }

        // nums[left]就是最小值
        return nums[left];
    }
}
```

## 227. 基本计算器 II

掌握程度：★☆☆

原题链接：[227. 基本计算器 II](https://leetcode.cn/problems/basic-calculator-ii/)

> 给你一个字符串表达式 s ，请你实现一个基本计算器来计算并返回它的值。
>
> 整数除法仅保留整数部分。
>
> 你可以假设给定的表达式总是有效的。所有中间结果将在 [-231, 231 - 1] 的范围内。
>
> 注意：不允许使用任何将字符串作为数学表达式计算的内置函数，比如 eval() 。
>
> 提示：
>
> - 1 <= s.length <= 3 * 105
> - s 由整数和算符 ('+', '-', '*', '/') 组成，中间由一些空格隔开
> - s 表示一个 有效表达式
> - 表达式中的所有整数都是非负整数，且在范围 [0, 231 - 1] 内
> - 题目数据保证答案是一个 32-bit 整数

### 1.迭代法

``` java
class Solution {

    public int calculate(String s) {
        int length = s.length();

        // 表示遇到数字时前一个运算符号
        char sign = '+';
        // 用stack来保存被转换为加法运算的数字
        LinkedList<Integer> stack = new LinkedList<>();
        // 用num表示当前遍历过程中得到的整数
        int num = 0;

        for (int i = 0; i < length; i++) {
            char c = s.charAt(i);

            if ('0' <= c && c <= '9') {
                // 将整数转换到num中
                num = (c - '0') + num * 10;
            }

            if ((c == ' ' || '0' <= c && c <= '9') && i < length - 1) {
                // 只要不是最后一个空格或者数字，那就continue
                continue ;
            }

            // 到了这里说明c是运算符
            // 这个时候需要判断当前正在num前一个运算符是什么
            // 然后决定放进栈stack的数字是否需要进行运算
            // 这个步骤的目的就是将所有复杂的运算都转为加法运算
            // 加法直接入栈；减法转为负数；乘除法先计算出结果再压入栈中
            if (sign == '+') {
                // 前一个运算符是+号
                // 直接将num压入栈中
                stack.push(num);
            } else if (sign == '-') {
                // 前一个运算符是-号
                // num取反再压入栈中
                stack.push(-num);
            } else if (sign == '*') {
                // 前一个运算符是*号
                // 根据运算法则，乘法优先于+-法的运算
                // 所以这里先将栈顶元素弹出与num相乘
                // 再将乘积压入栈中
                stack.push(stack.pop() * num);
            } else {
                // 到了这里说明前一个运算符是/号
                // 根据运算法则，除法优先于+-法的运算
                // 所以这里先将栈顶元素弹出与num相除
                // 再将商压入栈中
                stack.push(stack.pop() / num);
            }

            // 初始化num，以备下一次转换数字
            num = 0;
            // 然后将c作为下一个整数的前置运算符
            sign = c;
        }

        // 清空栈中的元素，然后累加结果到sum中
        int sum = 0;
        while (!stack.isEmpty()) {
            sum += stack.pop();
        }

        return sum;
    }
}
```

## 179. 最大数

掌握程度：★☆☆

原题链接：[179. 最大数](https://leetcode.cn/problems/largest-number/)

> 给定一组非负整数 `nums`，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数。
>
> **注意：**输出结果可能非常大，所以你需要返回一个字符串而不是整数。
>
> **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 109`

### 1.排序

``` java
class Solution {
    public String largestNumber(int[] nums) {
        int length = nums.length;

        // zero用于统计nums数组中0出现的次数
        int zero = 0;
        // 将nums数组中每一个整数转为字符串并放入list集合中
        List<String> list = new ArrayList<>();
        for (int i = 0; i < length; i++) {
            if (nums[i] == 0) {
                // 统计0出现的次数
                zero++;
            }

            // 将整数转为字符串并放入list集合中
            list.add(Integer.toString(nums[i]));
        }

        // 如果数组中0出现的次数等于数组长度
        if (zero == length) {
            // 那么能组成的最大整数就是0
            return "0";
        }

        // 对集合里面的字符串排序：字符串的字符从左往右比较
        // 左边字符较大的字符串排在前面
        Collections.sort(list, (a, b) -> (b + a).compareTo(a + b));

        // 然后将排序后的字符按list集合的顺序拼接起来就组成了最大的整数
        StringBuilder builder = new StringBuilder();
        for (String str : list) {
            builder.append(str);
        }

        return builder.toString();
    }
}
```

## 662. 二叉树最大宽度

掌握程度：★☆☆

原题链接：[662. 二叉树最大宽度](https://leetcode.cn/problems/maximum-width-of-binary-tree/)

> 给定一个二叉树，编写一个函数来获取这个树的最大宽度。树的宽度是所有层中的最大宽度。这个二叉树与满二叉树（full binary tree）结构相同，但一些节点为空。
>
> 每一层的宽度被定义为两个端点（该层最左和最右的非空节点，两端点间的null节点也计入长度）之间的长度。
>
> **注意:** 答案在32位有符号整数的表示范围内。

### 1.深度搜索

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

    // res用来记录结果
    int res = 0;
    // 给二叉树的每一层节点从左到右编号（从0开始）
    // 使用哈希结构memory用来记录每一层最左边节点的编号
    Map<Integer, Integer> memory;
    public int widthOfBinaryTree(TreeNode root) {
        res = 0;
        memory = new HashMap<>();

        dfs(root, 0, 0);

        return res;
    }

    /**
     * 深度搜索：先序遍历
     * depth：深度即二叉树节点的层数
     * position：当前层节点的编号
     */
    private void dfs(TreeNode root, int depth, int position) {
        if (root == null) {
            return ;
        }

        // 因为深度搜索的时候，每一层总是最左边的节点先被访问
        // 所以可以根据depth是否存在于memory中
        // 判断当前节点是不是当前层depth最左边的节点
        if (!memory.containsKey(depth)) {
            // 不在说明是最左边的节点，保存层数depth和编号position的关系
            memory.put(depth, position);
        }

        // position - memory.get(depth) + 1就是当前节点到最左边节点的距离
        // 用这个值来更新结果res
        res = Math.max(res, position - memory.get(depth) + 1);

        // 再继续搜索root的左右子节点
        // 深度depth + 1
        // 左子节点的位置position为当前position * 2
        dfs(root.left, depth + 1, position * 2);
        // 右子节点的位置position为当前position * 2 + 1
        dfs(root.right, depth + 1, position * 2 + 1);
    }
}
```

## 468. 验证IP地址

掌握程度：★☆☆

原题链接：[468. 验证IP地址](https://leetcode.cn/problems/validate-ip-address/)

> 给定一个字符串 queryIP。如果是有效的 IPv4 地址，返回 "IPv4" ；如果是有效的 IPv6 地址，返回 "IPv6" ；如果不是上述类型的 IP 地址，返回 "Neither" 。
>
> 有效的IPv4地址 是 “x1.x2.x3.x4” 形式的IP地址。 其中 0 <= xi <= 255 且 xi 不能包含 前导零。例如: “192.168.1.1” 、 “192.168.1.0” 为有效IPv4地址， “192.168.01.1” 为无效IPv4地址; “192.168.1.00” 、 “192.168@1.1” 为无效IPv4地址。
>
> 一个有效的IPv6地址 是一个格式为“x1:x2:x3:x4:x5:x6:x7:x8” 的IP地址，其中:
>
> - 1 <= xi.length <= 4
>
> - xi 是一个 十六进制字符串 ，可以包含数字、小写英文字母( 'a' 到 'f' )和大写英文字母( 'A' 到 'F' )。
>
> - 在 xi 中允许前导零。
>
> 例如 "2001:0db8:85a3:0000:0000:8a2e:0370:7334" 和 "2001:db8:85a3:0:0:8A2E:0370:7334" 是有效的 IPv6 地址，而 "2001:0db8:85a3::8A2E:037j:7334" 和 "02001:0db8:85a3:0000:0000:8a2e:0370:7334" 是无效的 IPv6 地址。
>
> **提示：**
>
> - `queryIP` 仅由英文字母，数字，字符 `'.'` 和 `':'` 组成。

### 1.迭代法

``` java
class Solution {
    
    // 组成十六进制的字符
    String hexStr = "0123456789abcdefABCDEF";
    public String validIPAddress(String queryIP) {
        int length = queryIP.length();
        
        if (queryIP.indexOf(".") > 0) {
            // 可能为IPv4
            // 以.切割字符串queryIP，传-1会匹配尽可能多的次数
            // 并且不会丢弃最后一个空格
            String[] strs = queryIP.split("\\.", -1);
            if (strs.length != 4) {
                // 长度不等于4不符合IPv4
                return "Neither";
            }

            for (String str : strs) {
                int len;
                if ((len = str.length()) == 0 || len > 3 || len > 1 && str.charAt(0) == '0') {
                    // 某一位长度为0或者大于3或者有前导0都不符合IPv4
                    return "Neither";
                }

                int num = 0;
                try {
                    num = Integer.valueOf(str);
                } catch (Exception e) {
                    num = -1;
                }

                if (num < 0 || num > 255) {
                    // 数值小于0或者大于255都不符合IPv4
                    return "Neither";
                }
            }

            // 到了这里表示当前符合IPv4
            return "IPv4";
        } else if (queryIP.indexOf(":") > 0) {
            // 可能为IPv6
            // 以:切割字符串queryIP，传-1会匹配尽可能多的次数
            // 并且不会丢弃最后一个空格
            String[] strs = queryIP.split(":", -1);
            if (strs.length != 8) {
                // 长度不等于8不符合IPv6
                return "Neither";
            }

            for (String str : strs) {
                int len = str.length();
                if (len < 1 || len > 4) {
                    // 某一位长度小于0或者大于4都不符合IPv6
                    return "Neither";
                }

                for (int i = 0; i < str.length(); i++) {
                    char c = str.charAt(i);
                    if (hexStr.indexOf(c) == -1) {
                        // 某一位只要有一个字符不是十六进制字符那就不符合IPv6
                        return "Neither";
                    }
                }
            }

            // 到了这里表示当前符合IPv6
            return "IPv6";
        }

        // 到了这里表示queryIP既不属于IPv4也不属于IPv6
        return "Neither";
    }
}
```

## 138. 复制带随机指针的链表

掌握程度：★★☆

原题链接：[138. 复制带随机指针的链表](https://leetcode.cn/problems/copy-list-with-random-pointer/)

> 给你一个长度为 n 的链表，每个节点包含一个额外增加的随机指针 random ，该指针可以指向链表中的任何节点或空节点。
>
> 构造这个链表的 深拷贝。 深拷贝应该正好由 n 个 全新 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 next 指针和 random 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。复制链表中的指针都不应指向原链表中的节点 。
>
> 例如，如果原链表中有 X 和 Y 两个节点，其中 X.random --> Y 。那么在复制链表中对应的两个节点 x 和 y ，同样有 x.random --> y 。
>
> 返回复制链表的头节点。
>
> 用一个由 n 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 [val, random_index] 表示：
>
> val：一个表示 Node.val 的整数。
> random_index：随机指针指向的节点索引（范围从 0 到 n-1）；如果不指向任何节点，则为  null 。
> 你的代码 只 接受原链表的头节点 head 作为传入参数。
>
> **提示：**
>
> - `0 <= n <= 1000`
> - `-104 <= Node.val <= 104`
> - `Node.random` 为 `null` 或指向链表中的节点。

### 1.迭代法

``` java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/

class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) {
            return null;
        }

        // 先将每个节点复制一份
        // 复制的节点就接到原节点的后面
        Node node = head;
        while (node != null) {
            Node next = node.next;

            Node temp = new Node(node.val);
            node.next = temp;
            temp.next = next;

            node = next;;
        }

        // 填充随机指针
        node = head;
        while (node != null) {
            // 含有随机指针的节点
            if (node.random != null) {
                // 复制出来的新节点的随机指针指向原节点随机指针指向节点的next
                node.next.random = node.random.next;
            }

            node = node.next.next;
        }

        // 将原节点和新节点拆分为两个链表
        node = head;
        // 表示新链表的头节点
        Node newHead = new Node(0);
        Node cur = newHead;
        while (node != null) {
            Node next = node.next.next;
            // cur指向新节点
            cur.next = node.next;
            cur = cur.next;
            // node指向原节点
            node.next = next;
            node = node.next;
        }
        
        // newHead的next就是新链表的头节点
        return newHead.next;
    }
}
```

## 136. 只出现一次的数字

掌握程度：★★☆

原题链接：[136. 只出现一次的数字](https://leetcode.cn/problems/single-number/)

> 给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。
>
> 说明：
>
> 你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？
>

### 1.异或运算

``` java
class Solution {
    public int singleNumber(int[] nums) {
        int length = nums.length;

        // 根据异或运算法则：相同整数异或的结果为0
        int res = nums[0];
        for (int i = 1; i < length; i++) {
            // 逐个元素进行异或运算
            // 最终结果就是那个支出现一次的元素
            res ^= nums[i];
        }

        return res;
    }
}
```

## 198. 打家劫舍

掌握程度：★☆☆

原题链接：[198. 打家劫舍](https://leetcode.cn/problems/house-robber/)

> 你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
> 给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。
>
> **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 400`

### 1.动态规划

``` java
class Solution {
    public int rob(int[] nums) {
        int length;
        if ((length = nums.length) == 0) {
            return 0;
        }

        if (length == 1) {
            return nums[0];
        }

        // 用dp数组表示前 i 间房屋能偷窃到的最高总金额
        int[] dp = new int[length];
        // 只有一间房屋能偷窃到的最高总金额为num[0]
        dp[0] = nums[0];
        // 有两间房屋能偷窃到的最高总金额为max(nums[0], nums[1])
        dp[1] = Math.max(nums[0], nums[1]);
        // 超过两间房屋能偷窃到的最高金额为
        // dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1])
        // 偷窃第i间房时：dp[i] = dp[i - 2] + nums[i]
        // 不偷窃第i间房时：dp[i] = dp[i - 1]

        for (int i = 2; i < length; i++) {
            dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
        }

        // dp[length - 1]就是能够偷窃到的最高金额
        return dp[length - 1];
    }
}
```

