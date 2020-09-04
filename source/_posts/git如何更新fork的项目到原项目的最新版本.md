---
title: git如何更新fork的项目到原项目的最新版本
toc: true
date: 2019-08-04 11:06:42
categories: Git
tags: git
---

在 github 上 fork 了一个项目之后，如何使自己 fork 的项目和原先作者的项目分支保持同步呢，下面我使用 litemall的项目做示范，示范如何使 fork 的项目与原项目分支保持同步。

<!-- more -->

1. **查看远程的版本库地址**

   复制

   ```shell
   $ git remote -v 
   origin  git@github.com:tcxiaotudou/litemall.git (fetch)
   origin  git@github.com:tcxiaotudou/litemall.git (push)
   ```

2. **添加原项目 git 地址到本地版本库**

   ```shell
   $ git remote add upstream git@github.com:linlinjava/litemall.git
   ```

3. **检查版本库是否添加成功**

   ```shell
   $ git remote -v
   origin  git@github.com:tcxiaotudou/litemall.git (fetch)
   origin  git@github.com:tcxiaotudou/litemall.git (push)
   upstream   git@github.com:linlinjava/litemall.git (fetch)
   upstream   git@github.com:linlinjava/litemall.git (push)
   ```

4. **原项目更新内容同步到本地**

   ```shell
   $ git fetch upstream                             
   remote: Enumerating objects: 154, done.
   remote: Counting objects: 100% (154/154), done.
   remote: Total 346 (delta 153), reused 154 (delta 153), pack-reused 192
   Receiving objects: 100% (346/346), 53.14 KiB | 100.00 KiB/s, done.
   Resolving deltas: 100% (186/186), completed with 97 local objects.
   From github.com:linlinjava/litemall
    * [new branch]        dev        -> upstream/dev
    * [new branch]        master     -> upstream/master
    * [new tag]           v1.5.0     -> v1.5.0
   ```

5. **查看本地分支**

   ```shell
   $ git branch -a 
   * master
     remotes/origin/HEAD -> origin/master
     remotes/origin/dev
     remotes/origin/master
     remotes/upstream/dev
     remotes/upstream/master
   ```

6. **同步更新内容到本地对应分支**

   ```shell
   $ git merge upstream/master
   ```

7. **提交更新内容到 fork 地址**