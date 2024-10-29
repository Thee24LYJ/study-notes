[Windows 开启（WSL）Linux 子系统并远程连接 SSH_wsl ssh-CSDN 博客](https://blog.csdn.net/tzsm11/article/details/137093575)

[适用于 Linux 的 Windows 子系统文档 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/)

[使用 WSL 运行 Linux GUI 应用 | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/gui-apps)

WSL 安装 GPU 驱动
[CUDA Toolkit 12.6 Update 2 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network)

[深度学习 | 还在用双系统？一文教你 PyCharm+WSL2+CUDA 搭建开发环境\_pycharm wsl-CSDN 博客](https://blog.csdn.net/hxj0323/article/details/122026317)

# whisper 环境配置

开源项目：[GitHub - openai/whisper: Robust Speech Recognition via Large-Scale Weak Supervision](https://github.com/openai/whisper?tab=readme-ov-file#setup)
[Windows11 + WSL Ubuntu + Pycharm + Conda for deeplearning | 公孙启](https://www.gongsunqi.xyz/posts/3c995b2a/)

[深度学习 | 还在用双系统？一文教你 PyCharm+WSL2+CUDA 搭建开发环境\_pycharm wsl-CSDN 博客](https://blog.csdn.net/hxj0323/article/details/122026317)

1.conda 环境安装对应 Python 版本和 pytorch

2.安装 ffmpeg

3.安装 pyaudio
[linux 安装 pyaudio - CSDN 文库](https://wenku.csdn.net/answer/3j13e61gu3)

# WSL 环境语音唤醒

语音唤醒尝试一：mycroft-precise 失败
开源项目：[GitHub - MycroftAI/mycroft-precise: A lightweight, simple-to-use, RNN wake word listener](https://github.com/MycroftAI/mycroft-precise)

参考：
[一个轻量级的 RNN 语音唤醒引擎 - 李理的博客](https://fancyerii.github.io/books/mycroft-precise/)
[WSL 声音输入输出方案 - pulseaudio - 计算机视觉技术学习分享](https://gyrojeff.top/index.php/archives/WSL%E5%A3%B0%E9%9F%B3%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E6%96%B9%E6%A1%88-pulseaudio/)
[WSL 使用 pulseaudio 播放声音](https://www.lance.moe/archives/post-335/)
[No Default Input Device Available · Issue #3 · conda-forge/pyaudio-feedstock · GitHub](https://github.com/conda-forge/pyaudio-feedstock/issues/3#issuecomment-2359677622)

直接使用 pip install pyaudio 安装报错 fatal error: portaudio.h: No such file or directory 是因为缺少支持库 portaudio19-dev

pyaudio 语音支持报错：
ALSA lib confmisc.c:855:(parse_card) cannot find card '0'
OSError: [Errno -9996] Invalid input device (no default output device)
AttributeError: 'NoneType' object has no attribute 'stop_stream'

语音唤醒尝试二：snowboy 成功
开源项目：[GitHub - Kitt-AI/snowboy: Future versions with model training module will be maintained through a forked version here: https://github.com/seasalt-ai/snowboy](https://github.com/Kitt-AI/snowboy)

Ubuntu 22 系统

```bash
$ sudo apt-get install swig python3-pyaudio sox
$ pip install pyaudio
```
