# CentOS 7 安全策略

|   | 检测  |  加固 |   回退  |
| --- | --- | --- | --- |
| 系统信息 |  |
| 用户管理 |  |
| 服务管理 |  |
| 网络管理 |  |
| 日志审计 |  |
| 监控报警 |  |

----

## 一、系统信息

| 检查项 | 命令 |
| --- | --- |
| 查看内核信息 | uname -a |
| 查看所有软件包 | rpm -qa |
| 查看主机名 | hostname |
| 查看网络配置 | ifconfig -a |
| 查看路由表 | netstat -rn |
| 查看开放端口 | netstat -an |
| 查看当前进程 | ps -aux |

----

## 二、用户管理

### 2.1 检测弱口令

|   |   |
| --- | --- |
| 检查目的 | 检查弱口令 |
| 检查方法 | 使用 john the ripper和字典检测/etc/shadow文件中的弱口令 |
| 加固方法 | 使用passwd 用户名命令为用户设置复杂密码 |
| 回退方法 | 不需要回退 |
| 是否实施 |   |
| 备注 |   |

### 2.2 禁用无用账号

| 检查目的 | 减少系统无用账号，降低风险 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/passwd查看口令文件，与系统管理员确认不必要的账号FTP等服务的账号，如果不需要登录系统，shell应该/sbin/nologin |
| 加固方法 | 使用命令passwd -l <用户名>锁定不必要的账号 |
| 回退方法 | 使用命令passwd -u <用户名>解锁账号 |
| 是否实施 |   |
| 备注 | 需要与管理员确认此项操作不会影响到业务系统的登录 |

### 2.3 账号锁定策略

| 检查目的 | 防止口令暴力破解，降低风险 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/passwd查看口令文件，与系统管理员确认不必要的账号FTP等服务的账号，如果不需要登录系统，shell应该/sbin/nologin |
| 加固方法 | 设置连续输错10次密码，帐号锁定5分钟，使用命令vi /etc/pam.d/ system-auth修改配置文件，添加auth required pam_tally.so onerr=fail deny=10 unlock_time=300 |
| 回退方法 | 使用命令passwd -u 用户名 解锁账号 |
| 是否实施 |   |
| 备注 | 需要与管理员确认此项操作不会影响到业务系统的登录 |

### 2.4 检查特殊账号

| 检查目的 | 查看空口令和root权限的账号 |
| --- | --- |
| 检查方法 | 使用命令awk -F: '($2=="")' /etc/shadow查看空口令账号使用命令awk -F: ;($3==0); /etc/passwd查看UID为零的账号 |
| 加固方法 | 使用命令passwd  用户名为空口令账号设定密码UID为零的账号应该只有root，设置UID方法：usermod -u UID 用户名passwd 用户名设置空口令，只有在没有启用密u码策略的情况下可以使用passwd设置空口令或直接修改/etc/shadow文件，重设空口令usermod -u UID 用户名将UID设为加固前的值 |
| 回退方法 |   |
| 是否实施 |   |
| 备注 |   |

### 2.5 添加口令周期策略

| 检查目的 | 加强口令的复杂度等，降低被猜解的可能性 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/login.defs|grep PASS和cat /etc/pam.d/system-auth查看密码策略设置 |
| 加固方法 | 使用命令vi /etc/login.defs修改配置文件PASS_MAX_DAYS   90        #新建用户的密码最长使用天数PASS_MIN_DAYS   0          #新建用户的密码最短使用天数PASS_WARN_AGE   7         #新建用户的密码到期提前提醒天数使用chage命令修改用户设置，例如：chage -m 0 -M 30 -E 2000-01-01 -W 7 用户名表示：将此用户的密码最长使用天数设为30，最短使用天数设为0，帐号2000年1月1日过期，过期前7天里警告用户 |
| 回退方法 | vi /etc/login.defs将相关加固配置改回原来的值 |
| 是否实施 |   |
| 备注 | 锁定用户功能谨慎使用，密码策略对root不生效 |

### 2.6 添加口令复杂度策略

| 检查目的 | 加强口令的复杂度等，降低被猜解的可能性 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/pam.d/system-auth |grep pam_cracklib.so查看密码复杂度策略设置 |
| 加固方法 | 建议在/etc/pam.d/system-auth 文件中配置：password  requisite pam_cracklib.so difok=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=1 至少8位，包含一位大写字母，一位小写字母和一位数字 |
| 回退方法 | vi编辑 /etc/pam.d/system-auth，注释password  requisite pam_cracklib.so difok=3 minlen=8 ucredit=-1 lcredit=-1 dcredit=1 |
| 是否实施 |   |
| 备注 | 锁定用户功能谨慎使用，密码策略对root不生效 |

### 2.7 限制root远程登录

| 检查目的 | 限制root远程telnet登录，如果禁止telnet服务可忽略此项 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/securetty |grep CONSOLE查看是否禁止root远程登录 |
| 加固方法 | vi编辑/etc/securetty文件，配置：CONSOLE = /dev/tty01 |
| 回退方法 | vi编辑/etc/securetty文件，注释CONSOLE = /dev/tty01 |
| 是否实施 |   |
| 备注 | 如果禁止telnet服务可忽略此项 |

### 2.8  检查Grub/Lilo密码

| 检查目的 | 查看系统引导管理器是否设置密码 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/grub.conf|grep password查看grub是否设置密码使用命令cat /etc/lilo.conf|grep password查看lilo是否设置密码 |
| 加固方法 | vi编辑/etc/grub.confsplashimage这个参数下一行添加: password 密码如果需要md5加密，可以添加一行：password --md5 密码vi编辑/etc/lilo.confpassword=密码 |
| 回退方法 | vi编辑/etc/grub.conf，注释password 行vi编辑/etc/lilo.conf，注释password 行，/sbin/lilo –v更新lilo |
| 是否实施 |   |
| 备注 |   |

### 2.9 限制用户su

| 检查目的 | 限制能su到root的用户 |
| --- | --- |
| 检查方法 | 使用命令cat /etc/pam.d/su | grep pam_wheel.so查看配置文件，确认是否有相关限制 |
| 加固方法 | 使用命令vi /etc/pam.d/su修改配置文件，添加行例如：只允许test组用户su到rootauth    requiredch    pam_wheel.so group=test |
| 回退方法 | 注释掉auth    required    pam_wheel.so group=test |
| 是否实施 |   |
| 备注 |   |

### 2.10 SSH登录设置

|  
| ---
|  禁止密码登录PasswordAuthentication no ，采用密钥认证
|  SSH 禁止root登录PermitRootLogin no
|  SSH 双因素验证登录
|  SSH 限制登录用户
|  Fail2Ban 屏蔽暴力攻击IP
|  SSH 登录IP限制 AllowUsers User@ip

### 2.11 禁用或删除无用用户

|  |  | | | | | | | | | | |
| ---| --- | --- | --- | ---| --- | --- | --- | --- | --- | --- | --- |
|adm | lp | uucp | games | gopher | video | dip | ftp | audio | floppy | postfix

### 2.12 修改特殊文件权限

禁止非root用户执行/etc/rc.d/init.d/下的系统命令
chmod -R 700 /etc/rc.d/init.d/*
chmod -R 777 /etc/rc.d/init.d/*    #恢复默认设置

| 文件 | 防止非授权用户获得权限 | 加固 | 回退
| --- | --- | --- | --- |
| /etc/passwd | | chattr +i | chattr -i
| /etc/shadow | | chattr +i | chattr -i
| /etc/group  | | chattr +i | chattr -i
| /etc/gshadow| | chattr +i | chattr -i
| /etc/services | 系统服务端口列表文件加锁,防止未经许可的删除或添加服务 | chattr +i | chattr -i
| .bash_history | 避免删除.bash_history或者重定向到/dev/null | chattr +a |
| /usr/bin/vim | chmod 700 | chmod 755 |

### 2.13 禁止使用Ctrl+Alt+Del快捷键重启服务器

注释掉/etc/inittabbak文件中的 \#ca::ctrlaltdel:/sbin/shutdown -t3 -r now

----

## 三、服务管理

|  
| ---
| 密切关注所用软件安全事件、及时更新安全补丁
| 系统最小化安装、卸载删除不必要的服务
| 服务以最小权限运行，禁止root权限运行tomcat
| 启用 SELinux 模块 /etc/selinux/config  SELINUX=enforcing
| 禁用tomcat后台管理界面

----

## 四、网络管理

|  
| ---
| 限制SSH 端口及登录IP
| 屏蔽常见攻击
| 数据库只可内网连接，设置bind-address为内网地址并添加防火墙规则
| 只为相应的服务开放对应的端口 netstat -tunlp
| 本机localhost的任何请求 iptables -A INPUT -i lo -j ACCEPT
| zabbix-agent、zabbix-server、zabbix-web服务监听在内网地址，禁止公网访问

|  端口限制
| ---
| iptables -A INPUT -p tcp --dport 22 -j ACCEPT      #SSH
| iptables -A INPUT -p tcp --dport 80 -j ACCEPT      #HTTP
| iptables -A INPUT -p tcp --dport 443 -j ACCEPT     #HTTPS
| iptables -A INPUT -p tcp --dport 3306 -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT #MySQL
| iptables -I INPUT -m state  --state ESTABLISHED, RELATED -j ACCEPT
| iptable -L -n
| service iptables save
| service iptables restart

----

## 五、日志审计

| 远程备份安全日志 |
| --- |
| /var/log/message – 记录系统日志或当前活动日志。
| /var/log/auth.log – 身份认证日志。
| /var/log/cron – Crond 日志 (cron 任务).
| /var/log/maillog – 邮件服务器日志。
| /var/log/secure – 认证日志。
| /var/log/wtmp 历史登录、注销、启动、停机日志和，lastb命令可以查看登录失败的用户
| /var/run/utmp 当前登录的用户信息日志，w、who命令的信息便来源与此
| /var/log/yum.log Yum 日志。

----

## 六、监控报警

网络流量
分析日志文件

----

## 七、WEB安全

|  
| ---
| 命令行注入，对用户输入内容进行转义
| 目录遍历，tomcat
| 重定向错误页面40x、50x，避免错误日志输出
| 避免敏感信息输出到日志
| 用户密码加密存储，使用bcrypt算法进行加盐密码哈希

----

## 八、日常运维

1、Tomcat后台弱口令上传war包

2、Tomcat的PUT的上传漏洞(CVE-2017-12615)

3、Tomcat反序列化漏洞(CVE-2016-8735)

4、Tomcat JMX服务器弱口令

5、Tomcat 样例目录session操控漏洞

6、Tomcat本地提权漏洞(CVE-2016-1240)

7、Tomcat win版默认空口令漏洞(CVE-2009-3548)