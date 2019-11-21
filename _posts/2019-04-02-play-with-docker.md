---
title: Play-with-Docker --在线使用/学习Docker
mathjax: true
copyright: true
comment: true
date: 2019-04-02
tags:
    - Docker
slug: Play-with-Docker
---

`Play With Docker` 是一个运行在浏览器中的Docker   Playground，只需要服务端部署好pwd服务，客户端无需安装任何环境，使用浏览器就可以在线体验 Docker。类似的还有近期上线的`instantbox`在线体验Linux发行版。
按照官方readme或wiki部署起来，会有不少坑，接下来就开始填坑。

----
1.安装Docker 以及docker-compose，相信你已经完成了。

```bash
apt-get install -y docker.io docker-compose
```

2.开启swarm需要指定ip

```bash
docker swarm init --advertise-addr you-ip
```

3.安装golang、dep、项目依赖:

```bash
apt-get install golang go-dep
export GOPATH=/root/go
mkdir -p $GOPATH/src/github.com/play-with-docker/
cd !$
git clone https://github.com/play-with-docker/play-with-docker.git  
cd play-with-docker

# 安装项目依赖
dep ensure -v
```

4.拉取dind镜像，也即是工作台运行实例的模板镜像

```bash
docker pull franela/dind
```

5.修改监听地址和域名，如果部署在VPS上需要把localhost修改为域名或IP
vi api.go
在api.go文件的 config.ParseFlags()下面添加config.PlaygroundDomain = "YOU-IP or DOMAIN"

```golang
func main() {
    config.ParseFlags()
    config.PlaygroundDomain = "YOU-IP or DOMAIN"
```

另外附上shell脚本中获取本机公网ip的方法

```shell
ips=`ifconfig | grep inet | grep -v inet6 | grep -v 127 | grep -v 172 |  sed 's/^[ \t]*//g' | cut -d ' ' -f2`
IPADDR=$ips
```

5.最后一步`docker-compose up`走起！😋

几个坑：
1.服务器RAM低于1GB 经常会提示```fatal error: runtime: out of memory```,代码没问题，是你的服务器内存太少了，开启`SWAP`可解决。如果自己编译`go build`的话也会遇到同样的错误，debug了好几遍发现是物理内存不足的问题。

```bash
    dd if=/dev/zero of=/swapfile bs=4MB count=512
    mkswap /swapfile
    chmod 600 /swapfile
    swapon /swapfile
```

2.有些环境下会提示找不到`$GOPATH`，而`docker-compose.yml`里使用了`$GOPATH`指定目录，可以换成绝对路径。
