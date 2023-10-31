+++
author = "coucou"
title = "开发工具——Git"
date = "2023-08-01"
description = "开发工具专题之Git"
categories = [
    "开发工具"
]
tags = [
    "开发工具","Git"
]
+++

![](1.jpg)

## Git学习记录

### 一、Git基本命令

#### 1. 设置用户名和邮箱

```gas
git config --global user.name "coucou"

git config --global user.email "coucou"
```

#### 2. 初始化和查看日志

```gas
git init
git log
```

#### 3. 添加文件和提交文件

```gas
git add test.txt    // 表示文件放到暂存区
git commit -m "注释" test.txt   // -m 表注释
```

#### 4. 查看工作区和暂存区状态

```gas
git status
```

#### 5.删除文件

```gas
rm test.txt
git add test.txt
git commit -m "删除test.txt" test.txt
```

### 二、Git分支命令

#### 1. 查看、创建、切换分支

```gas
git branch branch01  // 创建分支
git branch -v  // 查看分支
git branch branch01 //  切换分支
```

### 三、GitHub相关

#### 1. 为URL起别名以及查看别名

```gas
git remote add origin [url]
git remote -v
```

#### 2. 推送操作和克隆操作

```gas
git push [url] master  // master表示要推送的分支
git clone [url]
```

#### 3. 远程库修改的拉取

```gas
git pull [url] master
```

### 四、IDEA使用Git



### 五、实例

```gas
git remote add cs_notes https://github.com/CyC2018/CS-Notes.git  // 起别名
git pull cs_notes master  // 拉项目
git add .  // 添加到暂存区
git commit -m "modify test"  // 提交给git管理
git push cs_notes master  // 提交到GitHub
```



