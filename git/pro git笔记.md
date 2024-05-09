## 起步

### 关于版本控制

#### 本地版本控制系统

​	在不使用任何工具的情况下, 我们的备份往往是直接对文件或者文件目录的复制.

​	后来开发了很多种==本地版本控制系统==, 大多都是采==用某种简单的数据库来记录文件的历次更新差异==. 其中最流行的一种叫做RCS, 它主要关注单个文件的版本控制. 工作原理是在硬盘上保存补丁集.



#### 集中化的版本控制系统

​	接下来遇到的问题是, 如何让在不同系统上的开发者协同工作. 于是==集中化的版本控制系统(CVCS)应运而生==. 如CVS, SVN以及Perforce等, 都有一个单一的集中管理的服务器, 保存所有文件的修订版本, 协同工作的人们通过客户端连到这台服务器.

​	这种做法的好处是每个人都可以在一定程度上看到项目中的其他人正在做什么. 管理员也可以掌控每个开发者的权限. 并且管理一个统一的CVCS要比在各个客户端上维护本地数据库更轻松容易.

​	缺点是中央服务器的单点故障. 如宕机时间内无法提交更新, 也就无法协同工作; 若磁盘发生损坏, 也没有备份, 将丢失所有数据, 只剩下在各自机器上保留的单独快照.



#### 分布式版本控制系统

​	于是诞生了==分布式版本控制系统(DVCS)==, 像Git, Mervurial, Bazaar, Darcs等. 客户端并==不只是提取最新版本的快照, 而是把代码仓库完整地镜像下来==. 这样如果服务器发生故障, 可以用任何一个镜像出来的本地仓库恢复. 因为每一次克隆, 都是对仓库代码的完整备份, 包括整个变更历史. 

​	更进一步, 许多这类系统可以指定和若干不同的远端代码仓库进行交互.



### Git简史

1991-2002: linux内核开源项目的绝大多数维护工作都花在了提交补丁和保存归档的繁琐事务上.

2002-2005: 整个项目组开始启用分布式版本控制系统BitKeeper来管理和维护代码.

2005: BitKeeper同Linux内核开源社区的合作关系结束, Linux Torvalds主导开发自己的版本系统Git.



### 认识Git

#### 直接记录快照, 而非差异比较

​	SVN以文件变更列表的方式存储数据, 将保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异.

![image-20240424105708294](assets/image-20240424105708294.png)

​	

​	Git则更像是把数据看作是对==小型文件系统==的一组快照. 每次提交更新时, 它主要对当时的全部文件制作一个快照并保存这个快照的索引. 如果某个文件没有修改, 就不会重新存储该文件, 而是保留一个链接指向之前存储的文件. Git对待数据更像是一个快照流.

![image-20240424105730739](assets/image-20240424105730739.png)

​		Git更像一个小型的文件系统, ==提供了许多以此为基础构建的超强工具, 而不只是一个简单的VCS==.



#### 几乎所有操作都是本地执行

​	绝大多数操作只需访问本地文件和资源.

​	SVN(其他集中式版本管理系统也是如此)如果要切换到某一个过去的版本, 则需要访问服务器, 通过保存在服务器中的原始文件和增量差异, 计算出该版本.



#### Git保证完整性

​	Git中的所有数据在存储前都计算==校验和==, 然后以校验和来引用. 

​	用以计算校验和的机制叫做SHA-1(hash, 哈希). 是一个由40个十六进制字符组成的字符串.基于Git中文件的内容或目录结构计算出来.

​	Git数据库中, 保存的信息都是以文件内容的哈希值来索引, 而不是文件名.



#### Git一般只添加数据

​	我们执行的Git操作, 几乎只往Git数据库中增加数据. 很难让Git执行任何不可逆操作, 或者让它以任何方式清除数据.



#### 三种状态

​	三种状态, 文件可能处于其中之一:

- 已提交(committed)
- 已修改(modified)
- 已暂存(staged)



​	已提交表示数据已经安全的保存在本地数据库中. 

​	已修改表示修改了文件, 但还没保存到数据库中.

​	已暂存表示一个已修改文件的当前版本做了标记, 使之包含在下次提交的快照中.



#### 三个工作区域

由此引出三个工作区域的概念: 工作目录, Git仓库, 暂存区域.

![image-20240424142320066](assets/image-20240424142320066.png)



- 工作目录是对项目的某个版本从Git仓库的==压缩数据库==中独立提取出来的内容. 
- Git仓库目录是Git用来保存项目的元数据和==对象数据库==的地方.从其它计算机克隆仓库时, 拷贝过来的就是这里的数据.
- 暂存区域是一个文件, 保存了下次将提交的文件列表信息, 一般在Git仓库目录中. 有时也被称作'索引', 不过一般说法还是叫暂存区域.



#### 基本的Git工作流程

1. 在工作目录中修改文件
2. 暂存文件, 将文件的索引放入暂存区(.git/index). 索引通过哈希值指向文件快照(.git/objects)
3. 提交更新, 找到暂存区域的文件, 将快照永久性存储到Git仓库目录



​	如果Git目录中保存着特定版本文件, 就属于已提交状态. 如果做了修改并已放入暂存区域, 就属于已暂存状态. 如果自上次取出后, 做了修改但还没有放到暂存区域, 就是已修改状态.(后面会了解到, 可以跳过暂存直接提交).



### 命令行

​	只有在命令行模式下才能执行Git的所有命令, 而大多数的GUI软件只实现了Git所有功能的一个子集以降低操作难度. 如果学会了在命令行下如何操作, 那么在操作GUI软件时, 也不会遇到什么困难.

​	不同的人可能会安装不同的GUI软件, 但所有人一定会有命令行工具.



### 安装Git

​	推荐安装最新版, 但这篇笔记的参考书pro git用的是2.0.0版本.

​	因为Git在保持向后兼容方面表现很好, 因此后面介绍的命令在2.0之后的版本应当有效.



### 初次运行Git前的配置

​	定制Git环境. 每台计算机上只需要配置一次, 程序升级时会保留配置信息. 

1. `/etc/gitconfig`文件, 包含系统上每一个用户及他们仓库的通用配置. 如果使用带有`--system`选项的`git config`时, 它会从此文件读写配置变量
2. `~/.gitconfig`或`~/.config/git/config`文件, 只针对当前用户. 可以传递`--global`选项让Git读写此文件
3. 当前使用仓库的Git目录中的`.git/config`文件, 只针对该仓库



​	每一个级别覆盖上一级别的配置.

​	在Windows系统中, Git会查找`$HOME`目录下的`.gitconfig`文件. 以及`/etc/gitconfig`文件, 但只限于MSys的根目录下, 即安装Git时所选的目标位置.



#### 用户信息

​	==安装完后的第一件事==就是应该设置用户名称和邮件地址.  这很重要, 因为每一个提交都会使用这些信息, 并且会写入到每一次提交中, 不可更改.

```shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```



#### 文本编辑器

​	还可以配置文本编辑器. 当git需要你输入信息时会调用它(比如需要你输入提交的comment信息时). 如果未配置, Git会使用操作系统默认的文本编辑器, 通常是Vim. 如果想使用emacs:

```shell
git config --global core.editor emacs
```



#### 检查配置信息

​	使用`git config --list`命令来列出所有Git当前能找到的配置.		

​	如果看到重复的变量名, 是因为Git从不同的文件中读取同一个配置, 这种情况下, Git会使用它找到的每一个变量的最后一个配置.

​	可以通过`git config <key>`来检查Git的某一项配置.

```shell
git config user.name
```



### 获取帮助

​	如果使用Git时需要获取帮助, 有三种方法可以找到Git命令的使用手册:

```shell
git help <verb>
git <verb> --help
man git-<verb>

git help config
```

​	除了上述命令, 还可以尝试在Freenode IRC服务器(irc.freenode.net)的`#git`或`#github`频道寻求帮助.



## Git基础

### 获取Git仓库

#### 在现有目录中初始化仓库

​	在项目目录下输入`git init`.  该命令将会创建一个名为`.git`的子目录, 这个子目录含有初始化的Git仓库中所有的必须文件, 这些文件是Git仓库的主干. 但此时, 我们只是做了一个初始化操作, 项目里的文件还没有被跟踪.

​	如果对已经存在的项目进行了Git仓库的初始化, 接下来应该开始跟踪这些文件并提交. 可以通过`git add`命令实现对指定文件的跟踪, 通过`git commit`命令进行提交.

#### 克隆现有的仓库

​	如果要获得一份已经存在了的Git仓库的拷贝, 就要用到`git clone`命令. 之所以使用clone而不是checkout, 是因为Git仓库克隆的是该Git仓库服务器上的==几乎所有数据==, 而不仅仅是复制完成工作所需的文件.当执行`git clone`命令的时候, ==默认配置下远程Git仓库中的每一个文件的每一个版本都将被拉下来.==

​	克隆仓库的命令是`git clone [rurl]`. 例如:

```shell
git clone https://github.com/libgit2/libgit2
```

​	这会在当前目录下创建一个名为“libgit2"的目录, 并在这个目录下初始化一个`.git`文件夹, 从远程仓库拉取的所有数据会放入这个`.git`文件夹, 然后从中读取最新版本的文件的拷贝. 点进这个新建的libgit2文件夹, 就会发现所有的项目文件已经在里面了, 准备就绪等待后续的开发和使用. 如需在克隆远程仓库时自定义本地仓库的名字, 可以使用如下命令:

```shell
git clone https://github.com/libgit2/libgit2 mylibgit
```

​	Git支持多种数据传输协议, 上面的例子使用的是`https://`协议, 不过你也可以使用`git://`协议或者使用SSH传输协议, 各种方式各有利弊.

### 记录每次更新到仓库

​	首先, 工作目录下的每一个文件都不外乎两种状态: 已跟踪或未跟踪.已跟踪的文件是指那些被纳入了版本控制的文件, 在上一次快照中有它们的记录, 在工作一般时间后, 它们的状态可能处于未修改, 已修改或已放入暂存区. 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件, 它们既不存在于上次快照的记录中, 也没有放入暂存区.初次克隆某个仓库的时候, 工作目录中的所有文件都属于已跟踪文件, 并处于未修改状态.

​	对某些文件进行编辑后, Git将它们标记为已修改文件. 然后逐步将这些已修改的文件放入暂存区, 然后提交所有暂存了的修改, 如此反复. 所以使用Git时文件的生命周期如下:

![image-20240424233139799](./assets/image-20240424233139799.png)



#### 检查当前文件状态

​	要查看哪些文件处于什么状态, 可以`git status`命令. 如果在克隆仓库后立即使用此命令, 会看到类似如下==输出==:

```shell
On branch master
nothing to commit, working directory clean
```

​	这说明现在的工作目录==相当干净==. 也就是==所有已跟踪文件在上次提交后都未更改过==. 此外, 上面信息还表明, 当前目录下没有出现任何未跟踪状态文件, 否则Git会在这里列出来. 最后, 该命令还显示了当前所在分支, 并且这个分支同远程服务器上对应的分支没有偏离. 

​	当前分支是“master", 这是默认的分支名, 最新版本Git已将默认分支名改为“main"了.

​	在该项目下创建一个新的文件README后, 再次输入`git status`命令, 将会看到==一个新的未跟踪文件==:

```
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
    	README
nothing added to commit but untracked files present (use "git add" to track)
```

​	可以看到新建的README文件出现在`Untracked files`下面. 未跟踪的文件意味着Git在之前的快照(提交)中没有这些文件, Git不会自动将之纳入跟踪范围.



#### 跟踪新文件

​	使用`git add`开始跟踪一个文件.

```shell
git add README
```

​	此时, 再运行`git status`命令, 会看到README文件已被跟踪, 并处于暂存状态:

```shell
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
	  new file: README
```

​	只要在`Changes to be committed`这行下面的, 就说明是已暂存状态. 如果此时提交, 那么该文件此时此刻的版本将被留存在历史记录中.

​	`git add (files)`命令的参数可以是文件或者目录, 如果参数是目录的路径, 该命令将递归地跟踪该目录下的所有文件.



#### 暂存已修改文件

​	修改一个已被跟踪的文件. 然后运行`git status`. 将会看到如下内容:

```shell
On branch master
Changes to be committed:
	(use "git reset HEAD <file>..." to unstage)
	   new file: README
Changes not staged for commit:
	(use "git add <file>..." to update what will be committed)
	(use "git checkout -- <file>..." to discard changes in working directory)
	   modified: CONTRIBUTING.md
```

​	文件CONTRIBUTING.md出现在`Changes not staged for commit:`这行下面, 说明已跟踪文件的内容发生了变化, 但还没有放到暂存区. 要暂存这次更新, 需要运行`git add`命令. ==这是个多功能命令: 可以用它开始跟踪新文件, 或者把已跟踪的文件放到暂存区, 还能用于合并时把有冲突的文件标记为已解决状态等.==将这个命令理解为==添加内容到下一次提交中==, 而不是==将一个文件添加到项目中==要更加合适. 现在运行`git add`把CONTRIBUTING.md放入暂存区, 再看一下`git status`输出:

```shell
On branch master
Changes to be committed:
	(use "git reset HEAD <file>..." to unstage)
	  new file: README
	  modified: CONTRIBUTING.md
```

​	现在暂存区中有两个文件, 下次提交时会一并记录到仓库. 如果此时再次修改CONTRIBUTING.md, 再次运行`git status`看看:

```shell
On branch master
Changes to be committed:
	(use "git reset HEAD <file>..." to unstage)
	  new file: README
	  modified: CONTRIBUTING.md
Changes not staged for commit:
	(use "git add <file>..." to update what will be committed)
	(use "git checkout -- <file>..." to discard changes in working directory)
	  modified: CONTRIBUTING.md
```

​	哈? CONTRIBUTING.md同时出现在暂存区和非暂存区. 是的, Git只暂存了刚才运行`git add`命令时的版本, 如果现在提交, CONTRIBUTING.md的版本是最后一次运行`git add`时的那个版本, 而不是在工作目录中的当前版本. 所以, 如果运行了`git add`之后又修改了文件, 需要重新运行`git add`把最新版本重新暂存起来, 调用`git add CONTRIBUTING.md` 后再次观察输出:

```shell
On branch master
Changes to be committed:
	(use "git reset HEAD <file>..." to unstage)
	  new file: README
	  modified: CONTRIBUTING.md
```

#### 状态简览

​	从上面可以看到使用`git status`命令时的输出十分详细, 但是有时略显繁琐. 如果想要得到一种更为紧凑的格式输出, 则可以使用`git status -s`或者`git status --short`. 可能的输出如下:

```shell
 M README
MM Rakefile   # 在工作区被修改提交单暂存区后又在工作区中被修改了, 所以在暂存区和工作区都有该文件被修改了的记录.
A lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt
```

各种标记代表的状态如下:

- ??: 新添加的未跟踪文件
- A: 新添加到暂存区中的文件
- M: 修改过的文件. 右M表示文件被修改但还没有放入暂存区, 左M表示被修改了并放入暂存区.



#### 忽略文件

​	一个项目中总会有些文件无需纳入Git管理, 甚至不希望总是在未跟踪列表中看到它们. 比如日志文件, 编译过程中产生的文件等. 这时, 可以创建一个名为`.gitignore`的文件, 列出要忽略的文件模式:

```shell
*.[oa]   # 忽略所有.o或.结尾的文件. 一般这类对象文件和存档文件都是编译过程中出现的.
*~   	 # 忽略所有以~结尾的文件. 如Emacs会以这样的文件名来保存副本.
```

​	最好养成一开始就设置好.gitignore文件的习惯, 以免将来误提交这类无用的文件.

​	文件`.gitignore`的格式规范如下:

- 所有空行或者以#开头的行都会被Git忽略
- 可以使用标准的glob模式匹配
- 匹配模式可以以(/)开头防止递归
- 匹配模式可以以(/)结尾指定目录
- 要忽略指定模式以外的文件或目录, 可以在模式前加上惊叹号(!)取反



​	所谓的glob模式是shell所使用的简化了的正则表达式.

1. \* 匹配0个或多个任意字符
2. [abc] 匹配任何一个列在方括号中的字符(要么a, 要么b, 要么c)
3. ? 只匹配一个任意字符
4. \- 例如[0-9], 所有在这两个字符范围内的都可以匹配, 这里表示匹配所有0到9的数字字符
5. ** 匹配任意中间目录, 如a/**/z, 可以匹配a/z, a/b/z, 或者a/b/c/z等



​	GitHub有一个十分详细的针对各种项目及语言的.gitignore文件列表, 可以在https://github.com/github/gitignore找到它.



#### 查看已暂存和未暂存的修改

​	`git status`命令看到有文件修改时, 想知道具体修改了什么地方. 可以使用`git diff`命令.

​	加入此时有已暂存的修改README和未暂存的修改CONTRIBUTING.md:

```shell
On branch master
Changes to be committed:
	(use "git reset HEAD <file>..." to unstage)
	  modified: README
Changes not staged for commit:
	(use "git add <file>..." to update what will be committed)
	(use "git checkout -- <file>..." to discard changes in working directory)
	  modified: CONTRIBUTING.md
```



​	要查看尚未暂存的文件更新了哪些部分, 可以直接输入`git diff`

```shell
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md
index 8ebb991..643e24f 100644
--- a/CONTRIBUTING.md
+++ b/CONTRIBUTING.md
@@ -65,7 +65,8 @@ branch directly, things can get messy.
Please include a nice description of your changes when you submit your PR;
if we have to read the whole diff to figure out why you're contributing in the first place, you're less likely to get feedback and have your change
-merged in.
+merged in. Also, split your changes into comprehensive chunks if your
patch is
+longer than a dozen lines.
If you are starting to work on a particular area, feel free to submit a PR that highlights your work in progress (and note in the PR title that it's
```

​	此命令比较的是==工作目录中当前文件==和==暂存区域快照==之间的差异, 也就是修改之后还没有暂存起来的变化内容.

​	若要查看已暂存的将要提交的内容, 可以用`git diff --cached`命令. (Git1.6.1及更高版本还允许使用`git diff --staged`, 效果是相同的, 但更好记)

​	

​	如果上面的内容看起来还是很不清楚, 可以重点理解以下两句话:

1. git diff比较的是工作目录下的当前文件和暂存区域快照之间的差异, 如果暂存区没有对应的文件, 就与上一次提交的文件比较
2. git diff --cached比较的是暂存区中的文件和上一次提交的文件的差异



​	除了使用`git diff`来分析文件差异, 如果想要通过图形化的方式或其它格式输出方式的话, 可以使用`git difftool`命令来使用Araxis, emerge或vimdiff等软件输出diff分析结果. 使用`git difftool --tool-help`查看你的系统支持哪些Git Diff插件.



#### 提交更新

​	先调用`git status`查看有没有什么已修改的文件或者新创建的文件还没有`git add`到暂存区, 将要提交的文件及文件修改提交到暂存区后, 就可以调用`git commit`来进行提交.

​	如果只调用`git commit`, 则会启动文本编辑器以输入本次提交的说明. (默认启动shell的环境变量`$EDITOR`所指定的软件, 一般是vim或emacs, 也可以按照之前介绍的方式`git config --global core.editor`来设定自己喜欢的编辑软件.)

​	编辑器显示类似如下的信息:

```shell
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
# new file: README
# modified: CONTRIBUTING.md
#~~~
".git/COMMIT_EDITMSG" 9L, 283C
```

​	默认的提交信息包含最后一次运行`git status`的输出, 如果想要在编辑器里显示更详细的修改信息, 可以使用`-v`选项. 当然, 也可以使用`-m`选项, 将提交信息与命令放在一行, 直接完成提交.

​	完成提交后将打印如下信息:

```shell
git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
```

​	其中包含了在哪个分支(master)提交的, 本次提交的完整SHA-1校验和是什么(463dc4f), 以及在本次提交中, 有多少文件修改过, 多少行的添加和删改.

​	每一次运行提交操作, 都是对项目做一次快照, 以后可以回到这个状态, 或者进行比较.



#### 跳过使用暂存区域

​	尽管使用暂存区域的方式, 可以精心准备要提交的内容, 但有时候会略显繁琐. Git提供了一个跳过使用暂存区域的方式, 只要在提交的时候, 给`git commit`加上`-a`选项, Git就会自动把所有已跟踪文件暂存起来一并提交, 从而==跳过已修改文件的`git add`步骤==(未跟踪的文件需要`git add`来进行跟踪).

```shell
git commit -a -m 'added new benchmarks'
```



#### 移除文件

​	要从Git中移除某个文件, 就必须要从已跟踪文件清单中移除(确切地说, 是从暂存区移除), 然后提交. 可以用`git rm`命令完成此项工作. 并连带从工作目录中删除指定的文件, 这样也不会在未跟踪文件清单看到它了. 

​	如果只是简单地从工作目录中手工删除文件, 运行`git status`时就会在未暂存清单中看到:

```shell
git status
On branch master
Your branch is up-to-date with 'origin/master'.
  Changes not staged for commit:
    (use "git add/rm <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)
          deleted:    PROJECTS.md
no changes added to commit (use "git add" and/or "git commit -a")
```

​	然后再运行`git rm`记录此次移除文件的操作.:

```shell
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Changes to be committed:
   (use "git reset HEAD <file>..." to unstage)
      deleted:    PROJECTS.md
```

​	下一次提交时, 该文件就不会纳入版本管理了. 如果删除之前修改过并已经放入到暂存区域的文件的话, 则必须要用强制删除选项`-f`. 这是一种安全特性, 用于防止误删除还没有添加到快照的数据, 这样的数据被Git恢复.

​	另外一种情况, 如果只是想把文件从Git仓库删除, 但仍然希望保留在当前工作目录中. 比如忘记添加`.gitignore`文件, 不小心把一个很大的日志文件或一堆`.a`这样的编译生成文件添加到暂存区时, 尤其适合这一做法. 这时, 需要使用`--cached` 选项:

```shell
git rm --cached README
```

​	`git rm`命令后面可以列出文件或者目录的名字, 也可以使用`glob`模式, 比如

```shell
git rm log/\*.log
```

​	这里的反斜杠`\`是因为Git有它自己的文件模式扩展匹配方式, 所以不用shell帮忙展开, 此处命令删除`log/`目录下扩展名为`.log`的所有文件. 类似的比如:

```shell
git rm \*~
```

​	该命令为删除以`~`结尾的所有文件.



#### 移动文件

​	移动文件, 也同时可以用来给文件改名:

```shell
git mv file_from file_to
```

​	此时查看`git status`的输出:

```shell
$ git mv README.md README
$ git status
On branch master
Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)
      renamed:    README.md -> README
```

​	其实, 运行`git mv`相当于运行了下面三条命令:

```shell
mv README.md README
git rm README.md
git add README
```

​	这样分开操作, Git也会意识到这是一次改名, 所以不管何种方式结果都一样. 所以直接用`git mv`轻便得多. 不过如果用了其它工具进行了批处理改名, 别忘了执行后面的两条命令. 因为此时相当于已经做了第一步.



### 查看提交历史

​	clone专门用于演示的simplegit项目, 运行`git log`, 观察输出:

```shell
git clone https://github.com/schacon/simplegit-progit
```

```shell
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700
      changed the version number
commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700
      removed unnecessary test
commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700
			first commit

```

​	默认不用任何参数的话, `git log`会按提交时间列出所有更新, 最近的更新排在最上面. 每一条更新记录包括SHA-1校验和, 作者的名字和电子邮件地址, 提交时间, 以及提交说明.

​	可以使用`-p`选项, 显示每次提交的内容差异. 也可以加上`-2`来显示最近两次提交. 该选项除了显示基本信息外, 还附带每次commit的变化. 

​	如果想看到每次提交的简略的统计信息, 可以使用`--stat`选项.

​	另一个选项是`--pretty`, 这个选项可以指定使用不同于默认格式的方式展示提交历史.这个选项有一些内建的子选项可以使用. 比如`oneline`将每个提交放在一行显示, 查看的提交数很大时很有用. 另外还有`short`, `full`,`fuller`可以使用, 展示的信息或多或少有些不同.

```shell
git log --pretty=oneline
```

​	最有意思的是`format`, 可以定制要显示的记录格式. 这样的输出对后期提取分析格外有用. 因为自己指定的格式可以确定不会随着Git的更新而发生改变.

```shell
$ git log --pretty=format:"%h - %an, %ar : %s"
ca82a6d - Scott Chacon, 6 years ago : changed the version number
085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
a11bef0 - Scott Chacon, 6 years ago : first commit
```

​	```git log --pretty=fomat```, 常用的选项列出了常用的格式占位符写法及其代表的意义:

![image-20240427211243672](./assets/image-20240427211243672.png)

​	这里分别提到了==作者==和==提交者==, 作者指的是实际做出修改的人, 提交者指的是最后将此工作成果提交到仓库的人. 比如, 当你为某个项目发布补丁, 然后某个核心成员将你的补丁并入项目时, 你就是作者, 而那个核心成员就是提交者. 这是适用于==分布式Git==的概念, 后面会详细介绍.

​	当oneline或format与另一个log选项``--graph`结合使用时尤其有用, 这个选项添加了一些ASCII字符串来形象地展示分支和合并历史. 

```shell
$ git log --pretty=format:"%h %s" --graph
* 2d3acf9 ignore errors from SIGCHLD on trap
*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
|\
| * 420eac9 Added a method for getting the current branch.
* | 30e367c timeout code and tests
* | 5a09431 add timeout protection to grit
* | e1193f8 support for heads with slashes in them
|/
* d6016bc require time for xmlschema
*  11d191e Merge branch 'defunkt' into local
```



​	`git log`的常用选项

![image-20240427233414100](./assets/image-20240427233414100.png)



#### 限制输出长度

​	除了定制输出格式的选项之外, `git log`还有许多非常实用的限制输出长度的选项, 也就是只输出部分提交信息. 比如之前看到的`-2`, 它只会显示最近的两条提交.  实际上, 这是`-<n>`选项的写法, 其中n可以是任何整数, 表示仅显示最近的若干条提交. 不过实践中并不太常用这个选项, Git在输出所有提交时会自动调用分页程序.

​	`--since`和`--until`是按照时间作限制的选项, 下面的命令时列出所有最近两周内的提交:

```shell
git log --since=2.weeks
```

​	这个命令可以在多个格式下工作, 比如说具体的某一天`2008-01-15`, 或者相对多久以前`2 years 1 day 3 minutes ago`.

​	还可以给出若干==搜索==条件, 列出符合的提交. 用`--author`选项显示指定作者的提交, 用`--grep`选项搜索提交说明中的关键字. 如果要同时满足上面两个选项的搜索条件, 就必须用`--all-match`选项. 否则, 满足任意一个条件的提交就会被匹配出来.

​	另一个非常有用的筛选选项是`-S`, 可以列出那些添加或移除了某些字符串的提交, 比如, 想找出添加或移除了某一个特定函数的引用的提交, 可以像下面这样使用:

```shell
git log -Sfunction_name
```

​	最后一个很实用的`git log`选项时路径(path), 如果只关心某些文件或目录的历史提交, 可以在git log选项的最后指定它们的路径. 因为这是放在最后位置上的选项, 所以用两个(--)隔开之前的选项和后面限定的路径名.

​	以下是限制git log输出的常用选项:

![image-20240428002327822](./assets/image-20240428002327822.png)



### 撤销操作 

​	接下来, 学习撤销操作. 注意: ==有些撤销操作是不可逆的, 这是在使用Git的过程中, 会因为操作失误而导致之前的工作丢失的少有的几个地方之一==

​	比如有时候提交完了发现有几个文件漏掉了没有添加, 或者==提交信息填写错了==, 此时可以运行带有`--amend`选项的提交命令尝试==重新提交==:

```shell
git commit amend
```

​	这个命令会再次将暂存区中的文件提交, 只不过会合并到上次提交中. 比如再刚刚执行的`git commit`之后, 再次执行带有该选项的commit, 那么快照会保持不变, 只是会修改提交信息. 使用了`amend`选项后, 就无需再使用`-m`选项, 因为Git会打开文本编辑器, 显示上次的提交信息, 以便我们对提交信息进行修改.  如果漏掉了文件, 则需要把文件添加到暂存区, 然后再通过`amend`选项进行提交.

```shell
$ git commit -m 'initial commit' 
$ git add forgotten_file 
$ git commit --amend
```

​	上面的三条命令最后只有一个提交, 第二次提交会代替第一次提交的结果.





#### 取消暂存的文件

​	`git reset`来取消暂存. 这个命令还有一个可选项`--hard`, 不过这会使`git reset`成为一个危险的命令, 因为这可能导致工作目录中的所有当前进度丢失. 不加选项地调用`git reset`并不危险, 它只会修改暂存区域.



#### 撤销对文件的修改

​	如果在工作目录中修改了一个文件, 但是此时想要还原成上次提交时的样子, 可以使用`git checkout`命令. 不过, 这也是一个非常危险的命令, 因为对那个文件做的那些修改都会消失. 这里的丢失意味着无法再通过其它操作恢复了, 所以非常危险.

​	如果想的是保留对那个文件所做的修改, 但是现在仍然需要撤销, 也并不需要复制那个文件, Git可以提供==分支==来保存这个进度.

​	最后记住, Git中任何==已提交==的东西几乎总是可以恢复的, 甚至那些被删除的分支中的提交或使用`--amend`选项覆盖的提交也可以恢复. 但是, 任何为提交的东西, 被撤销后, 很可能再也找不到了.



### 远程仓库的使用

​	远程仓库是指==托管==在因特网或其它网络中的你的项目的版本库. 通常有些仓库对你只读, 有些则可以读写. 与他人协作涉及管理远程仓库以及根据需要推送或拉取数据. 管理远程仓库包括了解如何添加远程仓库, 移除无效的远程仓库, 管理不同的远程分支, 并定义它们是否被跟踪等.



#### 查看远程仓库

​	查看已经配置的远程仓库服务器, 可以使用`git remote`, 它会列出你指定的每一个远程服务器的简写. 如果克隆了自己的仓库, 那么至少可以看到origin, 这是Git给你克隆的仓库服务器的默认名字. 也可以指定选项`-v`, 会显示需要读写远程仓库使用的Git保存的简写与其对应的Url.

​	

#### 添加远程仓库

​	运行`git remote add <shortname> <url>`添加一个新的远程仓库, 同时可以指定一个简写.



#### 从远程仓库中抓取与拉取

​	从仓库中获得数据, 可以执行`git fetch [remote-name]`,  这个命令会去访问远程仓库, 从中拉取==所有==你还没有的数据. 执行完成后, 你将会拥有那个远程仓库中所有分支的引用, 可以随时合并或查看.

​	如果使用`clone`命令克隆了一个仓库, 命令会自动将其添加为远程仓库并默认以origin为简写.

​	必须注意, `git fetch`只会拉取克隆(或上一次抓取)后推送的所有新工作到本地仓库, 但是并不会自动合并或修改你当前的工作. 你必须手动将其合并入你的工作.

​	如果有一个分支设置为跟踪一个远程分支, , 可以使用`git pull`, 自动抓取然后合并远程分支到当前分支. 默认情况下, `git clone`会自动设置本地main分支跟踪克隆的远程仓库的main分支.



#### 推送到远程仓库

​	`git push [remote-name] [branch-name]`, 当你有所克隆服务器的写入权限并且之前没有人推送过时, 这条命令才能生效. 



#### 查看远程仓库

​	如果想要查看远程仓库的更多信息, 可以使用`git remote show [remote-name]`.

```	shell
$ git remote show origin 
* remote origin   
  Fetch URL: https://github.com/schacon/ticgit   
	Push  URL: https://github.com/schacon/ticgit   
	HEAD branch: master   
	Remote branches:     
		master tracked     
		dev-branch tracked   
	Local branch configured for 'git pull':     
		master merges with remote master   
	Local ref configured for 'git push':     
		master pushes to master (up to date)
```

​	它同样会列出远程仓库的url与跟踪分支的信息. 上述信息说明正处于master分支, 以及执行`git pull`和`git push`时所进行的操作.



#### 远程仓库的移除与重命名

​	如果想要重命名引用的名字可以运行`git remote rename`去修改一个远程仓库的简写名.

```shell
git remote rename pb paul
```

​	如果想要移除一个远程仓库, 可以使用`git remote rm`.

```shell
git remote rm paul
```



### 打标签

​	Git可以给历史中的某一个提交打上标签, 以示重要. 比较有代表性的是人们经常使用这个功能来标记发布结点(v1.0)等. 



#### 列出标签

​	`git tag`. 这个命令以字母顺序列出标签, 但是它们出现的顺序并不重要.

​	也可以使用特定的模式查找标签, 例如Git自身的源代码仓库包含标签数量超过500个, 如果只对1.8.5系列感兴趣, 可以运行:

```shell
git tag -l 'v1.8.5*'
```



#### 创建标签

​	Git使用两种主要类型的标签:

-  轻量标签(lightweight)
- 附注标签(annotated)

​	一个轻量标签很像一个不会改变的分支, 它只是一个特定提交的引用.

​	附注标签是存储在Git仓库中的一个完整对象. 它们是可被校验的; 其中包含打标签者的名字, 电子邮件地址, 日期时间; 还有一个标签信息; 并且可以使用GNU Privacy Guard(GPG)签名与验证. 

​	通常建议创建附注标签, 这样可以拥有以上所有信息, 如果只是想用一个临时的标签, 或者不想保存那些信息, 轻量标签也是可以的.



#### 附注标签

​	最简单的创建方式是运行`tag`命令时指定`-a`选项:

```shell
git tag -a v1.4 -m 'my version 1.4'
```

​	使用`git show`命令可以查看到标签信息与对应的提交信息:

```shell
git show v1.4
```



#### 轻量标签

​	轻量标签本质上是将提交校验和存储到一个文件中, 没有保存任何其它信息.创建轻量标签不需要使用`-a`, `-s`, `-m`选项, 只需要提供标签名字.

```shell
git tag v1.4-lw
```

​	如果运行`git show v1.4-lw`, 将不会看到额外信息, 只会看到提交信息.



#### 后期打标签

​	也可以对过去的提交打标签.

```shell
git tag -a v1.2 校验和(部分校验和)
```

​	



#### 共享标签

​	默认情况下, `git push`不会传送标签到远程仓库服务器上. 必须显示地推送标签到共享服务器上. 这个过程就像共享远程分支一样, 你可以运行`git push origin [tagname]`.

```shell	
git push origin v1.5
```

​	如果想要一次性推送很多标签, 也可以使用带有`--tags`选项的`git push`命令, 将会把所有不在远程仓库服务器上的标签全部传送到那里.

```shell
git push origin --tags
```

​	现在, 当其他人从仓库中克隆或拉取, 也能得到你的那些标签.





#### 检出标签

​	如果你想要工作目录与仓库中特定的标签版本完全一样, 可以使用 `git checkout -b [branchname] [tagname]`在特定的标签上创建一个新分支:

```shell
git checkout -b version2 v2.0.0
```

​	如果在此基础上进行了一次提交, version2分支会因为改动向前移动了, 那么version2分支就会和v2.0.0标签稍微有些不同.



### Git别名

​	Git并不会在你输入部分命令时自动推断出你想要的命令. 如果不想每次都输入完整的Git命令, 可以通过`git config`文件来轻松地为每一个命令设置一个别名. 如:

```shell
$ git config --global alias.co checkout 
$ git config --global alias.br branch 
$ git config --global alias.ci commit 
$ git config --global alias.st status
```

 	利用此技术创建一个你认为应该存在的命令. 如, 为了解决取消暂存文件的易用性问题, 可以向Git中添加你自己的取消暂存别名:

```shell
$ git config --global alias.unstage 'reset HEAD --'
```

​	这会使下面的两个命令等价:

```shell
$ git unstage fileA 
$ git reset HEAD -- fileA
```

​	为了更轻松地看到最后一次提交:

```shell
$ git config --global alias.last 'log -1 HEAD'
```

​	如果想要执行外部命令, 而不是一个Git子命令, 可以在命令前面加入`!`符号. 如下是将`git visual`定义为`gitk`的别名:

```shell
$ git config --global alias.visual '!gitk'
```





## Git分支

​	几乎所有的版本控制系统都以某种形式支持分支. 利用分支意味着你可以把你的工作从开发主线上分离出来, 以免影响开发主线. 在很多版本控制系统中这是一个略微低效的过程, 因为常常需要完全创建一个源代码目录的副本. 对于大项目来说, 这样的过程会耗费许多时间.

​	有人把Git的分支模型称为它的‘必杀技特性’, 使Git从众多版本控制系统中脱颖而出. 为何Git的分支模型如此出众呢? Git处理分支的方式可谓是难以置信的轻量, 创建新分支这一操作几乎能在瞬间完成, 并且在不同分支之间的切换操作也是一样便捷. 与其它版本控制工具不同, Git鼓励在工作流程中频繁地使用分支与合并, 哪怕一天之内进行许多次. 理解和精通这一特性, 你便会意识到Git是如此的强大而又独特, 并且==从此真正改变你的开发方式==.



### 分支简介

​	为了更好地理解Git处理分支的方式, 我们先回顾一下Git保存数据的方式. Git保存的不是文件的变化或者差异, 而是一系列不同时刻的文件快照.

​	在进行提交操作时, Git会保存一个提交对象(commit object), 该提交对象包含一个指向暂存内容快照的指针. 而且不仅仅是这样, 该提交对象还包含了作者的姓名和邮箱, 提交时输入的信息以及指向它的父对象的指针. 首次提交产生的提交对象没有父对象, 普通提交操作产生的提交对象有一个父对象, 而由多个分支合并产生的提交对象有多个父对象. 

​	假设现在有一个工作目录, 里面包含了三个将要被暂存和提交的文件. 暂存操作会为每一个文件计算校验和, 然后会把当前版本的文件快照保存到Git仓库中(Git使用blob对象来保存它们), 最后将校验和加入到暂存区域等待提交.

​	当使用`git commit`进行提交操作时, Git会先计算每一个子目录的校验和, 然后在Git仓库中这些校验和保存为树对象. 随后, Git便会创建一个提交对象, 它除了包含上面提到的信息, 还包含指向这个树对象的指针. 如此一来, Git就可以在需要的时候重现此次保存的快照.

​	==现在, Git仓库中有五个对象: 三个blob对象(保存着文件快照), 一个树对象(记录着目录结构和blob对象索引)以及一个提交对象(包含着指向前述树对象的指针和所有提交信息).==

![image-20240428214016896](./assets/image-20240428214016896.png)



​	做些修改后再次提交, 那么这次产生的提交对象会包含一个指向上次提交对象(父对象)的指针.

![image-20240428214142048](./assets/image-20240428214142048.png)

​	Git的分支, 其实本质上仅仅==是指向提交对象的可变指针==. Git的默认分支是`master`, 在多次提交操作之后, 你其实已经有一个指向最后那个提交对象的`master`分支. 它会在每次的提交操作中自动向前移动. 

![image-20240428220010946](./assets/image-20240428220010946.png)

 	另外, master分支并不特殊, 它跟其它分支没有什么区别, 之所以几乎每个仓库都含有master分支, 是因为`git init`命令默认创建它, 并且大多数人都懒得去改动它.



#### 分支创建

​	`git branch`是创建新分支的命令, 运行这个命令后, Git所做的很简单, 它只是创建了一个可以移动的新的指针.

```shell
git branch testing
```

​	这会在==当前所在的提交对象上==创建一个指针.

![image-20240428220856928](./assets/image-20240428220856928.png)

​	那么Git又是怎么知道当前在哪一个分支上呢? 也很简单, 它还有一个名为`HEAD`的特殊指针(注意它和SVN里的HEAD概念完全不同). 在Git中, 它是一个指针, 指向当前所在的本地分支. 当前, 仍然在`master`分支上, 因为`git branch`仅仅创建一个新分支, 并不会自动切换到新分支中去.

![image-20240428221819383](./assets/image-20240428221819383.png)

​	可以使用`git log --oneline --decorate`命令查看各个分支当前所指的对象, 提供这一功能的选项是`--decorate`.

```shell
$ git log --oneline --decorate 
f30ab (HEAD, master, testing) add feature #32 - ability to add new 
34ac2 fixed bug #1328 - stack overflow under certain conditions 
98ca9 initial commit of my project
```



#### 分支切换

​	要切换到一个已存在的分支, 使用命令`git checkout`命令. 

```shell
git checkout testing
```

​	这样`HEAD` 就切换到`testing`分支了.

![image-20240428222450563](./assets/image-20240428222450563.png)

​	

​	现在不妨再提交一次.

![image-20240428222601815](./assets/image-20240428222601815.png)

​	可以看到, `testing`分支向前移动了, 但是`master`分支没有. 如果现在再切换回`master`分支.

```shell
git checkout master
```



![image-20240428222749605](./assets/image-20240428222749605.png)

​	这条命令做了两件事: ==一是使 `HEAD`指针指回`master`分支, 二是将工作目录恢复成`master`分支所指向的快照内容.== 此时将忽略`testing`分支所做的修改, 以便于向另一个方向进行开发.

​	不妨再做些修改并提交.

![image-20240428223417554](./assets/image-20240428223417554.png)

​	现在, 提交历史发生了分叉. 现在你可以在不同分支间不断地来回切换和工作, 并在时机成熟时将它们合并起来.	

​	运行`git log --oneline --decorate --graph --all`, 它将会输出你的提交历史, 各个分支的指向以及项目的分支分叉情况.

```txt
$ git log --oneline --decorate --graph --all 
* c2b9e (HEAD, master) made other changes 
| * 87ab2 (testing) made a change 
|/ 
* f30ab add feature #32 - ability to add new formats to the 
* 34ac2 fixed bug #1328 - stack overflow under certain conditions 
* 98ca9 initial commit of my project
```



​	由于Git的分支实质上仅是包含所指对象校验和的文件, 所以它的创建和销毁都异常高效, 创建一个新分支就像是往一个文件中写入41个字节(40个字符和1个换行符), 如此简单怎能不快?



### 分支的新建与合并

​	实际工作中, 可能会有如下步骤:

1. 开发某个网站
2. 为实现某个新的需求, 创建一个分支
3. 在这个分支上开展工作



​	正在此时, 你突然街道一个电话说有个很严重的bug需要修复, 将按照如下方式处理:

1. 切换到你的线上分支(production branch)
2. 为这个紧急任务新建一个分支, 并在其中修复它
3. 在测试通过后, 切回线上分支, 然后合并这个修补分支, 最后将改动推送到线上分支
4. 切回最初工作的分支上, 继续工作



####  新建分支

​	首先, 假设目前的工作状态如下:

![image-20240429094914423](assets/image-20240429094914423.png)



​	现在, 准备解决公司使用的问题追踪系统中的#53问题, 想要新建一个分支并同时切换到那个分支上, 你可以运行一个带有`-b`参数的`git checkout`命令:

```shell
$ git checkout -b iss53
Switched to a new branch "iss53"
```

​	它是如下两条命令的简写:

```shell
$ git branch iss53
$ git checkout iss53
```

![image-20240429095314652](assets/image-20240429095314652.png)

​	针对这个问题做了一些工作后, 进行了提交. `iss53`分支不断向前推进, 因为你已经检出了该分支.

![image-20240429095502189](assets/image-20240429095502189.png)

​	现在, 接到那个电话, 有个紧急问题需要你来解决. 有了Git的帮助, 就==不用把这个紧急问题和`iss53`的修改混在一起==, 也不用花大力气来还原关于#53问题的修改, 然后再添加关于这个紧急问题的修改, 最后将这个修改提交到线上分支. 你要做的仅仅是切换回`master`分支.

​	有一点要注意, 切分支之前, 要留意你的工作目录和暂存区里那些没有被提交的修改, 它可能和你即将检出的分支产生冲突从而阻止Git切换分支. 最好的方法是, 在切换分支前, 保持一个好的干净的状态. 有一些方法可以绕过这个问题(保存进度stashing和修补提交commit amending), 在储藏和清理中会学习这两个命令. 现在暂且假设已经把你的修改全部提交了. 然后进行分支切换.

```shell
$ git checkout master
Switched to branch 'master'
```

​	切回之后, ==工作目录和在开始#53问题之前一模一样==, 现在可以专心修复紧急问题了.  请牢记, 当切换分支时, ==Git会重置你的目录==, 使其看起来像==回到了你在那个分支上最后一次提交的样子==. 

​	接下来为了解决该紧急问题, 建立一个针对该问题的分支(hotfix branch), 在该分支上工作并解决问题:

```shell
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ vim index.html
$ git commit -a -m 'fixed the broken email address'
[hotfix 1fb7853] fixed the broken email address
1 file changed, 2 insertions(+)
```

![image-20240429102910603](assets/image-20240429102910603.png)

​	进行测试通过后, 将其合并回`master`分支来部署到线上, 可以使用`git merge`.

```shell
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
  index.html | 2 ++
  1 file changed, 2 insertions(+)
```

​	这里需要注意执行`git merge`之后的快进(fast-forward)这个词, 由于当前`master`分支所指向的提交是`hotfix`的直接上游, 所以Git只是简单地将指针向前移动. 也就是说, 当执行合并操作时, 如果顺着一个分支下去是另一个分支, 那么Git在合并时, 只会简单地移动指针. 因为这种情况下的合并操作没有需要解决的分歧, 这就叫做"快进(fase-foward)".

![image-20240429103738430](assets/image-20240429103738430.png)

​	解决完这个紧急问题后, 准备回到被打断之前的工作中, 不过在此之前, 你==应该先删除`hotfix`分支==, 因为你已经不再需要它了, `master`分支已经指向了同一个位置. 可以使用带`-d`选项的`git branch`命令来删除分支.

```shell
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```



​	现在可以切回针对#53问题的那个分支.

```shell
$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'finished the new footer [issue 53]'
[iss53 ad82d7a] finished the new footer [issue 53]
1 file changed, 1 insertion(+)
```

![image-20240429104442288](assets/image-20240429104442288.png)

​	==在`hotfix`分支上做的工作并没有包含到`iss53`分支中, 如果你需要拉取`hotfix`分支所做的修改, 可以使用`git merge master`命令将`master`分支合并入`iss53`分支, 或者也可以等到`iss53`分支完成其使命, 再将其合并回master分支.==



#### 分支的合并

​	假设已经修正了#53的问题, 打算将你的工作合并入`master`分支, 这和之前向`master`合并`hotfix`分支差不多, 只需要==检出到==想要合并入的分支, 然后运行`git merge`命令. 

```shell
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html | 1 +
1 file changed, 1 insertion(+)
```

​	这和之前的合并有点不一样, 这是因为你的开发历史从一个更早的地方开始分叉开来(diverged). 因为`master`分支所在提交并不是`iss53`所在提交的直接上游, Git不得不做一些额外工作. ==出现这种情况的时候, Git会使用两个分支的末端所指的快照(`C4`和`C5`)以及这两个分支的工同祖先(`C2`), 做一个简单的三方合并.==

![image-20240429111013304](assets/image-20240429111013304.png)

​	和之前的快进不同, 这里Git将此次三方合并的结果做了一个新的快照并且自动创建了一个新的提交指向它. 这个被称作一次合并提交, 它的特别之处在于他不止有一个父提交.

![image-20240429111353773](assets/image-20240429111353773.png)



​	需要指出的是, Git会自行决定选取哪一个提交作为最优的共同祖先, 并以此作为合并的基础. 更古老的CVS系统或者SVN(1.5之前)需要用户自己选择最佳的合并基础. Git的这个优势使其在合并操作上比其他系统要简单很多.

​	合并进来之后, 同样不再需要`iss53`分支了. 现在可以==在任务追踪系统中关闭此项任务==, 并删除这个分支.

```shell
$ git branch -d iss53
```



#### 遇到冲突时的分支合并

​	有些时候合并操作不会如此顺利. 如果在两个分支中对==同一个文件的同一个部分==进行了不同的修改, Git就无法干净的合并它们. 如果你对#53问题的修改和紧急问题的修改都涉及到同一个文件的同一处, 在合并它们时就会产生合并冲突:

```shell
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

​	此时, Git做了合并, 但是没有自动创建一个新的合并提交. Git会暂停下来, 等待你去解决合并产生的冲突. 你可以在合并冲突后的任意时刻用`git status`命令来查看那些==因包含合并冲突而处于未合并(unmerged)状态==的文件:

```shell
$ git status
On branch master
You have unmerged paths.
	(fix conflicts and run "git commit")
Unmerged paths:
	(use "git add <file>..." to mark resolution)
		both modified: index.html
no changes added to commit (use "git add" and/or "git commit -a")
```

​	Git会在有冲突的文件中加入标准的==冲突解决标记==, 以方便你打开合并冲突的文件来手动解决冲突. 含有冲突的文件中会包含一些如下的特殊区段:

```shell
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
  please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

​	这表示`HEAD`所指的版本, 对于目前的例子来说也就是`master`分支所在的位置在这个区段的上半部分, 而`iss53`分支所指示的版本在下半部分. 为了解决冲突, 必须选择使用由`=======`分割的两个部分中的一个, 或者也可以自行决定合并这些内容. 例如, 你可以通过把这段内容换成下面的样子来解决冲突:

```shell
<div id="footer">
please contact us at email.support@github.com
</div>
```

​	上述的冲突解决方案仅保留了其中一个分支的修改, 并且`<<<<<<<`, `=======`,`>>>>>>>`这些行被完全删除了. 在解决了这些文件的冲突后, 对每个文件使用`git add`命令来将其标记为冲突已解决. 一旦暂存这些原本有冲突的文件, Git就会将它们标记为冲突已解决.

​	如果你想使用图形化工具来解决冲突, 可以运行`git mergetool`. 它会启动一个合适的可视化合并工具, 并带领你一步一步解决这些冲突.

​	你可以再次运行`git status`来确认所有的合并冲突都已被解决:

```shell
$ git status
On branch master
All conflicts fixed but you are still merging.
	(use "git commit" to conclude merge)
Changes to be committed:
	modified: index.html
```



​	这时可以输入`git commit`来完成合并提交. 默认情况下提交信息看起来像下面这个样子:

```txt
Merge branch 'iss53'
Conflicts:
index.html
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
# .git/MERGE_HEAD
# and try again.
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# All conflicts fixed but you are still merging.
#
# Changes to be committed:
# modified: index.html
#
```

​	如果你觉得还不够充分, 可以修改上述信息, 添加一些细节给未来检视这个合并的读者一些帮助, 告诉他们你是如何解决合并冲突的, 以及理由是什么.



### 分支管理

​	现在已经创建, 合并, 删除了一些分支, 接下来了解一些常用的分支管理工具. 

​	`git branch`不只可以创建和删除分支. 如果不加任何参数地运行它, 会得到当前所有分支的一个列表:

```shell
$ git branch
  iss53
* master   # * 代表现在检出的那一个分支. 也就是当前HEAD指针指向的分支
  testing
```

​	如果要查看每一个分支的最后一次提交, 可以运行`git branch -v`命令:

```shell
$ git branch -v
  iss53 93b412c fix javascript issue
* master 7a98805 Merge branch 'iss53'
  testing 782fd34 add scott to the author list in the readmes
```



​	如果想要查看已经合并或者尚未合并到当前分支的分支可以使用`--merged`与`--no-merged`选项.

```shell
$ git branch --merged
  iss53
* master
```

​	这个列表中没有`*`号的分支通常可以通过`-d`选项删除掉, 因为你已经将它们合并到了当前分支, 所以不会失去任何东西.

```shell
$ git branch --no-merged
  testing
```

​	这里显示了还未合并到当前分支的分支, 如果要使用`-d`选项删除它会失败:

```shell
$ git branch -d testing
error: The branch 'testing' is not fully merged.
If you are sure you want to delete it, run 'git branch -D testing'.
```

​	如果确实想删除并丢掉那些工作, 可以使用`-D`选项强制删除.



### 分支开发工作流

​	这里会介绍一些常见的利用分支开发的工作流程. 这是一些典型的工作模式.

#### 长期分支

​	在整个项目开发周期的不同阶段, 你可以同时拥有多个开放的分支, 可以定期把某些特性分支合并入其他分支中.

​	比如只在`master`分支上保留完全稳定的代码, 还有一些名为`develop`或`next`的平行分支, 被用来做后续开发或者测试稳定性, 这些分支不必保持绝对稳定, 但是一旦达到稳定状态, 它们就可以被合并入`master`分支了.

​	稳定分支的指针总是在提交历史中落后一大截, 而前沿分支的指针往往比较靠前.

![image-20240429141834059](assets/image-20240429141834059.png)

​	通常把他们想象成流水线(work silos)可能更好理解一点, 那些经过测试考研的提交会被遴选到更加稳定的流水线上去.

![image-20240429142947152](assets/image-20240429142947152.png)

​	一些大型项目还有一个`proposed(建议)`或`pu:proposed updates(建议更新)`分支, 它可能因包含一些不成熟的内容而不能进入`next`或`master`分支.

​	这么做的目的是使你的分支具有不同级别的稳定性, 当它们具有一定程度的稳定性后, 再把它们合并入具有更高级别稳定性的分支中. 



#### 特性分支

​	特性分支是一种短期分支, 对任何规模的项目都适用, 它被用来实现单一特性或其相关工作. 

​	考虑一个例子, 你在`master`分支上工作到`C1`, 这时为了解决一个问题而新建`iss91`分支, 在`iss91`分支上工作到`C4`, 然后对于那个问题你又有了新的想法, 于是又创建了一个`iss91v2`分支试图用另一种方法解决那个问题, 接着又回到`master`分支工作了一会, 你又冒出了一个不太确定的想法, 你便在C10的时候新建一个dumbidea分支, 并在上面做些实验. 提交历史看起来像下面这个样子:

![image-20240429145126128](assets/image-20240429145126128.png)

​	最后, 你决定使用第二个方案来解决那个问题, 即使用在`iss91v2`分支中的方案, 另外, 你将`dumbidea`分支拿给同事看后, 结果发现这是个==惊人之举==. 这时, 你可以抛弃`iss91`分支(即丢弃`C5`和`C6`提交), 然后把另外两个分支合并入主干分支. 最后提交历史看起来如下:

![image-20240429145446034](assets/image-20240429145446034.png)



### 远程分支

​	远程引用是对仓库的引用(指针), 包括分支, 标签等等. 可以通过`git ls-remote (remote)`来显示地获得远程引用的完整列表, 或者通过`git remote show (remote)`获得远程分支的更多信息. 不过, 一个更常见的做法是利用远程跟踪分支.

​	==远程跟踪分支是远程分支状态的引用.== 它们是你==不能移动的本地引用==, 当你做任何网络通信操作时, 它们会自动移动. 远程跟踪分支像是你上次连接到远程仓库时, 那些分支所处状态的书签. 

​	它们以`(remote)/(branch)`形式命名. 例如, 如果你想看最近一次与远程仓库`origin`通信时`master`分支的状态, 可以查看`origin/master`分支.

​	例子: 假设你的网络里有一个在`git.ourcompany.com`的Git服务器. 如果你从这里克隆, Git的`clone`命令会为你自动将其命名为`origin`, 拉取它的所有数据, 创建一个指向它的`master`分支的指针, 并且在本地将其命名为`origin/master`. Git也会给你一个与origin的`master`分支在指向同一个地方的本地`master`分支, 这样你就有工作的基础. 

​	`git int`会指定`master`做为默认的起始分支的名字; `git clone`会指定`origin`作为默认的远程仓库的名字. 如果运行`git clone -o booyah`, 那么远程仓库名字会被指定为`booyah`.

![image-20240429155525864](assets/image-20240429155525864.png)

​	如果你在本地`master`分支上进行了提交的同时, 远程仓库的`master`被其他人的提交更新了, 那么你的提交历史将向不同方向前进.

![image-20240429155915721](assets/image-20240429155915721.png)

​	如果要同步你本地的工作, 运行`git fetch origin`命令. 从远程服务器抓取本地没有的数据, 并且更新本地数据库, ==移动`origin/master`指针指向新的, 更新后的位置==.

![image-20240429160450104](assets/image-20240429160450104.png)



​	为了演示多个远程仓库与远程分支的情况, 我们假定有另一个内部Git服务器, 仅用于你的sprint小组的开发工作. 这个服务器位于`git.team1.ourcompany.com`. 你可以运行`git remote add` 命令添加一个新的远程仓库引用到当前的项目. 将这个远程仓库命名为`teamone`, 将其作为整个URL的缩写.

![image-20240430101045771](assets/image-20240430101045771.png)

​	现在可以运行`git fetch teamone`来抓取远程仓库`teamone`有而本地没有的数据. ==因为那台服务器上现有的数据是`origin`服务器上的一个子集, 所以Git并不会抓取数据而是会设置远程跟踪分支`teamone/master`指向`teamone`的`master`分支.==

![image-20240430141544492](assets/image-20240430141544492.png)



#### 推送

​	当想要公开分享一个分支时, 需要将其推送到有写入权限的远程仓库上. 

​	如果希望和别人一起在名为`serverfix`的分支上工作, 你可以像推送第一个分支那样推送它. 运行`git push (remote) (branch)`:

```shell
$ git push origin serverfix
Counting objects: 24, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
Total 24 (delta 2), reused 0 (delta 0)
To https://github.com/schacon/simplegit
  * [new branch] serverfix -> serverfix
```



​	注意, 这里你可能已经对每次推送时输入用户名与密码感到了厌烦. 如果不想在每一次推送时都设置用户名密码, 可以设置一个'credential cache'(验证缓存). 最简单的方式就是将其保存在内存中几分钟, 可以简单地运行`git config --global credential.helper cache`来设置它.

​	

​	下一次其他协作者从服务器上抓取数据时, 他们会在本地生成一个远程分支`origin/serverfix`, 指向服务器的`serverfix`分支:

```shell
$ git fetch origin
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 3 (delta 0)
Unpacking objects: 100% (3/3), done.
From https://github.com/schacon/simplegit
  * [new branch] serverfix -> origin/serverfix
```



​	要特别注意的一点是当抓取到新的远程跟踪分支时, 本地不会自动生成一份可编辑的副本(拷贝). 也就是, 在这种情况下, 不会有一个新的`serverfix`分支, 只有一个不可以修改的`origin/serverfix`指针.

​	可以运行`git merge origin/serverfix`将这些工作合并到当前所在的分支. 如果想要在自己的`serverfix`分支上工作, 可以将其建立在远程跟踪分支之上:

```shell
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

​	这会给你一个用于工作的本地分支, 并且起点位于`origin/serverfix`.



#### 跟踪分支

​	跟踪分支是与远程分支有直接关系的本地分支. 如果在一个跟踪分支上输入`git pull`, Git能自动识别去哪个服务器上抓取, 合并到哪个分支.

​	当克隆一个仓库时, 它通常会自动地创建一个跟踪`origin/master`的`master`分支. 也可以手动设置其它的跟踪分支, 或者不跟踪`master`分支. 最简单的就是刚刚看到的例子`git checkout -b [branch] [remotename]/[branch]`. 这是一个十分常用的操作所以Git提供了`--track`快捷方式:

```shell
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```



​	设置已有的本地分支跟踪一个刚刚拉取下来的远程分支, 或者想要修改正在跟踪的上游分支, 可以在任意时间使用`-u`或`--set-upstream-to`选项运行`git branch`来显式地设置:

```shell
$ git branch -u origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
```

​	当设置好跟踪分支后, 可以通过`@{upstream}`或`@{u}`快捷方式来引用它. 所以在`master`分支时并且它正在跟踪`origin/master`时, 如果愿意的话可以使用`git merge @{u}`来代替`git merge origin/master`.

​	查看设置的所有跟踪分支, 可以使用`git branch`的`-vv`选项. 这会将所有的本地分支列出来并且包含更多的信息, 比如每一个分支正在跟踪哪个远程分支与本地分支是否是领先, 落后或是都有.

```shell
$ git branch -vv
  iss53 7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master 1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing 5ea463a trying something new
```

​	`ahead`表示领先, `behind`表示落后.

​	需要重点注意的一点是这些数字的值来自于你从每个服务器上最后一次抓取的数据. 这个命令并没有连接服务器, 它只会告诉你关于本地缓存的服务器数据. 如果想要统计最新的领先与落后数字, 需要在运行此命令前抓取所有的远程仓库. `git fetch --all; git branch -vv`.



#### 拉取

​	当`git fetch`命令从服务器上抓取本地没有的数据时, 它并不会修改工作目录中的内容. 它只会获取数据, 然后让你自己合并. 

​	`git pull`在大多数情况下它的含义是一个`git fetch`紧接着一个`git merge`命令. 如果有一个分支设置好跟踪哪个远程分支, `git pull`会查找当前分支所跟踪的服务器与分支, 从服务器上抓取数据然后尝试合并入那个远程分支.

​	Pro Git的作者建议单独显式地使用`fetch`和`merge`命令更好.



#### 删除远程分支

​	如果你确保某一个远程分支不会再被使用了, 可以通过带有`--delete`的`git push`命令来删除一个远程分支. 注意这不会删除本地分支, 删除本地分支的命令前面学习过.

```shell
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
 - [deleted] serverfix
```





### 变基

​	在Git中整合来自不同分支的修改主要有两种方式: `merge`和`rebase`.



#### 变基的基本操作

​	先来回顾一下合并.

![image-20240430165612985](assets/image-20240430165612985.png)

​	之前面对这样的分叉, 我们使用的是`merge`命令. 它会把两个分支的最新快照(C3和C4)以及二者最近的共同祖先(C2)进行三方合并, 合并的结果是生成一个新的快照(并提交).

![image-20240430165753082](assets/image-20240430165753082.png)

​	其实, 还有一种做法: ==你可以提取在`C4`中引入的补丁和修改, 然后在`C3`的基础上再应用一次. 在Git中, 这种操作就叫做变基.== 你可以使用`rebase`命令将提交到某一分支上的所有修改都移至另一分支上, 就好像"重新播放"一样.

​	在上面这个例子中, 运行:

```shell
$ git checkout experiment 
$ git rebase master 
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

​	==它的原理是首先找到这两个分支(即当前分支`experiment`、变基操作的目标基底分支`master`)的最近共同祖先`C2`, 然后对比当前分支相对于该祖先的历次提交, 提取相应的修改并存为临时文件, 然后将当前分支指向目标基地`C3`, 最后以此将之前另存为临时文件的修改依序应用.==

![image-20240430212607019](./assets/image-20240430212607019.png)

​	现在回到`master`分支, 进行一次快进合并.

```shell
$ git checkout master 
$ git merge experiment
```

![image-20240430214232671](./assets/image-20240430214232671.png)

​	此时, `C4'`指向的快照就和上面使用`merge`命令的例子中`C5`指向的快照一模一样了. 这两种整合方法的最终结果没有任何区别, 但是变基使得提交历史更加整洁. 你在查看一个经过变基的分支的历史记录时会发现, 尽管实际的开发工作是并行的, 但它们看上去就像是先后串行一样, 提交历史是一条直线没有分叉.

​	一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁, 例如向某个别人维护的项目贡献代码时, 你应该先在自己的分支里开发, 当开发完成时你需要先将你的代码变基到`origin/master`上, 然后再向主项目提交修改. 这样项目的维护者就不再需要进行整合工作, 只需要快进合并便可.

​	请注意, 无论是通过变基, 还是通过三方合并, 整合的最终结果所指向的快照始终是一样的, 只不过提交历史不同罢了. ==变基是将一系列提交按照原有次序依次应用到另一分支上, 而合并是把最终结果合在一起.==



#### 更有趣的变基例子

​	在对两个分支进行变基时, 所生成的“重演"并不一定要在目标分支上应用, 也可以指定另外的一个分支进行应用. 就像从一个特性分支里再分出一个特性分支的提交历史中的例子这样. 你创建了一个特性分支`server`, 为服务端添加了一些功能, 然后提交了`C3`和`C4`. 然后从`C3`上创建了特性分支`client`, 为客户端添加了一些功能, 提交了`C8`和`C9`. 最后, 又回到`server`分支, 又提交了`C10`.

![image-20240501215346145](./assets/image-20240501215346145.png)

​	假设你希望将`client`中的修改合并到主分支并发布, 但暂时不想合并`server`中的修改. 这时, 就可以使用`git rebase`命令的`--onto`选项, 选中在`client`分支里但不在`server`分支里的修改(即`C8`和`C9`), 将它们在`master`分支上重演:

```shell
git rebase --onto master server client
```

​	以上命令的意思是: 取出`client`分支, 找出处于`client`分支和`server`分支的共同祖先之后的修改, 然后把它们在`master`分支上重演一遍. 这理解起来有点复杂, 不过效果非常酷. 

![image-20240501215757569](./assets/image-20240501215757569.png)

​	现在可以快进合并`master`分支了.

![image-20240501220558170](./assets/image-20240501220558170.png)

​	接下来你决定将`server`分支中的修改也整合进来. 使用`git rebase [basebranch] [topicbranch]`命令可以直接将特性分支(即本例中的`server`)变基到目标分支(即`master`)上. 这样做能省去你先切换到`server`分支, 再对其执行变基命令的多个步骤.

```shell
git rebase master server
```

​	如图将server中的修改变基到master上所示, server中的代码被“续"到了master后面.

![image-20240505185741106](./assets/image-20240505185741106.png)

​	然后就可以快进合并主分支master了:

```shell
$ git checkout master 
$ git merge server
```

​	至此, `client`和`server`分支中的修改都已经合并到主分支里去了, 你可以删除这两个分支, 最终提交历史会变成下图中的样子:

```shell
$ git branch -d client 
$ git branch -d server
```

![image-20240505190232122](./assets/image-20240505190232122.png)



#### 变基的风险

​	要记住一条准则: ==不要对在你的仓库外有副本的分支执行变基.==

​	变基操作的实质是丢弃一些现有的提交, 然后相应地新建一些内容一样但实际上不同的提交. 如果你已经将提交推送至某个仓库, 而其他人也已经从该仓库拉取提交并进行了后续工作, 此时, 如果你用`git rebase`命令重新整理了提交并再次推送, 你的同伴因此将不得不再次将他们手头的工作与你的提交进行整合, 如果接下来你还要拉取并整合他们修改过的提交, 事情就会变得一团糟.

​	例子: 假设你从一个中央服务器克隆然后在它的基础上进行了一些开发. 你的提交历史如图所示:

![image-20240505191301575](./assets/image-20240505191301575.png)

​	然后, 某人又向中央服务器提交了一些修改, 其中还包括一次合并. 你抓取了这些在远程分支上的修改, 并将其合并到你本地的开发分支, 然后你的提交历史就会变成这样:

![image-20240505191542836](./assets/image-20240505191542836.png)

​	接下来, 这个人又决定把合并操作回滚, 改用变基; 继而又用`git push --force`命令覆盖了服务器上的提交历史. 之后, 你从服务器抓取更新, 会发现多出来一些新的提交.

![image-20240505192335177](./assets/image-20240505192335177.png)

​	结果就是你们两人的处境都十分尴尬. 如果你执行`git pull`命令, 你将合并来自两条提交历史的内容, 生成一个新的合并提交, 最终仓库会如图所示:

![image-20240505214448873](./assets/image-20240505214448873.png)

​	此时, 如果你执行`git log`命令, 你会发现有两个提交的作者, 日期, 日志居然是一样的, 这会令人感到混乱. 此外, 如果你将这一堆又推送到服务器上, 你实际上是将那些已被变基抛弃的提交又找了回来, 这会令人感到更加混乱. 很明显对方并不想在提交历史中看到`C4`和`C6`, 因为之前就是他们把这两个提交通过变基丢弃的.



#### 用变基解决变基

​	如果你真的遭遇了类似的处境, Git还有一些高级魔法可以帮到你. 如果团队中的某人强制推送并覆盖了一些你所基于的提交, 你需要做的就是检查你做了哪些修改, 以及他们覆盖了哪些修改. 

​	实际上, ==Git除了对整个提交计算SHA-1校验和以外, 也对本次提交所引入的修改计算了校验和, 即“patch-id".==

​	如果你拉取被覆盖过的更新并将你手头的工作基于此进行变基的话, 一般情况下Git都能成功分辨出哪些是你的修改, 并把它们应用到新分支上.

​	比如, 如果遇到前面提到的有人推送了经过变基的提交, 并丢弃了你的本地开发所基于的一些提交那种情境, 如果我们==不是执行合并==, 而是执行`git rebase teamone/master`, Git将会:

- 检查哪些提交是我们的分支上独有的(C2, C3, C4, C6, C7)
- 检查其中哪些提交不是合并操作的结果(C2, C3, C4)
- 检查哪些提交在对方覆盖更新时并没有被纳入目标分支(只有C2和C3, 因为C4其实就是C4‘)
- 把查到的这些提交应用在`teamone/master`上面

​	从而我们将得到与你将相同的内容又合并了一次, 生成了一个新的提交中不同的结果, 如图在一个被变基然后强制推送的分支上再次执行变基所示:

![image-20240505220238511](./assets/image-20240505220238511.png)

​	要想上述方案奏效, 还需要对方在变基时确保C4'和C4是几乎一样的.否则变基操作将无法识别, 并新建另一个类似C4的补丁(而这个补丁很可能无法整洁的整合入历史, 因为补丁中的修改已经存在于某个地方了).

​	在本例中另一种简单的方法是使用`git pull --rebase`, 而不是直接使用`git pull`. 又或者也可以手动完成这个过程, 先`git fetch`, 再`git rebase teaming/master`.

​	如果你习惯使用`git pull`, 同时又希望默认使用选项`--rebase`, 你可以执行这些语句`git config --global pull.rebase true`来更改`pull.rebase`的默认配置.

​	只要你把变基命令当作是在推送前清理提交使之整洁的工具, 并且只在从未推送至公用仓库的提交上执行变基命令, 就不会有事. 假如你在那些已经被推送至公用仓库的提交上执行变基命令, 并因此丢弃了一些别人的开发所基于的提交, 那就有大麻烦了, 你的同事也会因此鄙视你. 

​	如果你或你的同事在某些情形下决意要这么做, 请一定要通知每个人执行`git pull --rebase`命令, 这样尽管不能避免伤痛, 但能有所缓解. 



#### 变基VS合并

​	至此, 已经学习了变基和合并的用法. 那么, 到底哪种方式更好. 在回到这个问题之前, 先讨论一下提交历史到底意味着什么.

​	有一种观点认为, 仓库的提交历史即是==记录实际发生过什么==. 它是针对历史的文档, 本身就有价值, 不能乱改. 从这个角度来说, 改变提交历史是一种亵渎.那么这些痕迹就应该被保留下来, 让后人能够查阅.

​	另一种观点相反, 他们认为提交历史是==项目过程中发生的故事==. 没人会出版一本书的第一批草稿, 软件维护手册也是需要反复修订才能方便使用. 持这一观点的人会使用rebase及filter-branch等工具来编写故事, 怎么方便后来的读者就怎么写.

​	所以, 到底是合并好还是变基好.  事实上, 并没有一个简单的答案. Git是一个非常强大的工具, 它允许你对提交历史做许多事情, 但每个团队, 每个项目对此的需求并不相同. 我们应当根据实际情况做出明智的选择. 

​	总的原则是, ==只对尚未推送或分享给别人的本地修改执行变基操作清理历史, 从不对已推送至别处的提交执行变基操作, 这样, 你才能享受到两种方式带来的便利.==





## 服务器上的Git

​	到目前为止, 我们已经可以使用Git来完成日常工作. 不过为了使用Git协作功能, 你还需要有远程的Git仓库. 尽管在技术上可以从个人仓库进行推送(push)和拉取(pull)来修改内容, 但最好不要使用这种方式, 因为很容易弄混其他人的进度. 此外, 你希望你的合作者们即使在你的电脑未联机时也能存取仓库. 因此, 与他人合作的最佳方法是建立一个你与合作者们==都有权力访问==, 且==可从那里推送和拉取资料==的共用仓库. 

​	架设一台Git服务器. 首先, 选择服务器使用的通讯协议. 下面将从介绍可用的协议展开, 然后解释使用不同协议的典型设置及如何在你的服务器上运行. 

​	==一个远程仓库通常只是一个裸仓库(bare repository)==, 即一个没有当前工作目录的仓库. 因为该仓库仅仅作为合作媒介, 不需要从磁碟检查快照; 存放的只有Git的资料. 简单地说, 裸仓库就是你专案目录内的`.git	`子目录内容, 不包含其他资料.



### 协议

​	Git可以使用四种主要的协议来传输资料: 本地协议(Local), HTTP协议, SSH(Secure Shell)协议及Git协议. 在此, 我们将会讨论那些协议及哪些情形应该使用或避免使用它们.



#### 本地协议

​	最基本的就是本地协议(Local protocol). 其远程版本库就是硬盘内的另一个目录. 这常见于团队每一个成员都对一个共享大的文件系统(例如一个挂载的NFS)拥有访问权, 或者较少见的多人共用一台电脑的情况. 后者并不理想, 因为你的所有代码版本库如果长存于同一台电脑, 更可能发生灾难性的损失.

​	 如果使用的是共享文件系统, 就可以从本地版本库克隆, 推送, 拉取. 像如下方式去克隆一个版本库, 使用版本库路径作为URL.

```shell
$ git clone /opt/git/project.git
```

​	或如下命令:

```shell
$ git clone file:///opt/git/project.git
```

​	如果在URL开头明确的执行`file://`, 那么Git的行为会略有不同. 如果仅是指定路径, Git会尝试使用硬链接(hard link)或直接复制所需要的文件. 但如果指定`file://`, Git会触发平时用于网络传输资料的进程, 那通常是传输效率较低的方法. 指定`file://`的主要目的是取得一个没有外部参考(extraneous references)或对象(object)的干净版本库副本, 通常是在从其他版本控制系统导入后或一些类似情况(参见 Git内部原理for maintenance tasks)需要这么做. 在此我们将使用普通路径, 因为这样通常更快.

​	要增加一个本地版本库到现有的Git项目, 可以执行如下命令:

```shell
$ git remote add local_proj /opt/git/project.git
```

​	然后, 就可以像在网络上一样从远端版本库推送和拉取更新了. 



##### 优点

- 简单, 可以直接利用现有的共享文件系统(如果有)
- 直接使用了现有的文件权限和网络访问权限
- 只需像设置其它共享目录一样, 把一个裸版本库的副本放到大家都可以访问的路径, 设置好读写权限即可.

##### 缺点

- 共享文件系统比较难配置, 并且与基本的网络连接访问相比, 不方便从多个位置访问.
- 例如从在家中工作时, 需要挂在一个远程磁盘, 配置不方便, 速度也慢.

- 每个用户都有"远程"目录的完整shell权限, 没有方法可以阻止他们修改或删除Git内部文件和损坏仓库.



#### HTTP协议

​	Git1.6.6之前只有哑HTTP协议可用, 十分简单且通常是只读模式.

​	Git1.6.6引入了智能HTTP协议, 让Git可以像通过SSH那样智能地协商和传输数据.



##### 智能(Smart)HTTP协议

​	智能HTTP协议的运行方式和SSH及Git协议类似, 只是运行在标准的HTTP/S端口上并且可以使用各种HTTP验证机制, 这意味着使用起来会比SSH协议简单的多, 比如可以使用HTTP协议的用户名/密码的基础授权, 免去设置SSH公钥.

​	智能HTTP协议或许已经是最流行的使用Git的方式了, 它既支持像`git://`协议一样设置匿名服务, 也可以像SSH协议一样提供传输时的授权和加密. 而且只用一个URL就可以都做到, 省去了为不同的需求设置不同的URL. 如果你要推送到一个需要授权的服务器上(一般来讲都需要), 服务器会提示你输入用户名和密码. 从服务器获取数据时也一样.

​	事实上, 类似GitHub的服务, 你在网页上看到的URL, 和你在克隆, 推送时使用的是一样的.



##### 哑(Dumb)HTTP协议

​	如果服务器没有提供智能HTTP协议的服务, Git客户端会尝试使用更简单的"哑"HTTP协议. 哑HTTP协议里web服务器仅把裸版本库当作普通文件来对待, 提供文件服务. 哑HTTP协议的优美之处在于设置起来简单. 基本上, 只需要把一个裸版本库放在HTTP根目录, 设置一个叫做`post-update`的挂钩就可以了. 此时, 只要能访问web服务器上你的版本库, 就可以克隆你的版本库. 下面是设置从HTTP访问版本库的方法:

```shell
$ cd /var/www/htdocs/
$ git clone --bare /path/to/git_project gitproject.git
$ cd gitproject.git
$ mv hooks/post-update.sample hooks/post-update
$ chmod a+x hooks/post-update
```

​	这样就可以了. Git自带的`post-update`挂钩会默认执行合适的命令(`git update-server-info`), 来确保通过HTTP的获取和克隆操作正常工作. 这条命令会在你通过SSH向版本库推送之后被执行, 然后别人就可以通过类似下面的命令来克隆:

```shell
$ git clone https://example.com/gitproject.git
```

​	这里我们用了Apache里设置了常用的路径`/var/www/htdocs`, 不过你可以使用任何静态web服务器, 只需要把裸版本库放到正确的目录下就可以. Git的数据是以基本的静态文件形式提供的.



​	通常会在可以==提供读写的智能HTTP服务==和简单的==只读的哑HTTP服务==之间选一个. 极少会将二者混合提供服务. 



##### 优点

​	以下只关注智能HTTP协议的优点.

- clone, push, pull等不同访问方式只需要一个URL
- 服务器只在需要授权时提示输入授权信息
- 使用HTTPS协议时, 可以给传输的数据加密, 甚至可以让客户端使用指定的SSL证书



##### 缺点

- 在一些服务器上, 架设HTTP/S协议的服务端会比SSH协议的棘手一些
- 管理凭证会比使用SSH密钥认证麻烦一些



#### SSH协议

​	架设Git服务器常用SSH协议作为传输协议. 因为大多数环境已支持通过SSH访问. 即使没有也比较容易架设. SSH协议也是一个验证授权的网络协议.

​	通过SSH协议克隆版本库, 需要指定一个`ssh://`的URL:

```shell
$ git clone ssh://user@server/project.git
```

​	或者使用一个简短的scp式的写法:

```shell
$ git clone user@server:project.git
```

​	也可以不指定用户, Git会使用当前登录的用户名.



##### 优点

- SSH架设相对简单. SSH守护进程很常见, 多数管理员都有使用经验, 并且多数操作系统都包含了它及相关的管理工具.
- 通过SSH访问是安全的. 所有传输数据都要经过授权和加密.
- 高效, 在传输前也会尽量压缩数据.



##### 缺点

- 不能通过它实现匿名访问. 即便只是读取数据, 使用者也要有通过SSH访问你的主机的权限, 这使得SSH协议不利于开源项目.
- 如果你只在公司网络使用, SSH协议可能是你唯一要用到的协议.
- 如果你要同时提供匿名只读访问和SSH协议, 那么除了架设SSH服务以外, 还得架设一个可以让其他人访问的服务.



#### Git协议

​	Git里有一个特殊的守护进程, 它监听一个特定的端口(9418), 类似于SSH服务, 但访问无需任何授权. 要让版本库支持Git协议, 需要先创建一个`git-daemon-export-ok`文件, 它是Git协议守护进程为这个版本库提供服务的必要条件, 但是除此之外, 没有任何安全措施. 要么谁都可以克隆这个版本库, 要么谁都不能. 这意味着, 通常不能通过Git协议推送. 由于没有授权机制, 一旦开放推送操作, 意味着网络上知道这个项目URL的人都可以向项目推送数据.



##### 优点

- Git协议是Git使用的网络传输协议里最快的.
- 它使用与SSH相同的数据传输机制, 但是省去了加密和授权的开销.



##### 缺点

- 缺乏授权机制.
- Git协议也许是最难架设的. 它要求有自己的守护进程, 这就要配置`xinetd`或者其他的程序, 这些工作并不简单.
- 它还要求防火墙开放9418端口, 但是企业防火墙一般不会开放这个非标准端口. 大型企业的防火墙通常会封锁这个端口.



### 在服务器上搭建Git

​	以下示例在Linux服务器上演示.

​	在开始架设Git服务器前, 需要把现有仓库导出为裸仓库, 即一个不包含当前工作目录的仓库. 这只需要在clone命令加上`--bare`选项. 按照惯例, 裸仓库目录名以`.git`结尾:

```shell
$ git clone --bare my_project my_project.git
Cloning into bare repository 'my_project.git'...
done.
```

​	现在, 你的`my_project.git`目录中应该有Git目录的副本了.

​	整体上效果大致相当于

```shell
$ cp -Rf my_project/.git my_project.git
```

​	

#### 把裸仓库放到服务器上

​	现在要做的就是把裸仓库放到服务器上并设置你的协议. 假设一个域名为`git.example.com`的服务器已经架设好, 并可以通过SSH连接, 你想把所有的Git仓库放在`/opt/git`目录下. 假设服务器上存在`/opt/git/`目录, 你可以通过以下命令复制你的裸仓库来创建一个新仓库:

```shell
$ scp -r my_project.git user@git.example.com:/opt/git
```

​	此时, 其他通过SSH连接这台服务器并对`/opt/git`目录拥有可读权限的使用者, 通过运行以下命令就可以克隆你的仓库.

```shell
$ git clone user@git.example.com:/opt/git/my_project.git
```

​	如果一个用户, 通过使用SSH连接到一个服务器, 并且对`/opt/git/my_project.git`目录拥有可写权限, 那么他将自动拥有推送权限.

​	如果到该项目目录中运行`git init`命令, 并加上`--shared`选项, 那么Git会自动修改该仓库目录的组权限为可写.

```shell
$ ssh user@git.example.com
$ cd /opt/git/my_project.git
$ git init --bare --shared
```

​	由此可见, 根据现有的Git仓库创建一个裸仓库, 然后把它放上你和协作者都有SSH访问权的服务器是多么容易. 现在你们已经准备好在同一项目上开展合作了.

​	以上的确是架设一个几个人拥有连接权的Git服务的全部, 只要在服务器上加入可以用SSH登录的账号, 然后把裸仓库放在大家都有读写权限的地方. 这就是一切, 无需更多. 

​	接下来了解一些更复杂的扩展. 如避免为每一个用户建立一个账户, 给仓库添加公共读取权限, 架设网页界面等.



#### 小型安装

​	架设Git服务最复杂的地方在于用户管理. 如果需要仓库对特定的用户可读, 而给另一部分用户读写权限, 那么访问和许可安排就会比较困难.



##### SSH连接

​	如果已经有一台所有开发者都可以用SSH连接的服务器, 架设Git仓库就十分简单了. 如果想在你的仓库上设置更复杂的访问控制权限, 只要使用服务器操作系统的普通的文件系统权限就行了.

​	如果需要团队里每个人都对仓库有写权限. 第一个就是给团队里每个人创建账号, 这种方法很直接但也很麻烦. 你应该不会想为每一个人运行一次`adduser`并且设置临时密码.

​	第二个办法是在主机上建立一个git账户, 让每个需要写权限的人发送一个SSH公钥, 然后将其加入git账户的`~/.ssh/authorized_keys`文件. 这样所有人都将通过git账户访问主机. 这一点也不会影响提交的数据, 因为访问主机用的身份不会影响提交对象的提交者信息.

​	另一个办法是让SSH服务器通过某个LDAP服务, 或者其他已经设定好的集中授权机制, 来进行授权. 只要每个用户可以获得主机的shell访问权限, 任何SSH授权机制你都可视为是有效的.



### 生成SSH公钥

​	许多Git服务器都是用SSH公钥进行认证. 为了向Git服务器提供SSH公钥, 如果某系统用户尚未拥有密钥, 必须事先为其生成一份. 

​	首先, 确认自己是否拥有密钥. 默认情况下, 用户的SSH密钥存储在其`~/.ssh`目录下. 进入该目录并列出其中内容, 你便可以快速确认自己是否拥有密钥:

```shell
$ cd ~/.ssh
$ ls
authorized_keys2 id_dsa known_hosts
config id_dsa.pub
```

​		我们需要寻找一对以`id-dsa`或`id_rsa`名为的文件, 其中一个带有`.pub`扩展名. `.pub`文件是你的公钥, 另一个是私钥. 如果找不到这样的文件, 你可以通过`ssh-keygen`程序来创建它们. 在Linux/Mac系统中, `ssh-keygen`随SSH软件包提供, 在Windows上, 该程序包含于MSysGit软件包中.

```shell
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

​	首先, `ssh-keygen`会确认密钥的存储位置(默认是`.ssh/id_rsa`), 然后它会要按求你输入两次密钥口令. 如果你不想在使用密钥时输入口令, 将其留空即可.

​	 现在, 用户需要将的得到的公钥发送给Git服务器管理员即可. 公钥看起来是这样的:

```shell
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@mylaptop.local
```



### 配置服务器

​	接下来看看如何配置服务器端的SSH访问. 本例中, 我们将使用`authorized_keys`方法来对用户进行认证. 同时假设操作系统是标准的Linux发行版, 比如Ubuntu. 首先, 创建一个操作系统用户`git`, 并为其建立一个`.ssh`目录.

```shell
$ sudo adduser git 
$ su git 
$ cd 
$ mkdir .ssh && chmod 700 .ssh 
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

​	接着, 我们需要为系统用户`git`的`authorized_keys`文件添加一些开发者SSH公钥. 假如, 我们已经获得了一些受信任的公钥, 并保存在临时文件中:

```shell
$ cat /tmp/id_rsa.john.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq dAv8JggJICUvax2T9va5 gsg-keypair
```

​	将这些公钥加入系统用户`git`的`.ssh`目录下`authorized_keys`的末尾:

```shell
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys 
$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys 
$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys
```

​	现在我们来为开发者==新建==一个空仓库, 可以借助带`--bare`选项的`git init`命令做到, 该命令在初始化仓库时不会创建工作目录:

```shell
$ cd /opt/git 
$ mkdir project.git 
$ cd project.git 
$ git init --bare 
Initialized empty Git repository in /opt/git/project.git/
```

​	接着John, Josie, Jessica中的任意一人可以将他们项目的最初版本推送到这个仓库中. 他只需将此仓库设置为项目的远程仓库并向其推送分支. ==请注意, 每添加一个新项目, 都需要有人登录服务器取得shell, 并创建一个裸仓库.== 我们假定上述服务器使用`gitserver`作为主机名, 且该服务器运行在内网, 并且你已经在DNS配置中将`gitserver`指向此服务器. 假定`myproject`是已有的本地项目:

```shell
# on John's computer 
$ cd myproject 
$ git init 
$ git add .
$ git commit -m 'initial commit' 
$ git remote add origin git@gitserver:/opt/git/project.git 
$ git push origin master
```

​	此时, 其他开发者可以克隆此仓库, 并推送各自的改动, 步骤很简单:

```shell
$ git clone git@gitserver:/opt/git/project.git 
$ cd project 
$ vim README 
$ git commit -am 'fix for the README file' 
$ git push origin master
```

​	通过这种方法, 你可以快速搭建一个具有读写权限, 面向多个开发者的Git服务器.

​	注意, 目前所有获得授权的开发者用户都能以系统用户 `git`的身份登录服务器从而获得一个普通shell. 如果想要对此加以限制, 则需要修改`passwd`文件中`git`用户所对应的shell值. 

​	借助一个名为`git-shell`的受限shell工具, 可以方便地将用户`git`的活动限制在与Git相关的范围内. 该工具随Git软件包一同提供. 如果将`git-shell`设置为用户`git`的登录shell, 那么用户`git`便不能获得此服务器的普通shell访问权限. 若要使用`git-shell`, 需要用它替换掉bash或csh, 使其成为系统用户的登录shell. 为进行上述操作, 首先必须确保`git-shell`已存在于`/etc/shells`文件中:

```shell
$ cat /etc/shells   # see if `git-shellìs already in there.  If not...
$ which git-shell   # make sure git-shell is installed on your system.
$ sudo vim /etc/shells  # and add the path to git-shell from last command
```

​	现在可以使用`chsh <username>`命令修改任一系统用户的shell:

```shell
$ sudo chsh git  # and enter the path to git-shell, usually: /usr/bin/gitshell
```

​	这样, 用户`git`就只能利用SSH连接对Git仓库进行推送和拉取操作, 而不能登录机器并取得普通shell. 如果试图登录, 你会发现尝试被拒绝, 像这样:

```shell
$ ssh git@gitserver 
fatal: Interactive git shell is not enabled.
hint: ~/git-shell-commands should exist and have read and execute access.
Connection to gitserver closed.
```

​	现在, 网络相关的Git命令依然可以正常工作, 但是开发者用户已经无法得到一个普通shell了. 正如输出信息所提示的, 你也可以在`git`用户的家目录下建立一个目录, 来对`git-shell`命令进行一定程度的自定义. 比如, 可以限制掉某些本应被服务器接受的Git命令, 或者对刚才的SSH拒绝登录信息进行自定义, 这样, 当有开发者用户以类似方式尝试登录时, 便会看到你的信息. 要了解更多有关自定义shell的信息, 请运行`git help shell`.



### Git守护进程

​	接下来通过Git协议建议一个基于守护进程的仓库. 对于快速且无需授权的Git数据访问, 这是一个理想之选. 

​	Git协议十分容易设定, 通常, 只需要以守护进程的形式运行该命令:

```shell
git daemon --reuseaddr --base-path=/opt/git/ /opt/git/
```

​	`--reuseaddr`允许服务器在无需等待旧连接超时的情况下重启, `--base-path`选项允许用户在未完成指定路径的条件下克隆项目, 结尾的路径将告诉Git守护进程从何处寻找仓库来导出. 如果有防火墙正在运行, 需要开放端口9418的通信权限.

​	在Ubuntu上也可以使用脚本启动该守护进程, 先找到如下文件:

```shell
/etc/event.d/local-git-daemon
```

​	并添加下列脚本内容:

```shell
start on startup 
stop on shutdown 
exec /usr/bin/git daemon \     
		--user=git --group=git \     
		--reuseaddr \     
		--base-path=/opt/git/ \     
		/opt/git/ 
respawn
```

​	出于安全考虑, 强烈建议使用一个对仓库拥有只读权限的用户身份来运行该守护进程. 为简便起见, 这里将像之前`git-shell`一样, 同样使用git用户来运行它.

​	当你重启机器时, 你的Git守护进程将会自动启动, 并且如果进程被意外结束它会自动重新运行. 为了在不重启的情况下直接运行, 你可以运行以下命令:

```shell
initctl start local-git-daemon
```

​	在其他系统中, 你可以使用`sysvinit`系统中的`xinetd`脚本, 或者另外的方式来实现-只要你能够将其命令守护进程化并实现监控.

​	接下来, 需要告诉Git哪些仓库允许基于服务器的无授权访问. 你可以在每个仓库下创建一个名为`git-daemon-export-ok`的文件来实现.

```shell
$ cd /path/to/project.git 
$ touch git-daemon-export-ok
```

​	该文件将允许Git提供无需授权的项目访问服务. 



### Smart HTTP

​	我们一般通过SSH进行授权访问, 通过git://进行无授权访问, 但还有一种协议可以同时实现以上两种方式的访问. 设置Smart HTTP一般只需要在服务器上启用一个Git自带的名为`git-http-backend`的CGI脚本. 该CGI脚本将会读取由`git fetch`或`git push`命令向HTTP URL发送的请求路径和头部信息, 来判断客户端是否支持HTTP通信(要求1.6.6版本以上). 如果CGI发现该客户端支持智能模式, 它将会以智能模式与它通信, 否则它将会回落到哑模式下.

​	完成上述步骤后, 将用Apache来作为CGI的服务器. 如果还没有安装Apache, 你可以在Linux环境下执行如下或类似的命令来安装:

```shell
$ sudo apt-get install apache2 apache2-utils 
$ a2enmod cgi alias env
```

​	该操作将会启用`mod_cgi`, `mod_alias`, `mod_env`等Apache模块, 这些模块都是使该功能正常工作所必须的.

​	接下来我们要向Apache配置文件添加一些内容, 来让`git_http_backend`作为Web服务器对`/git`路径请求的处理器.

```shell
SetEnv GIT_PROJECT_ROOT /opt/git 
SetEnv GIT_HTTP_EXPORT_ALL 
ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
```

​	如果留空`GIT_HTTP_EXPORT_ALL`这个环境变量, Git将只对无授权客户端提供带`git-daemon-export-ok`文件的版本库, 就像Git守护进程一样.

​	接着需要让Apache接受通过该路径的请求, 添加如下的内容至Apache配置文件:

```shell
<Directory "/usr/lib/git-core*">
	Options ExecCGI Indexes
	Order allow,deny
	Allow from all
	Require all granted
</Directory>
```

​	最后, 如果想实现写操作授权验证, 使用如下的未授权屏蔽配置即可:

```shell
<LocationMatch "^/git/.*/git-receive-pack$">
	AuthType Basic
	AuthName "Git Access"
	AuthUserFile /opt/git/.htpasswd
	Require valid-user
</LocationMatch>
```

​	这需要你创建一个包含所有合法用户密码的`.htaccess`文件. 以下是一个添加"schacon"用户到此文件的例子:

```shell
$ htdigest -c /opt/git/.htpasswd "Git Access" schacon
```

​	你可以通过许多方式添加Apache授权用户, 选择其中一种方式即可. 以上仅仅是我们可以找到的最简单的一个例子. 如果愿意的话, 你也可以通过SSL运行它, 以保证所有数据是在加密状态下进行传输的.

​	主要原理就是使用一个Git附带的, 名为`git-http-backend`的CGI. 它被引用来处理协商通过HTTP发送和接收的数据. 它本身不包含任何授权功能, 但是授权功能可以在Web服务器层引用它时被轻松实现. 你可以在任何所有可以处理CGI的Web服务器上办到这点.



### GitWeb

​	如果你有了对项目的读写权限或只读权限, 你可能需要建立起一个基于网页的简易查看器. Git提供了一个叫做GitWeb的CGI脚本来做这项工作.

![image-20240509112929250](assets/image-20240509112929250.png)

​	如果你想要查看GitWeb如何展示你的项目, 并且在服务器上安装了轻量级网络服务器比如`lighttpd`或`webrick`, Git提供了一个命令来让你启动一个临时服务器. 在linux上`lighttpd`通常已经安装了, 只需要在项目目录里执行`git instaweb`命令即可. 如果是Mac系统, 通常已经预安装了`Ruby`, 所以`webrick`或许是最好的选择. 如果不想使用lighttpd启动`instaweb`命令, 只需要在执行时加入`--httpd`参数.

```shell
$ git instaweb --httpd=webrick
[2009-02-21 10:02:21] INFO WEBrick 1.3.1
[2009-02-21 10:02:21] INFO ruby 1.8.6 (2008-03-03) [universal-darwin9.0]
```

​	这个命令启动了一个监听1234端口的HTTP服务器, 并且自动打开了浏览器. 如果想关闭这个服务器, 可以执行同一个命令, 并加上`--stop`选项:

```shell
$ git instaweb --httpd=webrick --stop
```

​	如果你想持续运行这个页面, 而不只是临时查看. 需要通过普通的Web服务器来设置CGI脚本. 一些Linux发行版的软件库有`gitweb`包, 可以通过`apt`或`yum`来安装. 接下来我们来了解一下如何手动安装`Gitweb`包.

​	首先, 获得Git的源代码, 它包含了GitWeb, 并可以生成自定义的CGI脚本:

```shell
$ git clone git://git.kernel.org/pub/scm/git/git.git
$ cd git/
$ make GITWEB_PROJECTROOT="/opt/git" prefix=/usr gitweb
	SUBDIR gitweb
	SUBDIR ../
make[2]: `GIT-VERSION-FILE' is up to date.
	GEN gitweb.cgi
	GEN static/gitweb.js
$ sudo cp -Rf gitweb /var/www/
```

​	需要注意的是, 你需要在命令中指定`GITWEB_PROJECTROOT`变量来让程序知道你的Git版本库的位置. 现在, 需要在Apache中使用这个CGI脚本, 为此你需要添加一个虚拟主机:

```xml
<VirtualHost *:80>
	ServerName gitserver
	DocumentRoot /var/www/gitweb
	<Directory /var/www/gitweb>
		Options ExecCGI +FollowSymLinks +SymLinksIfOwnerMatch
		AllowOverride All
		order allow,deny
		Allow from all
		AddHandler cgi-script cgi
		DirectoryIndex gitweb.cgi
	</Directory>
</VirtualHost>
```

 再次提醒, GitWeb可以通过任何一个支持CGI或Perl的网络服务器架设. 现在你可以访问`http://gitserver/`在线查看你的版本库. 



### GitLab

​	GitWeb简单, 但是不够现代, 功能也不够齐全. GitLab是几个开源的解决方案中最出名的一个. 

#### 安装

​	GitLab是一个数据库支持的web应用, 所以安装也就涉及更多的东西.

​	可以下载虚拟机镜像, 或者在https://bitnami.com/stack/gitlab 上获取一键安装包, 同时调整配置使之符合你特定的环境.

#### 管理

​	管理界面通过网络进入. 在浏览器中输入已经安装GitLab的主机名或IP地址, 然后以管理员身份登录即可. 登陆后, 点击主栏上方靠右位置的"Admin area"图标进行管理.

![image-20240509141418718](assets/image-20240509141418718.png)



##### 使用者

​	GitLab上的用户指的是对应协作者的账号. 每一个用户账号都有一个命名空间, 即该用户项目的逻辑集合. 如果一个叫jane的用户拥有一个名称是project的项目, 那么这个项目的url会是http://server/jane/project .

![image-20240509141836152](assets/image-20240509141836152.png)

​	移除一个用户有两种方法.

- “屏蔽(Blocking)" 一个用户阻止他登录 GitLab 实例,但是该用户命名空间下的所有数据仍然会被保存,并且仍可以通过该用户提交对应的登录邮箱链接回他的个人信息页.
- “销毁(Destroying)" 一个用户,会彻底的将他从数据库和文件系统中移除.他命名空间下的所有项目和数据都会被删除,拥有的任何组也会被移除.这显然是一个更永久且更具破坏力的行为,所以很少用到这种方法.

##### 组

​	一个GitLab的组是一些项目的集合, 连同关于多少用户可以访问这些项目的数据. 每一个组都有一个项目命名
空间(与用户一样), 如果一个叫training的组拥有一个名称是materials的项目, 那么这个项目的url会是http://server/training/materials .

![image-20240509144323587](assets/image-20240509144323587.png)

​	每一个组都有许多用户与之关联, 每一个用户对组中的项目以及组本身的权限都有级别区分. 



##### 项目

​	一个 GitLab 的项目相当于 git 的版本库. 每一个项目都属于一个用户或者一个组的单个命名空间. 

​	在 GitLab 中, 一个项目不能直接同时属于个人和组, 它必须属于一个具体的命名空间. 然而, 通过正确配置权限, 可以实现项目在归属于一个组的同时, 由一个或多个个人管理. 这种设置提供了组织和权限管理的灵活性, 同时也保持了项目管理的效率和安全性. 

​	每一个项目都有一个可视级别, 控制着谁可以看到这个项目页面和仓库. 如果一个项目是私有的, 这个项目的
拥有者必须明确授权从而使特定的用户可以访问. 一个内部的项目可以被所有登录的人看到, 而一个公开的项
目则是对所有人可见的. 注意, 这种控制既包括git "fetch"的使用也包括对项目web用户界面的访问.



##### 钩子

​	GitLab在项目和系统级别上都支持钩子程序. 对任意级别, 当有相关事件发生时, GitLab的服务器会执行一个包含描述性JSON数据的HTTP请求. 这是自动化连接你的git版本库和GitLab实例到其他的开发工具, 比如CI服务器, 聊天室, 或者部署工具的一个极好方法.



#### 基本用法

​	创建一个新项目. 通过点击工具栏上的"+"图标完成. 需要填写项目名称和可视层级. 然后点击"Create Project"就完成了. 

​	项目存在后, 再与本地的Git版本库连接. 复制HTTPS或者SSH连接, 在本地版本库执行如下命令:

```shell
$ git remote add gitlab https://server/namespace/project.git
```

​	如果本地尚没有版本库, 可以执行如下命令:

```shell
$ git clone https://server/namespace/project.git
```

​	web 用户界面提供了几个有用的获取版本库信息的网页. 每一个项目的主页都显示了最近的活动, 并且通过顶
部的链接可以使你浏览项目文件以及提交的日志.



#### 一起工作

​	在一个 GitLab 项目上一起工作的最简单方法就是赋予协作者对 git 版本库的直接 push 权限. 你可以通过项目设定的"Members(成员)" 部分向一个项目添加协作者,并且将这个新的协作者与一个访问级别关联(不同的访问级别在组中已简单讨论). 通过赋予一个协作者"Developer(开发者)" 或者更高的访问级别,这个用户就可以毫无约束地直接向版本库或者向分支进行提交.
​	另外一个让合作更解耦的方法就是使用合并请求.它的优点在于让任何能够看到这个项目的协作者在被管控的情况下对这个项目作出贡献.可以直接访问的协作者能够简单的创建一个分支, 向这个分支进行提交, 也可以开启一个向 master 或者其他任何一个分支的合并请求. 对版本库没有推送权限的协作者则可以"fork" 这个版本库(即创建属于自己的这个库的副本), 向那个副本进行提交,然后从那个副本开启一个到主项目的合并请求. 这个模型使得项目拥有者完全控制着向版本库的提交, 以及什么时候允许加入陌生协作者的贡献.

​	这个部分主要聚焦于在 GitLab 中与 Git 相关的特性, 但是 GitLab 作为一个成熟的系统, 它提供了许多其他产品
来帮助你协同工作, 例如项目 wiki 与系统维护工具. GitLab 的一个优点在于, 服务器设置和运行以后,你将很少需要调整配置文件或通过 SSH 连接服务器; 绝大多数的管理和日常使用都可以在浏览器界面中完成.



### 第三方托管的选择

​	现在, 有非常多的第三方托管供你选择,每个选择都有不同的优缺点. 欲查看最新列表, 请浏览 Git 维基的GitHosting页面 https://git.wiki.kernel.org/index.php/GitHosting
​	我们会在GitHub的章节详细讲解GitHub, 作为目前最大的 Git 托管平台, 你很可能需要与托管在 GitHub 上的项目进行交互, 而且你也很可能并不想去设立你自己的 Git 服务器.







## 分布式Git



### 分布式工作流程

#### 集中式工作流

#### 集成管理者工作流

#### 司令官与副官工作流

#### 工作流程总结

### 向一个项目贡献

#### 提交准则

#### 私有小型团队

#### 私有管理团队

#### 派生的公开项目

#### 通过邮件的公开项目

#### 总结

### 维护项目

#### 在特性分支中工作

#### 应用来自邮件的补丁

##### 使用`apply`命令应用补丁

##### 使用`am`命令应用补丁

#### 检出远程分支

#### 确定引入了哪些东西

#### 将贡献的工作整合进来

##### 合并工作流

##### 大项目合并工作流

##### 变基与拣选工作流

##### Rerere

#### 为发布打标签

#### 生成一个构建号

#### 准备一次发布

#### 制作提交简报

### 总结





## GitHub





### 账户的创建和配置

#### SSH访问

#### 头像

#### 邮件地址

#### 两步验证

### 对项目做出贡献

#### 派生(Fork)项目

#### GitHub流程

##### 创建合并请求

##### 利用合并请求

#### 合并请求的进阶用法

##### 将合并请求制作成补丁

##### 与上游保持同步

##### 参考

#### Markdown

##### GitHub风格的Markdown

##### 任务列表

##### 摘录代码

##### 引用

##### 表情符号(Emoji)

##### 图片

### 维护项目

#### 创建新的版本库

#### 添加合作者

#### 管理合并请求

##### 邮件通知

##### 在合并请求上进行合作

##### 合并请求引用

##### 合并请求之上的请求

#### 提醒和通知

##### 通知页面

##### 网页通知

##### 邮件通知

#### 特殊文件

##### README

##### 贡献CONTRIBUTING

#### 项目管理

##### 改变默认分支

##### 移交项目

#### 管理组织

##### 组织的基本知识

##### 团队

##### 审计日志

### 脚本GitHub

#### 钩子

##### 服务

##### 钩子

#### GitHub API

##### 基本用途

在一个问题上评论

修改Pull Request的状态

Octokit



总结





## Git工具

