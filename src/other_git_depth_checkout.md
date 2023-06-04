# depth 1 情况下切换分支
在大型工程，特别是 git commit 记录特别多的工程中，如 linux，clone 时如果加入 `--depth 1`选项将使速度大大减小下载内容，同时提高成功率、加快完成速度。但这种情况下本地仓库将只有主分支，没有其它分支的记录，这里记录进一步添加远程分支，并在本地同步的方法。

## 操作步骤
1. clone 远程仓库
```bash
$ git clone xxx.git --depth 1
```

2. 查看本地仓库的分支记录
```bash
$ git branch -a
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
```
可以看到远程分支的信息只有 origin/main，此时无法 checkout 到其它分支中工作。

3. 添加远程分支
``` bash
$ git remote set-branches origin dev1
$ git fetch origin dev1 --depth 1
$ git remote set-branches origin dev2
$ git fetch origin dev2 --depth 1
$ git branch -a
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
  remotes/origin/dev1
  remotes/origin/dev2
```
这样本地仓库就有远程分支的信息

4. 创建对应的本地分支
```bash
$ git checkout -b dev1 origin/dev1
$ git branch -a
  main
* dev1
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
  remotes/origin/dev1
  remotes/origin/dev2
```
这样即可在本地创建与远程分支关联的本地分支，并 checkout 到次分支中。

5. 合并远程分支
此时往往需要将远程某个分支的内容拉取到本地，并继续做开发，如将 `origin/dev2` 拉取到 `dev1`，使用简单 `merge` 命令，可能会提示 `unrelated histories` 相关的错误，这是因为本地仓库的 commit 记录只有1层，git 认为这两个分支是不相关，默认不可以合并，可以添加 `--allow-unrelated-histories` 忽略此检查
```bash
$ git branch
  main
* dev1
$ git merge origin/dev2 --allow-unrelated-histories
```
