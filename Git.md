# Git命令

[git教程](https://blog.csdn.net/weixin_42152081/article/details/80558282)

> 拉取项目

git clone https://github.com/wanwanpp/spring-framework-4.2.0.git

> 设置用户名和邮件

git config --global user.name "lijiaqing"

git config --global user.email "1013960792@qq.com"

> 查看用户名和邮件

git config user.name

git config user.email



> 创建空目录

mkdir learngit

cd learngit 	//进入当前目录

pwd	//查看当前路径

> 变为git可以管理的仓库

git init	//当前目录会多了一个.git文件，是Git用来跟踪管理版本库的

> 添加文件到版本库(在learngit下先创建一个文件readme.txt)

git add readme.txt	//把文件添加到仓库

git commit -m "本次提交的说明"	//把文件提交到仓库



> 修改文件readme.txt

git status	//查看仓库当前的状态

git diff readme.txt	//查看修改内容

// 之后就是add和commit命令提交修改后文件

> 版本回退

git log	//（--prett=oneline）查看历史纪录

git reset --hard HEAD^	//HEAD表示当前版本，HEAD^表示上一个版本，那么上上版本就是HEAN^^

cat readme.txt	//查看文件的内容

git reset --hard 1094a...  //回到最新版本，使用commit id（git log命令可查到）





### 工作区和暂存区 

工作区（Working Directory）

learngit 文件夹就是一个工作区。

版本库（Repository）

工作区有个隐藏目录 .git ，这个不算工作区，而是 Git 的版本库。

版本库里面的 index(stage) 文件叫暂存区，还有Git为我们自动创建的第一个分支 master ，以及指向 master 的一个指针叫做 HEAD。

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\20180603202659766)

前面我们提到过，如果我们想把文件添加到Git里面时，需要分两步：

第一步是用 git add 把文件添加进去，实际上就是把文件修改添加到暂存区。

第二步是用 git commit 提交更改，实际上就是把暂存区的所有内容提交到当前分支。（我们现在只有唯一一个分支 master，所以现在就是往 master 分支上提交更改）

### 管理修改

Git 如此的优秀是因为，Git 跟踪并管理的不是文件，而是修改。

在工作区的第一次修改被放入暂存区，准备提交；而在工作区的第二次修改并没有被放入暂存区，所以， git commit 命令只负责把暂存区的修改提交了。

### 撤销修改

(1) 没有 git add 之前

> 可以手动删除最后一行，手动把文件恢复到上一个版本的状态。然后再用 git checkout -- file 命令丢弃工作区的修改：

$ git checkout -- readme.txt     //把readme.txt文件在工作区的修改全部撤销。

(2) git add了，但没有git commit

> 这时候的修改添加到了暂存区，但没有提交到分支。这时候我们可以使用 git reset HEAD file 命令把把暂存区的修改撤销掉，重新放回工作区。这时候再丢弃工作区的修改就OK了

git reset HEAD readme.txt //git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区，HEAD表示最新版本。

(3) 既 git add 了，也 git commit 了

> 可以回退到上一个版本

### 删除文件

> 可用 rm 命令删除

rm test.txt

这时工作区和版本库就不一样了。

现在又分两种情况：

(1) 确实要从版本库中删除该文件，那就用 git rm 命令删除，并且 git commit：

(2) 文件被删错了。因为版本库里有，所以很好恢复：

$ git checkout -- test.txt //用版本库里的版本替换工作区的版本。

------------

上传文件到GitHub

　**1、在GitHub上建立远程仓库**

**![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414145214624-382234296.png)**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414145604535-1550208425.png)

 

 **2、新建完成之后，接下来就是生成SSH密钥部分**

　　win10系统下 可以点击win标志，找到Git--Git Bash

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414150009442-694621626.png)

在命令行跳出之后，输入如下命令：

**git config --global user.name "你注册GitHub账号的名字"**

**git config --global user.email "你注册GitHub账号用的邮箱.com"**

 ![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414150832753-1743851588.png)

 键入命令：**cd ~/.ssh**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151020381-270509611.png)

一般会有密钥生成，我当时是没有的，然后再次键入命令：**ssh-keygen -t rsa -C** "你注册GitHub账号时用的邮箱"

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151249491-1340535611.png)

然后去到用户根目录下寻找 **.ssh 文件夹**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151438912-897266727.png)

用记事本打开**id_rsa.pub文件**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151528384-1364961206.png)

里面是生成的SSH密钥，复制此密钥

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151653882-689150763.png)

**3、接下来为你的Github账户配置密钥**

　　**首先找到settings，点击进入**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151801285-1014582742.png)

先找到 SSH and GPG keys，点击 NEW SSH key

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414151925405-1031100359.png)

 然后写上你定义的名字，并粘贴上你的SSH 密钥

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414153503511-267630760.png)

**4、接下来在你要上传的目录上右键，点击 Git Bash Here 新打开一个窗口**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414154007910-482767924.png)

或者直接在之前打开的窗口中一步步使用cd xxx命令读进你需要上传的本地文件目录

然后对本地文件进行初始化，输入命令：**git init**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414154230049-580749290.png)

这时会发现目录中多出一个.git 文件夹

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414154446570-1070231457.png)

 输入命令：git init

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414160832496-1127130538.png)

输入命令：**git commit -m "你的说明注释"**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414160918799-1767795152.png)

 **5、接下来要连接GitHub仓库**

　　首先打开本地仓库

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414161116085-2000834194.png)

复制你的远程仓库地址

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414162741125-445483928.png)

 

 输入命令： **git remote add origin 你的仓库地址**

**这里便是连接仓库**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414161403075-1843055203.png)

如果你的远程仓库有README.md文件，请执行命令：**git pull --rebase origin master**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414161829852-936402048.png)

再执行命令：**git push -u origin master**

![img](C:\Users\李佳庆\Desktop\APP\笔记本\图片库\1317672-20190414161857950-475153528.png)

这一步执行成功之后，刷新GitHub仓库，可看到本地文件成功上传至远程仓库。

 