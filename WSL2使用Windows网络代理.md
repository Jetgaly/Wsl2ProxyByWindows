**WSL2使用Windows网络代理**

### **代理是什么？**

正常上网的情况下，输入网址，根据这个网址找到一个服务器，从服务器上把该网址上存储的资源发到本机，就完成了上网的过程。代理就是一个转接口，如果输入一个被wall掉的国外的网址，会先找到代理，代理是你的客户端可以访问到的服务器，然后代理帮助你进行网址查询，将查询到的信息返回给你。 

### **什么是http_proxy和https_proxy?**

这是两个环境变量，http_proxy ， http_proxy，程序会自动识别这两个环境变量，决定是否走代理。

**环境变量**（如 `http_proxy`、`https_proxy`、`all_proxy`）：
许多命令行工具（`curl`、`wget`、`git`、`apt` 等）会读取这些变量，自动通过代理发送请求

**`no_proxy` 环境变量**：
指定不经过代理的地址（如内网、本地服务）（访问这些地址的时候不会走代理）

```bash
export no_proxy="localhost,127.0.0.1,192.168.1.0/24,*.internal.com"
```



### 步骤

windows设置防火墙

clash for windows

clash-win64

都打勾

![](Images\be94cbc2560a157521d46b249f7f21d.png)

clash 开启局域网 ALLOW LAN

![](D:\study_res\mdNotes\WSL2网络代理\Images\1744812194426.jpg)

172.27.16.1（windows的ip） 可以在wsl2中查到（重启后可能不同）

WSL2 会将 Windows 宿主机的 IP 设置为默认网关，可以通过路由表查询：

```bash
jjet@jet:~/test$ ip route show | grep -i default | awk '{ print $3}'
172.27.16.1
```

临时生效（重启失效）注意ip和代理端口

```
export http_proxy="172.27.16.1:7890"
export https_proxy="172.27.16.1:7890"
```

永久生效

#### 将脚本写在/etc/profile.d下

在用户登录时，系统会先读取 /etc/profile 文件，而 /etc/profile 脚本里包含了对 /etc/profile.d/ 目录的处理逻辑，它会依次执行该目录下所有以 .sh 结尾的脚本文件。这样一来，系统管理员就可以将一些环境变量设置、别名定义等配置拆分成多个小脚本存放在这个目录中，便于管理和维护

#### 或者直接改变环境变量http_proxy，https_proxy



script

执行后对当前终端当前用户有效

```bash
#!/bin/bash

# 正则表达式匹配IP地址
IP_PATTERN="^([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9]{2}|2[0-4][0-9]|25[0-5])$"

proxy_path=$(ip route show | grep -i default | awk '{ print $3}')
if [[ $proxy_path =~ $IP_PATTERN ]]
then
        echo "proxy_path is "$proxy_path
        export http_proxy="$proxy_path:7890"
        export https_proxy="$proxy_path:7890"
        echo "http_proxy is "$http_proxy
        echo "https_proxy is "$https_proxy
else
        echo proxy_path pattern is not right
        echo $proxy_path
fi
```

测试

```bash
jjet@jet:~$ curl -I www.google.com
HTTP/1.1 200 OK
Transfer-Encoding: chunked
Cache-Control: private
Connection: keep-alive
Content-Security-Policy-Report-Only: object-src 'none';base-uri 'self';script-src 'nonce-wBT-jtfkr18hgCCm3MqHpg' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
Content-Type: text/html; charset=ISO-8859-1
Date: Thu, 17 Apr 2025 03:19:51 GMT
Expires: Thu, 17 Apr 2025 03:19:51 GMT
Keep-Alive: timeout=4
P3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Proxy-Connection: keep-alive
Server: gws
Set-Cookie: AEC=AVcja2f0HTVuJSk4aFesArgAwzkeDWTuEt0QjDCNbF3n9L6IVzRTF_jnbA; expires=Tue, 14-Oct-2025 03:19:51 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
Set-Cookie: NID=523=ODmgHygnwf6kaqn0OUhFmCfBym3D0-kqqvQRo5IP4SJQ6Tp1V2vSBjuE4YV7MTV95H1Khrrg1Kmr7meq88s3EglsG2sB7OgwJ3F1Myiq40sRWNZCejDCUUpBvoMvuMH1VXyYe841N5-Z7-SrIqK39zfzn3bdfdwBFuieQrWMRvcJ9271emcdN9GmTMw0xiG8ggXQSrw9riG4O-A; expires=Fri, 17-Oct-2025 03:19:51 GMT; path=/; domain=.google.com; HttpOnly
X-Frame-Options: SAMEORIGIN
X-Xss-Protection: 0
```

