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