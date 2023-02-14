---
title: jenkins构建golang及mysql
date: 2022-04-29 16:34:25
cover: /img/thumbnail/学习/后端/jenkins/jenkins.png
thumbnail: /img/thumbnail/学习/后端/jenkins/jenkins.png
tags:
- jenkins
- docker
- golang
- mysql
---

## <font color=#a862ea>简介</font>

这是一个简单的通过jenkins构建golang接口的教程，前提是在安装完[jenkins](http://http://175.178.107.242/2022/04/25/%E5%AD%A6%E4%B9%A0/%E5%90%8E%E7%AB%AF/jenkins/%E5%AE%89%E8%A3%85/)及[docker](http://http://175.178.107.242/2021/12/15/%E5%AD%A6%E4%B9%A0/%E5%90%8E%E7%AB%AF/docker/study/)后。

[项目地址](https://github.com/qianduanzhou/beego_study)

<!--more-->

## <font color=#a862ea>项目</font>

我是通过beego生成的golang项目，在根目录下新建一个docker-compose.yml文件，其中相关的含义可以查看[教程](https://www.runoob.com/docker/docker-compose.html)。

主要看golang中的command命令，使用到了[wait-for-it](https://github.com/vishnubob/wait-for-it)这个sh，主要是等待mysql容器完全建立完成，才去执行beego_test这个go可执行文件。

还有就是docker是相对路径执行，context是定义相对的路径，之后dockerfile中的路径可以通用相对路径来判断。

**docker-compose.yml**

```yaml
version: '2'
services: 
  golang:
    build: 
      context: .
      dockerfile: ./docker/go/Dockerfile
    container_name: "beego_test"
    ports: 
     - "8080:8080"
    network_mode: "test-net"
    depends_on:
      - mysql
    restart: on-failure
    command: ["./wait-for-it", "-t", "30", "mysql:3307", "--", "./beego_test"]
  mysql:
    build: ./docker/mysql
    container_name: "mydb_test"
    network_mode: "test-net"
    environment:
      MYSQL_ROOT_PASSWORD: 123456 #你的root密码
      MYSQL_ROOT_HOST: "%"
    ports: 
     - "3307:3306"
```

在根目录新建一个文件夹为docker，里面放入dockerfile及wait-for-it等相关文件，搭配上面的docker-compose.yml使用。

**/docker/go/Dockerfile**

```dockerfile
FROM golang:latest

#定义基础路径
ENV GO_PATH /go/src/go_test

#创建工作目录
RUN mkdir -p $GO_PATH
 
#进入工作目录
WORKDIR $GO_PATH
 
#安装netcat wait-for依赖nc命令
RUN sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt-get -q update
RUN apt-get -qy install netcat

#将当前目录下的所有文件复制到指定位置
COPY . $GO_PATH

# #复制等待sh
COPY ./docker/sh/wait-for-it $GO_PATH
#给予等待sh权限
RUN chmod 777 ./wait-for-it

#给予build之后的可执行文件权限
RUN chmod 777 ./beego_test

#端口
EXPOSE 8080
```

**/docker/mysql/Dockerfile**

```dockerfile
#基础镜像使用 mysql:latest
FROM mysql:latest

#定义会被容器自动执行的目录
ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d
 
#定义初始化sql文件
ENV INSTALL_DB_SQL init_database.sql
 
#把要执行的sql文件放到/docker-entrypoint-initdb.d/目录下，容器会自动执行这个sql
COPY ./$INSTALL_DB_SQL $AUTO_RUN_DIR/
 
#给执行文件增加可执行权限
RUN chmod a+x $AUTO_RUN_DIR/$INSTALL_DB_SQL
```

**/docker/mysql/init_database.sql**

```sql
-- 创建数据库
drop database if exists go_test;
create database go_test;
```

**/docker/sh/wait-for-it**

```sh
#!/usr/bin/env bash
# Use this script to test if a given TCP host/port are available

WAITFORIT_cmdname=${0##*/}

echoerr() { if [[ $WAITFORIT_QUIET -ne 1 ]]; then echo "$@" 1>&2; fi }

usage()
{
    cat << USAGE >&2
Usage:
    $WAITFORIT_cmdname host:port [-s] [-t timeout] [-- command args]
    -h HOST | --host=HOST       Host or IP under test
    -p PORT | --port=PORT       TCP port under test
                                Alternatively, you specify the host and port as host:port
    -s | --strict               Only execute subcommand if the test succeeds
    -q | --quiet                Don't output any status messages
    -t TIMEOUT | --timeout=TIMEOUT
                                Timeout in seconds, zero for no timeout
    -- COMMAND ARGS             Execute command with args after the test finishes
USAGE
    exit 1
}

wait_for()
{
    if [[ $WAITFORIT_TIMEOUT -gt 0 ]]; then
        echoerr "$WAITFORIT_cmdname: waiting $WAITFORIT_TIMEOUT seconds for $WAITFORIT_HOST:$WAITFORIT_PORT"
    else
        echoerr "$WAITFORIT_cmdname: waiting for $WAITFORIT_HOST:$WAITFORIT_PORT without a timeout"
    fi
    WAITFORIT_start_ts=$(date +%s)
    while :
    do
        if [[ $WAITFORIT_ISBUSY -eq 1 ]]; then
            nc -z $WAITFORIT_HOST $WAITFORIT_PORT
            WAITFORIT_result=$?
        else
            (echo -n > /dev/tcp/$WAITFORIT_HOST/$WAITFORIT_PORT) >/dev/null 2>&1
            WAITFORIT_result=$?
        fi
        if [[ $WAITFORIT_result -eq 0 ]]; then
            WAITFORIT_end_ts=$(date +%s)
            echoerr "$WAITFORIT_cmdname: $WAITFORIT_HOST:$WAITFORIT_PORT is available after $((WAITFORIT_end_ts - WAITFORIT_start_ts)) seconds"
            break
        fi
        sleep 1
    done
    return $WAITFORIT_result
}

wait_for_wrapper()
{
    # In order to support SIGINT during timeout: http://unix.stackexchange.com/a/57692
    if [[ $WAITFORIT_QUIET -eq 1 ]]; then
        timeout $WAITFORIT_BUSYTIMEFLAG $WAITFORIT_TIMEOUT $0 --quiet --child --host=$WAITFORIT_HOST --port=$WAITFORIT_PORT --timeout=$WAITFORIT_TIMEOUT &
    else
        timeout $WAITFORIT_BUSYTIMEFLAG $WAITFORIT_TIMEOUT $0 --child --host=$WAITFORIT_HOST --port=$WAITFORIT_PORT --timeout=$WAITFORIT_TIMEOUT &
    fi
    WAITFORIT_PID=$!
    trap "kill -INT -$WAITFORIT_PID" INT
    wait $WAITFORIT_PID
    WAITFORIT_RESULT=$?
    if [[ $WAITFORIT_RESULT -ne 0 ]]; then
        echoerr "$WAITFORIT_cmdname: timeout occurred after waiting $WAITFORIT_TIMEOUT seconds for $WAITFORIT_HOST:$WAITFORIT_PORT"
    fi
    return $WAITFORIT_RESULT
}

# process arguments
while [[ $# -gt 0 ]]
do
    case "$1" in
        *:* )
        WAITFORIT_hostport=(${1//:/ })
        WAITFORIT_HOST=${WAITFORIT_hostport[0]}
        WAITFORIT_PORT=${WAITFORIT_hostport[1]}
        shift 1
        ;;
        --child)
        WAITFORIT_CHILD=1
        shift 1
        ;;
        -q | --quiet)
        WAITFORIT_QUIET=1
        shift 1
        ;;
        -s | --strict)
        WAITFORIT_STRICT=1
        shift 1
        ;;
        -h)
        WAITFORIT_HOST="$2"
        if [[ $WAITFORIT_HOST == "" ]]; then break; fi
        shift 2
        ;;
        --host=*)
        WAITFORIT_HOST="${1#*=}"
        shift 1
        ;;
        -p)
        WAITFORIT_PORT="$2"
        if [[ $WAITFORIT_PORT == "" ]]; then break; fi
        shift 2
        ;;
        --port=*)
        WAITFORIT_PORT="${1#*=}"
        shift 1
        ;;
        -t)
        WAITFORIT_TIMEOUT="$2"
        if [[ $WAITFORIT_TIMEOUT == "" ]]; then break; fi
        shift 2
        ;;
        --timeout=*)
        WAITFORIT_TIMEOUT="${1#*=}"
        shift 1
        ;;
        --)
        shift
        WAITFORIT_CLI=("$@")
        break
        ;;
        --help)
        usage
        ;;
        *)
        echoerr "Unknown argument: $1"
        usage
        ;;
    esac
done

if [[ "$WAITFORIT_HOST" == "" || "$WAITFORIT_PORT" == "" ]]; then
    echoerr "Error: you need to provide a host and port to test."
    usage
fi

WAITFORIT_TIMEOUT=${WAITFORIT_TIMEOUT:-15}
WAITFORIT_STRICT=${WAITFORIT_STRICT:-0}
WAITFORIT_CHILD=${WAITFORIT_CHILD:-0}
WAITFORIT_QUIET=${WAITFORIT_QUIET:-0}

# Check to see if timeout is from busybox?
WAITFORIT_TIMEOUT_PATH=$(type -p timeout)
WAITFORIT_TIMEOUT_PATH=$(realpath $WAITFORIT_TIMEOUT_PATH 2>/dev/null || readlink -f $WAITFORIT_TIMEOUT_PATH)

WAITFORIT_BUSYTIMEFLAG=""
if [[ $WAITFORIT_TIMEOUT_PATH =~ "busybox" ]]; then
    WAITFORIT_ISBUSY=1
    # Check if busybox timeout uses -t flag
    # (recent Alpine versions don't support -t anymore)
    if timeout &>/dev/stdout | grep -q -e '-t '; then
        WAITFORIT_BUSYTIMEFLAG="-t"
    fi
else
    WAITFORIT_ISBUSY=0
fi

if [[ $WAITFORIT_CHILD -gt 0 ]]; then
    wait_for
    WAITFORIT_RESULT=$?
    exit $WAITFORIT_RESULT
else
    if [[ $WAITFORIT_TIMEOUT -gt 0 ]]; then
        wait_for_wrapper
        WAITFORIT_RESULT=$?
    else
        wait_for
        WAITFORIT_RESULT=$?
    fi
fi

if [[ $WAITFORIT_CLI != "" ]]; then
    if [[ $WAITFORIT_RESULT -ne 0 && $WAITFORIT_STRICT -eq 1 ]]; then
        echoerr "$WAITFORIT_cmdname: strict mode, refusing to execute subprocess"
        exit $WAITFORIT_RESULT
    fi
    exec "${WAITFORIT_CLI[@]}"
else
    exit $WAITFORIT_RESULT
fi
```

值得注意的是，在mysql的dockerfile中定义了自动执行sql文件，创建数据库。

## <font color=#a862ea>jenkins</font>

在jenkins中需要先添加go插件，可以在插件管理中下载并重启jenkins。然后就是对其进行配置，进入全局工具配置里面添加golang相关配置，主要是安装目录的设置，因为golang官网速度很慢，不一定能拉下来，所以可以在本地先安装好golang，配置好环境，然后设置下安装目录即可。

### <font color=#a862ea>golang安装</font>

golang的安装可以先在[官网](https://golang.google.cn/dl/)下载，然后通过rz上传到服务器上，接下来通过解压即可

```shell
tar -C /usr/local -xzf go1.12.5.linux-amd64.tar.gz
```

环境变量则是简单的设置添加到path中

```shell
export PATH=$PATH:/usr/local/go/bin // 添加go/bin到系统环境变量PATH中
```

然后验证go version即可。

### <font color=#a862ea>新增任务</font>

接下来就是新建一个自由风格的任务。

主要是在构建环境那块勾选Add timestamps to the Console Output及Set up Go programming language tools（并选则go版本）。

然后就是执行shell，

```shell
#设置代理，go get获取依赖速度会快些
export GO111MODULE=on
export GOPROXY=https://goproxy.cn
go get
#打包一个go可执行文件名为beego_test
go build -o beego_test
#删除相关容器及镜像
[ ! "$(docker network ls | grep test-net)" ] && docker network create -d bridge test-net
[ "$(docker ps -a | grep beego_test)" ] && docker rm -f beego_test
[ "$(docker ps -a | grep mydb_test)" ] && docker rm -f mydb_test
[ "$(docker image ls | grep beego_test_mysql)" ] && docker rmi -f beego_test_mysql
[ "$(docker image ls | grep beego_test_golang)" ] && docker rmi -f beego_test_golang
#执行docker-compose
docker-compose up -d
```

## <font color=#a862ea>总结</font>

以上的构建并部署golang项目比较简单，相关操作也是按部就班来，还有很多细化的操作如手动编写shell脚本来控制容器等目前暂未学习。
