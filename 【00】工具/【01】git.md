# 忽略文件

**`.gitignore` 忽略文件**

- **全局忽略文件`**：

  如果你希望某些忽略规则对所有 Git 项目生效，可以配置全局 .gitignore 文件

  ```shell
  touch ~/.gitignore_global
  git config --global core.excludesfile ~/.gitignore_global		# 使用该忽略文件
  ```

- **忽略模板**：

  我们可以从 http://github.com/github/gitignore 中下载常用的忽略模板（Android、Python 等等）

- .gitignore 只对未跟踪的文件生效，对已提交的文件无效（git add）

<br/>

1. **忽略单个文件**

   ```yaml
   file.txt
   ```

2. **忽略指定文件夹**

   ```yaml
   logs/
   ```

3. **通配符**

   ```tex
   *.log
   *.tmp
     
   build/**/*.o
   ```

4. **取消忽略**

   用 `!` 表示不忽略某个已被忽略的文件

   ```tex
   *.log
   !important.log
   ```

5. **取消跟踪**：

   如果在提交忽略文件之前（git add .gitignore），其他文件已经被 Git 跟踪了（已通过 git add 添加），那么 .gitignore 对它们无效

   这个时候要先取消跟踪 `git rm`，然后重新提交（git add），.gitignore 就会重新生效

   ```shell
   git rm -r --cached <file-or-directory>
   ```

   如果误执行了 git rm --cached，可以用 `git restore --staged` 将文件重新加回暂存区，前提是还没提交 git add

   ```shell
   git restore --staged <file>
   ```

6. **检查某个文件是否被忽略**

   `git check-ignore -v`

   ```shell
   git check-ignore -v path/to/file
   ```

<br/>

---

<br/>

# 命令

## 仓库

### 远程仓库

**`git remote`**：操作远程仓库

<br/>

1. 查看所有远程仓库信息 `git remote -v`

   ```shell
   git remote -v
   ```

   *输出*

   ```tex
   origin  https://github.com/user/repo.git (fetch)
   origin  https://github.com/user/repo.git (push)
   ```

   查看指定远程仓库信息 `git remote show`

   ```shell
   git remote show <name>
   ```

   *输出*

   ```c
    remote origin
     Fetch URL: git@github.com:xzqbetter/Note.git
     Push  URL: git@github.com:xzqbetter/Note.git
     HEAD branch: main													// 远程仓库的第一个分支
     Remote branch:														// 远程仓库的所有分支
       main tracked
     Local branch configured for 'git pull':
       main merges with remote main
     Local ref configured for 'git push':
       main pushes to main (up to date)
   ```

2. 添加远程仓库 `git remote add`

   ```shell
   git remote add <name> <url>
   ```

   *案例*

   ```shell
   git remote add origin https://github.com/user/repo.git
   ```

3. 删除远程仓库 `git remote remove`

   ```shell
   git remote remove <name>
   ```

4. 修改远程仓库的 url `git remote set-url`

   ```shell
   git remote set-url <name> <new-url>
   ```

<br/>

<br/>

## 分支

**`本地分支`**：通常以 branch-name 形式存在，当前分支前面会有一个 `*` 或者 `HEAD` 表示

**`远程分支`**：通常以 origin/branch-name 形式存在

- 命名：

  为每个新功能创建一个新分支，如 feature/login

  修复 bug 创建一个新分支，如 bug/issue-123

- 分支必须指向一个 `commit`，没有 commit 就没有任何分支（git branch -a 不会显示）

  提交第一个 commit 后，Git 会自动创建第一个分支 master/main

<br/>

1. 查看分支 `git branch`

   `-r、--remote` 查看远程分支

   `-a` 查看所有分支（远程 + 本地）

   - 本地分支显示 main
   - 远程分支显示 remotes/\<storage-name>/\<branch-name>

   `-v` 查看分支的合并状态

   ```shell
   git branch -a
   ```

2. 创建新分支 `git branch <name>`

   它会基于当前 HEAD 创建一个新分支，但不会切换到该分支

   ```shell
   git branch <branch-name>
   ```

3. 删除分支 `git branch -d`

   ```shell
   git branch -d <branch-name>		# 分支必须已合并到其他分支
   git branch -D <branch-name>		# 强制删除：如果分支未合并，可以使用强制删除
   ```

4. 重命名分支 `git branch -m`

   ```shell
   git branch -m <old-name> <new-name>		# 如果省略 <old-name>，则重命名当前分支
   ```

5. 切换分支 `git checkout`、`git switch`

   切换分支时，Git 会将未提交的更改带到新分支。如果有冲突，需要先提交或暂存（git stash）。

   ```shell
   git checkout <branch-name>					# 切换
   git checkout -b <branch-name>				# 切换 + 创建
   ```

   ```shell
   git switch <branch-name>
   git switch -c <branch-name>					# 切换 + 创建
   ```

6. 合并分支 `git merge`

   ```shell
   git merge <other-branch>
   ```

7. 比较分支 `git diff`

   ```shell
   git diff main origin/main					# 比较本地分支和远程分支的差异
   ```

<br/>

### git fetch

**`git fetch`**：获取远程仓库的分支列表

- git fetch 从远程仓库获取最新数据（提交、分支等），但它并不会将这些更改自动 "合并" 到你的本地工作分支中

- `--all` 所有
- `--prune` 清理本地分支（在远程分支不存在的）

```shell
git fetch <remote> <branch>
```

<br/>

### 上游分支

**`上游分支`**：指的是与本地分支关联的远程分支

- `--set-upstream-to`：通常配合 <u>git branch</u> 使用，设置本地分支的上游分支

  设置完后 push/pull 默认就是推送到该远程分支上

  如果远程分支不存在，会报错

  如果本地分支没有上游，或者上游设置错误，都可以使用该命令进行设置/修改

- `--set-upstream`：简写 `-u`，通常配合 <u>git push</u> 使用，明确指定将本地分支推送到远程仓库的哪个分支上，并且对两者进行关联

  如果远程分支不存在，会创建一个新的分支

```shell
git branch --set-upstream-to=<远程分支> <本地分支>
```

```shell
git push --set-upstream <远程仓库> <远程分支名>
```

- 查看每个分支的上游分支的信息

```shell
git branch -vv

# 输出：feature  abc1234 [origin/feature] Commit message
# 表示本地分支 feature 的上游分支是 origin/feature
```

<br/>

## 提交

**`git commit`**：提交信息

- 可以多次使用 -m，每条对应一行

```shell
git commit -m "修复登录页面" -m "解决了用户无法登录的 bug，优化了错误提示信息"
```

<br/>

---

<br/>

# 选项

**`-r`**：表示递归操作

- 如果目标是一个目录，-r 会递归地移除目录下的所有文件和子目录
- 如果不加 -r，尝试删除目录会报错

```shell
git rm -r --cached <file-or-directory>
```

<br/>

**`--cached`**：表示 "暂存区"

- 这个选项告诉 Git 只从索引（暂存区）中移除文件，而不会影响工作目录中的实际文件
- 换句话说，文件会保留在磁盘上，但 Git 将不再跟踪它

```shell
git rm -r --cached <file-or-directory>
```

<br/>

**`--force`**：强制

<br/>

**`-a`**：是 --all 的缩写，表示 "所有"

```shell
git branch -a
```

<br/>

**`-v`**

**`-vv`**：是 --verbose --verbose 的缩写，表示“非常详细”（double verbose）

- 它通常与 git branch 命令一起使用，用于显示更详细的分支信息：

  分支名称

  最近一次提交的哈希值和提交信息

  与上游分支（upstream）的跟踪关系及其同步状态】

- **git branch**：只列出分支名称。

  **git branch -v**：显示分支名、最近提交哈希和提交信息，但不包括上游信息。

  **git branch -vv**：在 -v 的基础上增加上游分支信息和同步状态。

```shell
git branch -vv

# 输出：
#			feature    abc1234 [origin/feature: ahead 2, behind 1] Add new feature
#			* main       def5678 [origin/main] Update readme
#			test       ghi9012 No upstream set
# 说明：
#			本地分支 feature 的上游分支是 origin/feature
#			本地比远程领先 2 个提交（ahead 2），落后 1 个提交（behind 1）
#			最近提交哈希：abc1234
#			提交信息：Add new feature
```

