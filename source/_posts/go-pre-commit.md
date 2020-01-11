---
title: Go项目配置pre-commit hook
comments: true
date: 2019-08-04 21:27:47
updated: 2019-08-04 21:27:47
tags:
categories: [GO]
permalink:
---

# 一、前言
Go语言官方提供有：
- gofmt 来进行Go代码格式化
- go tool vet 检查Go源码中静态错误
第三方提供有:
- golint 检查GO代码风格(注释，命名等)
- goimports 自动导入需要的import statement和自动移除未被使用import statement

下面的pre-commit hook将上述的四个工具集成在一起。在将git暂存区中的代码commit到仓库之前，会对被改动的Go文件(不包括vendor目录下的文件)做下面的工作:
+ 自动调整被改动文件的 import statement
+ 自动格式化被改动文件
+ 展示代码中的静态错误
+ 展示不符合规范的代码风格问题
上述1和2是自动进行的, 而3和4则要求开发者按照提示进行修正，只有静态错误和代码风格全部修正完成之后，才能够commit成功。

# 二、用法
1. 下载第三方的golint, goimports工具
``` bash
go get golang.org/x/tools/cmd/goimports
go get golang.org/x/lint/golint
```
2. 将下面的脚本拷贝到项目仓库的 `.git/hooks/pre-commit`文件中(没有该文件新建后一个)

3. 赋予文件可行性权限，并且重新加载git配置
``` bash
chmod +x .git/hooks/pre-commit
git init
```

# 三、脚本
``` bash
#!/bin/sh

STAGED_GO_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep ".go$")

if [[ "$STAGED_GO_FILES" = "" ]]; then
    exit 0
fi

PASS=true

for FILE in $STAGED_GO_FILES
do
    # 跳过vendor目录下的文件
    if [[ $FILE == "vendor"* ]];then
        continue
    fi

    # goimports 检查并调整导入语句
    goimports -w $FILE
    if [[ $? != 0 ]]; then
        PASS=false
    fi

    # golint 检查代码风格,给出提示
    golint "-set_exit_status" $FILE
    if [[ $? == 1 ]]; then
        PASS=false
    fi

    # go tool vet 检查代码中的静态错误
    go tool vet $FILE
    if [[ $? != 0 ]]; then
        PASS=false
    fi

  # 如果当前文件没有被格式化，就格式化它
    UNFORMATTED=$(gofmt -l $FILE)
    if [[ "$UNFORMATTED" != "" ]];then
        gofmt -w $PWD/$UNFORMATTED
        if [[ $? != 0 ]]; then
            PASS=false
        fi
    fi

    # 上述 goimports, gofmt可能会对文件作出改动，
    # 所以此处将更改提交至暂存区
    git add $FILE

done

if ! $PASS; then
    printf "\033[31m COMMIT FAILED \033[0m\n"
    exit 1
else
    printf "\033[32m COMMIT SUCCEEDED \033[0m\n"
fi

exit 0
```