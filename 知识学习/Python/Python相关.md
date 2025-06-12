# 使用 from utils import check_in 导入自定义 Python 包报错

[腾讯元宝 - 轻松工作 多点生活](https://yuanbao.tencent.com/bot/app/share/chat/CoOV28DqLxcJ)

```python
# check.py
import sys
import os

# 获取当前脚本的父目录（即 scripts 目录的父目录，即工程根目录）
root_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(root_dir)

from utils import check_in  # 此时可以正确导入
```
