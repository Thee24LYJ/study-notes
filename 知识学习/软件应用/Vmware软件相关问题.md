[VMware 打开运行一段时间后卡死，CPU 占比增至 100%\_vm 占用 cpu 过高-CSDN 博客](https://blog.csdn.net/hxinyu6666/article/details/127893227)

# 1.无法打开内核设备"\.\Globa 小 vx86”:系统找不到指定的文件。是否在安装 VMware Workstation 后重新引导？未能初始化监视器设备。

解决办法：

[vmware 启动虚拟机时报错“无法打开内核设备\\.\Global\vmx86”的解决办法](https://blog.csdn.net/xinbadar/article/details/119530002)

[Vmware 虚拟机无法打开内核设备“\\.\Global\vmx86“的解决方法](https://developer.aliyun.com/article/1135403)

# 2.无法打开内核设备“\.\VMCIDev\VMX”: 操作成功完成。是否在安装 VMware Workstation 后重新引导? 模块“DevicePowerOn”启动失败。 未能启动虚拟机。

解决办法：

[无法打开内核设备“\.\VMCIDev\VMX”: 操作成功完成。是否在安装 VMware Workstation 后重新引导? 模块“DevicePowerOn”启动失败。 未能启动虚拟机。](https://blog.csdn.net/qq_43674360/article/details/120911532)

[无法打开内核设备"\.\VMCIDev\VMX"是否在安装 VM 后重新引导](https://www.bilibili.com/read/cv34411010/?jump_opus=1)

# 3.VMware 虚拟机网络模式

[终于有人把 VMware 虚拟机三种网络模式讲清楚了！](https://zhuanlan.zhihu.com/p/666987647)
[VmWare 网络配置，只此一篇就够了](https://segmentfault.com/a/1190000024580532)
[深入解析 VMware 虚拟机网络设置：NAT 与桥接模式](https://blog.csdn.net/weixin_42509888/article/details/142956953)
[vmware 虚拟机三种网络连接方式](https://www.cnblogs.com/wushuaishuai/p/9258849.html)

> 本文是一篇实验记录，目的是为测试局域网内的两台主机，主机 A 与主机 B 上虚拟机能否 ping 通，通过更改虚拟机的网络配置模式实现主机 A 与主机 B 上虚拟机的双向 ping 通

### NAT 模式

> 虚拟机 IP：192.168.177.153/24
> 主机 IP：
> ​ 以太网：192.168.1.100/24 192.168.0.100/24 192.168.137.1/24 正常 ping 通(虚拟机 ping 主机)
> ​ WLAN：113.54.202.33/19 正常 ping 通(虚拟机 ping 主机)

源 MAC 地址：Src: VMware_72:67:9e (00:0c:29:72:67:9e) -> 虚拟机的网卡 MAC 地址

目的 MAC 地址：Dst: VMware_f4:c6:f7 (00:50:56:f4:c6:f7) -> 均不属于虚拟机和主机上的网卡 MAC 地址

远程主机其实是通过网线直接连接的，也算远程主机

> 虚拟机 IP：192.168.177.153/24
> 远程主机 IP(香橙派)：
> ​ 以太网：192.168.1.68/24 正常 ping 通(虚拟机 ping 远程主机)
> ​ WLAN：113.54.214.129 /19 正常 ping 通(虚拟机 ping 远程主机)
> 虚拟机->远程主机 正常 ping 通
> 远程主机->虚拟机 无法 ping 通

### 桥接模式

> 虚拟机 IP：113.54.202.223/19 手动配置 且虚拟网络编辑器中桥接至 WLAN
> 主机 IP：
> ​ 以太网：192.168.1.100/24 192.168.0.100/24 192.168.137.1/24 无法 ping 通(虚拟机 ping 主机)
> ​ WLAN：113.54.202.33/19 正常 ping 通(虚拟机 ping 主机)
> 只能 ping 通手动配置的子网内的 IP 地址(前提是虚拟网络编辑器中需桥接至对应的网卡)，若 ping 其他子网 IP 地址则显示网络不可达

> 虚拟机 IP：192.168.1.126/24 手动配置 且虚拟网络编辑器中桥接至以太网
> 主机 IP：
> ​ 以太网：192.168.1.100/24 正常 ping 通(虚拟机 ping 主机)
> ​ WLAN：113.54.202.33/19 无法 ping 通(虚拟机 ping 主机)
> 远程主机 IP：192.168.1.68/24 **双向 ping 通**

桥接模式下：可以实现同一局域网内，主机 Aping 通主机 B 上的虚拟机，目前测试网线直连的方式双向 ping 通成功；但是使用了多个无线路由器感觉还不太可行，校园网环境测试失败，无法实现双向 ping 通

NAT 模式下：可以实现主机 B 上的虚拟机 ping 通主机 A，但是无法 ping 通主机 B 上的虚拟机，可能得添加类似于 NAT 的技术实现，若主机 A 有 TCP 客户端需要主动连接主机 B 上的虚拟机，这时会出现一些问题
