---
title: reality的初次搭建
date: 2025-12-04 14:24:33
tags: reality vps 开始
---


# 代理服务器

## 过墙原理
### 传统协议
<p>在客户端与服务器连接后，使用tls协议加密，将流量的传输隐藏在tls下，在明面上是客户端与网站的超文本连接，规避了防火墙对返回流量的拦截<br>
<p>缺点：<br>
1.个人使用服务器ip及域名通常固定，频繁连接明显<br> 
2.证书使用acme客户端获取证书，需要维护，同时与主流网站不同<br>
<p>防火墙视角：<br>
客户端--------->服务器<br>
客户端<---------服务器

### reality协议
<p>在客户端与服务器握手后，reality借用预先设定网站A的证书伪装成网站A，在加密下传输流量，在明面上是客户端借助服务端访问合法网站A，完全合规<br>
<p>优势:<br>
1.减少域名花费与证书管理<br>
2.把流量转发伪装为合规传输,隐藏在对网站A的访问中

<p>防火墙视角:<br>
客户端-------->服务器-------->网站A<br>
客户端<--------服务器<--------网站A







## 使用docker compose搭建
### 下载docker
### 手动配置文件在/opt/xray/目录下
#### 1 *.env*   (environment文件)
`XRAY_IMAGE=teddysun/xray:latest`
`SERVER_HOST=************        `
`SERVER_PORT=443`
`TZ=Asis/Shanghai`
`UUID=**********    `
`DESTINATION_DOMAIN=www.apple.com`
`DESTINATION_PORT=443`
`SHORT_IDS=88,8888,1234`
`LOG_LEVEL=debug`
`NETWORK_MODE=host`
<p>好处<br>
1.环境文件使xray配置文件中的具体信息输入简化，免去c+v的步骤<br>
2.当docker compose内存在多个实例时，可一次性更改<br>

#### 2 *config.json*
直接copy服务器本地部署的文件，把具体信息替换为.env中的变量（不然不是不需要环境文件了吗：）

#### 3 *docker compose.yml* ()
管理docker
#### 4 *xray-manager.sh*
`cd /opt/xray`

`case $1 in`
`    start)`
`        docker compose up -d`
`        ;;`
`    stop)`
`        docker compose down`
`        ;;`
`    restart)`
`        docker compose restart`
`        ;;`
`    status)`
`        docker compose ps`
`        docker compose logs --tail=5`
`        ;;`
`    logs)`
`        docker compose logs -f`
`        ;;`
`    update)`
`        docker compose pull`
`        docker compose up -d`
`        ;;`
`    backup)`
`        cp config.json "config-$(date +%Y%m%d-%H%M).json.bak"`
`        ;;`
`    *)`
`        echo "命令 $0 {start|stop|restart|status|logs|update|backup}"`
`       ;;`
`esac`
<p>使linux上的命令在docker compose中实现以管理xray

#### 使用
<p>和本地部署相同，但要把命令改为xray-manager.sh中设置的(真让人不习惯)<br>
如：docker compose up -d=systemctl start

## 踩坑及解決

### ssh登入
#### 1.1
<p>坑：使用用户名与密码登入，服务器收到未知ip请求<br>
解决：使用xshell的rsa非对称加密，服务器仅允许签名远程登入<br>
学习：初步了解了公钥密码的使用过程与用途（信息加密，签名）

#### 2.1
<p>坑：修改ssh监听端口后，未修改防火墙放行新端口，导致xshell无法登入<br>
解决：进入救援模式，挂载后进入chroot,修改防火墙放行端口<br>

### 入站出站监听回环
<p>坑：使用dokodemo-door监听入站，freedom出站后流量被入站再次监听，字节描述词超级大爆,耗尽系统资源，ssh无法连接 ：（<br>
解决: 救援模式进入chroot后无果，重装alma9系统，satrt_again >_< <br>
教训 :<br>
 1.ssh连接后，出现问题不要重新连接，大概率发生小概率连接不上的情况 <br>
2.服务器没有任何预防,docker中就不会有<br>
3.服务器没有快照, <br>
学习: <br>
1.先sock测试<br>
2.连接频率设置以及预警脚本(好朋友ds帮我的:)

### reality使用自身域名
<p>坑：使用自身域名，reality协议混淆,tls握手错误<br>
解决：换用apple,发挥reality原本的作用 <br>
学习：reality原理和使用知名域名的好处

## Other
### arch实现终端代理
完全参考https://blog.linioi.com/posts/clash-on-arch/ 这篇博客

### api监听
<p>在入站中添加了监听本地用于api流量查询

### policy分组
<p>添加了客户level,区别上行下行缓存等资源占用

