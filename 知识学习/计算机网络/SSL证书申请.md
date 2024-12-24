> 给个人博客的域名申请 SSL 证书，从而实现个人博客的 https 访问

# 申请单域名证书

> 单域名证书指的单个子域名，例如 abc.example.com 为单域名，def.example.com 为另外一个单域名

- 给域名添加 ssl 证书
  参考：[Certbot Instructions \| Certbot](https://certbot.eff.org/instructions)

- 追加域名 ssl 证书
  参考：
  [User Guide — Certbot 3.1.0.dev0 documentation](https://eff-certbot.readthedocs.io/en/latest/using.html#changing-a-certificate-s-domains)
  [Let's Encrypt 证书的一些操作（Certbot） \| 菜包子](https://caibaoz.com/blog/2018/07/22/some-task-about-lets-encrypt-cert-certbot/)
  [Certbot 为新子域名添加证书](https://www.lefer.cn/posts/9482/)

```
$ sudo certbot -d xxx.xxx.com --expand

# 原本已有域名example.com和www.example.com，接着添加一个域名abc.example.com
$ sudo certbot certonly --cert-name example.com -d example.com -d www.example.com -d abc.example.com
```

- certbot 删除已有 ssl 证书

参考：[let's encrypt 如何用 certbot 删除一个证书](https://viencoding.com/article/311)

```bash
$ sudo certbot delete
```

# 申请泛域名证书

> 范域名证书指的多个子域名，写作\*.example.com，包含 abc.example.com、def.example.com 等子域名

参考：[使用 Let’s Encrypt 免费申请泛域名 SSL 证书，并实现自动续期](https://www.cnblogs.com/michaelshen/p/18538178)

笔者在阿里云购买的域名并完成 DNS 解析，因此这里通过阿里云 DNS 来给泛域名申请 SSL 证书，Github 脚本为[GitHub - justjavac/certbot-dns-aliyun: 阿里云 DNS 的 certbot 插件，用来解决阿里云 DNS 不能自动为通配符证书续期的问题](https://github.com/justjavac/certbot-dns-aliyun)，步骤如下：

1. 安装 aliyun cli 工具

   ```shell
   $ wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz
   $ tar xzvf aliyun-cli-linux-latest-amd64.tgz
   $ sudo cp aliyun /usr/local/bin
   $ rm aliyun
   ```

   安装完成后需要配置[凭证信息](https://help.aliyun.com/document_detail/110341.html)，其中地域 ID 可以参考：[地域和可用区](https://help.aliyun.com/zh/ecs/regions-and-zones?spm=a2c4g.11186623.0.0.212c33afWSL8IS#concept-2459516)

2. 安装 certbot-dns-aliyun 插件

   ```shell
   $ wget https://cdn.jsdelivr.net/gh/justjavac/certbot-dns-aliyun@main/alidns.sh
   $ sudo cp alidns.sh /usr/local/bin
   $ sudo chmod +x /usr/local/bin/alidns.sh
   $ sudo ln -s /usr/local/bin/alidns.sh /usr/local/bin/alidns
   $ rm alidns.sh
   ```

3. 申请证书

   测试是否能正确申请：

   ```sh
   $ certbot certonly -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run
   ```

   正式申请时去掉 `--dry-run` 参数：

   ```sh
   $ certbot certonly -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean"
   ```

4. 一般情况下，证书会自动续期，可通过以下命令测试是否正常实现自动续期，否则需要添加定时任务每个三个月执行一次

   ```sh
   $ certbot renew --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --dry-run
   ```
