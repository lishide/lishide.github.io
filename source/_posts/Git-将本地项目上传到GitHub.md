---
title: Git-将本地项目上传到GitHub
date: 2018-02-28 13:09:05
tags: [git, GitHub, 码云]
---

1、在本地创建一个版本库（即文件夹），初始化为 Git 仓库
``` bash
git init
```
<!--more-->

2、把项目文件复制到这个文件夹里面（也可在已经存在项目文件的文件夹里执行第 1 步 Git 初始化操作）

3、把项目添加到仓库
``` bash
git add .
```
> 在这个过程中其实可以一直使用 `git status` 来查看当前的状态。

4、把项目提交到仓库
``` bash
git commit -m "注释内容"
```

5、在 GitHub 上设置好 SSH 密钥后，新建一个远程仓库
> SSH 配置步骤此处不再赘述，自行查找；
> 在 GitHub 或码云等平台新建项目的过程也可提前完成。

6、将本地仓库和远程仓库进行关联
``` bash
git remote add origin https://github.com/username/repo.git
```
> 注意：origin 后面加的是 GitHub 上创建好的仓库的地址。

7、把本地仓库的项目推送到远程仓库上
``` bash
git push -u origin master
```
由于新建的远程仓库是空的，所以要加上 `-u` 这个参数，等远程仓库里面有了内容之后，下次再从本地仓库上传内容的时候只需下面这样就可以了：
``` bash
git push origin master
```

**重要！！！**如果新建远程仓库的时候勾选了“*创建 README 文件*”会报错：`failed to push some refs to  https://github.com/username/repo.git`，这是由于新创建的那个仓库里面的 README 文件不在本地仓库目录中，这时可以通过以下命令先将内容合并一下：
``` bash
git pull --rebase origin master
```
这时再 push 就能成功了。

等待一段时间，即可完成。此时再刷新项目的仓库就会发现工程文件已经上传成功了。

至此就完成了将本地项目上传到 GitHub 等仓库的整个过程。
