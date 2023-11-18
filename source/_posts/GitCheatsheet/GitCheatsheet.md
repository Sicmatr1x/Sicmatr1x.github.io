title: Git Cheetsheet
date: 2023-10-20 10:04:40
categories:
    - Git
tags: 
    - Git
---

# Git

Reference:
- [Pro Git](https://book.git-scm.com/book/en/v2)
- [git - 简明指南](https://rogerdudler.github.io/git-guide/index.zh.html)

### Dictionary
1. VCS: Version Control Systems
2. CVCS: Centralized Version Control System
3. DVCS: Distributed Version Control System

## basic knowladge

#### The Three States: modified, staged, and committed
1. Modified means that you have changed the file but have not committed it to your database yet.
2. Staged means that you have marked a modified file in its current version to go into your next commit snapshot.
3. Committed means that the data is safely stored in your local database.

#### The basic Git workflow goes something like this:
1. You modify files in your working tree.
2. You selectively stage just those changes you want to be part of your next commit, which adds only those changes to the staging area.
3. You do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

工作目录下的每一个文件都存在两种状态：已跟踪 或 未跟踪 tracked or untracked

工作目录中除已跟踪文件外的其它所有文件都属于未跟踪(untracked)文件，它们既不存在于上次快照(snapshot)的记录中，也没有被放入暂存区(staging area)。

### git branch
Git 保存的不是文件的变化或者差异，而是一系列不同时刻的 快照(snapshot)

在进行提交操作时，Git 会保存一个提交对象（commit object）。 知道了 Git 保存数据的方式，我们可以很自然的想到——该提交对象会包含一个指向暂存内容快照的指针。 但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。 首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象， 而由多个分支合并产生的提交对象有多个父对象

1. 暂存操作会为每一个文件计算校验和（使用我们在 起步 中提到的 SHA-1 哈希算法）
2. 然后会把当前版本的文件快照保存到 Git 仓库中 （Git 使用 blob 对象来保存它们）
3. 最终将校验和加入到暂存区域等待提交

由于 Git 的分支实质上是指向commit对象的指针，而这个指针仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符）

#### `git branch`
展示当前所有分支的一个列表
`git branch -v` 会在每个分支后带上对应分支最新的一个commit的简短信息

#### `git branch <branchName>`
分支创建: 
分支创建的本质是为你在当前的提交对象上创建了一个可以移动的新的指针
```
$ git branch testing
```

**注: git branch 命令仅仅 创建 一个新分支，并不会自动切换到新分支中去**

如果要创建的同时切换过去可以用`git checkout -b <newbranchname>`

Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 HEAD 的特殊指针。且该指针指向当前所在的本地分支

#### `git log --oneline --decorate`
可以使用 `git log --oneline --decorate` 命令查看各个分支当前所指的对象

#### `git log --oneline --decorate --graph --all`
它会输出你的提交历史、各个分支的指向以及项目的分支分叉情况

#### `git checkout <branchName>`
分支切换:
本质是更改HEAD指针指向的分支指针指向的commit对象
```
$ git checkout testing
```

#### `git merge <branchName>`
合并`<branchName>`分支到当前分支:
eg: 合并`hotfix`分支到`master`分支: 
1. 检出(checkout)到你想合并入的分支，即`master`分支
2. 运行 `git merge` 命令
```
$ git checkout master
$ git merge hotfix
```

注: 若这里的`hotfix`分支所指向的提交 `C4` 是当前`master`分支指针所指向的提交 `C2` 的直接后继，那么 Git 会直接将指针最新的commit方向移动，换句话说，当你试图合并两个分支
时， 如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候， 只会简单的将指针向最新的commit方向移动，因为这种情况下的合并操作没有需要解决的分歧——这就叫做 快进(fast-forward)

## set-up environment

### git config
lets you get and set configuration variables that control all aspects of how Git looks and operates.
location:
1. `[path]/etc/gitconfig` file: Contains values applied to every user on the system and all their repositories. If you pass the option `--system` to git config, it reads and writes from this file specifically. Because this is a system configuration file, you would need administrative or superuser privilege to make changes to it.
2. `~/.gitconfig` or `~/.config/git/config` file: Values specific personally to you, the user. You can make Git read and write to this file specifically by passing the `--global` option, and this affects all of the repositories you work with on your system.
3. `config` file in the Git directory (that is, `.git/config`) of whatever repository you’re currently using: Specific to that single repository. You can force Git to read from and write to this file with the `--local` option, but that is in fact the default. Unsurprisingly, you need to be located somewhere in a Git repository for this option to work properly.

在 Windows 系统中，Git 会查找 $HOME 目录下（一般情况下是 C:\Users\$USER ）的 .gitconfig 文件。 如果你在 Windows 上使用 Git 2.x 以后的版本，那么还有一个系统级的配置文件，Windows XP 上在 C:\Documents and Settings\All Users\Application Data\Git\config ，Windows Vista 及更新的版本在 C:\ProgramData\Git\config 。此文件只能以管理员权限通过 `git config -f <file>` 来修改。

查看所有的配置以及它们所在的文件: `git config --list --show-origin`
```
file:C:/Program Files/Git/etc/gitconfig diff.astextplain.textconv=astextplain
file:C:/Program Files/Git/etc/gitconfig filter.lfs.clean=git-lfs clean -- %f
file:C:/Program Files/Git/etc/gitconfig filter.lfs.smudge=git-lfs smudge -- %f
file:C:/Program Files/Git/etc/gitconfig filter.lfs.process=git-lfs filter-process
file:C:/Program Files/Git/etc/gitconfig filter.lfs.required=true
file:C:/Program Files/Git/etc/gitconfig http.sslbackend=openssl
file:C:/Program Files/Git/etc/gitconfig http.sslcainfo=C:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
file:C:/Program Files/Git/etc/gitconfig core.autocrlf=true
file:C:/Program Files/Git/etc/gitconfig core.fscache=true
file:C:/Program Files/Git/etc/gitconfig core.symlinks=false
file:C:/Program Files/Git/etc/gitconfig credential.helper=manager
file:D:/Users/GUOYAN708/.gitconfig      user.name=GUOYAN708
file:D:/Users/GUOYAN708/.gitconfig      user.email=GUOYAN708@pingan.com.cn
file:D:/Users/GUOYAN708/.gitconfig      credential.helper=store
file:D:/Users/GUOYAN708/.gitconfig      http.https://github.com.proxy=45.40.234.179:3128
file:D:/Users/GUOYAN708/.gitconfig      http.https://android.googlesource.com.proxy=socks5://127.0.0.1:1080
```

设置你的用户名和邮件地址: 
如果使用了 --global 选项，
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```

检查 Git 的某一项配置: `git config <key>`

git config 命令的手册: `git help config`


#### Git 别名
如果不想每次都输入完整的 Git 命令，可以通过 git config 文件来轻松地为每一个命令设置一个别名
```
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

即: 当要输入 git commit 时，只需要输入 git ci

eg: 取消暂存别名：
```
$ git config --global alias.unstage 'reset HEAD --'

等价于以下两条命令

$ git unstage fileA
$ git reset HEAD -- fileA
```

eg: 轻松查看最后一次提交信息`git last`
```
$ git config --global alias.last 'log -1 HEAD'
```

### 登录与凭证存储
如果你正在使用 HTTPS URL 来推送，Git 服务器会询问用户名与密码。 默认情况下它会在终端中提示服务器是否允许你进行推送。
如果不想在每一次推送时都输入用户名与密码，你可以设置一个 `credential cache`。 最简单的方式就是将其保存在内存中几分钟，可以简单地运行 `git config --global credential.helper cache` 来设置它。

Git 的凭证系统
- 默认所有都不缓存。 每一次连接都会询问你的用户名和密码
- `cache` 模式会将凭证存放在内存中一段时间。 密码永远不会被存储在磁盘中，并且在15分钟后从内存中清除
  - `--timeout <seconds>` 参数，可以设置后台进程的存活时间（默认是 “900”，也就是 15 分钟）
- `store` 模式会将凭证用明文的形式存放在磁盘中，并且永不过期。 这意味着除非你修改了你在 Git 服务器上的密码，否则你永远不需要再次输入你的凭证信息。 这种方式的缺点是你的**密码是用明文的方式**存放在你的 `home` 目录下
  - `--file <path>` 参数，可以自定义存放密码的文件路径（默认是 `~/.git-credentials` ）
- 如果你使用的是 Mac，Git 还有一种 `osxkeychain` 模式，它会将凭证缓存到你系统用户的`钥匙串`中。这种方式将凭证存放在磁盘中，并且永不过期，但是是被加密的，这种加密方式与存放 HTTPS 凭证以及 Safari 的自动填写是相同的
- 如果你使用的是 Windows，你可以安装一个叫做 [Git Credential Manager for Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) 的辅助工具。这和上面说的 `osxkeychain` 十分类似，但是是使用 `Windows Credential Store` 来控制敏感信息

可以设置 Git 的配置来选择上述的一种方式
```
$ git config --global credential.helper cache
```

`store` 模式自定义路径的例子：
```
$ git config --global credential.helper 'store --file ~/.my-credentials'
```

Git 甚至允许你配置多个辅助工具。 当查找特定服务器的凭证时，Git 会按顺序查询，并且在找到第一个回答时停止查询。 当保存凭证时，Git 会将用户名和密码发送给 所有 配置列表中的辅助工具，它们会按自己的方式处理用户名和密码。 如果你在闪存上有一个凭证文件，但又希望在该闪存被拔出的情况下使用内存缓存来保存用户名密码，`.gitconfig` 配置文件如下：
```
[credential]
  helper = store --file /mnt/thumbdrive/.git-credentials
  helper = cache --timeout 30000
```

### Git 凭证辅助工具
### ` git credential`
这个命令接收一个参数，并通过标准输入获取更多的参数


### .gitignore 忽略文件
`.gitignore`用于列出要忽略的文件的模式

格式规范如下：
- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

glob 模式是指 shell 所使用的简化了的正则表达式
- 星号（`*`）匹配零个或多个任意字符；
- `[abc]` 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 
- 问号（`?`）只匹配一个任意字符；
- 如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 
- 使用两个星号（`**`）表示匹配任意中间目录，比如 `a/**/z` 可以匹配 `a/z` 、 `a/b/z` 或 `a/b/c/z` 等

```
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO
# 忽略任何目录下名为 build 的文件夹
build/
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

## 常用git命令
### 本地git命令
#### `git clone <url>`
克隆已经存在于远程的仓库`git clone <url>`

eg: `git clone https://github.com/libgit2/libgit2`

#### `git status`
检查当前文件状态`git status`

#### `git add 文件名`
跟踪新文件` git add 文件名`  
这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等  
其作用是:精确地将内容添加到下一次提交中

#### `git reset HEAD <file>`
取消暂存的文件  
eg:取消暂存 CONTRIBUTING.md 文件：
```
git reset HEAD CONTRIBUTING.md
```

**注: `git reset` 是个危险的命令，如果加上了 `--hard` 选项则更是如此。 然而在上述场景中，工作目录中的文件尚未修改，因此相对安全一些**

#### `git checkout -- <file>`
撤消对文件的修改，将这个文件回滚到上个提交了的版本，相当于idea里的revent  
eg:
```
git checkout -- CONTRIBUTING.md
```

**注: ` git checkout -- <file>` 是个危险的命令， 你对那个文件在本地的任何修改都会消失——Git 会用最近提交的版本覆盖掉它。 除非你确实清楚不想要对那个文件的本地修改了，否则请不要使用这个命令。**

#### `git diff`
查看尚未暂存的文件更新了哪些部分，不加参数直接输入 `git diff`  
此命令比较的是工作目录中当前文件和暂存区域快照之间的差异。

若要查看已暂存的将要添加到下次提交里的内容，可以用 `git diff --staged`(和` git diff --cached`是同义词) 命令。 这条命令将比对已暂存文件与最后一次提交的文件差异

#### `git commit`
提交更新  
这样会启动你选择的文本编辑器来输入提交说明。 
注: 启动的编辑器是通过 Shell 的环境变量 EDITOR 指定的，一般为 vim 或 emacs。 当然也可以使用 `git config --global core.editor` 命令设置你喜欢的编辑器

以在 commit 命令后添加 -m 选项，将提交信息与命令放在同一行: ` git commit -m "Story 182: Fix benchmarks for speed"`

提交时记录的是放在暂存区域的快照

跳过使用暂存区域: Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候使用`git commit -a`，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交

#### `git commit --amend`
撤消操作  
这个命令会将暂存区中的文件提交。 如果自上次提交以来你还未做任何修改（例如，在上次提交后马上执行了此命令）， 那么快照会保持不变，而你所修改的只是提交信息  
eg:
```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

#### `git rm 文件名`
暂存区域移除特定文件
如果只是简单地从工作目录中手工删除文件，运行 git status 时就会在 `Changes not staged for commit` 部分（也就是 未暂存清单）看到  
下一次提交时，该文件就不再纳入版本管理了

**注: 如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 -f（译注：即 force 的首字母）。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这样的数据不能被 Git 恢复。**

#### `git mv file_from file_to`
移动文件  
Git 并不显式跟踪文件移动操作，如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。不过 Git 非常聪明，会自动进行推断  
以下2条命令等价:
`$ mv README.md README`

```
$ git rm README.md
$ git add README
```

#### `git log`
查看提交历史  
不传入任何参数的默认情况下，git log 会按时间先后顺序列出所有的提交(SHA-1 校验和, 作者的名字和电子邮件地址, 提交时间, 提交说明)  

| 选项            | 说明                                                                                                       |
| --------------- | ---------------------------------------------------------------------------------------------------------- |
| -p              | 按补丁格式显示每个提交引入的差异。                                                                         |
| --stat          | 显示每次提交的文件修改统计信息。                                                                           |
| --shortstat     | 只显示 --stat 中最后的行数修改添加移除统计。                                                               |
| --name-only     | 仅在提交信息后显示已修改的文件清单。                                                                       |
| --name-status   | 显示新增、修改、删除的文件清单。                                                                           |
| --abbrev-commit | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。                                                          |
| --relative-date | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。                                            |
| --graph         | 在日志旁以 ASCII 图形显示分支与合并历史。                                                                  |
| --pretty        | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和format（用来定义自己的格式）。 |
| --oneline       | --pretty=oneline --abbrev-commit 合用的简写。                                                              |

`-p` 或 `--patch`: 它会显示每次提交所引入的差异，或使用 `-数字` 例如 `-2` 选项来只显示最近的两次提交
```
git log -p -2
```

看每次提交的简略统计信息: `--stat`
```
git log --stat
```

使用不同于默认格式的方式展示提交历史: `--pretty`
```
git log --pretty=oneline
```

git log --pretty=format 常用的选项
| 选项 | 说明                                         |
| ---- | -------------------------------------------- |
| %H   | 提交的完整哈希值                             |
| %h   | 提交的简写哈希值                             |
| %T   | 树的完整哈希值                               |
| %t   | 树的简写哈希值                               |
| %P   | 父提交的完整哈希值                           |
| %p   | 父提交的简写哈希值                           |
| %an  | 作者名字                                     |
| %ae  | 作者的电子邮件地址                           |
| %ad  | 作者修订日期（可以用 --date=选项来定制格式） |
| %ar  | 作者修订日期，按多久以前的方式显示           |
| %cn  | 提交者的名字                                 |
| %ce  | 提交者的电子邮件地址                         |
| %cd  | 提交日期                                     |
| %cr  | 提交日期（距今多长时间）                     |
| %s   | 提交说明                                     |

使用ASCII 字符串来图形化展示你的分支、合并历史：`git log --pretty=format:"%h %s" --graph`

### 远程仓库相关git命令

远程跟踪分支(Remote-tracking branches)是远程分支状态的引用。它们是你无法移动的本地引用(local references)。一旦你进行了网络通信， Git 就会为你移动它们以精确反映远程仓库的状态。
一般以 `<remote>/<branch>` 的形式命名，例如：`origin/master`
你的同事有分支`iss53`，那对应的远程为`origin/iss53`

注: 远程仓库名字 `origin` 与分支名字 `master` 一样，在 Git 中并没有任何特别的含义。`origin` 是当你运行 `git clone` 时默认的远程仓库名字。 如果你运行 `git clone -o booyah`，那么你默认的远程分支名字将会是 `booyah/master`

#### `git clone <url>`
克隆仓库到本地

#### `git remote`
查看远程仓库  
eg: `-v`: 显示详细的URL
```
$ git remote -v
origin https://github.com/schacon/ticgit (fetch)
origin https://github.com/schacon/ticgit (push)
```

#### `git ls-remote <remote>`
显式地获得远程引用的完整列表

#### `git remote show <remote>`
获得远程分支的更多信息

#### `git remote add <shortname> <url>`
添加远程仓库  
添加一个新的远程 Git 仓库，同时指定一个方便使用的简写：
```
$ git remote add pb https://github.com/paulboone/ticgit
$ git remote -v
origin https://github.com/schacon/ticgit (fetch)
origin https://github.com/schacon/ticgit (push)
pb https://github.com/paulboone/ticgit (fetch)
pb https://github.com/paulboone/ticgit (push)
```

#### `git fetch <remote>`
从远程仓库中抓取与拉取分支信息: 这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看  
eg: 
```
git fetch origin
```

#### `git push <remote> <branch>`
推送到远程仓库: 
只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先抓取他们的工作并将其合并进你的工作后才能推送。 
```
$ git push origin master
```

#### `git remote show <remote>`
查看某个远程仓库: 查看某一个远程仓库的更多信息(会列出远程仓库的 URL 与跟踪分支的信息)
```
$ git remote show origin
```

#### `git remote rename <branch>`
远程仓库的重命名: 来修改一个远程仓库的简写名
```
$ git remote rename pb paul
$ git remote
origin
paul
```

#### `git remote remove <branch>`
远程仓库的移除: 如果因为一些原因想要移除一个远程仓库——你已经从服务器上搬走了或不再想使用某一个特定的镜像了， 又或者某一个贡献者不再贡献了——可以使用 `git remote remove` 或 `git remote rm`
```
$ git remote remove paul
$ git remote
origin
```

**注: 一旦你使用这种方式删除了一个远程仓库，那么所有和这个远程仓库相关的远程跟踪分支以及配置信息也会一起被删除。**

#### ` git push <remote> <branch>`
推送本地分支的commit到有写入权限的远程仓库上
eg: 推送到 serverfix 的分支上:
```
$ git push origin serverfix
```

等价于
```
git push origin serverfix:serverfix
```

其含义是推送本地的 `serverfix` 分支，将其作为远程仓库的 `serverfix` 分支

### 标签(Tag)
Git 可以给仓库历史中的某一个提交(commit)打上标签(tag)，通常用于标记发布结点(v1.0 、 v2.0 等等)

**注： 默认情况下，git push 命令并不会传送标签到远程仓库服务器上。**

标签分为两种
1. 轻量标签(lightweight): 轻量标签很像一个不会改变的分支——它只是某个特定提交的引用
2. 附注标签(annotated): 而附注标签是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息，并且可以使用 GNU Privacy Guard(GPG)签名并验证。 通常会建议创建附注标签，这样你可以拥有以上所有信息。但是如果你只是想用一个临时的标签， 或者因为某些原因不想要保存这些信息，那么也可以用轻量标签

#### `git tag`
列出已有的标签: 可带上可选的 `-l` 选项 `--list`
```
$ git tag
v1.0
v2.0
```

模糊搜索特定的tag:
```
$ git tag -l "v1.8.5*"
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5-rc2
v1.8.5-rc3
v1.8.5.1
v1.8.5.2
v1.8.5.3
v1.8.5.4
v1.8.5.5
```

#### `git tag <tag>`
轻量标签: 
轻量标签本质上是将提交校验和存储到一个文件中——没有保存任何其他信息
```
$ git tag v1.4-lw
```

#### `git tag -a <tag> -m "<message>"`
创建附注标签: 
-m 选项指定了一条将会存储在标签中的信息。 如果没有为附注标签指定一条信息，Git 会启动编辑器要求你输
入信息。
```
$ git tag -a v1.4 -m "my version 1.4"
$ git tag
v0.1
v1.3
v1.4
```

#### `git tag -a <tag> <shortHash>`
补打标签:
给历史提交打标签
eg: 给`updated rakefile`` 这个提交补打标签
```
$ git log --pretty=oneline
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
4682c3261057305bdd616e23b64b0857d832627b added a todo file
166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo

$ git tag -a v1.2 9fceb02
```

#### ` git show`
看到标签信息和与之对应的提交信息:
```
$ git show v1.4
```

#### `git push origin <tagname>`
共享标签:
push标签信息到远程分支
```
$ git push origin v1.5
```

如果想要一次性推送很多标签，也可以使用带有 --tags 选项的 git push 命令。 这将会把所有不在远程仓库服务器上的标签全部传送到那里。
```
$ git push origin --tags
```

#### `git tag -d <tagname>`
删除标签: 
eg: 删除一个轻量标签
```
$ git tag -d v1.4-lw
```

注意上述命令并不会从任何远程仓库中移除这个标签，你必须用 `git push <remote>:refs/tags/<tagname>` 或 ` git push <remote> :refs/tags/<tagname>`(将冒号前面的空值推送到远程标签名，效果等同于删除) 来更新你的远程仓库
```
$ git push origin :refs/tags/v1.4-lw
```

或者直接删:
```
$ git push origin --delete <tagname>
```

####  `git checkout`
检出标签:
```
$ git checkout 2.0.0
```

**注: 该命令会使你的仓库处于分离头指针(detached HEAD)的状态——这个状态有些不好的副作用(果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。 因此，如果你需要进行更改，比如你要修复旧版本中的错误，那么通常需要创建一个新分支)**




