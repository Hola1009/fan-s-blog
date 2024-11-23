# 初级
## 前期准备
### 设置 SSH key
通过一下方式来创建
```bash
$ ssh-keygen -t rsa -C "your_email@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key
(/Users/your_user_directory/.ssh/id_rsa): 按回车键
Enter passphrase (empty for no passphrase): 输入密码
Enter same passphrase again: 再次输入密码
```
创建完成之后到 对应目录下的 id_rse.pub 文件里复制公钥内容, 然后添加到 github
![[Pasted image 20241030113022.png]]

测试是否有效
![[Pasted image 20241017124806.png]]

如果克隆仓库的时候失败了, 可能是因为git 所设端口与系统代理不一致, 需要重新设置
![[Pasted image 20241017131318.png|400]]

```bash
# 注意修改成自己的IP和端口号
git config --global http.proxy http://127.0.0.1:7897 
git config --global https.proxy http://127.0.0.1:7897
```

## 实际动手
写好了代码后
git status 可以看到未提交的代码
git add 将文件假如到站暂存区,
git commit 将文件提交
git push 将文件提交到远程仓库

# 通过实际操作来学习
## 基本操作
### git init
使用 git 进行版本控制需要使用 `git init` 对仓库进行初始化, 初始化后会生成 `.git` 目录, 这个git目录里存储着管理当前目录内容所需的仓库数据
在 Git 中，我们将这个目录的内容称为“附属于该仓库的工作树”。 文件的编辑等操作在工作树中进行，

### git status
用来查询 git 仓库状态, 只要对工作树和仓库进行操作, status 就会发生变化

### git add
要想让文件成为 Git 仓库的管理对象，就需要用 git add命令将其
加入暂存区（Stage 或者 Index）中。暂存区是提交之前的一个临时区域

### git commit——保存仓库的历史记录
git commit命令可以将当前暂存区中的文件实际保存到仓库的历
史记录中。通过这些记录，我们就可以在工作树中复原文件。

-m 添加一下提交的描述信息
```bash
git commit -m "First commit"
```

#### 记录详情信息
刚才我们只简洁地记述了一行提交信息，如果想要记述得更加详细，请不加 -m，直接执行 git commit命令。

在编辑器中记述提交信息的格式如下。
- 第一行：用一行文字简述提交的更改内容
- 第二行：空行
- 第三行以后：记述更改的原因和详细内容

###  git log——查看提交日志
- git log命令可以查看以往仓库中提交的日
- git log --pretty=short 只显示提交信息的第一行
- git log docName 只显示指定目录, 文件的日志
- git log -p 显示文件改动

### git dif 查看更改前后变化
- git diff命令可以查看工作树、暂存区的差别。我们可以尝试修改下 README.md 文件
![[Pasted image 20241017134857.png]]
这里解释一下显示的内容。“+”号标出的是新添加的行，被删除的
行则用“-”号标出。我们可以看到，这次只添加了一行。

- git diff head
查看与工作树与最新提交的区别

## 分支操作
在进行多个并行作业时，我们会用到分支。在这类并行开发的过程中，往往同时存在多个最新代码状态。如下图所示，从 master 分支创建 feature-A 分支和 fix-B 分支后，每个分支中都拥有自己的最新代码。master 分支是 Git 默认创建的分支，因此基本上所有开发都是以这个分支为中心进行的。

![[Pasted image 20241017140448.png|400]]
通过灵活运用分支，可以让多人同时高效地进行并行开发。



### git branch——显示分支一览表
- git branch命令可以将分支名列表显示，同时可以确认当前所在`*`分支。
![[Pasted image 20241017140859.png]]

- git branch \[branchName\] 可以用来创建分支
### git checkout  -b ——创建, 切换分支
- git checkout -b 切换分支, 如果没有分支就创建分支
```bash
git chechout -b -dev
```

git brach 查看下操作后的结果
```bash
$ git branch
* dev
  master
```

处于 dev 分支下修改并提交代码, 就会提交至 feature-A 分支, 像这样在分支下提交代码称为`培育分支`
对当前分支的修改和提交不会影响到其他分支

- git checkout - 切换回上一分支

### 特性分支
主分支的代码应该是随时可以发布的稳定版本, 如果想在开发新的功能可以创建一个特性分支进行操作, 完成后再与mster分支合并

### 主干分支
主干分支是刚才我们讲解的特性分支的原点，同时也是合并的终点。通常人们会用 master 分支作为主干分支。主干分支中并没有开发到一半的代码，可以随时供他人查看。有时我们需要让这个主干分支总是配置在正式环境中，有时又需要
用标签 Tag 等创建版本信息，同时管理多个版本发布。拥有多个版本发布时，主干分支也有多个.

### git merge 合并分支
我们确定dev分支已经开发完成了, 想要把它合并到主分支下, 在这样一个场景下, 需要完成如下操作来合并代码
1. 先 checkout 到主分支
2. 执行 `git merge --no-ff dev` 为了在历史记录中明确记录下本次分支合并，我们需要创建合并提交。因此，在合并时加上 --no-ff参数
![[Pasted image 20241017144020.png]]
这样一来，feature-A 分支的内容就合并到 master 分支中了, 打开 README.md 文件可以看到, 文件已经发生了变化
![[Pasted image 20241017144108.png]]

### git log --graph 以图标形式查看分支
执行后的结果
![[Pasted image 20241017144444.png|400]]

## 更改提交操作
### git reset 回溯历史版本
































