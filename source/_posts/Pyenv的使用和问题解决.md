

## 问题

python3.6使用tkinter提示 `No module named ‘_tkinter’` 问题：

```
import _tkinter # If this fails your Python may not be configured for Tk
ModuleNotFoundError: No module named ‘_tkinter’
```

解决方法如下：

```bash
sudo apt update
sudo apt install python3-tk  # （如果是python2.x，安装sudo apt-get install python-tk即可）
sudo apt install tk-dev
```

安装完成之后，重新安装Python：

```
pyenv uninstall 3.6.6  # 卸载原来安装的版本
pyenv install 3.6.6  # 重新安装
```
