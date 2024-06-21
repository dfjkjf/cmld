---
sort: 12
---

# Docker

## 安装

- [推荐](https://docs.docker.com/desktop/install/ubuntu/)
- [普通](https://docs.docker.com/engine/install/ubuntu/)

## 以nodered为例使用docker

`docker run -it -p 18800:1880 -v node_red_data:/data --name mynodered nodered/node-red:latest`

```bash
docker run              在 Docker 环境中运行一个容器。
-it                     交互式终端 (interactive terminal)。
-d                      后台运行 (daemon mode)。
-p 18800:1880           端口映射 (port mapping)。此标志将容器的 1880 端口映射到主机的 18800 端口。这意味着可以通过访问主机上的 18800 端口来访问容器中的 Node-RED 实例。
-p 1880                 容器端口 1880 随机映射到主机端口上。要找出哪个端口，请运行 `docker ps`
-v node_red_data:/data  数据卷挂载 (volume mounting)。此标志将主机的 node_red_data 目录挂载到容器中的 /data 目录。这意味着容器中的数据将存储在主机上的 node_red_data 目录中。（docker自动创建/var/lib/docker/volumes/node_red_data）
--name mynodered        容器名称 (container name)。此标志为容器指定了一个名称。
nodered/node-red        镜像名称 (image name)。此标志指定要运行的 Docker 镜像。在该示例中，我们正在运行 Node-RED 官方镜像的最新版本。
```

- 数据卷：
    - `docker volume create --name node_red_data`
    - `docker volume ls`
    - `docker cp  mynodered:/data /your/backup/dir`
    - `docker cp ./install.txt mynodered:/data`
    - `docker volume rm node_red_data`
- 分离终端：`Ctrl-p + Ctrl-q`
- 重新连接到终端：`docker attach mynodered`
- 更新容器：
    - `docker stop mynodered`
    - `docker rm mynodered`
    - `docker run -d -p 1880 -v node_red_data:/data --name mynodered nodered/node-red`
    - `docker ps`
- 重启容器：
    - `docker ps -a`
    - `docker start mynodered`
- 容器中启动新终端：
    - `docker exec -it mynodered /bin/bash`

## 一般使用

- 制作镜像：
	1. Dockerfile：
    ```dockerfile
    FROM ubuntu:18.04

    # 将宿主机的 aout/ 文件夹复制到容器的 / 目录下
    COPY aout/ /aout

    # 设置容器的工作目录为 /aout/
    WORKDIR /aout/

    # 容器启动时执行的命令
    CMD bash -c 'source release.com && ./throughput-sub 20'
    ```
    `docker build --pull=false -t my_custom_image .`

	2. `docker save -o ./rtsb-lite.tar`
- 加载镜像：
	- `docker load -i ./rtsb-lite.tar`
- 启动容器：
	1. 进入终端：
	    - `docker run --cpus="1" --memory="64m" --rm -it --name rtsb_pub rtsb_lite bash`
	2. 容器应用，应用结束容器退出：
	    - `docker run --cpus="1" --memory="64m" --rm -it --name rtsb_sub rtsb_lite bash -c "cd aout/ && source release.com && ./throughput-sub 20"`
	3. 镜像自带执行应用
	    - `docker run --cpus="1" --memory="64m" --rm -it --name rtsb_sub my_custom_image`
- 监控容器：
    - `docker stats`

## 限制网络流量

在宿主机上执行。$arg是容器在宿主机上的网卡：
1.	设置网络2Mbps
```bash
tc qdisc add dev $arg root handle 1: htb default 11
tc class add dev $arg parent 1: classid 1:1 htb rate 250kbps
tc filter add dev $arg protocol ip parent 1:0 prio 1 u32 match ip dst 0.0.0.0/0 flowid 1:1
```
2.	查看
```bash
tc -s qdisc show dev $arg
tc -s class show dev $arg
tc -s filter show dev $arg
```
3.	删除
```bash
tc filter del dev $arg parent 1:0 prio 1
tc class del dev $arg classid 1:1
tc qdisc del dev $arg root
```

已做成脚本，使用：./setdockernet.sh 1(/0) <net-name1> <net-name2>
```bash
#!/bin/bash

# 检查是否有至少一个参数
if [ $# -lt 2 ]; then
    echo "errot：at least two args"
    exit 1
fi

# 获取第一个参数
first_arg=$1

# 移除第一个参数，以便后续处理剩余的参数
shift

# 根据第一个参数的值，执行不同的命令
if [ "$first_arg" == "1" ]; then
        # 遍历所有参数
        for arg in "$@"; do
            echo "set：$arg"

            tc qdisc add dev $arg root handle 1: htb default 11
            tc class add dev $arg parent 1: classid 1:1 htb rate 250kbps
            tc filter add dev $arg protocol ip parent 1:0 prio 1 u32 match ip dst 0.0.0.0/0 flowid 1:1

            tc -s qdisc show dev $arg
            tc -s class show dev $arg
            tc -s filter show dev $arg

            echo "done：$arg"
            echo "--------------------------------------"
        done
elif [ "$first_arg" == "0" ]; then
        # 遍历所有参数
        for arg in "$@"; do
            echo "unset：$arg"

            tc filter del dev $arg parent 1:0 prio 1
            tc class del dev $arg classid 1:1
            tc qdisc del dev $arg root

            echo "done：$arg"
            echo "--------------------------------------"
        done

else
    echo "第一个参数既不是0也不是1，不知道该执行什么命令。"
fi
echo "All args done."
```