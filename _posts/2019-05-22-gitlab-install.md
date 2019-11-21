---
title: CentOS7 install gitlab
date: 2019-05-22
slug:
tag:
  - gitlab
copyright: true
comment: true
---

目前gitlab官方给出的安装方式有很多种，普遍采用Omnibus包、Docker安装。官方说的😂```我们强烈建议使用 Omnibus 包安装 GitLab ，因为它安装起来更快、更容易升级版本，而且包含了其他安装方式所没有的可靠性功能。```

## 1.Omnibus包安装，也就是rpm包、deb包安装😂

## 1.安装

### 1.CentOS

```bash
# 安装依赖
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd

# 配置防火墙
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld

# 添加官方的软件包源
# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

# 国内用户添加清华大学镜像站的源，下载速度会快些。
sudo cat > /etc/yum.repos.d/gitlab-ce.repo <<EOF
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
EOF
sudo yum makecache
# 然后安装
yum install -y gitlab-ee
```

### 2.Ubuntu

```bash
sudo apt-get install openssh-server
curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null

# 添加清华大学的镜像站源 bionic是Ubuntu18.04 xenial是16.04，根据自己的Ubuntu发行版本修改一下下
deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu bionic main
sudo apt-get update
sudo apt-get install gitlab-ce
```

----

## 2.打补丁，补充汉化的补丁

```bash
git clone https://gitlab.com/xhang/gitlab.git
cd gitlab

# 获取当前安装的版本
gitlab_version=$(cat /opt/gitlab/embedded/service/gitlab-rails/VERSION)

# 生成对应版本补丁文件
git diff v${gitlab_version} v${gitlab_version}-zh > ../${gitlab_version}-zh.diff

gitlab-ctl stop

# 打补丁的时候会提示一些补丁文件不存在，一定要跳过这些文件，不然后面reconfig的时候会报错的。
# 不要一路狂奔按Enter，要一个个按Y跳过这些补丁😂
patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < ${gitlab_version}-zh.diff

# 重启gitlab
gitlab-ctl start
gitlab-ctl reconfigure
```

## 3.进行一些配置，gitlab的配置文件在/etc/gitlab/gitlab.rb

```rb
# 修改为你自己的域名或者IP，是单引号，而且前面的http不要改
external_url  'http://gitlab.domain'

# 邮件配置，选用外部SMTP服务器
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.office365.com"
gitlab_rails['smtp_port'] =  587
gitlab_rails['smtp_user_name'] = "xxxx@outlook.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'xxxx@outlook.com'
```
