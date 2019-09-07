---
title: Minimax+Alpha-Beta pruning及其应用
tags:
  - Minimax
  - Alpha-Beta Pruning
categories:
  - algorithm
mathjax: true
abbrlink: 8fcfbd4c
date: 2018-10-07 22:03:26
---

{% note default %}

最近在淘宝做一份assignment的时候用到了Minimax+Alpha-Beta剪枝，一般使用在棋类等双方较量的游戏和程序中

{% endnote %}

<!-- more -->

### Minimax算法

极小化极大算法常用于棋类等由两方较量的游戏和程序。该算法是一个[零总和](https://zh.wikipedia.org/wiki/%E9%9B%B6%E5%92%8C%E5%8D%9A%E5%BC%88)算法，即一方要在可选的选项中选择将其优势最大化的选择，另一方则选择令对手优势最小化的方法。而开始的时候总和为0。很多棋类游戏可以采取此算法，例如[井字棋](https://zh.wikipedia.org/wiki/%E4%BA%95%E5%AD%97%E6%A3%8B)（tic-tac-toe）

1. Minimax是一种悲观算法，即假设对手每一步都会将我方引入从当前看理论上价值最小的格局方向，即对手具有完美决策能力
2. Minimax不找理论最优解，Minimax中我方完全掌握主动，如果对方每一步决策都是完美的，则我方可以达到预计的最小损失格局

### Minimax算法流程

1. 确定最大搜索深度$D$
2. 在$D$的格局树的叶子结点上，使用预定义的价值函数对叶子结点进行评价
3. 自底向上为非叶子节点赋值。其中max节点取子节点最大值，min节点取子节点最小值
4. 每次轮到我方时（此时必处在格局树的某个max节点），选择价值等于此max节点价值的那个子节点路径

### Alpha-Beta Pruning

Alpha-Beta剪枝用于裁剪搜索树中没有意义的不需要搜索的树枝，以提高运算速度

假设$\alpha$为下界，$\beta$为上界，对于$\alpha<=N<=\beta$:

- 若$\alpha<=\beta$ 则$N$有解

- 若 $\alpha>\beta$ 则$N$无解

简单来说就是不是先生成整棵树，而是先探索到第一种叶子格局，以此记录$\alpha$和$\beta$的值，以限制之后进行剪枝搜索。

1. 对手想要我方获利小，则轮到min节点要使$\beta$尽可能小
2. 我方想要获利大，则轮到max节点要使$\alpha$尽可能大
3. 若 $\alpha>\beta$ 则表明我方一定选择之前$\alpha$的那一种，不会选择接下来的搜索到的这一种最大只有$\beta$的收益，则进行剪枝

### Example

Dominoes Game，双方在二维棋盘上放1*2大小的木块，一方只能竖着放，另一方只能横着放

```python
import random
import sys


def create_dominoes_game(rows, cols):
    Total = [[False for i in range(cols)] for i in range(rows)]
    return DominoesGame(Total)


class DominoesGame(object):

    # Required
    def __init__(self, board):
        self.board = board
        self.rows = len(board)
        self.cols = len(board[0])

    def get_board(self):
        return self.board

    def reset(self):
        self.board = [[False for i in range(self.cols)] for i in range(self.rows)]

    def is_legal_move(self, row, col, vertical):
        if vertical:
            if row >= 0 and row < self.rows and col >= 0 and col < self.cols \
            and row+1 < self.rows and not self.board[row][col] and not self.board[row+1][col]:
                return True
        else:
            if row >= 0 and row < self.rows and col >= 0 and col < self.cols \
            and col+1 < self.cols and not self.board[row][col] and not self.board[row][col+1]:
                return True
        return False

    def legal_moves(self, vertical):
        for i in range(self.rows):
            for j in range(self.cols):
                if self.is_legal_move(i, j, vertical):
                    yield (i, j)

    def perform_move(self, row, col, vertical):
        if vertical:
            self.board[row][col] = self.board[row+1][col] = True
        else:
            self.board[row][col] = self.board[row][col+1] = True

    def game_over(self, vertical):
        if list(self.legal_moves(vertical)):
            return False 
        return True

    def copy(self):
        return DominoesGame([row[:] for row in self.board])

    def successors(self, vertical):
    '''当前棋盘可能的所有存放情况，生成器函数'''
        for move in self.legal_moves(vertical):
            g = self.copy()
            g.perform_move(move[0], move[1], vertical)
            yield move, g

    def get_random_move(self, vertical):
        return random.choice(list(self.legal_moves(vertical)))

    # Required
   
    def get_best_move(self, vertical, limit):
        best_move = ()
        total_num = 0
        is_max = True
        # alpha为最小值，beta为最大值
        alpha = -sys.maxint
        beta = sys.maxint
        best_move, value, total_num = self.alpha_beta_search(is_max, alpha, beta, vertical, limit)
        return (best_move, value, total_num)
    
    def alpha_beta_search(self, is_max, alpha, beta, vertical, limit):
        total_num = 0
        best_move = ()
        if limit == 0:
            # 我方的可放的情况数 - 对手的可放的情况数
            value = len(list(self.legal_moves(vertical))) - len(list(self.legal_moves(not vertical)))
            return (), value, 1
        else:
            # xor用来计算当前是竖着还是横着
            for move, g in self.successors(not is_max ^ vertical):
                if alpha < beta:
                    _, value, num = g.alpha_beta_search(not is_max, alpha, beta, vertical, limit-1)
                    total_num += num
                    # 更新alpha值
                    if is_max and value > alpha:
                        alpha = value
                        best_move = move
                    # 更新beta的值
                    elif not is_max and value < beta:
                        beta = value
        return best_move, alpha if is_max else beta, total_num
```


