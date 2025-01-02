# frp 实现 ssh 内网穿透

> 主要为了实现笔记本电脑通过 frp 实现香橙派的远程 ssh 访问，前提需要有一个具有公网 IP 的云服务器

### 安装 frp

[安装 \| frp](https://gofrp.org/zh-cn/docs/setup/)

> 注意在云服务器防火墙开放对应的端口

- 云服务器安装 frps (frp 服务端)
- 香橙派&笔记本电脑安装 frpc (frp 客户端)

### 配置 frp

[安全地暴露内网服务 \| frp](https://gofrp.org/zh-cn/docs/examples/stcp/)

> 这里的 frp 配置使得只有授权用户能够访问的 SSH 服务代理，实现内网服务的安全暴露
> 配置完成后记得运行云服务器的 frps 服务端，笔记本电脑和香橙派的 frpc 客户端，如果需要长期运行记得放到后台
>
> - 使用以下命令启动服务器：`./frps -c ./frps.toml`
> - 使用以下命令启动客户端：`./frpc -c ./frpc.toml`

##### 云服务器 frps 服务配置

```toml
bindAddr = "0.0.0.0"
bindPort = 7000

# token身份认证 确保只有授权用户能够建立连接
auth.method = "token"
auth.token = "token123456"

# 服务端 Dashboard 可通过http://[server addr]:7500进行访问
# 默认为 127.0.0.1，如果需要公网访问，需要修改为 0.0.0.0
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin"
```

##### 香橙派 frpc 配置

```toml
# 云服务器的公网ip地址
serverAddr = "x.x.x.x"
serverPort = 7000

auth.method = "token"
auth.token = "token123456"

# 客户端管理界面 可通过http://127.0.0.1:7400进行访问
webServer.addr = "127.0.0.1"
webServer.port = 7400
webServer.user = "admin"
webServer.password = "admin"

[[proxies]]
name = "secret_ssh"
type = "stcp"
# 只有与此处设置的 secretKey 一致的用户才能访问此服务
secretKey = "abcdefghijk"
localIP = "127.0.0.1"
localPort = 22
```

##### 笔记本电脑 frpc 配置

```toml
# 云服务器的公网ip地址
serverAddr = "x.x.x.x"
serverPort = 7000

auth.method = "token"
auth.token = "token123456"

webServer.addr = "127.0.0.1"
webServer.port = 7400
webServer.user = "admin"
webServer.password = "admin"

[[visitors]]
name = "secret_ssh_visitor"
type = "stcp"
# 要访问的 stcp 代理的名字即香橙派代理的名字
serverName = "secret_ssh"
secretKey = "abcdefghijk"
# 绑定本地端口以访问 SSH 服务
bindAddr = "127.0.0.1"
bindPort = 6000
```

### 笔记本电脑 ssh 远程连接香橙派

```bash
# user为远程访问的用户
$ ssh -o Port=6000 user@127.0.0.1
```
