# Git使用笔记

- `git reflog`

> reflog是Git操作的一道安全保障，它能够记录几乎所有本地仓库的改变。包括所有分支commit提交，已经删除（其实并未被实际删除）commit都会被记录

当rebase导致commit提交记录发生丢失时，可以通过git reflog 查看本地仓库的改变记录，并且找到相应commit-sha,然后git checkout -b 分支名 [commit-sha] 找回相应的代码。