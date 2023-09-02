---
layout: post
title: Git使用技巧——不同的Git仓库使用不同的提交人
date: 2023-09-02
catalog: true
categories: 日常工具
tags: 
- Git
- GitHub
- GitLab
- 多用户
---

# 背景

在日常开发中，我们可能会从事多个项目的开发工作，一些在Gitlab上，一些在GitHub，或者公司私有的git服务。在不同git服务，你的用户、email以及ssh的配置可能存在差别，本文将介绍如何进行相关的配置。

# 操作方法

操作步骤主要包括两步：1）配置git提交的相关信息（提交人）；2）配置git相关的ssh

## 步骤一. 配置git提交的相关信息

### 方法1：基于git config 命令行的方法
一种常见的配置方法是通过命令行`git config`来解析配置；

设置提交用户名：`git config user.name example`
设置提交邮箱：`git config user.email  user@example.org`

默认为本地模式（仅在当前目录下生效），添加--global后，可以在全局生效

这种方法适合对部分仓库进行的特殊操作，需要用户在项目clone 进行手动操作。


### 方法2：基于.gitconfig 文件的方法

另一种方法为基于文件`.gitconfig`的配置，可以支持host维度的用户配置，全局自动生效。

git 默认会从本地文件`.gitconfig`加载配置信息。在该文件中，使用includeIf配置可以选择性加载相关配置，可以进行配置的定向修改，对应的文档地址如下：[https://git-scm.com/docs/git-config#_conditional_includes](https://git-scm.com/docs/git-config#_conditional_includes)。

具体支持基于git远程地址（hasconfig:remote.*.url）、本地git地址（gitdir）、分支（onbranch）等维度进行git config的信息修改。

例如，使用如下配置，对于远程地址为`https://github.com/work_org`开头的仓库，gitconfig_work这个文件的配置会生效。

```
[includeIf "hasconfig:remote.*.url:https://github.com/work_org/**"]
  path = .gitconfig_work
```

如果需要支持不同的Git仓库使用不同的提交人，只需要添加的不同配置即可

例如需要支持github和gitlab使用不同提交人，配置如下：

用户目录下的.gitconfig文件：
```
# 支持github的https协议
[includeIf "hasconfig:remote.*.url:https://github.com/**"]
  path = .git-config/github-config

# 支持github的ssh协议
[includeIf "hasconfig:remote.*.url:git@github.com:**/**"]
  path = .git-config/github-config

# 支持gitlab的https协议
[includeIf "hasconfig:remote.*.url:https://gitlab.com/**"]
  path = .git-config/gitlab-config

# 支持gitlab的ssh协议
[includeIf "hasconfig:remote.*.url:git@gitlab.com:**/**"]
  path = .git-config/gitlab-config
```

用户目录下的.git-config/github-config文件：
```
[user]
  name = Mi&Jack(GitHub)
  email = mijackstudio@gmail.com
```

用户目录下的.git-config/gitlab-config文件：
```
[user]
  name = Mi&Jack(GitLab)
  email = mijackstudio@gmail.com
```




## 步骤二. 配置git相关的ssh

在文件夹`~/.ssh`的新加config文件，可以添加如下信息，即可为对应的仓库添加用户信息。


```
Host mijack-GitHub
HostName github.com
User Mi&Jack
PreferredAuthentications publickey
IdentityFile ~/.ssh/mijack-GitHub 
AddKeysToAgent yes
```

各字段含义如下：


- Host:用来定义特定主机的配置信息的。它指定了一个主机的别名和相关的SSH连接参数。
- HostName:指定远程主机名。
- Port：指定远程主机端口号，默认为 22 。
- User：指定登录用户名。
- IdentityFile:指定密钥认证使用的私钥文件路径
- AddKeysToAgent字段是用于配置SSH密钥管理的选项。它的作用是决定在首次连接时是否将密钥添加到SSH代理中。