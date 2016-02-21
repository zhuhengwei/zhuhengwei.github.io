---
title: Git管理空文件夹
date: 2016-02-19 20:35:45
tags:
    - Git
---
把文件夹中的所有文件忽略不进去git仓库，但是让这个文件夹进去git。例如项目里的upload文件夹，里面的文件和项目本身没有任何关系，而且特别容易发生冲突，拉取推送都会浪费时间。
### 方法
(以upload目录为例)在upload目录下下新建.gitignore文件中，将下面两行写入文件。新建时可以从其他地方复制一个.gitignore文件或者新建文本文件，重命名为.gitignore.(注意前后都有一个点，windows会自动命名为.gitignore)
```
/*  
!/.gitignore  
```
### 解释
    1.忽略该目录下所有文件;
    2.不忽略.gitignore文件，就是跟踪该文件，而在myeclispe和tomcat中，该文件透明。
于是可以保证upload文件夹不是空，而且忽略其他文件.
如果想让git跟踪其他文件，在.gitignore中添加
```
!/xxx.xxx  
```