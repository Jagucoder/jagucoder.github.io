---
layout: post
title: "ctags & cscope"
categories: [tools]
tags: [ctags, cscope, linux]
redirect_from:
  - /2019/05/11/
---
## ctags

用法：

```
ctags . -R //生成ctag文件
ctrl + ] //下一个，需在ctag文件所在目录执行vim
ctrl + T //上一个
```



## cscope

1. cscope的用法很简单，首先需要为你的代码生成一个cscope数据库。在你的项目根目录运行：

````
cscope -Rbqk
````



2. 这个命令会生成三个文件：cscope.out, cscope.in.out, cscope.po.out。其中cscope.out是基本的符号索引，后两个文件是使用"-q"选项生成的，可以加快cscope的索引速度。"-k"选项用于取消自动到/usr/include目录中查找项目目录中未找到的头文件。


3. 在vim中使用cscope非常简单，首先调用"cscope add"命令添加一个cscope数据库，然后就可以调用"cscope find"命令进行查找了。

4. vim支持8种cscope的查询功能，如下：

   1. s: 查找C语言符号，即查找函数名、宏、枚举值等出现的地方
   2. g: 查找函数、宏、枚举等定义的位置，类似ctags所提供的功能
   3. d: 查找本函数调用的函数
   4. c: 查找调用本函数的函数
   5. t: 查找指定的字符串
   6. e: 查找egrep模式，相当于egrep功能，但查找速度快多了
   7. f: 查找并打开文件，类似vim的find功能
   8. i: 查找包含本文件的文件

5. 常用命令

   ```
   :cs show   //显示当前连接
   :cs reset   //重新初始化连接
   :cs kill  {number|partial_name}   //终止某个连接
   :cs find c do_cscope //查找调用do_cscope()函数的函数
   :cs find s do_cscope //查找这个C符号出现的位置
   ```

   ​