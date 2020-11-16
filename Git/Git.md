# Git相关技术总结
## 1.环境搭建
+ 安装包下载  
官网下载安装包安装  
+ 配置用户名  
git config --global user.name “your_user_name” 
+ 配置邮箱  
git config --global user.email “your_email

## 2.常用Git命令
### 2.1 撤销commit
git reset --soft HEAD^

### 2.2 代码提交到远程仓库
git push origin HEAD:refs/for/分支名
### 2.3 删除已经添加到暂存区的无用的文件
git rm -r --cached

### 2.4 从远程仓库拉取最新代码
+ 查看远程仓库  
git remote -v
+ 新建一个temp分支来拉取远程仓库的代码  
git fetch origin master:temp
+ 对比差异  
git diff temp
+ 合并差异  
git merge temp
+ 删除临时分支  
git branch -d temp

### 2.5 提交本地新建的分支到远程仓库
git push --set-upstream origin your_branch

### 2.6 提交本地已有项目到远程仓库
 **注意:远程仓库得先建立对应的项目仓库**  
git remote add origin https://github.com/EricWord/项目名.git  
git branch -M master  
git push -u origin master

### 2.7 直接将本地项目推送到远程仓库
**注意:这种情况是远程仓库没有对应名称的仓库**  
echo "# 项目名" >> README.md  
git init  
git add README.md  
git commit -m "first commit"  
git branch -M master  
git remote add origin https://github.com/EricWord/项目名.git  
git push -u origin master

## 3. 常见问题解决方案
### 3.1 您的分支领先 'origin/master' 共 x 个提交
+ 更新本地分支和远程master分支同步（但是不会丢失本地更改)  
git reset --soft origin/master
+ 推送到远程仓库  
git push origin HEAD:refs/for/master
