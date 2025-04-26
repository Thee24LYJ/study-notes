# CUDA 编程

### WSL 上的 CUDA 环境搭建(WSL2)

WSL2 上 GPU 的 CUDA 环境下载安装
[CUDA Toolkit 12.6 Update 2 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network)

[深度学习 | 还在用双系统？一文教你 PyCharm+WSL2+CUDA 搭建开发环境\_pycharm wsl-CSDN 博客](https://blog.csdn.net/hxj0323/article/details/122026317)

C++语法编译报错：
address of a host variable "std::cout" cannot be directly taken in a device function

原因：不能使用 C++的 iostream 库的 cout 实现输出，只能使用 stdio.h 库的 printf 实现输出

### Windows 安装支持 GPU 的 pytorch

[CUDA 学习（一）——如何查看自己 CUDA 版本？\_cuda version-CSDN 博客](https://blog.csdn.net/bruce_zhao1407/article/details/109580835)
[nvcc -V 和 nvidia-smi 出现的 cuda 版本不同-CSDN 博客](https://blog.csdn.net/weixin_39518984/article/details/111406728)

pytorch 安装(CUDA 版本)：[Get Started](https://pytorch.org/get-started/locally/)

验证 pytorch 是否支持 CUDA：

```python
import torch
print(torch.cuda.is_available())
```

# CUDA 学习

### CUDA 基础

[CUDA 编程之快速入门 - 最难不过二叉树 - 博客园](https://www.cnblogs.com/skyfsm/p/9673960.html)
[CUDA 编程入门极简教程](https://zhuanlan.zhihu.com/p/34587739)

### WSL2 上使用 CUDA 遇到的问题

##### 使用`nvprof 可执行程序`报错：

Warning: nvprof is not supported on devices with compute capability 8.0 and higher.Use NVIDIA Nsight Systems for GPU tracing and CPU sampling and NVIDIA Nsight Compute for GPU profiling.
Refer https://developer.nvidia.com/tools-overview for more details

解决方法：
使用`nsys nvprof 可执行程序`即可

[nvprof --metrics 参数含义-CSDN 博客](https://blog.csdn.net/u013378687/article/details/130114918)

##### 使用`ncu 可执行程序`报错：

==ERROR== ERR_NVGPUCTRPERM - The user does not have permission to access NVIDIA GPU Performance Counters on the target device 0. For instructions on enabling permissions and to get more information see https://developer.nvidia.com/ERR_NVGPUCTRPERM

解决方法：
在 NVIDIA 控制面板里的“桌面”选项卡中,确保勾选了“启用开发人员设置”，然后在“开发人员”>“管理 GPU 性能计数器”中,选择“允许所有用户访问 GPU 性能计数器”,以授予不受限制的分析权限

[NVIDIA Development Tools Solutions - ERR_NVGPUCTRPERM: Permission issue with Performance Counters | NVIDIA Developer](https://developer.nvidia.com/nvidia-development-tools-solutions-err_nvgpuctrperm-permission-issue-performance-counters)
[在 WSL 上使用 Nsight System 和 Nsight Compute 常见问题与解决方案](https://zhuanlan.zhihu.com/p/644905434)
