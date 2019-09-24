---
title: Docker 部署 FastDFS
data: 2019-09-25
---

如果你的 FastDFS 文件系统需要高可用，需要部署在多台机器上的话，并且你的这些服务器上只跑 FastDFS 这个服务，那么 FastDFS 可能并不适合用 Docker 来部署，按照官方文档直接部署在机器上就可以，没必要采用容器来部署。一是增加了 Docker 安装管理监控的成本、二是

其实 FastDFS 并不适合容器化部署，因为 tracker 服务器向 storage 服务器报告自己的 IP， 而这个 IP 是容器内的 IP 。是 Docker 的一个私有 IP 段，这将导致客户端无法访问 storage 服务器。当然如果使用 host 网络或者能打通客户端到 storage 的网络解决方案，比如 flannel ，calico 等，这都可以，在 Kubernetes 基于服务发现，客户端也可以访问到 storage 服务器。

