# 剑指offer JAVA题解

# JZ1 二维数组中的查找
在一个二维数组中（每个一维数组的长度相同），每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
## 思路
1. 暴力解
2.仿照二分的思想
先找一个基准（二分一般是找mid位置），关键在于找到一致的变化方向（有序的情况下，如果小于mid，往左；大于，则往右）。
所以关键是需要对下一个方向进行判断，要调整一个基准值，在判断和target的大小后，能有清晰的变化方向，所以会采用从左下往右上或者从右上王左下的方向。
以左下为基准，如果大于target，需要往小走，即行--；如果小于target，需要往大走，即列++。
## 题解
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
        int rows = array.length;
        int cols = array[0].length;
        // 测试用例:1.taeget小于最小值
        if((rows == 0) || (cols == 0) )return false;
        if(array[0][0] > target) return false;
        if(array[rows - 1][cols - 1] < target) return false;
        for (int row = rows - 1, col = 0;row >= 0 && col < cols;){
            if(array[row][col] == target){
                return true;
            }else if(array[row][col] > target){
                row--;
            }else if(array[row][col] < target){
                col++;
            }
        }
        
        return false;
    }
}
```

# JZ2 
请实现一个函数，将一个字符串中的每个空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。
## 思路
存在的坑：如果从左往右不用辅助空间的话，每遇到一个空格都改成%20，会出现后面2个字符被覆盖的问题，所以不能这么操作。
解决方法：
1.自带的函数replace。String str; str.replace(“ ”， “%20”)；注意是替换字符串，不是字符char，不用‘’
2.使用一个辅助的数组，从左往右遍历，每次遇到空格就换成%20添加如数组，其他的不变直接添加。
3.不使用辅助数组。这种思想下需要对str变成的char数组长度进行修改，每次出现一个空格，后面都需要往后移2个大小。这时候就展现出换方向的好处了，如果从后往前，直接把最后一个字符一步到位移动完毕2*空格数个位置，就可以避免每次都重复移动了。
-->这种也出现在合并数组/合并字符串等问题中，因为你需要判断大小并插入，插入过程如果是从前往后那么后面的内容会出现大量的重复移动。所以从后往前一次性修改位置可以加快算法的速度。
## 题解
1. 使用辅助数组
```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        StringBuffer str1 = new StringBuffer();
        char[] ch = str.toString().toCharArray();
        for(char chars : ch){
            if(chars == ' ') str1.append("%20");
            else str1.append(chars);
        }
        return str1.toString();
    }
}
```

2. in-place解法，不使用新的数组
在当前字符串上进行替换。
先计算替换后的字符串需要多大的空间，并对原字符串空间进行扩容；
从后往前替换字符串的话，每个字符串只需要移动一次；
如果从前往后，每个字符串需要多次移动，效率较低。
```java
public class Solution {
    public String replaceSpace(StringBuffer str) {
        int spacenum = 0;
        for(int i = 0; i < str.length(); i++){
            if(str.charAt(i) == ' '){
                spacenum++;
            }
        }
        int oldLength = str.length();
        int oldIndex = oldLength - 1;
        int newLength = oldLength + spacenum*2;
        str.setLength(newLength);
        int newIndex = newLength - 1;
        for(; oldIndex >= 0 && oldLength < newLength; oldIndex--){
            if(str.charAt(oldIndex) == ' '){
                str.setCharAt(newIndex--, '0');
                str.setCharAt(newIndex--, '2');
                str.setCharAt(newIndex--, '%');
            }else{
                str.setCharAt(newIndex--, str.charAt(oldIndex));
            }
        }
        return str.toString();
    }
}
```

# JZ3 从尾到头打印链表
输入一个链表，按链表从尾到头的顺序返回一个ArrayList。
## 思路
从尾到头打印链表的值，实际上就是后进先出的思想，所以可以直接用栈来实现。
把所有值依次放入栈Stack st，然后依次pop出来就好了。
## 题解
1. 栈思想，或者利用list.add(index,value)，可以指定 index 位置插入 value 值
```java
import java.util.*;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        Stack<ListNode> st = new Stack<>();
        ArrayList<Integer> ans = new ArrayList<>();
        while(listNode != null){
            st.push(listNode);
            listNode = listNode.next;
        }
        while(!st.isEmpty()){
            ListNode tmp = st.pop();
            ans.add(tmp.val);
        }
        return ans;
    }
}
```
```
import java.util.*;
public class Solution {
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<>();
        ListNode tmp = listNode;
        while(tmp!=null){
            list.add(0,tmp.val);
            tmp = tmp.next;
        }
        return list;
    }
}
```
2. 递归
```java
import java.util.*;
public class Solution {
    ArrayList<Integer> list = new ArrayList();
    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        if(listNode!=null){
			//递归到结尾，然后一个个加入值
            printListFromTailToHead(listNode.next);
            list.add(listNode.val);
        }
        return list;
    }
}
```



# JZ35 数组中的逆序对
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007
## 思路
实际上就是归并排序，只是需要子啊排序过程中计数，并且要处理count过大取余的问题。
1. 归并中计数
归并到最后一层时肯定都是有序的，你只需要判断在换的过程中，有多少次出现逆序的可能就好了
左右两边都是排好序的，如果左>右，因为左右各自已经有序，所以左值后面的内容一定也大于当前右值，直接统计了一排逆序对
统计一批后，右值拿出来放进数组，再比较下一个，直到结束
2. count取余
一般是设置好一个MOD，在计算过程中只要涉及count变化，就直接%MOD，避免在运算过程中直接超出范围而值出错。
## 题解
```
public class Solution {
    private int count = 0;
    private int MOD = (int) Math.pow(10, 9) + 7;
    public int InversePairs(int [] array) {
        if(array == null || array.length < 2) return count;
        mergeSort(array, 0, array.length - 1);
        return (int) (count % MOD);
    }
    
    public void mergeSort(int[] array, int low, int high){
        if(low >= high) return;
        int mid = low + ((high - low) >> 1);
        mergeSort(array, low, mid);
        mergeSort(array, mid + 1, high);
        merge(array, low, mid, high);
    }
    public void merge(int[] array, int low, int mid, int high){
        int[] temp = new int[high - low + 1];
        int index = 0;
        int p1 = low, p2 = mid + 1;
        while (p1 <= mid && p2 <= high) {
            // 换到最后肯定都是有序的，你只需要判断在换的过程中，有多少次出现逆序的可能就好了
            // 左右两边都是排好序的，如果左>右，因为左右各自已经有序，所以左值后面的内容一定也大于当前右值，直接统计了一排逆序对
            // 统计一批后，右值拿出来放进数组，再比较下一个，直到结束
            if (array[p1] > array[p2]) {
                count = (count + mid - p1 + 1) % MOD;
                temp[index++] = array[p2++];
            } else {
                temp[index++] = array[p1++];
            }
        }
        while (p1 <= mid) {
            temp[index++] = array[p1++];
        }
        while (p2 <= high) {
            temp[index++] = array[p2++];
        }
        for (int i = 0; i < temp.length; i++) {
            array[low+i] = temp[i];
        }
    }
}
```
# JZ36 两个链表中的第一个公共结点
输入两个链表，找出它们的第一个公共结点。（注意因为传入数据是链表，所以错误测试数据的提示是用其他方式显示的，保证传入数据是正确的）
## 思路
第一反应是双指针，每次判断完往后走。但是出现的一个问题是：他这个东西长度不一定是一样的，如果不一样，每次都往后走一步，总会有一个提前结束，导致后面的内容无法串在一起。
这里看到的一个技巧，直接把两个链表合在一起a+b=b+a。从头往前走，如果没走到尾就一直next，如果走到尾那就用另一个链表补上。
为什么两个连起来就能？因为如果有公共结点，肯定有一段末尾是连起来的，也就是说前面少点多点没啥问题，只要保证运行的头到公共点的长度一样就好了。
## 题解
```java
public class Solution {
    public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
        ListNode p1 = pHead1, p2 = pHead2;
        while(p1!=p2){
            p1 = (p1!=null) ? p1.next: pHead2;
            p2 = (p2!=null) ? p2.next: pHead1;
        }
        return p1;
    }
}
```

# JZ16 合并两个排序的链表
输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
## 思路
两个链表都是递增的，也就是可以考虑用两个指针来指出内容，每次判断后确定下一个next是哪个，直到有一个指针走到结尾，然后把剩余的链表连上就好了。
解决方法上也可以有两种模式，递归的和循环的。
这里还要注意一个问题：返回的内容是一个ListNode，那么也就是说我们需要返回链表头，这就涉及到怎么做的问题：
**一般都是先新建一个**
**	ListNode answer = new ListNode(-1);**
**   ListNode ans = answer;**
**	对ans的整个链接做完操作后**
**	return answer.next;**
**	这样就能回到结果的头结点了**
## 题解
1. 循环
```
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        ListNode answer = new ListNode(-1);
        ListNode ans = answer;
        while(list1 != null && list2 != null){
            if(list1.val <= list2.val){
                ans.next = list1;
                list1 = list1.next;
            }else{
                ans.next = list2;
                list2 = list2.next;
            }
            ans = ans.next;
        }
        if (list1 == null){
            ans.next = list2;
        }
        if (list2 == null){
            ans.next = list1;
        }
        return answer.next;
    }
}
```
2. 递归
要明白递归函数的功能。可以不必关心递归函数的具体实现。
比如这个ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
函数功能：合并两个单链表，返回两个单链表头结点值小的那个节点。

如果知道了这个函数功能，那么接下来需要考虑2个问题：

1. 递归函数结束的条件是什么？
2. 递归函数一定是缩小递归区间的，那么下一步的递归区间是什么？
对于问题1. 对于链表就是，如果为空，返回什么?
对于问题2，跟迭代方法中的一样，如果PHead1的所指节点值小于等于pHead2所指的结点值，那么phead1后续节点和pHead节点继续递归

```
public class Solution {
    public ListNode Merge(ListNode list1,ListNode list2) {
        if(list1 == null) return list2;
        if(list2 == null) return list1;
        if(list1.val < list2.val){
            list1.next = Merge(list1.next, list2);
            return list1;
        }else{
            list2.next = Merge(list1, list2.next);
            return list2;
        }
    }
}
```

# JZ24 二叉树中和为某一值的路径
输入一颗二叉树的根节点和一个整数，按字典序打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。
##思路
回溯问题，每次往下找节点把节点放入结果，然后回溯到末尾，如果没合适结果，就回来把path回退，否则的话直接把这条路径加入到结果中去
## 题解
1. 递归
递归算法三部曲：
1)明白递归函数的功能：FindPath(TreeNode* root,int sum)，从root节点出发，找和为sum的路径
2)递归终止条件：当root节点为叶子节点并且sum==root->val, 表示找到了一条符合条件的路径
3)下一次递归：如果左子树不空，递归左子树FindPath(root->left, sum - root->val),如果右子树不空，递归右子树，FindPath(root->right, sum - root->val)
还要注意的是这是一道回溯题，如果不行的话，需要返回，然后把path的内容拿出来，使用list.remove(index)操作，remove结尾就是path.remove(path.size() - 1);
```
public class Solution {
    private ArrayList<ArrayList<Integer>> list = new ArrayList<ArrayList<Integer>>();
    private ArrayList<Integer> path = new ArrayList<Integer>();
    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
		// 走到底了，之前都没判断有个路径，直接返回list
        if(root == null) return list;
        path.add(root.val);
        if(target == root.val && root.left == null && root.right == null){
			// 判断有路径了，把path加上
            list.add(new ArrayList<>(path));
        }
        if(target > root.val){
            if(root.left != null){
				//继续走到下一个节点直到得到结果或结束，都return list
                FindPath(root.left, target - root.val);
				//回溯
                path.remove(path.size() - 1);
            }
            if(root.right != null){
                FindPath(root.right, target - root.val);
                path.remove(path.size() - 1);
            }
        }
        return list;
    }
}
```
# JZ25 复杂链表的复制
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针random指向一个随机节点），请对此链表进行深拷贝，并返回拷贝后的头结点。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）
##思路
这个题目就比较懵了。你可以去复制节点，但是为什么不搞清楚的话就复制不好呢？因为他有2个指针，指向next和random，所以是很有可能出现环进入死循环的。所以需要记录这个链表是否被访问到了。
## 题解
1.使用哈希表记录跳转关系
这样记录，只要有已经访问过的，就可以直接返回访问过的节点跳出当前流程，而这个节点的后续也在流程中复制过了，从而避免死循环。
```java
import java.util.*;
public class Solution {
    private HashMap<RandomListNode, RandomListNode> visited_map = new HashMap<>();
    public RandomListNode Clone(RandomListNode pHead)
    {
        if(pHead == null) return null;
        RandomListNode head = pHead;
        if(visited_map.containsKey(head)){
            return visited_map.get(head);
        }
        RandomListNode node = new RandomListNode(head.label);
        
        visited_map.put(head, node);
        node.next = Clone(head.next);
        node.random = Clone(head.random);
        return node;
    }
}
```
2.

# JZ26 二叉搜索树与双向链表

## 思路

## 题解

# JZ29 最小的K个数
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。
##思路
一个思路是最小堆或者最大堆，这样堆里能维持最大或最小的n个数，只要在堆大小超过n的时候poll出堆顶值就好了。
数组问题，注意数组大小不符合目标的情况，如K>length。
## 题解
1.堆
```
import java.util.*;
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> ans = new ArrayList<Integer>();
        PriorityQueue<Integer> q = new PriorityQueue<>(new Comparator<Integer>() {
            // 重定义比较器
            @Override
            public int compare(Integer a, Integer b) {
                return b - a;
            }}
        );
        if(k > input.length){
            return ans;
        }
        for(int num : input){
            q.offer(num);
            if(q.size()>k){
                q.poll();
            }
        }
        while(!q.isEmpty()){
            ans.add(q.poll());
        }
        return ans;
    }
}
```
2.快排
快排的思想也是可以借鉴的，快排的思路是先随便选一个基准（一般是index=0的值），然后将小于和大于基准的值分在两边，将基准放在中间。那么这实际上就是求基准最后被放在k时，基准左侧的所有值的内容。
下面这个就是topN问题的快排解法。与传统快排不同的是，用基准和k进行比较，通过比较来确定下一个快排顺序，而不是一直递归。
```
import java.util.*;
public class Solution {
    public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
        ArrayList<Integer> ans = new ArrayList<>();
        if (input == null || input.length == 0 || k == 0 || input.length < k) {
            return ans;
        }
        if(input.length == k){
            for(int i = 0; i < k; i++){
                ans.add(input[i]);
            }
            return ans;
        }
        int low = 0, high = input.length - 1;
        while(low <= high){
            int pivot = partition(input, low, high);
            // 参考值刚好在第k个
            if(pivot == k - 1){
                break;
            }else if(pivot < k - 1){
                //参考值在k左侧，说明参考值需要更大些，所以把范围往参考值右边调。
                low = pivot + 1;
            }else{
                //参考值在k右侧，说明参考值需要更小些，所以把范围往参考值左边调。
                high = pivot - 1;
            }
        }
        for(int i = 0; i < k; i++){
            ans.add(input[i]);
        }
        return ans;
    }
    
    public static int partition(int[] arr, int left, int right){
        int pivot = arr[left];
        int start = left;
        int end = right;
        while(true){
            while(arr[start] <= pivot){
                start++;
                if(start == end) break;
            }
            while(arr[end] > pivot){
                end--;
                if(start == end) break;
            }
            if(start > end) break;
            swap(arr, start, end);
        }
        swap(arr, left, end);
        return end;
    }
    
    public static void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }

}
```

# JZ65 矩阵中的路径
请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一个格子开始，每一步可以在矩阵中向左，向右，向上，向下移动一个格子。如果一条路径经过了矩阵中的某一个格子，则该路径不能再进入该格子。
##思路
同理，使用一个visited数组记录是否访问过，避免成环重复。
所以是做个dfs，按照dfs的思路做下去就好了。
dfs的框架：
dfs(输入参数，一般为位置，数组，当前匹配位置){

    // 第一步，检查下标范围是否满足条件

    // 第二步：检查是否被访问过，或者是否满足当前匹配条件

    // 第三步：检查是否满足返回结果条件

    // 第四步：都没有返回，说明当前满足递归条件，应该进行下一步递归
	
    // 标记
    dfs(下一次)
    // 回溯（对于一些要记录的结果，可能需要把结果删除来实现回溯，如把visit矩阵置零）
}  
## 题解
```java
public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str)
    {
        int[] visited = new int[rows * cols];
        for(int i = 0 ; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(find(matrix, rows, cols, visited, i, j, 0, str)) return true;
            }
        }
        return false;
    }
    
    public boolean find(char[] array, int rows, int cols, int[] visited, int i, int j, int pos, char[] str){
        int index = i * cols + j;
        if(i < 0 || i >= rows || j < 0 || j >= cols || array[index] != str[pos]) return false;
        if(visited[index] == 1) return false;
        if(pos == str.length - 1) return true;
        visited[index] = 1;
		// 有一个能判断正确，就直接判正。
        boolean isTarget =  find(array, rows, cols, visited, i - 1, j, pos + 1, str) ||
                            find(array, rows, cols, visited, i, j - 1, pos + 1, str) ||
                            find(array, rows, cols, visited, i + 1, j, pos + 1, str) ||
                            find(array, rows, cols, visited, i, j + 1, pos + 1, str);
		// 恢复访问数组，因为上一行是搜完了内容，搜完了为了方便下一个，所以要重新置零。
        visited[index] = 0;
        return isTarget;
    }
}
```

# JZ64 滑动窗口的最大值
给定一个数组和滑动窗口的大小，找出所有滑动窗口里数值的最大值。例如，如果输入数组{2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，他们的最大值分别为{4,4,6,6,6,5}； 针对数组{2,3,4,2,6,2,5,1}的滑动窗口有以下6个： {[2,3,4],2,6,2,5,1}， {2,[3,4,2],6,2,5,1}， {2,3,[4,2,6],2,5,1}， {2,3,4,[2,6,2],5,1}， {2,3,4,2,[6,2,5],1}， {2,3,4,2,6,[2,5,1]}。
窗口大于数组长度的时候，返回空
##思路

## 题解
1. 暴力方法
把左边界枚举出来，然后从左边界到右边界之间判断最大值。
```
import java.util.*;
public class Solution {
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> ans = new ArrayList<>();
        if(num == null || num.length <= 0 || size == 0) return ans;
        if(size > num.length) return ans;
        int left = 0, right = size + left - 1;
        while(size + left - 1 < num.length){
            int max = num[left];
            for(int i = left; i <= size + left - 1; i++){
                if(max < num[i]) max = num[i];
            }
            ans.add(max);
            left++;
        }
        return ans;
    }
}
```
2. 单调队列
可以看到1中有很多重复计算，因为在每次滑动窗口中都要比较最大值。
使用大顶堆，每次把大的值poll出来，留下的就是剩下的小值。
那么步骤就是：
1. 每次把size大小的窗口内容入堆
2. 每次把最大值拿出来，然后把窗口头部的值出堆（这样如果是最大值，就出来了，不是的话也出来了，但是最大值仍然记录在堆里）
3. 每次拿出来后把后面的值入堆一个，重新构建最大值
4. 重复直到所有内容都入过堆了
数组问题：判断数组为null+数字长度不够+其他标定值为0等。
if(num == null || num.length <= 0 || size == 0 || size > num.length) return ans;
```
import java.util.*;
public class Solution {
    PriorityQueue<Integer> queue = new PriorityQueue<>((o1, o2) -> o2 - o1);
    
    public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> ans = new ArrayList<>();
        if(num == null || num.length <= 0 || size == 0 || size > num.length) return ans;
        int count = 0;
        while(count < size){
            queue.add(num[count]);
            count++;
        }
        // count = size开始
        while(count < num.length){
            ans.add(queue.peek());
            //把左侧的内容拿出
            queue.remove(num[count - size]);
            queue.add(num[count]);
            count++;
        }
        //count = length, 最后一轮走到底，把前一轮的值放进去了，但是最后一轮的值没有放进去，所以需要额外添加。
        ans.add(queue.peek());
        return ans;
    }
}
```
# 

##思路

## 题解

# 

## 思路

## 题解

# 

##思路

## 题解

# 

##思路

## 题解

# 

##思路

## 题解



# JZ17 树的子结构
输入两棵二叉树A，B，判断B是不是A的子结构。（ps：我们约定空树不是任意一个树的子结构）
## 思路
主要要做两个内容：
1.判断当前节点和root2的根节点相不相同，也就是说要遍历root1看到底有没有节点能进入2的判断过程
2.相同的话，判断子节点是否相同
要注意特例条件带来的终止条件，root1为空，root2为空等，记住数的题目每次都要问问自己，这个节点为空了怎么办？
## 题解
public class Solution {
    public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root2 == null || root1 == null) return false;
        // 检查当前节点，并遍历其他节点
        return check(root1, root2) || HasSubtree(root1.left, root2) || HasSubtree(root1.right, root2);
    }
    
    public boolean check(TreeNode root1,TreeNode root2) {
        if(root2 == null) return true;
        if(root1 == null || root1.val!= root2.val) return false;
        return (check(root1.left, root2.left) && check(root1.right, root2.right));
    }
}

# JZ18 二叉树的镜像
操作给定的二叉树，将其变换为源二叉树的镜像。

## 思路
怎么变成镜像，可以通过画图来简化分析一下，实际上就是把当前节点的左右节点.left和.right互换一下，可以想到是一个递归的情况，当前换完之后去递归当前.left和当前.right。这实际上就和先序遍历的思路是一个道理嘛。
注意树形题的当前节点==null这一个特例。

## 题解
1. 递归版本
```java
public class Solution {
    public void Mirror(TreeNode root) {
        if(root == null) return;
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;
        Mirror(root.left);
        Mirror(root.right);
    }
}
```
2. 循环版本
树形问题一般都是有递归和循环版本的，递归版本很好写，循环版本则需要把思路倒一倒，通常会引入队列Queue<TreeNode> nodes = new LinkedList<>();
换成队列的话，正常的都是把当前节点入队，然后取出时判断接下来入队的是哪个节点，这道题在节点出队时，意味着要把左右节点入队顺序改一改。
```java
import java.util.*;
public class Solution {
    public void Mirror(TreeNode root) {
        if(root == null) return;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        while(!q.isEmpty()){
            int len = q.size();
            for(int i = 0; i < len; i++){
                //每次出队一个节点，把左右节点连接关系互换，然后入队
                TreeNode curr = q.poll();
                TreeNode tmp = curr.left;
                curr.left = curr.right;
                curr.right = tmp;
                if(curr.left != null) q.offer(curr.left);
                if(curr.right != null) q.offer(curr.right);
            }
        }
    }
}
```

# JZ19 顺时针打印矩阵
输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下4 X 4矩阵： 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.

## 思路
想清楚思路和边界，没别的什么技巧。
要进行顺时针打印，你需要判断的有：
1. 从左往右，起始和结尾
2. 从上往下，起始和结尾
3. 从右往左，起始和结尾
4. 从下往上，起始和结尾
以上4个内容均和边界值有关，所以可以维护4个值，LBound，RBound，UBound，DBound，每次打印时都更新边界情况就好了。
再一个要关注的是循环终止条件，边界重合还是边界交错？应该是边界交错，因为重合时对应只有一行的情况，仍然需要打印。
这时候会有一个特例，最后如果只有一行或者一列，就不需要循环回来了，所以要加入一个上下边界不相同/左右边界不相同时，才走1~4的3和4两步。
## 题解
```java
import java.util.ArrayList;
public class Solution {
    public ArrayList<Integer> printMatrix(int [][] matrix) {
        int upBoard = 0;
        int downBoard = matrix.length - 1;
        int rightBoard = matrix[0].length - 1;
        int leftBoard = 0;
        ArrayList<Integer> ans = new ArrayList<Integer>();
        
        // 使用等号，避免最后有单行单列的情况直接退出，从而导致缺少内容
        while(upBoard <= downBoard && leftBoard <= rightBoard){
            //操作四个方向为一循环，然后调整边界
            for(int col = leftBoard; col <= rightBoard; col++){
                ans.add(matrix[upBoard][col]);
            }
            // 边界上，如果只有一行row>downBoard，不会进入这列打印
            for(int row = upBoard + 1; row <= downBoard; row++){
                ans.add(matrix[row][rightBoard]);
            }
            // 不是单行的时候才顺时针从上到下输出
            if(upBoard < downBoard){
                for(int col = rightBoard - 1; col >= leftBoard; col--){
                    ans.add(matrix[downBoard][col]);
                }
            }
            // 不是单列的时候才顺时针从右到左输出
            if(leftBoard < rightBoard){
                for(int row = downBoard - 1; row >= upBoard + 1; row--){
                    ans.add(matrix[row][leftBoard]);
                }
            }
            // 修改边界
            upBoard++;
            downBoard--;
            rightBoard--;
            leftBoard++;
        }
        return ans;
    }
}
```

# JZ20 包含min函数的栈
定义栈的数据结构，包含正常的push，pop和top函数，并请在该类型中实现一个能够得到栈中所含最小元素的min函数（时间复杂度应为O（1））。
## 思路
这里的思路实际上看push，pop和top函数，应该都没啥问题，对于原生栈还是那一套的内容，但是对于min这个函数，就需要额外添加内容了。
为什么呢？如果你每次只是简单保存最小值信息，那么在最小值pop后，这个后续就连不上了，所以很自然的就想到要有一个辅助栈来保存min信息。
那么这个时候其他函数就不一样了，你现在有原生栈和辅助栈，原生栈是原来的入出的元素，辅助栈需要维护最小值。
可以模拟流程：
|操作|原生栈|辅助栈|最小值
|压入3|3|3|3
|压入4|3,4|3,3|
|压入2|3,4,2|3,3,2|2
|压入1|3,4,2,1|3,3,2,1|1
|弹出|3,4,2|3,3,2|2
|弹出|3,4|3,3|3
|弹出|3|3|3
|压入0|3,0|3,0|0
这样实际上逻辑就是：
每次压入，都入原生栈；同时判断这个值和辅助栈顶，把小的入辅助栈，这样维持了一个原生栈和最小栈
每次弹出，把原生栈弹出，辅助栈也弹出（如果弹出的是小值，辅助栈弹出的也是这个小值；如果是大值，辅助栈弹出的是重复的这个大值前的小值）
每次求min，就是找辅助栈的peek
求top，就是求原生栈的peek。

注意，涉及栈，判断st.isEmpty()，否则直接操作会出错。
## 题解
public class Solution {

    Stack<Integer> stackTotal = new Stack<Integer>();
    Stack<Integer> stackMin = new Stack<Integer>();
    
    public void push(int node) {
        stackTotal.push(node);
        if(stackMin.empty()){
            stackMin.push(node);
        }else{
            if(stackMin.peek() < node){
                stackMin.push(stackMin.peek());
            }else{
                stackMin.push(node);
            }
        }
    }
    
    public void pop() {
        stackTotal.pop();
        stackMin.pop();
    }
    
    public int top() {
        return stackTotal.peek();
    }
    
    public int min() {
        return stackMin.peek();
    }
}

# JZ21 栈的压入、弹出序列
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否可能为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列1,2,3,4,5是某栈的压入顺序，序列4,5,3,2,1是该压栈序列对应的一个弹出序列，但4,3,5,1,2就不可能是该压栈序列的弹出序列。（注意：这两个序列的长度是相等的）
##思路
模拟一下，会发现每次和popA不同的时候，全部入栈就好了，关键在和popA前面值相同是怎么办？
相同，那么就等于这个值push进去然后pop出来，这环对了。那么后面呢？
1. 后面的值跟栈顶一样，那么一直pop直到空
2. 后面的值跟栈顶不同，那么和之前不同的判断一样，持续入栈，即下一个循环
注意，栈题判断是否栈为空。
## 题解
import java.util.*;

public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
        if(pushA.length == 0 || popA.length == 0 || popA.length!=pushA.length) return false;
        Stack<Integer> st = new Stack<Integer>();
        int index1 = 0, index2 = 0;
        while(index1 < pushA.length){
            if(pushA[index1] != popA[index2]){
                st.push(pushA[index1++]);
            }else{
                index1++;
                index2++;
                // 如果栈顶是后续的值，那就pop到空，否则的话，不做啥事等下一个循环push
                while(!st.isEmpty() && st.peek() == popA[index2]){
                    st.pop();
                    index2++;
                }
            }
        }
        return st.isEmpty();
    }
}

# JZ22 从上往下打印二叉树
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
##思路
其实就是层次遍历，层次遍历比较特殊，可以直接用队列来做。
## 题解
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> ans = new ArrayList<Integer>();
        if(root == null){
            return ans;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        TreeNode current = root;
        queue.offer(current);
        while(!queue.isEmpty()){
            current = queue.poll();
            ans.add(current.val);
            if(current.left != null){
                queue.offer(current.left);
            }
            if(current.right != null){
                queue.offer(current.right);
            }
        }
        
        return ans;
    }
}

# JZ23 二叉搜索树的后序遍历序列
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
##思路
后序和前序的特点在于：他的root一个在后，一个在前。而二叉搜索树的特性就是左边小于root，右边大于root。这样就能把两边分开，然后继续进行递归了。
注意，数组题要先额外判断**数组为空null或者数组为[]，即length==0**，一共两个判断。
所以遍历相关的题实际上都大同小异，先找根节点，然后根据根节点找到左右子树，做操作，然后递归左右子树。
## 题解
```java
public class Solution {
    public boolean VerifySquenceOfBST(int [] sequence) {
        if(sequence == null || sequence.length <=0) return false;
        return helper(sequence, 0 , sequence.length - 1);
    }
    public boolean helper(int[] sequence, int left, int right){
        if(left >= right) return true;
        int root = sequence[right];
        int i = left;
        for(; i < right; i++){
            if(sequence[i] > root) break;
        }
        for(int j = i; j < right; j++){
            if(sequence[j] < root) return false;
        }
        boolean leftResult = helper(sequence, left, i - 1);
        boolean rightResult = helper(sequence, i, right - 1);
        return (leftResult && rightResult);
    }
}
```

# JZ55 链表中环的入口
给一个链表，若其中包含环，请找出该链表的环的入口结点，否则，输出null。
## 思路
环形链表问题，使用相遇点和入口相关知识。
注意找相遇点的while循环退出循环的条件pFast != null && pFast.next != null，因为Fast走得快，所以对fast进行判断，正常来说会一直在走，可能出现问题的就是他不是坏，fast直接走出去了结束了。fast是走两步，所以要把fast和fast.next都判断一下是不是null，是null的话就表明已经走到结尾了没有环。
链表问题，注意节点==null和节点.next==null这两个特殊用例，此外注意整个链不符合题意（如本题的没有环）
## 题解
```java
public class Solution {
    public ListNode EntryNodeOfLoop(ListNode pHead)
    {
        if(pHead == null || pHead.next == null) return null;
        ListNode pSlow = pHead;
        ListNode pFast = pHead;
        while(pFast != null && pFast.next != null){
            pFast = pFast.next.next;
            pSlow = pSlow.next;
            if(pFast == pSlow) break;
        }
        pSlow = pHead;
        while(pSlow != pFast){
            pFast = pFast.next;
            pSlow = pSlow.next;
        }
        return pSlow;
    }
}
```

# JZ58 对称的二叉树
请实现一个函数，用来判断一棵二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
## 思路
这个题画一下图就能解释，实际上就是对于root而言，判断它的左右子树是不是镜像的就好了，所以可以看到递归的起始位置是root.left和root.right，判断时也是对两个节点判断的，而不是一个节点。
注意树节点为null的情况。
注意if判断的先后顺序对结果的影响，
## 题解
```java
public class Solution {
    boolean isSymmetrical(TreeNode pRoot)
    {
        // 特殊测试用例
        if(pRoot == null) return true;
        return check(pRoot.left, pRoot.right);
    }
    
    public static boolean check(TreeNode pLeft, TreeNode pRight){
		// 两个都是空，说明走到结尾了，也算是对称
        if(pLeft == null && pRight == null) return true;
		// 注意这里if的判断顺序，先判断全空，然后判断各自是否为空，倒一下就没有这种效果了
        if(pLeft == null || pRight == null || pLeft.val != pRight.val) return false;
        return check(pLeft.left, pRight.right) && check(pLeft.right, pRight.left);
    }
}
```

# JZ59 按之字形打印二叉树
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
## 思路
这边也是层次遍历，只不过在层次中要判断是不是要翻转打印顺序，这里可以采用一个标志位和Collections.reverse(list);函数，把顺序修改成反序。
## 题解
public class Solution {
    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> ans = new ArrayList<ArrayList<Integer>>();
        if(pRoot == null) return ans;
        LinkedList<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(pRoot);
        boolean flag = true;
        while(!queue.isEmpty()){
            ArrayList<Integer> list = new ArrayList<Integer>();
            int size = queue.size();
            for(int i = 0; i < size; i++){
                TreeNode temp = queue.poll();
                list.add(temp.val);
                if(temp.left != null) queue.offer(temp.left);
                if(temp.right != null) queue.offer(temp.right);
            }
            if(!flag) Collections.reverse(list);
            flag = !flag;
            ans.add(list);
        }
        return ans;
    }
}

# JZ60 把二叉树打印成多行
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。
## 思路
也是典型的层次遍历，用队列实现。不同是每次要输出一行就好了，那么实际上就可以用到每次while中的size变量，i=0~size，poll完了，把这个list加入到结果中就好了。
树问题，注意树==null的特例。
## 题解
public class Solution {
    ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer> > ans = new ArrayList<>();
        if(pRoot == null) return ans;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(pRoot);
        while(!queue.isEmpty()){
            int size = queue.size();
            ArrayList<Integer> list = new ArrayList<>();
            for(int i = 0; i < size; i++){
                TreeNode curr = queue.poll();
                list.add(curr.val);
                if(curr.left != null) queue.offer(curr.left);
                if(curr.right != null) queue.offer(curr.right);
            }
            ans.add(list);
        }
        return ans;
    }
}

# JZ61 序列化二叉树
请实现两个函数，分别用来序列化和反序列化二叉树

二叉树的序列化是指：把一棵二叉树按照某种遍历方式的结果以某种格式保存为字符串，从而使得内存中建立起来的二叉树可以持久保存。序列化可以基于先序、中序、后序、层序的二叉树遍历方式来进行修改，序列化的结果是一个字符串，序列化时通过 某种符号表示空节点（#），以 ！ 表示一个结点值的结束（value!）。

二叉树的反序列化是指：根据某种遍历顺序得到的序列化字符串结果str，重构二叉树。
##思路
首先要挑选一个顺序，比如先序遍历。套用遍历模板
```
void pre_order(TreeNode *root) {
    if (!root) {
        return;
    }

    // process root
    // ...
    pre_order(root->left);
    pre_order(root->right);
}
```
以先序遍历为例：
那么在process中，要做什么？做序列化成串，遇到nullptr节点，str += "#"；遇到非空节点，str += "val" + "!"。可以更直接点直接返回串，串里边做递归。

反序列化上，要拿遍历的特性，比如先序的话，头一个内容是root节点，然后递归后面的字符串就好了。
这里引入了一个index来做头的判定，在递归中先把左侧全部递归完，这样轮到右侧的时候，index已经到右边的头部了。
## 题解
1.先序遍历
```
public class Solution {
    StringBuilder ans = new StringBuilder();
    public String Serialize(TreeNode root) {
        if(root == null){
            return "#";
        }else{
            return root.val + "!" +Serialize(root.left) + "!" + Serialize(root.right);
        }
    }
    
    int index = -1;
    public TreeNode Deserialize(String str) {
        index++;
        if(index >= str.length()) return null;
        String[] s = str.split("!");
        if(!(s[index].equals("#"))){
            TreeNode node = new TreeNode(Integer.valueOf(s[index]));
            node.left = Deserialize(str);
            node.right = Deserialize(str);
            return node;
        }else{
            return null;
        }
    }
}
```
2.层次遍历
层次遍历相对来说比较难一点，因为你的基准index不存在了。
序列化没有问题，是正常的使用队列。
反序列化里，怎么按照顺序重构呢？
实际上是一样的，把节点入队。每次把节点poll出来的同时，通过index把后面的两个节点连接上，然后把节点入队。把需要插入左右节点的元素放在队列中管理，一旦左右子树建立之后就从队列中移除
```
import java.util.*;
public class Solution {
    String Serialize(TreeNode root) {
        StringBuilder result = new StringBuilder("");
        if(root != null){
            //先访问根节点
            result.append(String.valueOf(root.val));
            //offer poll peek
            Queue<TreeNode> queue = new LinkedList<TreeNode>();
            queue.offer(root);//需要根据队列中存储的元素来作为访问下一个节点的依据。
            while(!queue.isEmpty()){
                //把队列元素的左右子树进行访问，然后加入到队列中。
                TreeNode head = queue.poll();
                if(head.left!= null || head.right!= null){
                    if(head.left == null)
                        result.append(",#");
                    else{
                        result.append("," + String.valueOf(head.left.val));
                        queue.offer(head.left);
                    }
 
                    if(head.right == null)
                        result.append(",#");
                    else{
                        result.append("," + String.valueOf(head.right.val));
                        queue.offer(head.right);
                    }
                }else{
                    //该节点为叶子节点，不需要进行任何处理。
                }
            }
        }
        return result.toString();
  }
    TreeNode Deserialize(String str) {
        TreeNode root = null;
        if(!str.equals("")){
            String[] list = str.split(",");
               //先建立根节点，是index=0处。
            root = new TreeNode(Integer.parseInt(list[0]));
            Queue<TreeNode> queue = new LinkedList<TreeNode>();
            queue.offer(root);//把需要插入左右节点的元素放在队列中管理，一旦左右子树建立之后就从队列中移除
            int index = 1;
            while(!queue.isEmpty() && index < list.length){
                    TreeNode temp = queue.poll();
					// 下面两个if，不管怎么样index必然+2，所以可以保证每个节点poll出来，他的左右子节点都被判断到了。
                    if(!list[index].equals("#")){
                        TreeNode newLeft = new TreeNode(Integer.parseInt(list[index]));
                        temp.left = newLeft;
                        index ++;
                        queue.offer(newLeft);
                    }else
						// 如果是# 那么也就是说是空的，这个空的就不用入队了，下次出队判断的时候也不会有判断他子节点的过程
                        index ++;//所有的if都应该配备else，不然真的很容易出错误
 
                    if(index < list.length && !list[index].equals("#")){
                        TreeNode newRight = new TreeNode(Integer.parseInt(list[index]));
                        temp.right = newRight;
                        index ++;
                        queue.offer(newRight);
                    }else
                        index++;//所有的if都应该配备else，不然真的很容易出错误
                }
            }
        return root;
    }
}
```

# JZ63 数据流中的中位数
如何得到一个数据流中的中位数？如果从数据流中读出奇数个数值，那么中位数就是所有数值排序之后位于中间的数值。如果从数据流中读出偶数个数值，那么中位数就是所有数值排序之后中间两个数的平均值。我们使用Insert()方法读取数据流，使用GetMedian()方法获取当前读取数据的中位数。
## 思路

## 题解

# JZ66 机器人的运动范围
地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？
## 思路
要做两个内容：
1. 从(0,0)开始，如果和超过，那么返回0,；否则的话往邻居遍历，把路径加上。
2. 判断各位和是否小于threshold
3. 这种类型的二维数组，要做的一个关键的事是记录是否访问过，这样可以避免重复访问。
4. 还可以优化，把二维数组改成一维的，索引是row * cols + col。
## 题解
```
public class Solution {
    public int movingCount(int threshold, int rows, int cols) {
        boolean[] visited = new boolean[rows*cols];
        return movingCountCore(threshold,rows,cols,0,0,visited);
    }
    private int movingCountCore(int threshold,int rows,int cols,int row,int col,boolean[] visited) {
        if(row<0||row>=rows||col<0||col>=cols)
            return 0;
        int i = row * cols + col;
        if(visited[i] || !checkSum(threshold,row,col))
            return 0;
        visited[i] = true;
		// 因为每次走过必然不会再走，所以直接加上四边能走到的格子数，所以没问题的。
        return 1 + movingCountCore(threshold,rows,cols,row-1,col,visited)
                 + movingCountCore(threshold,rows,cols,row+1,col,visited)
                 + movingCountCore(threshold,rows,cols,row,col-1,visited)
                 + movingCountCore(threshold,rows,cols,row,col+1,visited);
    }
    private boolean checkSum(int threshold,int row,int col) {
        int sum = 0;
        while(row != 0) {
            sum += row%10;
            row = row/10;
        }
        while(col != 0) {
            sum += col%10;
            col = col /10;
        }
        if(sum > threshold) return false;
        else return true;
    }
}
```

# JZ67 剪绳子
给你一根长度为n的绳子，请把绳子剪成整数长的m段（m、n都是整数，n>1并且m>1，m<=n），每段绳子的长度记为k[1],...,k[m]。请问k[1]x...xk[m]可能的最大乘积是多少？例如，当绳子的长度是8时，我们把它剪成长度分别为2、3、3的三段，此时得到的最大乘积是18。
## 思路
dp，但是要注意不要把一些特殊情况忘记了。
题目大意是说，绳子剪成2段以上，求乘积最大。
dp[n] = max(dp[i]*dp[n-i])，这是个最常看到的，但是要注意：
1. i=1~n-1,这里确实枚举了大部分，但是少了i和n-i都是一段的特殊情况
2. i和n-i其中有一个是一段的特殊情况
## 题解
```
import java.util.*;
public class Main {
    public static void main(String args[]){
        Scanner sc = new Scanner(System.in);
        int num = sc.nextInt();
        int out = cutRope(num);

    }
    public static int cutRope(int target) {
        int[] dp = new int[target + 1];
        dp[1] = 0;
        dp[2] = 1;
        dp[3] = 2;
        for(int i = 4; i <= target; i++){
            int max = 0;
            for(int j = 1; j <= (i - 1); j++){
                max = Math.max(max, Math.max((j * dp[i - j]), (j * (i - j))));
            }
            dp[i] = max;
        }
        System.out.println(dp[target]);
        return dp[target];
    }
}
```