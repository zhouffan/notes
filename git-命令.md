### 初始化

#### Git global setup

- git config --global user.name "xxx"
- git config --global user.email "[xxxx@androidmov.com](mailto:xxxx@androidmov.com)"

#### Create a new repository

- git clone http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- cd advert_hb
- touch README.md
- git add README.md
- git commit -m "add README"
- git push -u origin master

#### Push an existing folder

- cd existing_folder
- git init
- git remote add origin http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- git add .
- git commit -m "Initial commit"
- git push -u origin master

#### Push an existing Git repository

- cd existing_repo
- git remote rename origin old-origin
- git remote add origin http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- git push -u origin --all
- git push -u origin --tags



### [1、工具界面操作](https://blog.csdn.net/yisun123456/article/details/93643745)

#### 1.1checkout and rebase onto current

现有更改：app_1.0.x
即将打开：app_develop_1.0.x

操作：Checkout and rebase onto current

效果：app_1.0.x中更新的代码，现在在app_develop_1.0.x分支上也可以看到了。



**Rebase Current onto Selected**: 在当前分支做变基。（将所选分支提交加入到当前分支）

**Checkout with Rebase** : 检出所选分支并做变基。（将当前分支提交加入到所选分支）

**Merge into  Current**： 合并到当前分支（将所选分支合并到当前分支）

**储藏工作区**：项目右键 --> Git --> Repository --> Stash Changes



### 2.[Git 之 merge 与 rebase 的区别](https://www.cnblogs.com/zhangzhang-y/p/13682281.html)

实际操作体会



### 3、git撤销add操作

如果已经执行了git add dir_name

此时需要撤销 add操作，则需执行如下命令

**git rm -r dir_name --cached**

由于目录已经添加到git 暂存（stage）中了，所以需要加--cached参数



### 4、git 撤销commit

**git reset --soft HEAD^**： 仅仅是撤回commit操作，您写的代码仍然保留。

HEAD^：上一个版本，也可以写成HEAD~1。

HEAD~2：进行了2次commit，想都撤回。

- --mixed ：不删除工作空间改动代码，撤销commit，**撤销**git add . 

  这个为**默认参数**，git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。

- --soft：不删除工作空间改动代码，撤销commit，**不撤销**git add . 

- --hard：删除工作空间改动代码，撤销commit，撤销git add .。 恢复到了上一次的commit状态。

  

**只修改注释**：git commit --amend。 进入默认vim编辑器



