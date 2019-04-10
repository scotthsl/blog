# github中根据upstream更新fork的仓库



1. 先将fork的仓库拉取到本地

   git  clone xxx

2. 原始仓库添加为upstream

   git  remote add upstream <upstream的地址>

3. 本地创建一个新分支

   git  checkout -b  newbranch

4. 将upstream的master分支更新pull到刚才创建的新分支

   git  pull  upstream master

5. 将拉下来的改动退到fork出来的仓库

   git push origin newbranch