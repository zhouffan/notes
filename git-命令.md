### 0、 初始化

```java
//**Git global setup**
- git config --global user.name "xxx"
- git config --global user.email "[xxxx@androidmov.com](mailto:xxxx@androidmov.com)"

//**Create a new repository**
- git clone http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- cd advert_hb
- touch README.md
- git add README.md
- git commit -m "add README"
- git push -u origin master

//Push an existing folder
- cd existing_folder
- git init
- git remote add origin http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- git add .
- git commit -m "Initial commit"
- git push -u origin master

//**Push an existing Git repository**
- cd existing_repo
- git remote rename origin old-origin
- git remote add origin http://192.168.2.230/amt-gitlab/iptv-dev-tp/epg-fx/message/ad/advert_hb.git
- git push -u origin --all
- git push -u origin --tags
```



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



### 2、 [Git 之 merge 与 rebase 的区别](https://www.cnblogs.com/zhangzhang-y/p/13682281.html)

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



###  5、git 配置文件  （git clone慢 - 解决方案）

https://blog.csdn.net/qq_29811421/article/details/109348754

1.查看Git所有配置

```
git config --list

git config --list --local        //只对某个仓库有效
git config --list --global       //对当前用户的所有仓库有效
git config --list --system       //对系统所有登录用户有效
```

2.删除全局配置项

(1)终端执行命令：

```
git config --global --unset user.name

git config --global user.name "your_name"
git config --global user.email "your_email@domain.com"
```

(2)编辑配置文件：

```
git config --global --edit
```



```java
//–glboal: 选项指的是修改 Git 的全局配置文件 ~/.gitconfig，而非各个 Git 仓库里的配置文件 .git/config。
//protocol 指的是代理的协议，如 http，https，socks5 等。port 则为端口号。
对指定域名生效：git config –-global http.url.proxy protocol://127.0.0.1:port 
//url 为你需要走代理的仓库域名，url 以 http:// 和 https:// 打头的均用这个方法。
对所有域名生效：git config –-global http.proxy protocol://127.0.0.1:port


git config --global http.proxy http://127.0.0.1:7890
git config --global http.proxy socks5://127.0.0.1:7890
//为了避免从国内仓库或私有仓库clone时也走代理，配置只让github.com走代理
//clash 既支持HTTP/HTTPS，也支持socks v5协议代理
git config --global http.https://github.com.proxy http://127.0.0.1:7890
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
```

**注意：Git 不认 https.proxy ，设置 http.proxy 就可以支持 https 了。**



### 6、常用基础命令

#### 6.1 修改远程仓库地址  

**方法一**：

```java
//1.修改命令
git remote origin set-url [url]

//2.先删后加
git remote rm origin
git remote add origin [url]
```

**方法二**： 

直接修改config文件