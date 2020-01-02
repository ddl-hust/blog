---
title: Dockerfile 搓镜像的小技巧
date: 2020-01-02
update:
slug: 
tag:
  - docker
copyright: true
comment: true
---

## Dockerfile 最佳实践

关于 Dockerfile 最佳实践的博客，网上已经有很多很多啦，咱在这里就不赘述啦。在这里分享几个搓镜像的小技巧，尤其是针对于咱大陆的用户😂。

- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 推荐看一下官方博客上写的
- [Dockerfile 最佳实践](https://yeasy.gitbooks.io/docker_practice/appendix/best_practices.html#dockerfile-最佳实践) 

## 几个搓镜像的小技巧

#### 基础镜像

使用体积较小的基础镜像，比如 `alpine` 或者  `debian:buster-slim`，像 openjdk 可以选用`openjdk:xxx-slim` ，由于 openjdk 是基于 debian 的基础镜像构建的，所以向 debian 基础镜像一样，后面带个 `slim` 就是基于 `debian:xxx-slim` 镜像构建的。

```bash
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
debian                      buster-slim         e1af56d072b8        4 days ago          69.2MB
alpine                      latest              cc0abc535e36        8 days ago          5.59MB
```

不过需要注意的是，alpine 的 c 库是 `musl libc` ，而不是正统的 `glibc` ，另外对于一些依赖 `glibc` 的大型项目，像 openjdk 、tomcat、rabbitmq 等都不建议使用 alpine 基础镜像，因为 `musl libc` 可能会导致 jvm 一些奇怪的问题。这也是为什么 tomcat 官方没有给出基础镜像是 alpine 的 Dockerfile 的原因。

#### 国内软件源

使用默认的软件源安装构建时所需的依赖，对于绝大多数基础镜像来说，在国内网络环境构建时的速度较慢，可以通过修改软件源的方式更换为国内的软件源镜像站。目前国内稳定可靠的镜像站主要有，华为云、阿里云、腾讯云、163等。对于咱的网络环境，华为云的镜像站速度最快，平均 10MB/s，峰值可达到 20MB/s，极大的能加快构建镜像的速度。可以参考我曾经写过的一篇 [国内软件源镜像站伪评测](https://blog.502.li/archives/mirrors-test.html)

- 对于  alpine 基础镜像修改软件源

```bash
echo "http://mirrors.huaweicloud.com/alpine/latest-stable/main/" > /etc/apk/repositories ;\
echo "http://mirrors.huaweicloud.com/alpine/latest-stable/community/" >> /etc/apk/repositories ;\
apk update ;\
```

- debian 基础镜像修改默认原件源码

```bash
sed -i 's/deb.debian.org/mirrors.huaweicloud.com/g' /etc/apt/sources.list ;\
sed -i 's|security.debian.org/debian-security|mirrors.huaweicloud.com/debian-security|g' /etc/apt/sources.list ;\
apt update ;\
```

- Ubuntu 基础镜像修改默认原件源码

```bash
sed -i 's/archive.ubuntu.com/mirrors.huaweicloud.com/g' /etc/apt/sources.list
apt update ;\
```

- 对于 CentOS ??? 大哥，你确定要用 220MB 大小的基础镜像？

```bash
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
centos                      latest              0f3e07c0138f        3 months ago        220MB
```

建议这些命令就放在 RUN 指令的第一条，update 以下软件源，之后再 install 相应的依赖。

#### 时区设置

由于绝大多数基础镜像都是默认采用 UTC 的时区，与北京时间相差 8 个小时，这将会导致容器内的时间与北京时间不一致，因而会对一些应用造成一些影响，还会影响容器内日志和监控的数据。因此对于东八区的用户，最好在构建镜像的时候设定一下容器内的时区，一以免以后因为时区遇到一些 bug😂。可以通过环境变量设置容器内的时区。在启动的时候可以通过设置环境变量`-e TZ=Asia/Shanghai` 来设定容器内的时区。

##### alpine

- 但对于 alpine 基础镜像无法通过 `TZ` 环境变量的方式设定时区，需要安装 `tzdata` 来配置时区。

```bash
root@ubuntu:~/docke/alpine# docker run --rm -it -e TZ=Asia/Shanghai alpine date
Thu Jan  2 03:37:44 UTC 2020
```

- 对于 alpine 基础镜像，可以在 RUN 指令后面追加上以下命令

```bash
apk add --no-cache tzdata ;\
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ;\
echo "Asia/Shanghai" > /etc/timezone ;\
apk del tzdata ;\
```

- 通过 tzdate 设定时区

```bash
root@ubuntu:~/docke/alpine# docker build -t alpine:tz2 .
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM alpine
 ---> cc0abc535e36
Step 2/2 : RUN set -xue ;    echo "http://mirrors.huaweicloud.com/alpine/latest-stable/main/" > /etc/apk/repositories ;    echo "http://mirrors.huaweicloud.com/alpine/latest-stable/community/" >> /etc/apk/repositories ;    apk update ;    apk add --no-cache tzdata ;    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ;    echo "Asia/Shanghai" > /etc/timezone ;    apk del tzdata
 ---> Running in 982041a34dbf
+ echo http://mirrors.huaweicloud.com/alpine/latest-stable/main/
+ echo http://mirrors.huaweicloud.com/alpine/latest-stable/community/
+ apk update
fetch http://mirrors.huaweicloud.com/alpine/latest-stable/main/x86_64/APKINDEX.tar.gz
fetch http://mirrors.huaweicloud.com/alpine/latest-stable/community/x86_64/APKINDEX.tar.gz
v3.11.2-11-gd5cdcefa20 [http://mirrors.huaweicloud.com/alpine/latest-stable/main/]
v3.11.2-14-g973431591e [http://mirrors.huaweicloud.com/alpine/latest-stable/community/]
OK: 11261 distinct packages available
+ apk add --no-cache tzdata
fetch http://mirrors.huaweicloud.com/alpine/latest-stable/main/x86_64/APKINDEX.tar.gz
fetch http://mirrors.huaweicloud.com/alpine/latest-stable/community/x86_64/APKINDEX.tar.gz
(1/1) Installing tzdata (2019c-r0)
Executing busybox-1.31.1-r8.trigger
OK: 9 MiB in 15 packages
+ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
+ echo Asia/Shanghai
+ apk del tzdata
(1/1) Purging tzdata (2019c-r0)
Executing busybox-1.31.1-r8.trigger
OK: 6 MiB in 14 packages
Removing intermediate container 982041a34dbf
 ---> 3ec89f3e824d
Successfully built 3ec89f3e824d
Successfully tagged alpine:tz2
root@ubuntu:~/docke/alpine# docker run --rm -it alpine:tz2 date
Thu Jan  2 11:12:23 CST 2020
```

##### debian

- 通过启动时设定环境变量指定时区

```bash
root@ubuntu:~/docke/alpine# docker run --rm -it -e TZ=Asia/Shanghai debian date
Thu Jan  2 11:38:56 CST 2020
```

-  也可以再构建镜像的时候复制时区文件设定容器内时区

```bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ;\
echo "Asia/shanghai" > /etc/timezone ;\
```

##### ubuntu

- 通过启动时设定环境变量指定时区，发射失败😂，只能通过时区文件来设定时区了。

```bash
root@ubuntu:~/docke/alpine# docker run --rm -it -e TZ=Asia/Shanghai debian date
Thu Jan  2 11:38:56 CST 2020

root@ubuntu:~/docke/alpine# ^debian^ubuntu
docker run --rm -it -e TZ=Asia/Shanghai ubuntu date
Thu Jan  2 03:44:13 Asia 2020
```

> 在这里有个命令执行的小技巧，通过脱字符 `^` 来替换上一条命令中的 debian 为 ubuntu 然后执行相同的命令😂

- 通过时区文件来设定时区

```bash
apt 
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime ;\
echo "Asia/shanghai" > /etc/timezone ;\
```

#### 尽量使用 URL 添加源码

如果不采用分阶段构建，对于一些需要在容器内进行编译的项目，最好通过 git 或者 wegt 的方式将源码打入到镜像内，而非采用 ADD 或者 COPY ，因为源码编译完成之后，源码就不需要可以删掉了，而通过 ADD 或者 COPY 添加进去的源码已经用在下一层镜像中了，是删不掉滴啦。也就是说 `git & wget source ` 然后 `build` ，最后 `rm -rf source/` 这三部放在一条 RUN 指令中，这样就能避免源码添加到镜像中而增大镜像体积啦。

下面以 FastDFS 的 Dockerfile 为例

- 项目官方的 [Dockerfile](https://github.com/happyfish100/fastdfs/blob/52ac538a71fc9753c9dbcd4c75f581e9402f39a5/docker/dockerfile_local/Dockerfile)

```dockerfile
# centos 7
FROM centos:7
# 添加配置文件
# add profiles
ADD conf/client.conf /etc/fdfs/
ADD conf/http.conf /etc/fdfs/
ADD conf/mime.types /etc/fdfs/
ADD conf/storage.conf /etc/fdfs/
ADD conf/tracker.conf /etc/fdfs/
ADD fastdfs.sh /home
ADD conf/nginx.conf /etc/fdfs/
ADD conf/mod_fastdfs.conf /etc/fdfs

# 添加源文件
# add source code
ADD source/libfastcommon.tar.gz /usr/local/src/
ADD source/fastdfs.tar.gz /usr/local/src/
ADD source/fastdfs-nginx-module.tar.gz /usr/local/src/
ADD source/nginx-1.15.4.tar.gz /usr/local/src/

# Run
RUN yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y \
  &&  mkdir /home/dfs   \
  &&  cd /usr/local/src/  \
  &&  cd libfastcommon/   \
  &&  ./make.sh && ./make.sh install  \
  &&  cd ../  \
  &&  cd fastdfs/   \
  &&  ./make.sh && ./make.sh install  \
  &&  cd ../  \
  &&  cd nginx-1.15.4/  \
  &&  ./configure --add-module=/usr/local/src/fastdfs-nginx-module/src/   \
  &&  make && make install  \
  &&  chmod +x /home/fastdfs.sh
# export config
VOLUME /etc/fdfs


EXPOSE 22122 23000 8888 80
ENTRYPOINT ["/home/fastdfs.sh"]
```

- 经过本人优化后的 [Dockerfile](https://github.com/happyfish100/fastdfs/issues/327)

```bash
FROM alpine:3.10

RUN set -x \
    && echo "http://mirrors.huaweicloud.com/alpine/latest-stable/main/" > /etc/apk/repositories \
    && echo "http://mirrors.huaweicloud.com/alpine/latest-stable/community/" >> /etc/apk/repositories \
    && apk update \
    && apk add --no-cache --virtual .build-deps gcc libc-dev make perl-dev openssl-dev pcre-dev zlib-dev git \
    && mkdir -p /usr/local/src \
    && cd /usr/local/src \
    && git clone https://github.com/happyfish100/libfastcommon.git --depth 1 \
    && git clone https://github.com/happyfish100/fastdfs.git --depth 1    \
    && git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1  \
    && wget http://nginx.org/download/nginx-1.15.4.tar.gz \
    && tar -xf nginx-1.15.4.tar.gz \
    && cd /usr/local/src/libfastcommon \
    && ./make.sh \
    && ./make.sh install \
    && cd /usr/local/src/fastdfs/ \
    && ./make.sh \
    && ./make.sh install \
    && cd /usr/local/src/nginx-1.15.4/ \
    && ./configure --add-module=/usr/local/src/fastdfs-nginx-module/src/ \
    && make && make install \
    && apk del .build-deps \
    && apk add --no-cache pcre-dev bash \
    && mkdir -p /home/dfs  \
    && mv /usr/local/src/fastdfs/docker/dockerfile_network/fastdfs.sh /home \
    && mv /usr/local/src/fastdfs/docker/dockerfile_network/conf/* /etc/fdfs \
    && chmod +x /home/fastdfs.sh \
    && rm -rf /usr/local/src*
VOLUME /home/dfs
EXPOSE 22122 23000 8888 8080
CMD ["/home/fastdfs.sh"]
```

- 构建之后的对比

使用项目默认的 Dockerfile 进行构建的话，镜像大小接近 500MB 😂，而经过一些的优化，将所有的 RUN 指令合并为一条，最终构建出来的镜像大小为 30MB 😂。

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
fastdfs             alpine              e855bd197dbe        10 seconds ago      29.3MB
fastdfs             debian              e05ca1616604        20 minutes ago      103MB
fastdfs             centos              c1488537c23c        30 minutes ago      483MB
```

#### 使用虚拟编译环境

对于只在编译过程中使用到的依赖，我们可以将这些依赖安装在虚拟环境中，编译完成之后可以一并删除这些依赖，比如 alpine 中可以使用 `apk add --no-cache --virtual .build-deps`  ，后面加上需要安装的相关依赖。

```bash
apk add --no-cache --virtual .build-deps gcc libc-dev make perl-dev openssl-dev pcre-dev zlib-dev git
```

构建完成之后可以使用 `apk del .build-deps` 命令，一并将这些编译依赖全部删除。需要注意的是，`.build-deps` 后面接的是编译时以来的软件包，并不是所有的编译依赖都可以删除，不要把运行时的依赖包接在后面，最好单独 add 一下。

## docker 镜像分析工具

推荐阅读 [Docker 镜像分析工具 Dive(附视频)](https://www.qikqiak.com/post/docker-image-explore-tool-dive/) ，我就不赘述啦，其实是懒😂