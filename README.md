## 安装流程
### 1. 设置域名解析

在域名购买网站设置域名解析

例如现有域名: ``` example.com ```   !!!!!!!!! 具体域名换成自己的 !!!!!!!!!

在域名解析新增两条A记录并指向当前服务器
```
ngrok.example.com
*.ngrok.example.com
```
10分钟之后尝试 ``` ping ngrok.example.com ``` 如果ping通则代表设置成功

### 2.安装go
``` shell
yum install golang
```
如果没有权限，请使用 sudo 安装，安装完成之后，执行 ``` go version ``` 看到如下信息，证明安装成功：
go version go1.7.3 linux/amd64

设置go环境变量，在 ~/.bash_profile
``` shell
export GOPATH=$HOME/go
PATH=$PATH:$HOME/.local/bin:$HOME/bin:$GOPATH/bin
```
保存后，重新加载配置文件 source ~/.bash_profile
执行完成后，echo $GOPATH 可查看go路径
如果git版本小于1.7版本需要自行升级

### 3. 部署编译环境
``` shell
cd /home
git clone https://github.com/vok123/ngrok.git
cd ngrok
export GOPATH=/home/ngrok
```

### 4. 创建证书
``` shell
NGROK_DOMAIN="ngrok.example.com"
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt
```

覆盖文件

``` shell
cp server.key assets/server/tls/snakeoil.key
cp server.crt assets/server/tls/snakeoil.crt
cp base.pem assets/client/tls/ngrokroot.crt
```

### 5. 编译

#### 服务端:
``` shell
sudo make release-server
```
得到服务端 /home/ngrok/bin/ngrokd

#### 客户端按平台按需编译:

linux arm
``` shell
GOOS=linux GOARCH=arm make release-client
```
linux 32
``` shell
GOOS=linux GOARCH=386 make release-client
```
linux amd64
``` shell
GOOS=linux GOARCH=amd64 make release-client
```
windows 32
``` shell
GOOS=windows GOARCH=386 make release-client
```
windows 64
``` shell
GOOS=windows GOARCH=amd64 make release-client
```
mac 32
``` shell
GOOS=darwin GOARCH=386 make release-client
```
mac 64
``` shell
GOOS=darwin GOARCH=amd64 make release-client
```
执行完编译客户端指令之后将得到 ``` /home/ngrok/bin/*/ngrok ``` 并将其下载

### 6.启动服务端
httpAddr指定监听端口
``` shell
cd /home/ngrok/bin
nohup ./ngrokd -domain="ngrok.example.com" -httpAddr=":80" &
```
### 7.下载客户端并启动
新建 ngrok.cfg 配置文件 (这里固定4443端口)
```
server_addr: “ngrok.example.com:4443"
trust_host_root_certs: false
```
运行客户端，暴露本地8080端口, 控制台执行:
```
ngrok -subdomain test -config=./ngrok.cfg 8080
```
-subdomain 指令指定自定义域名
-subdomain test => test.ngrok.example.com

#### 看到以下结果及代表映射成功:
```
Tunnel Status                 online                                                                          
Version                       1.7/1.7                                                                         
Forwarding                    http://test.ngrok.example.com -> 127.0.0.1:8080                                         
Forwarding                    https://test.ngrok.example.com -> 127.0.0.1:8080                                        
Web Interface                 127.0.0.1:4040
```
#### 如果失败状态
```
Tunnel Status                 reconnecting
```
1. 检查服务器是否有启动``` ngrokd ```服务端程序 ```  ps -ef | grep ngrokd ```
2. 检查服务器防火墙是否有限制 ``` ngrokd ```
3. 检查 ``` ngrok.example.com ``` 是否能ping通






[![Build
status](https://travis-ci.org/inconshreveable/ngrok.svg)](https://travis-ci.org/inconshreveable/ngrok)

# ngrok - Introspected tunnels to localhost ([homepage](https://ngrok.com))
### ”I want to expose a local server behind a NAT or firewall to the internet.”
![](https://ngrok.com/static/img/overview.png)

## What is ngrok?
ngrok is a reverse proxy that creates a secure tunnel from a public endpoint to a locally running web service.
ngrok captures and analyzes all traffic over the tunnel for later inspection and replay.

## ngrok 2.x

ngrok 2.x is the successor to 1.x and the focus of all current development effort. Its source code is not available.

**NOTE** This repository contains the code for ngrok 1.x.

## Status of the ngrok 1.x project

ngrok 1.x is no longer developed, supported or maintained by its author, except to ensure that the project continues to compile. The contribution policy has the following guidelines:

1. All issues against this repository will be closed unless they demonstrate a crash or other complete failure of ngrok's functionality.
2. All issues against this repository are for 1.x only, any issues for 2.x will be closed.
3. No new features will be added. Any pull requests with new features will be closed. Please fork the project instead.
4. Pull requests fixing existing bugs or improving documentation are welcomed.

#### The ngrok 1.x hosted service

ngrok.com ran a pay-what-you-want hosted service of 1.x from early 2013 until April 7, 2016. Afterwards, it only runs 2.x service.

## Production Use

**DO NOT RUN THIS VERSION OF NGROK (1.X) IN PRODUCTION**. Both the client and server are known to have serious reliability issues including memory and file descriptor leaks as well as crashes. There is also no HA story as the server is a SPOF. You are advised to run 2.0 for any production quality system. 

## What can I do with ngrok?
- Expose any http service behind a NAT or firewall to the internet on a subdomain of ngrok.com
- Expose any tcp service behind a NAT or firewall to the internet on a random port of ngrok.com
- Inspect all http requests/responses that are transmitted over the tunnel
- Replay any request that was transmitted over the tunnel


## What is ngrok useful for?
- Temporarily sharing a website that is only running on your development machine
- Demoing an app at a hackathon without deploying
- Developing any services which consume webhooks (HTTP callbacks) by allowing you to replay those requests
- Debugging and understanding any web service by inspecting the HTTP traffic
- Running networked services on machines that are firewalled off from the internet

## Developing on ngrok
[ngrok developer's guide](docs/DEVELOPMENT.md)
