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

​	这会在当前目录下创建一个名为“libgit2”的目录, 并在这个目录下初始化一个`.git`文件夹, 从远程仓库拉取的所有数据会放入这个`.git`文件夹, 然后从中读取最新版本的文件的拷贝. 点进这个新建的libgit2文件夹, 就会发现所有的项目文件已经在里面了, 准备就绪等待后续的开发和使用. 如需在克隆远程仓库时自定义本地仓库的名字, 可以使用如下命令:

```shell
git clone https://github.com/libgit2/libgit2 mylibgit
```

​	Git支持多种数据传输协议, 上面的例子使用的是`https://`协议, 不过你也可以使用`git://`协议或者使用SSH传输协议, 各种方式各有利弊.

#### 记录每次更新到仓库

​	首先, 工作目录下的每一个文件都不外乎两种状态: 已跟踪或未跟踪.已跟踪的文件是指那些被纳入了版本控制的文件, 在上一次快照中有它们的记录, 在工作一般时间后, 它们的状态可能处于未修改, 已修改或已放入暂存区. 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件, 它们既不存在于上次快照的记录中, 也没有放入暂存区.初次克隆某个仓库的时候, 工作目录中的所有文件都属于已跟踪文件, 并处于未修改状态.

​	对某些文件进行编辑后, Git将它们标记为已修改文件. 然后逐步将这些已修改的文件放入暂存区, 然后提交所有暂存了的修改, 如此反复. 所以使用Git时文件的生命周期如下:

![image-20240424233139799](./assets/image-20240424233139799.png)



##### 检查当前文件状态

​	要查看哪些文件处于什么状态, 可以`git status`命令. 如果在克隆仓库后立即使用此命令, 会看到类似如下==输出==:

```shell
On branch master
nothing to commit, working directory clean
```

​	这说明现在的工作目录==相当干净==. 也就是==所有已跟踪文件在上次提交后都未更改过==. 此外, 上面信息还表明, 当前目录下没有出现任何未跟踪状态文件, 否则Git会在这里列出来. 最后, 该命令还显示了当前所在分支, 并且这个分支同远程服务器上对应的分支没有偏离. 

​	当前分支是“master”, 这是默认的分支名, 最新版本Git已将默认分支名改为“main”了.

​	在该项目下创建一个新的文件README后, 再次输入`git status`命令, 将会看到==一个新的未跟踪文件==:

```
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
    	README
nothing added to commit but untracked files present (use "git add" to track)
```

​	可以看到新建的README文件出现在`Untracked files`下面. 未跟踪的文件意味着Git在之前的快照(提交)中没有这些文件, Git不会自动将之纳入跟踪范围.



##### 跟踪新文件

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



##### 暂存已修改文件

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

##### 状态简览

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



##### 忽略文件

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



##### 查看已暂存和未暂存的修改

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



##### 提交更新

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



##### 跳过使用暂存区域

​	尽管使用暂存区域的方式, 可以精心准备要提交的内容, 但有时候会略显繁琐. Git提供了一个跳过使用暂存区域的方式, 只要在提交的时候, 给`git commit`加上`-a`选项, Git就会自动把所有已跟踪文件暂存起来一并提交, 从而==跳过已修改文件的`git add`步骤==(未跟踪的文件需要`git add`来进行跟踪).

```shell
git commit -a -m 'added new benchmarks'
```



##### 移除文件













