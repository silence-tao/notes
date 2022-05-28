# git常用命令总结

## 1.创建仓库

``` bash
git init 初始化git
git remote add origin <git地址> 连接远程仓库
git clone <git地址> 克隆远程仓库到本地
```

## 2.提交和推送

``` bash
git status 可以看到项目中有哪些文件发生了变化
git add . 把发生了变化的文件添加进来
git commit -am '' 提交文件到本地仓库
git pull 把远程分支上的文件拉取过来
git push 
git push -u origin master 把本地仓库文件推送到远程仓库中
git push -u -f origin master 把本地仓库文件强制推送到远程仓库中
git push origin HEAD -u 把新创建的分支推送到远程仓库
git cherry-pick (commit id) 合并某次commit到当前分支
git checkout .		清空工作区，只能清空全部已修改的问题件
git clean -d 		清空所有新建的文件和文件夹
git reset .			清空暂存区
```

## 3.分支管理

``` bash
git branch 查看本地分支
git branch -r 查看远程分支
git checkout <name> 切换分支
git branch <name> 创建分支
git checkout -b <本地分支名> origin/<远程分支名> 在<远程分支名>的基础上创建<本地分支名>
git fetch origin <远程分支名>:<本地分支名> 在<远程分支名>的基础上创建<本地分支名>
git checkout -b <name> 创建+切换分支
git merge <name> 合并某分支到当前分支
git branch -d <name> 删除分支
git push origin --delete [branch_name] 删除远程分支
git branch | [grep "<关键字>"] | xargs git branch -D 批量删除[有<关键字>的]本地分支
```

## 4.git配置

``` bash
git config --global user.email "你的git的注册邮箱" 设置邮箱
git config --global user.user "你的git用户名" 设置用户名
git config --global credential.helper cache 设置记住密码（默认15分钟）
git config credential.helper 'cache --timeout=3600' 设置保存一个小时时间
git config --global credential.helper store 长期保存本地配置
```

## 5.github生成token

1.打开 `Settings` -> `Developer settings`，然后选择 `Personal access tokens`，再点击 `Generate new token` 按钮，具体如下图：

![image-20220101172918949](../img/image-20220101172918949.png)

2.选择要授予此令牌token的范围或权限：

- 要使用token从命令行访问仓库，请选择repo
- 要使用token从命令行删除仓库，请选择delete_repo
- 其他根据需要进行勾选

![image-20220101173115898](../img/image-20220101173115898.png)

3.然后点击  `Generate token` 生成令牌。

![image-20220101173336010](../img/image-20220101173336010.png)

4.生成token后，记得把你的token保存下来，以便进行后面的操作。把token直接添加远程仓库链接中，这样就可以避免同一个仓库每次提交代码都要输入token了。

```bash
git remote set-url origin https://<your_token>@github.com/<USERNAME>/<REPO>.git
```

- `<your_token>`：换成你自己得到的token
- `<USERNAME>`：是你自己github的用户名
- `<REPO>`：是你的仓库名称

例如：

``` bash
git remote set-url origin https://ghp_WNjxcsp0N5fsdwcyvLombVUiUUssnoN05V8yM@github.com/silence-tao/notes.git
```

