---
title: git批量删除分支
url: https://www.yuque.com/endday/blog/zb5st4
---

<a name="U9Z8k"></a>

### 1、批量删除本地分支

    git branch |grep 'branchName' |xargs git branch -D

-  git branch   查看本地分支
-  | grep 'branchName'  匹配分支名
-  | xargs git branch -D 将匹配到的分支名一个一个传递给git branch -D
-  git branch -D branchName  删除本地分支

 
 <a name="iRKW4"></a>

### 2、批量删除远程分支

    git branch -r| grep 'branchName' | sed 's/origin\///g' | xargs -I {} git push origin :{}

-  git branch -r  查看远程分支
-  | sed 's/origin///g'  用 空 替换 /origin（关于sed替换文本在下面有补充）
-  -I {}  使用占位符来构造后面的命令
-  git push origin :branchName  删除远程分支
