---
title: 【Go语言例题】迷宫的广度优先搜索
toc: true
date: 2022-01-27 00:23:57
tags: golang 宽度优先 开发语言
categories: Golang
---
​​点击阅读更多查看文章内容<!--more-->

# 【Go语言例题】迷宫的广度优先搜索

- 用循环创建二维slice
- 使用slice来实现队列
- 用Fscanf读取文件
- 对Point的抽象
 >Fscanf在遇到\n才结束
遇到\r时就会把\r替换成0
这就有个问题，要注意自己的文本换行符是什么，在Windows下就是\r\n，在Linux,Mac下就是\n，也就是说这里有个坑，
代码在Linux和Mac下读取数据文件是正常的，在Windows下就会遇到各种行末尾有个0，
可以使用自带IDE将打开的数据文件集换行符改成LF（Linux,Mac换行符）即可

>广度优先搜索可以求得最短路径，深度优先搜索无法求得

>**例题：**
只能上下左右前进，1的位置是墙不能走，起点为左上角，终点为右下角
文件内容
>6 5
0 1 0 0 0
0 0 0 1 0
0 1 0 1 0
1 1 1 0 0
0 1 0 0 1
0 1 0 0 0

**示例代码**
```go
package main

import (
	"fmt"
	"os"
)

func readMaze(filename string) [][]int {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}
	var row, col int
	fmt.Fscanf(file, "%d %d", &row, &col)

	maze := make([][]int, row)
	for i := range maze {
		maze[i] = make([]int, col)
		for j := range maze[i] {
			fmt.Fscanf(file, "%d", &maze[i][j])
		}
	}
	return maze
}

type point struct {
	i, j int
}

var dirs = [4]point{
	{-1, 0}, {0, -1}, {1, 0}, {0, 1},
}

func (p point) add(r point) point {
	return point{p.i + r.i, p.j + r.j}
}

func (p point) at(grid [][]int) (int, bool) {
	if p.i < 0 || p.i >= len(grid) {
		return 0, false
	}
	if p.j < 0 || p.j >= len(grid[p.i]) {
		return 0, false
	}
	return grid[p.i][p.j], true
}

func walk(maze [][]int, start, end point) [][]int {
	//初始化steps数组，记录步数
	steps := make([][]int, len(maze))
	for i := range steps {
		steps[i] = make([]int, len(maze[0]))
	}
	Q := []point{start}
	for len(Q) > 0 {
		cur := Q[0]
		Q = Q[1:]
		//到达终点，退出循环
		if cur == end {
			break
		}
		for _, dir := range dirs {
			next := cur.add(dir)
			//next不是墙，maze为0
			val, ok := next.at(maze)
			if !ok || val == 1 {
				continue
			}
			//next没有走过，steps为0
			val, ok = next.at(steps)
			if !ok || val != 0 {
				continue
			}
			//next不是起点，next不等于start
			if next == start {
				continue
			}
			//步数加一
			curSteps, _ := cur.at(steps)
			steps[next.i][next.j] = curSteps + 1
			//添加到队列
			Q = append(Q, next)
		}
	}

	////打印路径
	//cur := end
	//for a, _ := cur.at(steps); a != 0; {
	//	fmt.Printf("(%d,%d)->", cur.i, cur.j)
	//	for _, dir := range dirs {
	//		next := cur.add(dir)
	//		if s, _ := next.at(steps); s == a-1 {
	//			cur = next
	//			break
	//		}
	//	}
	//	a, _ = cur.at(steps)
	//}
	//fmt.Printf("(%d,%d)\n", start.i, start.j)
	return steps
}
func main() {
	maze := readMaze("maze/maze.in")
	steps := walk(maze, point{0, 0}, point{len(maze) - 1, len(maze[0]) - 1})
	for _, row := range steps {
		for _, val := range row {
			fmt.Printf("%3d", val)
		}
		fmt.Println()
	}
}

```


