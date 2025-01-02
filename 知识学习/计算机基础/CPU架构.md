[x86 Arm64 Amd64 的区别与联系](https://zhuanlan.zhihu.com/p/639230175)
[windows-386、arm、arm64 和 windows-amd64 区别](https://zhuanlan.zhihu.com/p/667726188)

> cpu 架构分为 x86 架构和 Arm 架构
> intel (英特尔)与 amd (超威半导体)是 x86 架构 CPU 制造商
> ARM 公司是 arm 架构 CPU 制造商

### x86 架构: x86、x86_64 和 x64

x86 和 x86_64 : 基于 X86 架构的不同版本, 位数不同 32 位和 64 位
x86_64 = x64 = amd64
x86 版本是 Intel 率先研发出 x86 架构, x86_64 版本(也称 x64)是 amd 率先研发出 x86 的 64 位版本, 所以 x86_64 也叫 amd64，主流的桌面 PC，笔记本电脑，服务器都在用 X86_64 的 CPU

### Arm 架构: arm64 和 aarch64

arm64 = aarch64
arm：32 位 ARM
arm64：64 位 ARM，arm64 是 ARM 架构的 CPU，苹果新出的电脑在用 ARM 架构的 CPU

有些路由器和嵌入式设备在用 arm64 的 CPU，手机和安卓平板电脑最常用的 CPU 也是 ARM 架构的

arm 的历史遗留问题，arm64 和 aarch 都曾代指过 64 位 arm 程序，目前 arm64 和 aarch64 概念已合并，新版 64 位 arm 程序统称 aarch64

![](https://pic4.zhimg.com/v2-c4f79563470221227dd14405175ba71d_1440w.jpg)

---

386，指的是 i386，是 intel80386，32 位架构
MIPS 是 MIPS 架构的 CPU。有些嵌入式设备和家用路由器在用 MIPS 架构的 CPU
