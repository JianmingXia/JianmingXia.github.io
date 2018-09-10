---
title: Git技巧
date: 2017/12/1 12:00:00
tags:
  - Git
categories: Git
---

## git reset
之前有次在开发的过程，原定于下个版本需要发布的feature突然插了一个版本，导致我修改的这部分不可发布，但是此时内容已经被merge到develop分支，此时develop分支被merge的只有我这部分代码，找到merge之前的hash值，使用git reset强行回复：
- git reset --hard 28b1bd014509d62eb09e669c1c817cfc19bb7a74

强行推送到远程库：
- git push origin HEAD --force

<!-- more -->
