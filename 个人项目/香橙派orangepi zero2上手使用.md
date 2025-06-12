# 系统镜像安装

系统镜像下载：
[Orange Pi Zero2-Orange Pi 官网-香橙派（Orange Pi）开发板,开源硬件,开源软件,开源芯片,电脑键盘](http://www.orangepi.cn/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-Zero-2.html)

使用镜像：`Orangepizero2_3.1.0_ubuntu_jammy_desktop_xfce_linux6.1.31`

通过 SD 卡烧录软件将该镜像写入 SD 卡

# 启动香橙派

> 遇到的问题：插入 SD 卡后上电 LED 不亮，接上串口上电只打印如下消息：
>
> ```
> U-Boot SPL 2021.10-orangepi(Jun 29 2023 - 22:36:55 +0800)DRAM: 1024 MiBTrying to boot from MMC1
> MMC: no card presentspl: mmc init failed with error: -123
> SPL: failed to boot from all boot devices###ERROR ### Please RESET the board ###
> ```
>
> 根据参考文章，看起来像是从 sd 卡中加载了 uboot 的 spl 部分出来运行，但想加载后面一部分出来运行时却找不到 sd 卡，这里和**sd 卡槽的插入检测引脚有关**，应该是 SD 卡槽硬件损坏导致的(其实就是损坏了插槽的几块铁片组成的一个接触开关，当 sd 卡插入时，无法让那几块贴片压到一起，因此无法导通)
> 芯片的 bootrom 不检测这个引脚，所以能把 spl 部分从 sd 卡顺利载入运行。而 spl 需要使用这个引脚来检测 sd 卡是否插入，就报错说找不到 sd 卡了
> 参考文章：
> [uboot 报错：SPL: failed to boot from all boot devices-CSDN 博客](https://blog.csdn.net/weixin_43850980/article/details/136684770) > [SD 卡/SD 卡卡槽/TF 卡/TF 卡卡槽的引脚定义\_tf 卡引脚定义-CSDN 博客](https://blog.csdn.net/jiangfutao/article/details/124466153)

### 时区设置：使用`timedatectl`命令

参考：[腾讯元宝 - 轻松工作 多点生活](https://yuanbao.tencent.com/bot/app/share/chat/7zfKtrh3RRKn)

​**​ 适用场景 ​**​：支持`systemd`的系统（如 CentOS 7+、Ubuntu 16.04+等）

- **​ 查看当前时区 ​**​：`timedatectl`
  输出中检查`Time zone`字段是否显示为`Asia/Shanghai`（东八区）
- ​**​ 列出可用时区 ​**​：`timedatectl list-timezones | grep -i "Asia/Shanghai"`-->确认`Asia/Shanghai`在列表中

- **​ 设置时区 ​**​：`sudo timedatectl set-timezone Asia/Shanghai`-->此命令会自动更新`/etc/localtime`软链接并同步系统配置
- **​ 验证时区是否更改成功 ​**​

```
$ date -R  # 应显示+0800
$ timedatectl | grep "Time zone"  # 应显示Asia/Shanghai`
```
