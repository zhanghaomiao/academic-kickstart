---
title: GDB 调试
date: 2019-03-12
tags:
    - computer
categories:
    - Learn
summary: 主要介绍有关GDB一些基本命令
---

1. 要求
   - 所有的程序编译时需要使用 `-g` 参数， 这样gdb程序可以识别
   - 编译时不要使用优化，使用优化时一些变量的值无法跟踪， 即优化等级设置为 `O0`

2. 命令

   - `b` 后加行数， 设置断点
   - `li`(start, [end]) 显示在start 和 end 之间的代码， end 为可选参数， 如果没有 end, 则显示以start 为中心前后5句代码
   - `r` 重新开始调试程序, 在遇到断点处终止
   - `c` 开始调试程序，在遇到断点处终止, 与 `step into my code` 功能类似
   - `s` 可后接参数 N ， 表示step n 次， 与 `step into` 功能类似
   - `n` 可后接参数 N ， 表示next n 次， 与 `step over` 功能类似
   - `fin` 从子程序跳出， 与 `step out` 功能类似
   - `p` 打印变量值
      - 对于一维数组， 使用 p `temp(n)` 会显示temp第`n`个元素
      - 对于二维数组， 使用 p `temp(i:j)` 会显示第 `i` 和 第 `j` 行的元素
   - `where` 显示当前程序所处位置
   - `info` b 显示所设置的断点
   - `del` breakpoints  删除断点
