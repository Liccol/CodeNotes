# BFS

广度优先搜索，实际上就是水平展开所有内容（访问每层的邻居，完成后才去下一层），适合寻找最短路径这类最短问题。

他的模板是这样的：

1. 建立一个所有源点的**队列**，建立一个存储访问过的节点的HashSet避免死循环(或者其他方式，如二维数组可以直接用一个visited[][]来记录相应位置是不是被访问过了)

2. 队列不空时：
	提取出队列大小size，作为当前循环的长度
	对于每个当前循环，把队列头内容poll
	然后做需要做的事
	把边上的内容add入队列（前提是先判断是不是可行(如未越界)并且未访问过）
	继续循环到下一层
3. 技巧：对于二维matrix，一般都是四个方向，所以可以预存一个4方向的数组

实际上就是之前的树的层次遍历，就是最简单的树形的BFS，只有一个入口需要入队进行开始。难一点的就是多源的，二维数组/图的BFS则是多源的。

思路：
-->>对于 「Tree 的 BFS」 （典型的「单源 BFS」）：
首先把 root 节点入队，再一层一层无脑遍历就行了。
-->>对于 「图 的 BFS」 （「多源 BFS」） 做法其实也是一样滴～，与 「Tree 的 BFS」的区别注意以下两条就 ok 辣～

Tree 只有 1 个 root，而图可以有多个源点，所以首先需要把多个源点都入队；

Tree 是有向的因此不需要标识是否访问过，而对于无向图来说，必须得标志是否访问过！

并且为了防止某个节点多次入队，需要在其入队之前就将其设置成已访问！（看见很多人说自己的 BFS 超时了，BFS的超时坑坑就在这里哈哈哈.

BFS2：最优优先，这和之前的广度优先不一样，优先展开最优的节点（BFS1只是单纯的每层依次展开）。使用heap（BFS1用的是队列）来辅助计算最优解。

他的模板是这样的：

1. 建立一个所有源点的**heap堆**，也要加入最初的cost一般是0，并建立一个存储访问过的节点的HashSet避免死循环(或者其他方式，如二维数组可以直接用一个visited[][]来记录相应位置是不是被访问过了)

2. 堆不空时：

	拿出一个node
	
	如果访问过，跳过
	
	否则的话标记已经访问过
	
	如果是终点，返回；如果不是，对能加入堆的邻居，cost更新=当前cost+邻居cost
	
	继续循环到下一层
	
3. 技巧：因为这个添加邻居时，可能会添加同一个node的多个cost值，因为重复更新，所以需要构建是否访问过的数据结构。




// 二叉树的右视图
给定一棵二叉树，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

示例:

输入: [1,2,3,null,5,null,4]
输出: [1, 3, 4]
解释:

   1            <---
 /   \
2     3         <---
 \     \
  5     4       <---

树的内容就是每层都把最右边的拿出来，所以意思就是，我每次从右往左把内容放进队列，把队头的内容拿出来就是最右边的内容了。所以实际上就是树的BFS。
```
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        Queue<TreeNode> queue = new LinkedList<>();
        List<Integer> ans = new ArrayList<>();
        if(root == null) return ans;
        queue.offer(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            ans.add(queue.peek().val);
            for(int i = 0; i < size; i++){
                TreeNode pre = queue.poll();
                if(pre.right != null) queue.offer(pre.right);
                if(pre.left != null) queue.offer(pre.left);
            }
        }
        return ans;
    }
}
```

// 01矩阵
给定一个由 0 和 1 组成的矩阵，找出每个元素到最近的 0 的距离。

两个相邻元素间的距离为 1 。

示例 1:
输入:

0 0 0
0 1 0
0 0 0
输出:

0 0 0
0 1 0
0 0 0

1. 动态规划

2. BFS

矩阵的BFS实际上是和模板是一样的，把0的内容放进来，然后往周围扩展每一层，并标记是否访问过。

这里可以加一个技巧，用direction的二维数组来表示四个方向，这样可以直接把坐标进行变化，因为每次都得吧二维的坐标位置放进队列嘛毕竟。
```
class Solution {
    // 用来做坐标变化
    int[][] direction = {{0, 1}, {1, 0}, {-1, 0}, {0, -1}};
    public int[][] updateMatrix(int[][] matrix) {
        int rows = matrix.length;
        int cols = matrix[0].length;
        int[][] ans = new int[rows][cols];
        boolean[][] visited = new boolean[rows][cols];
        Queue<int[]> queue = new LinkedList<>();
        for(int i = 0 ; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(matrix[i][j] == 0){
                    // 把是0的源点放进去，然后标记已访问
                    queue.offer(new int[]{i, j});
                    visited[i][j] = true;
                }
            }
        }

        int cost = 0;
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int s = 0; s < size; s++){
                // 把当前内容 当前点的横纵坐标拿出来
                int[] cur = queue.poll();
                int i = cur[0], j =cur[1];
                // 我们是从cost=0开始BFS的，那么下一个必然每次都cost+1，所以同时你要查的实际上就该是上一层周围是1的点。
                if(matrix[i][j] == 1){
                    ans[i][j] = cost;
                }

                for(int[] dir : direction){
                    //查找四邻居的坐标
                    int x = i + dir[0], y = j+ dir[1];
                    // 判断是否出界+是否访问过
                    if(x >= 0 && x < rows&& y >=0 && y < cols && !visited[x][y]){
                        queue.offer(new int[]{x, y});
                        visited[x][y] = true;
                    }
                }
            }
            cost++;
        }
        return ans;
    }
}
```

 // 网络延迟时间（理解一下！）
 有 N 个网络节点，标记为 1 到 N。给定一个列表 times，表示信号经过有向边的传递时间。 times[i] = (u, v, w)，其中 u 是源节点，v 是目标节点， w 是一个信号从源节点传递到目标节点的时间。现在，我们从某个节点 K 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 -1。
输入：times = [[2,1,1],[2,3,1],[3,4,1]], N = 4, K = 2
输出：2

这个实际上能理解成从一个节点出发，到目标节点，找到最长cost路径，可以用BFS2。
0. 构建图
这里比较重要的是，他的数据结构不是常见的，是UVW三元，那么怎么构建呢，一般可以用Map来做。

```
Map<Integer, List<int[]>> graph = new HashMap();
for (int[] edge: times) {
	if (!graph.containsKey(edge[0]))
		graph.put(edge[0], new ArrayList<int[]>());
	// 如果不存在，新建一项，存在的话，把当前的这个关系添加到List中去
	graph.get(edge[0]).add(new int[]{edge[2], edge[1]});
}

1. 使用优先队列做bfs
public int networkDelayTime(int[][] times, int N, int K) {
	// 图存储方式的构建
	Map<Integer, List<int[]>> graph = new HashMap();
	for (int[] edge: times) {
		if (!graph.containsKey(edge[0]))
			graph.put(edge[0], new ArrayList<int[]>());
		graph.get(edge[0]).add(new int[]{edge[1], edge[2]});
	}
	// 建立堆，队里保存的是int数组，包含cost和起点标号，而比较的内容是加入的节点的[0]位置，即他的成本进行比较
	PriorityQueue<int[]> heap = new PriorityQueue<int[]>((info1, info2) -> info1[0] - info2[0]);
	heap.offer(new int[]{0, K});

	Map<Integer, Integer> dist = new HashMap();

	while (!heap.isEmpty()) {
		// 每次把节点poll出来，获得他的成本d和起点K
		int[] info = heap.poll();
		int d = info[0], node = info[1];
		if (dist.containsKey(node)) continue;
		// 如果没有记录node点的最大成本值，那就加进去，有的话跳过
		dist.put(node, d);
		if (graph.containsKey(node))
			for (int[] edge: graph.get(node)) {
				int nei = edge[0], d2 = edge[1];
				if (!dist.containsKey(nei))
					heap.offer(new int[]{d+d2, nei});
			}
	}

	if (dist.size() != N) return -1;
	int ans = 0;
	for (int cand: dist.values())
		ans = Math.max(ans, cand);
	return ans;
}
```

// 腐烂的橘子

在给定的网格中，每个单元格可以有以下三个值之一：

值 0 代表空单元格；
值 1 代表新鲜橘子；
值 2 代表腐烂的橘子。
每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 -1。

这个题目比较复杂，从题目可以看到主要由几个内容：
1. 把2的点找出来，然后2边上的1都会变成2，天数+1,
2. 继续把2的点拿出来，然后找边上的，实际上就是个BFS
3. BFS结束后，怎么判断要不要-1返回？看最后是不是所有橘子都腐烂了，所以可以引入一个计数器做最终判断是否全部腐烂
4. 天数怎么定？每次把queue里的一排poll出来，就是当天查到了附近的1，所以cost+1，注意这样的话cost初始值为-1，因为第一次poll出来时，只是把原有的2全部排出去了，没有经过天数。
5. 体会下矩阵中找四邻居的坐标的方法，即dirs的设置
```
int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
public int orangesRotting(int[][] grid) {
	int rows = grid.length;
	int cols = grid[0].length;
	Queue<int[]> queue = new LinkedList<>();
	int count = -1, numCount = 0, oneCount = 0;
	boolean[][] dead = new boolean[rows][cols];
	for(int i = 0; i < grid.length; i++){
		for(int j = 0; j < grid[0].length; j++){
			// 把腐烂的位置放出去
			if(grid[i][j] == 2){
				queue.offer(new int[]{i, j});
				dead[i][j] = true;
				oneCount++;
			}
			if(grid[i][j] == 1){
				oneCount++;
			}
		}
	}
	if(oneCount == 0) return 0;
	while(!queue.isEmpty()){
		int size = queue.size();
		count++;
		for(int s = 0; s < size; s++){
			int[] cur = queue.poll();
			numCount++;
			for(int[] dir : dirs){
				int x = dir[0] + cur[0], y = dir[1] + cur[1];
				if(x >=0 && x < rows && y >= 0 && y < cols && grid[x][y] == 1 && !dead[x][y]){
					queue.offer(new int[]{x, y});
					dead[x][y] = true;
				}
			}
		}
	}
	return numCount == oneCount ? count: -1;
}
```