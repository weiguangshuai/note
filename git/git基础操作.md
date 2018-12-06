# git基础

## 用户信息

安装git以后要设置自己的用户名和邮箱，这样每次提交的时候都会使用这些信息	，不可更改；具体添加操作如下：

```Shell
git config --global user.name "weigs"
git config --global user.email "xxxx@qq.com"
```



## 检查配置信息

使用`git config --list` 命令来列出所有的git当时能找到的配置

也可以使用`git config <key>` 来检查git的某一项配置



## 获取Git仓库

### 在现有目录中初始化仓库

```shell
git init
```

这个命令将会创建一个.git的子目录，这个目录里面包含了初始化Git仓库所有的必须文件，但是项目中的文件还没有被跟踪到，如果想跟踪项目中的文件，需要继续执行后面的操作。

可以通过	`git add` 命令来实现对指定文件的跟踪，然后通过执行`git commit` 命令提交，如果一次性跟踪所有的文件，可以执行`git add .` 命令来进行操作：

```shell
git add Weigs.java（跟踪Weigs.java这个文件）
git add .(跟踪所有的文件)
git commit -m weigs （提交）
```

### 克隆现有的仓库

```shell
git clone https://xxxx.com/weigs <myweigs>
```

Git克隆的是该Git仓库服务器上的几乎所有的数据，而不是仅仅复制完成你的工作所需要的文件。假如你想克隆一个叫weigs的库，首先，会在当前目录下创建一个名为“weigs"的目录，并在这个目录下初始化一个.git文件夹，从远程仓库中拉取所有的数据放入.git文件夹中，然后从中读取最新版本的文件拷贝。如果你想自定义一个本地库名，像上面那样，加个myweigs就可以了。



## 记录每次更新到仓库

工作目录下的文件的状态：已跟踪和未跟踪。

- 已跟踪的文件：指那些被纳入版本控制的文件，在上一次快照中有他们的记录，它们的状态可能处于未修改、已修改或已放入暂存区（`git add .`以后的文件都是已跟踪的文件）
- 未跟踪文件：除了已跟踪以外的所有文件

git的文件生命周期如下：

![git文件生命周期](https://github.com/weiguangshuai/note/blob/master/%E5%9B%BE%E5%BA%8A/git文件生命周期.png)

### 检查当前文件的状态

使用命令`git status` 来查看哪些文件处于什么状态：

```shell
git status
##输出结果如下所示：
On branch garnet_v2_nil                                                    
Your branch is up to date with 'origin/garnet_v2_nil'.                     
                                                                           
Changes not staged for commit:                                             
  (use "git add <file>..." to update what will be committed)               
  (use "git checkout -- <file>..." to discard changes in working directory)
                                                                           
        modified:   Dockerfile                                             
                                                                           
no changes added to commit (use "git add" and/or "git commit -a")     
##上面是我对Dockerfile文件进行了修改，所以显示出 modified： Dockerfile
```

### 跟踪新文件

使用命令`git add xxx` 来跟踪一个新文件，同时使用`git add`命令也是文件放入暂存区。

### 状态简览

```shell
git status -s
git status --short	
```

上面两个命令可以将状态以更为紧凑的格式输出：新添加未跟踪的文件前面有`？？`标记，新添加到暂存区的文件前面有`A`标记，修改过的文件前面有`M`标记。出现在靠左的`M` 表示该文件被修改了并放入了暂存区；出现在右边的`M` 表示文件被修改了但是还没有放入暂存区。



### 忽略文件

一般我们有些文件不需要纳入git进行管理，在这种情况下，我们创建一个名为`.gitignore` 文件，列出要忽略的文件模式。

`.gitignore` 文件的格式规范如下：

- 所有空行和以#开头的行都会被Git忽略
- 可以使用标准的glob模式匹配
- 匹配模式可以以（/ ）开头防止递归
- 匹配模式可以以（/）结尾指定目录
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反

```shell
# no .a files
*.a
# but do track lib.a, even though you're ignoring .a files above
!lib.a
# only ignore the TODO file in the current directory, not subdir/TODO
/TODO
# ignore all files in the build/ directory
build/
# ignore doc/notes.txt, but not doc/server/arch.txt
doc/*.txt
# ignore all .pdf files in the doc/ directory
doc/**/*.pdf
```

### 移除文件

要从`Git` 中移除某个文件，就必须要从已跟踪清单中移除，然后提交。可以使用命令`git rm filename` 来完成此项工作，并连带从工作目录中移除指定的文件。如果想保留工作目录中的文件，可以使用命令`git rm --cached filename` 来执行

















