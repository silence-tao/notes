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
git branch -r -d origin/<name> 删除远程分支
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