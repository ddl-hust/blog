#

## 1.使用postgres数据库

```docker pull  postgres:11-alpine```

## 2.安装Miniflux2

```bash
git clone https://github.com/miniflux/docker.git
cd docker && wget -O miniflux https://github.com/miniflux/miniflux/releases/download/2.0.5/miniflux-linux-amd64
chmod +x miniflux
make image version=2.0.5
vim docker-compose.yml
```
