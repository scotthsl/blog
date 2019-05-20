# github中根据upstream更新fork的仓库



1. 先将fork的仓库拉取到本地

   git  clone xxx

2. 原始仓库添加为upstream

   git  remote add upstream <upstream的地址>

3. 本地创建一个新分支

   git  checkout -b  newbranch

4. 将upstream的master分支更新pull到刚才创建的新分支

   git  pull  upstream master

5. 将新分支中的上游的commit合并到本地master分支中，需要注意是否会覆盖本地的改动。

6. 将新的commit合并到本地master分支后，再将改动推入到自己的远程仓库中

   git push origin master