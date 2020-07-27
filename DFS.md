# DFS 深度优先搜索


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

