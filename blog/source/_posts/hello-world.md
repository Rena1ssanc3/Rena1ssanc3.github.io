---
title: 第一篇POST  
date: 2022-10-22 00:00:00
tags: HelloWorld
---
第一次使用hexo，这是我的第一篇post。

# 怎么搭起来的

## 有个域名

首先是你需要买一个域名，比如你现在看到的这个域名就是我的：`9394968.xyz`。 确定一下你的域名，然后在控制台里面添加一个域名解析，比如Blog的域名是`blog.9394968.xyz`。 那么我就添加一个`CNAME`记录，域名是`blog.9394968.xyz`，地址指向`${username}.github.io`。

## 有个GitHub Pages

在GitHub上开个Repo，比如我的是`Rena1ssanc3.github.io`。 然后在`Settings`里面开启`GitHub Pages`，选择`master branch`，然后就可以看到你的域名了。 GitHub Pages的域名是`Rena1ssanc3.github.io`，这个域名是固定的，不能修改的。 然后我们希望使用的域名是`blog.9394968.xyz`，所以我们需要添加一个`CNAME`文件，文件内容是`blog.9394968.xyz`。 这样就可以了，你可以在`Settings`里面看到你的域名已经变成了`blog.9394968.xyz`。 访问`blog.9394968.xyz`，你就可以看到你的GitHub Pages了。

## 有个Hexo

在本地Clone你的`Rena1ssanc3.github.io`。首先安装Node:
```zsh
$ brew install node
```
然后安装Hexo:
```zsh
$ npm install -g hexo-cli
```
然后在目录下执行:
```zsh
$ hexo init blog
```
这样就可以了，你可以在`blog`目录下看到你的Hexo项目了。

## 有个About页面

直接在hello-world上，或者新开一个页面：
```zsh
$ hexo new page "about"
```
然后在`source/about/index.md`里面写你的内容，比如我写的是：
```markdown
---
title: 关于我
---
我是Rena1ssanc3，一个普通的程序员。
```

## 有个Hexo主题

[看这里](https://hexo.fluid-dev.com/docs/start/#%E4%B8%BB%E9%A2%98%E7%AE%80%E4%BB%8B)。

## 有个部署

本地启动就直接通过
```zsh
$ hexo server
```
就可以了，但是部署的话就需要执行
```zsh
$ hexo clean && hexo generate && hexo deploy
```
提交到GitHub看Deployment，然后就可以看到你的Blog了。
