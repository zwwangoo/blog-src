---
title: pyenv的安装使用和问题解决
date: 2019-08-24
tags: [Ubuntu, Python, pyenv]
---

使用`pyenv`和`pyenv-virtualenv`可以轻松的管理不同版本的Python，而且各个版本的环境完全独立，互不干扰，在Linux下结合shell，是十分推荐的。
这里记录Ubuntu系统下的安装、使用的一些步骤，同时也记录一些使用Pyenv的一些比较复杂的问题的解决方案。

## 安装

### 依赖安装

为了避免一些不必要的麻烦，这里建议是提前安装一些依赖：

```shell
sudo apt install -y git make wget curl build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libncurses5-dev libncursesw5-dev llvm 
```

### 安装pyenv

- clone项目到当前用户目录下
````
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
````

注：如果使用zsh则将下面 `~/.bash_profile` 改为 `~/.zshrc`

- 添加环境环境变量
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
```

- 添加 `pyenv init` 到shell中
```
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
```

- 重启shell
```
exec "$SHELL"
```
- 查看pyenv安装版本，可以查看本机安装Python版本
```
pyenv verson
```

### 安装pyenv-virtualenv

- clone插件到pyenv的安装路径
```
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```

- 添加`pyenv virtualenv-init`到shell中
```
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
```
注：如果使用zsh则将下面 `~/.bash_profile` 改为 `~/.zshrc`

- 重启shell
```
exec "$SHELL"
```

## 使用

- 查看当前版本
```
pyenv version
```
- 查看所有版本
```
pyenv versions
```
- 查看所有可安装的版本
```
pyenv install --list
```
- 安装指定版本
```
pyenv install 3.6.5
```
- 安装新版本后rehash一下
```
pyenv rehash
```
- 删除指定版本
```
pyenv uninstall 3.6.5
```
- 指定全局版本
```
pyenv global 3.6.5
```

- 创建一个3.6.5版本的虚拟环境, 命名为v365env
```
pyenv virtualenv 3.6.5 v365env
```
- 激活虚拟环境
```
pyenv activate v365env
```
- 关闭虚拟环境
```
pyenv deactivate v365env
```
- 指定局部版本，会在当前目录下创建 `.python-version`，如果是git仓库，请记得将该文件添加到`.gitignore`，下次再进入当前目录是，会自动激活该虚拟环境。
```
pyenv local v365env
```

## 问题

### python3.6使用tkinter提示 `No module named ‘_tkinter’` 问题：

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
