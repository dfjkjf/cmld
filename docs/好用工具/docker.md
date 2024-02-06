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


