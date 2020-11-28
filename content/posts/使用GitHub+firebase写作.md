---
title: "使用GitHub+firebase写作"
date: 2020-11-28T17:03:31+08:00
draft: false
tags: ["折腾"]
categories: ["生活"]
---

最近上手了GitHub action，感觉很好用，特别是用它自动化部署了hugo博客，以往需要在本地生成静态文件并且deploy到firebase的操作全都可以自动化地完成。



## 部署hugo博客

[我的博客项目地址](https://github.com/xinyongpeng/newBlog)

编写的action.yml文件如下：

```yaml
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.75.1'

      - name: Build
        run: hugo --minify
      - name: Deploy to Firebase
        uses: w9jds/firebase-action@master
        with:
          args: deploy --only hosting
        env:
          FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}
```

具体的细节可以参考阮师傅的[博客](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

具体的步骤：

1. checkout仓库到虚拟机上，注意需要添加参数 `submodules` 表示克隆子模块
2. 安装hugo
3. hugo生成静态文件到 `public` 目录下
4. firebase部署



之后如果换电脑了，clone仓库到本地即可

```
git clone --recursive https://github.com/caffe2/caffe2
```

注意是需要添加 `--recursive` 参数来递归克隆的



一句话总结使用GitHub action自动化部署博客的好处：将繁琐的静态文件生成、部署的流程自动化完成。

## 部署gitbook

yaml文件如下：

```yaml
name: 'Gitbook-Action'

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout action
      uses: actions/checkout@v2

    - name: Gitbook Action # https://github.com/ZanderZhao/gitbook-action/releases
      uses: ZanderZhao/gitbook-action@master  # -> or ZanderZhao/gitbook-action@v1.2.4, Just example, click above, use latest please
      with:                                   #    or fork this repo and use YourName/gitbook-action@master
        token: ${{ secrets.PERSONAL_TOKEN }}  # -> remember add this in settings/secrets as https://ZanderZhao.github.io/gitbook-action/
        time_zone: Asia/Shanghai
        source_dir: gitbook-source
        source_branch: main
        source_edit_time: true
        publish_commit_message: ${{ github.event.head_commit.message }}

```

遇到的一点问题：GitHub现在新建博客之后提供的命令指引是这样的

```
git remote add origin https://github.com/xinyongpeng/newBlog.git
git branch -M main
git push -u origin main
```

执行完后，主分支的名字为`main` 而不是通常的 `master`

