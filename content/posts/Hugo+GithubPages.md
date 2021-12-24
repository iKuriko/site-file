---
title: "Hugo+GithubPages 个人站点搭建"
date: 2021-09-07T14:20:48+08:00
draft: true
---



## Hugo 安装

首先在本地安装[hugo](https://github.com/gohugoio/hugo/releases)，选择适合自己系统的安装包

将hugo执行目录加入系统的环境变量

添加系统变量后，测试是否成功
```bash
hugo version
```

此时，我们就可以开始新建站点了

```bash
hugo new site blog
```

它会在当前目录，生成一个名为`blog`的文件夹，里面是新建的站点文件



## 生成静态页面

在生成开始之前，我们需要为hugo安装一个主题，这里选用的是MemE

进入`blog`目录安装主题

```
cd blog
```

使用git初始化仓库

```bash
git init
```

下载[MemE](https://github.com/reuixiy/hugo-theme-meme)主题文件
```bash
git submodule add --depth 1 https://github.com/reuixiy/hugo-theme-meme.git themes/meme
```

将主题文件`config.toml`替换为示例配置

```bash
mv config.toml config.toml.bak
```

```bash
cp themes/meme/config-examples/en/config.toml config.toml
```

新建一篇文章和一个关于页面

```bash
hugo new “posts/test.md”
```
```bash
hugo new "about/_index.md"
```

Game Start

```bash
hugo server -D
```

生成本地测试页面，在浏览器使用`localhost:1313`访问



## 推送到Github Pages

首先，你需要拥有一个Github账号

创建一个存储仓库，在仓库启用Github Pages

在本地配置你的Github信息

```bash
$ git config --global user.name "username"
$ git config --global user.email "useremail"
```

<font color=red>注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。</font>

生成SSH密钥对

```bash
$ ssh-keygen -t rsa -b 4096 -C "useremail"
```

```bash
$ cat ~/.ssh/id_rsa.pub
```

将生成的密钥复制，去 Github 上设置一个新的密钥对 [New SSH key](https://github.com/settings/keys)

测试连接

```bash
ssh -T git@github.com
```



将Github仓库克隆到本地的 ikuriko_blog 文件夹

```bash
git clone https://github.com/iKuriko/ikuriko.github.io.git ikuriko_blog
```

进入仓库目录

```bash
cd ikuriko_blog
```

初始化仓库

```bash
git init
```

将本地Git仓库和远程仓库关联起来，并设置远程仓库名称

```bash
git remote add origin git@github.com:iKuriko/ikuriko.github.io.git
```

添加所有文件到本地缓冲区

```bash
git add -A
```

提交到本地仓库，标签为`first commit`

```bash
git commit -m "first commit"
```

提交到远程仓库，第一次需指定用户和分支

```bash
git push -u origin main
```

\#非首次推送

```bash
git push
```

修改blog目录下的config.xoml文件，添加一行：

```bash
publishDir = "../ikuriko_blog"
```

将hugo生成的public目录改为刚才克隆的Github仓库的目录

生成页面

```bash
hugo -D
```

再次使用上面的<font color=red> git </font>命令将生成文件推送到远程仓库

```bash
git add -A

git commit -m "new"

git push -u origin main
```

Success！

访问你的Github Page

https://ikuriko.github.io



**PS:Git 的基本使用**

查看当前仓库的状态

```bash
git status
```

添加并提交改动

```bash
git add <文件> <文件> ……
git commit -m "first commit"
```

* <font color=red size=3fx>`git add -A`</font>的意思是一次添加所有文件。
* <font color=red size=3fx>`-m`</font> 的意思是此次提交的修改说明

查看所有的 commit 历史

```bash
git log
```

查看远程仓库，以及与本地仓库的关系
```bash
git remote show origin
```

抓取远程的一个分支到本地

```bash
git fetch <remote_name> <branch_name>
```
