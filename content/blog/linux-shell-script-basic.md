---
title: "Linux Shell Script Basic"
date: 2022-08-25T12:58:36+08:00
publishDate: 2022-08-25T12:58:36+08:00
draft: true
tags:
- linux
- basic
---

## 命令

执行子命令
```
1. 双顿号
`` 

2. 
$() 

```

计算数学表达式
```
expr

$[]

bc
```

输入输出重定向以及管道

- > 
- <
- |

## 结构化语句
``` sh
# 采用 if 后的 command 语句的退出状态是否为 0, 判断是否为 True
if command
then
	command
fi

#
if command
then 
	commands
else
	commands
fi

#
if command
then
	commands
elif command
then
	commands
fi

```

### test 语句
```
# test 语句
if test command
then 
	commands
fi

# 使用方括号代替 test 命令
if [ command ] 
then 
	commands 
fi
```

判断数值关系

- 相等 `n1 -eq n2`
- `n1 -gt n2`
- `n1 -ge n2` 大于等于
- `n1 -le n2`
- `n1 -lt n2`
- `n1 -ne n2` 不等于

```
if [ var1 -eq var2 ]
then 
	...
fi
```

字符串比较

- str1 = str2
- str1 != str2
- str1 > str2
- str1 < str2
- `-n str1` 长度不为0
- `-z str1` 长度是否为0

文件比较

- d file
- e file


