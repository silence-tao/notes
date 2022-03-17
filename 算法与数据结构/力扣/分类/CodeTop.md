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

