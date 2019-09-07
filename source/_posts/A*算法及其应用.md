---
title: A*算法及其应用
tags:
  - A*
categories:
  - algorithm
mathjax: true
abbrlink: ef3eeb15
date: 2018-10-07 22:03:28
---

{% note default %}

最近在淘宝店铺接到了一个关于A*算法的assignment，借此机会学习了一下并记录一下，以这个assignment作为例子

{% endnote %}

<!-- more -->

### Heuristic Search（启发式搜索）

启发式搜索就是会有目的地搜索，一般通过一个启发函数来进行选择，选择代价最少的结点作为下一步搜索结点。`DFS`和`BFS`都属于盲目型搜索，不会在下一步搜索时选择最优的结点进行跳转，存在需要试探整个解集空间的可能，只能适用于问题规模不大的搜索问题中

> 而与`DFS`和`BFS`不同的是，一个经过仔细设计的启发函数，往往在很快的时间内就可得到一个搜索问题的最优解，对于NP问题，亦可在多项式时间内得到一个较优解。是的，关键就是如何设计这个启发函数

### A* algorithm

A-Star算法，俗称A星算法，这是一种在图形平面上，有多个节点的路径，求出最低通过成本的算法。常用于游戏中的NPC的移动计算，或网络游戏中的BOT的移动计算上。

该算法综合了Best-First Search和[Dijkstra算法](https://zh.wikipedia.org/wiki/Dijkstra%E7%AE%97%E6%B3%95)的优点：在进行启发式搜索提高算法效率的同时，可以保证找到一条最优路径（基于评估函数）。

在此算法中，如果以$g(n)$表示从起点到任意顶点$n$的实际距离，$h(n)$表示任意顶点$n$到目标顶点的估算距离（根据所采用的评估函数的不同而变化），那么A*算法的估算函数为
$$
f(n)=g(n)+h(n)
$$
### 特性

- 如果$g(n)$为0，即只计算任意顶点$n$到目标的评估函数$h(n)$，而不计算起点到顶点$n$的距离，则算法转化为使用贪心策略的`Best-First Search`，速度最快，但可能得不出最优解
- 如果$h(n)$不高于顶点$n$到目标顶点的实际距离，则一定可以求出最优解，而且$h(n)$越小，需要计算的节点越多，算法效率越低，常见的评估函数有——[欧几里得距离](https://zh.wikipedia.org/wiki/%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E8%B7%9D%E7%A6%BB)、[曼哈顿距离](https://zh.wikipedia.org/wiki/%E6%9B%BC%E5%93%88%E9%A0%93%E8%B7%9D%E9%9B%A2)、[切比雪夫距离](https://zh.wikipedia.org/wiki/%E5%88%87%E6%AF%94%E9%9B%AA%E5%A4%AB%E8%B7%9D%E7%A6%BB)
- 如果$h(n)$为0，即只需求出起点到任意顶点$n$的最短路径$g(n)$，而不计算任何评估函数$h(n)$，则转化为[单源最短路径](https://zh.wikipedia.org/w/index.php?title=%E5%8D%95%E6%BA%90%E6%9C%80%E7%9F%AD%E8%B7%AF%E5%BE%84&action=edit&redlink=1)问题，即[Dijkstra算法](https://zh.wikipedia.org/wiki/Dijkstra%E7%AE%97%E6%B3%95)，此时需要计算最多的定点

简单来解释，就是使用$g(n)$计算路径长度，$h(n)$估计到目标的距离，用$f$值维护优先队列。

### A*算法流程

首先将起始结点`S`放入`open_list`，`close_list`置空

1. 如果`open_list`不为空，从表头取一个结点`n`，如果为空算法失败
2. 判断`n`是否为目标解。是，找到一个解（继续寻找，或终止算法）
3. 对于`n`的所有后继结点（可以走到的结点），如果不在`close_list`中，就放入`open_list`，同时计算每一个后继结点的$f(n)$值，将`open_list`按$f$值从小到大排序维护优先队列，并把`n`放入`close_list`，重复算法，回到1

### 问题及代码

1. N数码问题（类似华容道）

   对于任意大小的数码问题，有一格是空的，向四个方向滑块，目标状态为0在最右下角，并且1~N-1按顺序摆放

   ```python
   import operator
   import copy
   
   def create_tile_puzzle(rows, cols):
       puz=[[i+1+cols*j if i+1+cols*j != rows*cols else 0 for i in range(cols)]for j in range(rows)]
       return TilePuzzle(puz)
   
   global visit
   
   class TilePuzzle(object):
       
       # Required
       def __init__(self, board, g=0, h=0, f=0, op=None, parent=None):
           self.board=board
           self.row=len(board)
           self.col=len(board[0])
           self.g = g
           self.h = h
           self.f = f
           self.op = op
           self.parent = parent
   
       def get_board(self):
           return self.board
                               
       def perform_move(self, direction):
           for row in self.board:
               if 0 not in row:
                   pass
               else:
                   a= self.board.index(row)
                   b= row.index(0)
           if direction == "up":
               if a-1 >= 0:
                   self.board[a][b],self.board[a-1][b] = self.board[a-1][b],self.board[a][b]
                   return True
               else:
                   return False                  
           if direction == "down":
               if a+1 < self.row:
                   self.board[a+1][b],self.board[a][b] = self.board[a][b],self.board[a+1][b]
                   return True
               else:
                   return False
           if direction == "left":
               if b-1 >= 0:
                   self.board[a][b],self.board[a][b-1] = self.board[a][b-1],self.board[a][b]
                   return True
               else:
                   return False        
           if direction == "right":
               if b+1 < self.col:
                   self.board[a][b],self.board[a][b+1] = self.board[a][b+1],self.board[a][b]
                   return True
               else:
                   return False
   
       def is_solved(self):
           if self.board[-1][-1] != 0:
               return False
           for i in range(self.row):
               for j in range(self.col):
                   if (i,j) == (self.row-1, self.col-1):
                       break
                   if self.board[i][j] != i*self.col+j+1:
                       return False
           return True
   
       def copy(self):
           new = copy.deepcopy(self.board)
           return TilePuzzle(new)
   
       def successors(self):
       '''求可能的后继节点'''
           newBoard = self.copy()
           for x in ["up","down","left","right"]:
               if newBoard.perform_move(x) == True:
                   yield (x,newBoard)
                   newBoard=self.copy()
               else:
                   pass
               
       def iddfs_helper(self, limit, moves):
       '''此为dfs解法'''
           global visit
           if limit == 0:
               yield (moves, True) if self.is_solved() else ([], False)
           for move, board in self.successors():
               if board.board not in visit:
                   moves.append(move)
                   if not board.is_solved():
                       visit.append(board.board)
                   # dfs
                   m, f = next(board.iddfs_helper(limit-1, moves))
                   if f:
                       yield (m, True)
                   # 回溯
                   moves.pop()
                   visit.pop()
           yield (moves, True) if self.is_solved() else ([], False)
               
                                  
       # Required
       def find_solutions_iddfs(self):
       '''此为dfs解法'''
           limit = 0
           f = False
           global visit 
           visit = [self.board]
           while not f:
               moves, f = next(self.iddfs_helper(limit, []))
               if f:
                   while f:
                       yield moves
                       moves, f = next(self.iddfs_helper(limit, []))
                   break
               limit += 1
           return
               
       
       def print_path(self):
           path = []
           puzzle = self
           while puzzle.parent:
               path.append(puzzle.op)
               puzzle = puzzle.parent
           return list(reversed(path)) if path else None
       
       def heuristic_cost_estimate(self):
           h = 0
           for i in range(self.row):
               for j in range(self.col):
                   if self.board[i][j] == 0:
                       h += self.row-i+self.col-j-2
                       continue
                   h += abs((self.board[i][j]-1) // self.col - i) + abs((self.board[i][j]-1) % self.col - j)
           return h    
   
       # Required
       def find_solution_a_star(self):
           open_list = [self]
           close_list = []
           while open_list:
               puzzle = open_list.pop(0)
               if puzzle.is_solved():
                   return puzzle.print_path()
               for move, p in list(puzzle.successors()):  
                   if p.board not in [_.board for _ in close_list]:
                       p.g = puzzle.g + 1
                       p.h = p.heuristic_cost_estimate()
                       p.f = p.g + p.h
                       p.parent = puzzle
                       p.op = move
                       open_list.append(p)
                   open_list.sort(key=operator.attrgetter('f'))
                   close_list.append(puzzle)
           return
   ```

2. Linear Disk Movement

   在一行长为$L$的格子中，开头摆放1-n的$n$个块，一个块有两种方式移动，第一种为向左或右移动一格，第二种为向左或向右隔着一个块跳到第二格，目标是全部摆放在最右边，并且为逆序n-1

   ```python
   import operator
   import math
   
   class Line(object):
       def __init__(self, line, g=0, h=0, op=None, parent=None):
           self.line = line
           self.g = g
           self.h = h
           self.f = g+h
           self.op = op
           self.parent = parent
       
   def cal_h(line):
       h = 0
       for i, v in enumerate(line):
           if v:
               # 估算为距离除2，因为最多跳2格
               h += math.ceil(abs((len(line)-v-i))/2.0)
       return 1+h
          
   def possible_line(line):
       # 求4种可能的跳法
       possible_list = []
       for i, v in enumerate(line.line):
           if v and i+2 < len(line.line) and line.line[i+1] and line.line[i+2] == 0:
               new_line = line.line[:]
               new_line[i] = 0
               new_line[i+2] = v
               possible_list.append(Line(new_line, g=line.g+1, h=cal_h(new_line), op=(i, i+2), parent=line))
           if v and i+1 < len(line.line) and line.line[i+1] == 0:
               new_line = line.line[:]
               new_line[i] = 0
               new_line[i+1] = v
               possible_list.append(Line(new_line, g=line.g+1, h=cal_h(new_line), op=(i, i+1), parent=line))
           if v and i-1 >= 0 and line.line[i-1] == 0:
               new_line = line.line[:]
               new_line[i] = 0
               new_line[i-1] = v
               possible_list.append(Line(new_line, g=line.g+1, h=cal_h(new_line), op=(i, i-1), parent=line))      
           if v and i-2 >= 0 and line.line[i-1] and line.line[i-2] == 0:
               new_line = line.line[:]
               new_line[i] = 0
               new_line[i-2] = v
               possible_list.append(Line(new_line, g=line.g+1, h=cal_h(new_line), op=(i, i-2), parent=line))    
       return possible_list
               
   def is_solved(line, ans):
       return True if line == ans else False
   
   def print_path(line):
       # 打印路径
       path = []
       while line.parent:
           path.append(line.op)
           line = line.parent
       return list(reversed(path))
               
   def solve_distinct_disks(length, n):
       line = [i+1 if i < n else 0 for i in range(length)]
       ans = list(reversed(line))
       open_list = [Line(line, 0)]
       close_list = []
       while open_list:
           line = open_list.pop(0)
           if is_solved(line.line, ans):
               return print_path(line)
           for l in possible_line(line):
               if l.line not in [_.line for _ in close_list]:
                   open_list.append(l)
           open_list.sort(key=operator.attrgetter('f'))
           close_list.append(line)
       return 
   ```




