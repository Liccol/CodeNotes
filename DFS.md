# DFS 深度优先搜索

DFS的概念实际上是往下走或者往边上走，走到低，然后回来走上一个分叉口继续走到底。
DFS模板：
1. 最简单的（很多都是从上往下的，利用父问题的值来计算子问题，跟2.不同）
1.1 当前要处理的内容，一般会在dfs函数一开始额外加一个直接return的内容
1.2 然后dfs其他位置
2. 从下往上做DFS，当前递归层利用子问题的值（通常要min或者max取出子问题集的极值）操作并返回，一定会有返回值！这种一般都是和递归一起的，这种最好是找一个中间状态，然后根据中间状态来写递归的函数
2.1 base case 一般是null等特例
2.2 从子问题要答案
2.3 利用答案构建当前答案
2.4 额外操作
2.5 返回答案给父问题，一般就是return值，这样父问题调用能直接得到结果


// 岛屿数量
给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。此外，你可以假设该网格的四条边均被水包围。
示例 1:

输入:
[
['1','1','1','1','0'],
['1','1','0','1','0'],
['1','1','0','0','0'],
['0','0','0','0','0']
]
输出: 1
示例 2:

输入:
[
['1','1','0','0','0'],
['1','1','0','0','0'],
['0','0','1','0','0'],
['0','0','0','1','1']
]
输出: 3
解释: 每座岛屿只能由水平和/或竖直方向上相邻的陆地连接而成。

接下来三题都是十分典型的网格搜索问题，需要用到DFS或者BFS。
将二维网格看成一个无向图，竖直或水平相邻的1之间有边相连。
为了求出岛屿的数量，我们可以扫描整个网格。如果一个位置为1，则以其为起始节点开始进行深度优先搜索。在深度优先搜索的过程中，每个搜索到的1都会被重新标记为0。最终岛屿的数量就是我们进行深度优先搜索的次数。
为什么这样会对？每当找到一个1，我们通过DFS尽可能的找到他最终的边界，找到之后这就是一个完整的岛了，已经找到过了，所以置0，这样下一次寻找的时候就不会重复。

class Solution {
    void dfs(char[][] grid, int r, int c) {
        int nr = grid.length;
        int nc = grid[0].length;
        // 如果已经超过了边界,或者当前是0不是1,那么就可以直接返回,即这个DFS结束
        if (r < 0 || c < 0 || r >= nr || c >= nc || grid[r][c] == '0') {
            return;
        }
        // 向四个方向进行DFS
        grid[r][c] = '0';
        dfs(grid, r - 1, c);
        dfs(grid, r + 1, c);
        dfs(grid, r, c - 1);
        dfs(grid, r, c + 1);
    }

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) {
            return 0;
        }

        int nr = grid.length;
        int nc = grid[0].length;
        int num_islands = 0;
        for (int r = 0; r < nr; ++r) {
            for (int c = 0; c < nc; ++c) {
                if (grid[r][c] == '1') {
                    ++num_islands;
                    dfs(grid, r, c);
                }
            }
        }
        return num_islands;
    }
}

// 岛屿的周长
给定一个包含 0 和 1 的二维网格地图，其中 1 表示陆地 0 表示水域。网格中的格子水平和垂直方向相连（对角线方向不相连）。整个网格被水完全包围，但其中恰好有一个岛屿（或者说，一个或多个表示陆地的格子相连组成的岛屿）。岛屿中没有“湖”（“湖” 指水域在岛屿内部且不和岛屿周围的水相连）。格子是边长为 1 的正方形。网格为长方形，且宽度和高度均不超过 100 。计算这个岛屿的周长。

基本的网格DFS框架：每次搜索四个相邻方格，二维的应该也是类似的，每次搜索左右内容。
// 标记已遍历过的岛屿，不做重复遍历
void dfs(int[][] grid, int r, int c) {
    // 判断是否在上下边界内
	if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return;
    }
    // 已遍历过（值为2）的岛屿在这里会直接返回，避免重复遍历(如果不这么操作，在边上网格会遍历到当前网格，进入死循环)
    if (grid[r][c] != 1) {
        return;
    }
    grid[r][c] = 2; // 将方格标记为"已遍历"
    dfs(grid, r - 1, c); 
    dfs(grid, r + 1, c);
    dfs(grid, r, c - 1);
    dfs(grid, r, c + 1);
}

那么本题的关键就在于，怎么对周长进行计算？
有一种很简单的思路：岛屿的周长就是岛屿方格和非岛屿方格相邻的边的数量。注意，这里的非岛屿方格，既包括水域方格，也包括网格的边界。
public int islandPerimeter(int[][] grid) {
    for (int r = 0; r < grid.length; r++) {
        for (int c = 0; c < grid[0].length; c++) {
            if (grid[r][c] == 1) {
                // 题目限制只有一个岛屿，计算一个即可
                return dfs(grid, r, c);
            }
        }
    }
    return 0;
}

int dfs(int[][] grid, int r, int c) {
	// 到边界时，会有一个碰撞，所以返回一个长度
    if (!(0 <= r && r < grid.length && 0 <= c && c < grid[0].length)) {
        return 1;
    }
	// 和0碰撞，会有一个长度出现
    if (grid[r][c] == 0) {
        return 1;
    }
	// 已遍历过（值为2）的岛屿在这里会直接返回，避免重复遍历，因为grid只有0 1 2 三种选择。
    if (grid[r][c] != 1) {
        return 0;
    }
    grid[r][c] = 2;
    return dfs(grid, r - 1, c)
        + dfs(grid, r + 1, c)
        + dfs(grid, r, c - 1)
        + dfs(grid, r, c + 1);
}

// 岛屿的最大面积
给定一个包含了一些 0 和 1 的非空二维数组 grid 。一个 岛屿 是由一些相邻的 1 (代表土地) 构成的组合，这里的「相邻」要求两个 1 必须在水平或者竖直方向上相邻。你可以假设 grid 的四个边缘都被 0（代表水）包围着。找到给定的二维数组中最大的岛屿面积。(如果没有岛屿，则返回面积为0)
示例 1:
[[0,0,1,0,0,0,0,1,0,0,0,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,1,1,0,1,0,0,0,0,0,0,0,0],
 [0,1,0,0,1,1,0,0,1,0,1,0,0],
 [0,1,0,0,1,1,0,0,1,1,1,0,0],
 [0,0,0,0,0,0,0,0,0,0,1,0,0],
 [0,0,0,0,0,0,0,1,1,1,0,0,0],
 [0,0,0,0,0,0,0,1,1,0,0,0,0]]
对于上面这个给定矩阵应返回 6。注意答案不应该是 11 ，因为岛屿只能包含水平或垂直的四个方向的 1 。

class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        int ans = 0;
        for(int rol = 0; rol < grid.length; rol++){
            for(int col = 0; col < grid[0].length; col++){
                if(grid[rol][col] == 1){
                    ans = Math.max(dfs(grid, rol, col), ans);
                }
            }
        }
        return ans;
    }

    public int dfs(int[][] grid, int rol, int col){
        // 到达边界 或者 已经遍历到了变成0了
        if(rol < 0 || col < 0 || rol >= grid.length || col >= grid[0].length || grid[rol][col] == 0) return 0;
        grid[rol][col] = 0;
        return 1 + dfs(grid, rol - 1, col) +
                dfs(grid, rol, col - 1) +
                dfs(grid, rol + 1, col) +
                dfs(grid, rol, col + 1);

    }
}

// 孤独像素||
给定一幅由黑色像素和白色像素组成的图像， 与一个正整数N, 找到位于某行 R 和某列 C 中且符合下列规则的黑色像素的数量:

行R 和列C都恰好包括N个黑色像素。
列C中所有黑色像素所在的行必须和行R完全相同。
图像由一个由‘B’和‘W’组成二维字符数组表示, ‘B’和‘W’分别代表黑色像素和白色像素。

示例:

输入:                                            
[['W', 'B', 'W', 'B', 'B', 'W'],    
 ['W', 'B', 'W', 'B', 'B', 'W'],    
 ['W', 'B', 'W', 'B', 'B', 'W'],    
 ['W', 'W', 'B', 'W', 'B', 'W']] 

N = 3
输出: 6

这道题实际上和dfs有一定关系，思路是这样的：
1. 判断是不是B，是的话查看列行是不是有N个B
2. 都有N个的话，判断B出现的行是不是相同，涉及到判断字符数字是否相同的方法
暴力解确实能做，但是有几个可以优化的地方：
1. 判断行列是不是有N个B-->这个方面实际上不需要通过dfs这样把每列都查下去，可以用空间换时间，一开始就建立两个数组，来标定数组i行和j列各自都有几个N，这样只需要在后面做一个判断就好了
2. 在1.的基础上，在查找时，先判断行有没有N个，有才继续查列，列对了，才查B所在行是不是一样
3. （看来的）**对于字符数组相同与否的判定**，一个比较可行的避免一个个判断的方法是把char连接成string，这里比较简单，因为是二维数组，把行数据直接String curr = new String(picture[i])就好了，实际上也同理，如果是char[] s = {'a','2'}这种，也可以用String S = String.valueOf(s);来实现转换。
4.（看来的）继续优化判断，哪里出现了大量重复动作？先判断有没有N，然后判断是不是一样。-->这个其实可以用哈希去优先判断行是否相等，Map<String,Integer> stringMap:将每一行char数组组成一个String，记录每一种String出现的次数（不等于N的肯定就不是目标行了，这样可以跳过判断。）
5. （继续优化）既然你能判断哪些行出现了N次，那么其实出现了N次不一定就一定对，你需要B的也对，所以你可以再多一个数二维数组来保存第i行的第一个B的列号，加快查找：int[][] colsRowIndex =new int[rows][N+1]：记录每一列出现X个B的行号。第0位不存值。

class Solution {
    public int findBlackPixel(char[][] picture, int N) {
        int row = picture.length;
        int col = picture[0].length;
        int rowNum[] = new int[row];
        int colNum[] = new int[col];
        int colsRowIndex[][] =new int[row][N+1];
        HashMap<String,Integer> stringMap = new HashMap<>();
        int count = 0;

        for(int i = 0; i < picture.length; i++){
            String str = new String(picture[i]);
            stringMap.put(str, stringMap.getOrDefault(str, 0) + 1);
            for(int j = 0; j < picture[0].length; j++){
                if(picture[i][j] == 'B'){
                    rowNum[i]++;
                    colNum[j]++;
                    // 这个赋值很灵性
                    // 如果目前还没有超过N个，那么把colsRowIndex赋值
                    // 怎么赋值?行号肯定是i，列指的是第几个B出现的列号，而此时rowNum[i]的值
                    // 刚好是第几个出现的位置，所以colsRowIndex[i][0]=0，内容从1列开始
                    if(rowNum[i] <= N) colsRowIndex[i][rowNum[i]] = j;
                }
            }
        }
        for(int i = 0; i < rowNum.length; i++){
            if(rowNum[i] == N){
                String curStr = new String(picture[i]);
                // 只有出现了N次，那么就满足行有N个B，有N个相同行，那么只需要判断
                // B出现的列是不是也有N个就好了，如果有那就是一个孤独像素
                if(stringMap.get(curStr) == N){
                    // 把B的列号信息拿出来
                    int indexCol[] = colsRowIndex[i];
                    // 因为colsRowIndex的0位置是0，所以直接从0开始遍历就好了
                    for(int j=1; j<indexCol.length; j++){
                        if(colNum[indexCol[j]]==N)
                        {
                            count++;
                        }
                    }
                }else continue;
            }else continue;
        }
        return count;
    }
    public boolean checkSame(char[] S, char[] R){
        if(S.length != R.length) return false;
        for(int i = 0; i < R.length; i++){
            if(S[i] != R[i]) return false;
        }
        return true;
    }
}

// 二叉树中的最大路径和
给定一个非空二叉树，返回其最大路径和。

本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径至少包含一个节点，且不一定经过根节点。

示例 1:

输入: [1,2,3]

       1
      / \
     2   3

输出: 6

注意理解这个题目，他指的路径不是指从root开始的，而是可以人字形的一个路径。
这道题实际上就是一个BFS，对于中间状态的某个root，要考虑这么几个内容：
1. 路径是人字形的，这个人字形（左+root+右）是不是新的最大路径？
2. 左孩子和右孩子的路径最大长度怎么得到，通过递归dfs每次返回到当前节点的路径最大值
3. 特殊条件，如果路径值是负数，那么实际上当前节点的返回值不应该把子树加进去，接直接以当前root为第一个路径，返回root.val的值就好了。
class Solution {
    int max = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        dfs(root);
        return max;
    }

    public int dfs(TreeNode root){
        if(root == null) return 0;
        int left = dfs(root.left);
        int right = dfs(root.right);
        left = left < 0 ? 0 :left;
        right = right < 0 ? 0 : right;
        max = Math.max(max, left + right + root.val);
        return Math.max(left + root.val, right + root.val);
    }
}