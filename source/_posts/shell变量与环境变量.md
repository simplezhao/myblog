---
title: shell变量与环境变量
tags:
  - linux
  - shell
categories: linux
description: shell变量与环境变量的区别
abbrlink: 77ae
keywords:
date: 2021-02-27 10:09:58
description: shell环境变量与变量的区别
---

> Shell 变量只在主shell有效，子进程无效，环境变量在主shell和子进程都有效，可以通过export命令将shell变量变为环境变量
>
> 使用set命令查看shell变量
>
> 使用printenv命令查看环境变量

<!-- more -->

## 背景

在python工程中，想通过环境变量获取参数值，并赋值给python变量

```python
import os

print(f"PID: {os.getpid()}, PPID: {os.getppid()}")
host = os.getenv('REDIS_HOST')
port = os.getenv('REDIS_PORT')
password = os.getenv('REDIS_PASSWORD')

print(f"{password}@{host}:{port}")
```

而参数值放在一个env文件里

```bash
REDIS_HOST=example.com
REDIS_PORT=5432
REDIS_PASSWORD=123456
```

通过执行`source .env` 后在终端查看

```bash
echo $REDIS_HOST $REDIS_PORT $REDIS_PASSWORD
> example.com 5432 123456
```

```bash
# 查看当前shell pid
echo $$
> 22650
# 运行python 文件
python variables.py
> PID: 23034, PPID: 22650
> None@None:None
```

发现python并未获取到存在shell环境变量中的值，python运行的父进程号为当前shell的进程号，而当时python运行是在子进程中运行

以上的`souce .env`操作是将变量存入到shell变量中，但未在环境变量中

## shell变量和环境变量

For [Standard UNIX variables](http://www.ee.surrey.ac.uk/Teaching/Unix/unix8.html) , says:

> Standard UNIX variables are split into two categories, environment variables and shell variables. In broad terms, shell variables apply only to the current instance of the shell and are used to set short-term working conditions; environment variables have a farther reaching significance, and those set at login are valid for the duration of the session. By convention, environment variables have UPPER CASE and shell variables have lower case names.
>
> 标准UNIX变量分为两类：环境变量和shell变量，shell变量仅在当前shell（主进程）有效，环境变量可以在子进程有效，通常环境变量为大写，shell变量为小写

For [POSIX-compatible shells](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_05_03) (including Bash), the standard says:

> **2.5.3 Shell Variables**
> Variables shall be initialized from the environment *[...]* If a variable is initialized from the environment, it shall be marked for export immediately; see the export special built-in. New variables can be defined and initialized with variable assignments, *[etc.]*
>
> 如果shell变量要转化为环境变量，执行export 命令

### 查看shell变量和环境变量

+ 查看shell变量使用`set`命令

```bash
set
# 截取部分
> ...
> status=0
> termcap
> terminfo
> userdirs
> usergroups
> watch=(  )
> widgets
> zle_bracketed_paste=( $'\C-[[?2004h' $'\C-[[?2004l' )
> zsh_eval_context=( toplevel )
> zsh_scheduled_events
> ...
```

另外可以通过set/unset 来设置/删除shell变量

```bash
set

  Display, set or unset values of shell attributes and positional parameters.

  - Display the names and values of shell variables:
    set

  - Mark variables that are modified or created for export:
    set -a

  - Notify of job termination immediately:
    set -b

  - Set various options, e.g. enable vi style line editing:
    set -o vi
```



```bash
unset

  Remove shell variables or functions.

  - Remove the variable foo, or if the variable doesn't exist, remove the function foo:
    unset foo

  - Remove the variables foo and bar:
    unset -v foo bar

  - Remove the function my_func:
    unset -f my_func
```

+ 查看环境变量使用env或者printenv

```bash
printenv
> VIRTUALENVWRAPPER_WORKON_CD=1
> VIRTUALENVWRAPPER_SCRIPT=/opt/anaconda3/bin/virtualenvwrapper.sh
> WORKON_HOME=/Users/simple/.virtualenvs
> VIRTUALENVWRAPPER_HOOK_DIR=/Users/simple/.virtualenvs
> ANDROID_HOME=/Users/simple/Library/Android/sdk
> _=/usr/bin/printenv
```

```bash
env

  Show the environment or run a program in a modified environment.

  - Show the environment:
    env

  - Run a program. Often used in scripts after the shebang (#!) for looking up the path to the program:
    env program

  - Clear the environment and run a program:
    env -i program

  - Remove variable from the environment and run a program:
    env -u variable program

  - Set a variable and run a program:
    env variable=value program

  - Set multiple variables and run a program:
    env variable1=value variable2=value variable3=value program
```

## 解决

方法就比较明显了

第一种，修改.env文件，加入export，然后执行`source .env`，最后运行python命令

```sh
export REDIS_HOST=example.com
export REDIS_PORT=5432
export REDIS_PASSWORD=123456
```

第二种，在运行python 命令前面加上env操作

```bash
env REDIS_HOST=example.com REDIS_PORT=5432 REDIS_PASSWORD=123456 python variables.py
> PID: 23517, PPID: 22650
> 123456@example.com:5432
```

## Tips

在shell中获取当前进程ID，父进程ID以及UID

```bash
echo "PID of this script: $$"
> 22650
echo "PPID of this script: $PPID"
> 22649
echo "UID of this script: $UID"
> 501
```

在python脚本中获取当前进程ID以及父进程ID

```python
import os
print(f"PID: {os.getpid()}, PPID: {os.getppid()}")
```



## 参考

[1] [UNIX Tutorial Eight](http://www.ee.surrey.ac.uk/Teaching/Unix/unix8.html)

[2] [difference-between-shell-and-environment-variables](https://stackoverflow.com/questions/3341372/difference-between-shell-and-environment-variables)

[3] [shell-variable-vs-environment-variable-which-one-is-preferred-if-both-have-the-same-name](https://unix.stackexchange.com/questions/364655/shell-variable-vs-environment-variable-which-one-is-preferred-if-both-have-the)

[4] [How to Set and List Environment Variables in Linux](https://linuxize.com/post/how-to-set-and-list-environment-variables-in-linux/)



