---
title: 使用GIT钩子自动生成提交注释
author: hypo
img: medias/featureimages/73.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2023-04-15 10:04:35
coverImg:
password:
summary: 使用GIT钩子自动生成提交注释
categories: 笔记
tags:
- GIT
- 钩子
- commit
---

# 使用GIT钩子自动生成提交注释

## 步骤

1. 在GIT仓库的.git/hooks目录下创建prepare-commit-msg文件
2. 向文件中添加如下代码:
```
   bash
   #!/bin/sh
   COMMIT_MSG_FILE=$1
   COMMIT_SOURCE=$2
```
# 获取最近一次git add的注释或解析diff信息

```
   msg=...
```
# 输出到提交注释文件
```
   echo "$msg" > $COMMIT_MSG_FILE
```
3. 设置文件为可执行:
```
   bash
   chmod +x .git/hooks/prepare-commit-msg
```
4. 禁止GIT的权限提示:
``` 
   bash
   git config --global advice.ignoredHook false
```

5. 提交代码,会自动生成注释

## 注意点

1. 确认默认shell,提供对应命令(如zsh)
2. 可以获取最近git add的注释,也可以解析git diff生成详细注释
3. 如果解析diff,需要在代码中标记接口等,如//[api]
4. 文件必须有执行权限,否则GIT会忽略钩子
5. 可以设置为多语言注释,提高易读性
6. 必须禁止GIT的ignoredHook提示,否则会出现warning
7. 提交前不需要自己输入注释,会自动生成
8. 定制化钩子可以大大提高开发效率,值得运用
9. 可以根据实践不断完善和增强定制钩子
10. 请提出使用问题和反馈,不断修复BUG和提高

# 解析git diff生成提交注释
## 步骤
1. 在prepare-commit-msg文件中,获取git diff结果:
```   
   bash
   diff=$(git diff --cached)
```
2. 解析diff,提取添加/删除/修改的接口、方法等信息:
```
   bash
   added=$(echo "$diff" | grep '^+    ' | grep '\[api\]')  
   removed=$(echo "$diff" | grep '^-    ' | grep '\[api\]')   
   modified=$(echo "$diff" | grep '^\+.*\[api\].*$' | grep '\-[api\]')
```
3. 根据解析结果生成详细注释:
```
   bash
   if [ "$added" != "" ]; then
   msg="Add API: $added"  
   elif [ "$removed" != "" ]; then
   msg="Remove API: $removed"  
   elif [ "$modified" != "" ]; then
   msg="Modify API: $modified"
   else
   msg="Update files"   
   fi
```

4. 输出生成的注释:
```
   bash
   echo "$msg" > $COMMIT_MSG_FILE
```
5. 提交代码,会自动生成详细注释

## 注意点
1. 需要在代码中标记接口、方法等,如//[api]
2. 不同语言有不同的标记方式,如Go使用//[api]
3. 提交时保留标记,否则无法解析生成详细注释
4. 详细注释会标明添加/删除/修改的接口、方法名称等
5. 其他步骤同上,如设置执行权限、禁用提示等
6. 定制化变得更加强大,但也需要更多标记维护
7. 灵活运用,根据需要选择获取最近git add注释或解析diff