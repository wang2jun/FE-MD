# 常用指令

配置基本用户信息
git config --global user.name  <你的用户名>
git config --global user.email  <你的邮箱地址>

创建仓库
git init

创建一个提交，并提交信息
git commit -m "提交的信息"

从远程仓库克隆一个仓库
git clone <远程仓库url>

显示提交历史
git log

显示当前的工作目录下的提交文件状态
git status

向远程仓库推送（push）
git push

将指定文件stage(标记为将要提交的文件)
git add <文件路径>
将指定文件unstage(取消标记为将要被提交的文件)
git reset <文件路径>

从远程仓库拉取
git  pull



# 本地项目上传至gitee

1、登录码云，在码云上新建仓库，填写相关内容，创建仓库

 2、在本地对应盘符下面，新建文件夹，例如：F:\gitRemote

 3、点击新建的文件夹，右键点击 Git Bush Here

 4、输入 git init 命令，此命令主要是为了初始化一个 git 本地仓库，此命令运行完之后，会在本地创建一个 .git 的文件夹

 5、输入 git remote add origin 码云仓库地址  例如：git remote add origin https://gitee.com/你的码云用户名/你创建的仓库名.git

 6、输入 git pull origin master 命令 ，这步的作用是：将码云上的仓库pull到本地你新建的文件夹中

​      注：在此命令运行期间，可能会要求输入你码云的用户名和密码，正确输入即可

7、将需要上传的文件添加到你新建的文件夹中，建议：在此文件夹中再建立一个 不带中文的文件夹用来存放需要上传的文件

8、输入 git add . （. 表示所有的）或者 git add + 文件名 （此命令可以将文件保存到缓存区） 注意：不要忘记 敲空格

 9、输入  git commit -m "此处填写上传文件描述"  ，此命令主要是为了添加文件描述

 10、输入 git push origin master  ，将本地代码push到码云仓库

 执行完此命令之后，就可以在码云远程仓库中看到你上传的项目了。

 至此，本地项目上传到码云步骤已完成，接下来，又可以愉快的玩耍啦

