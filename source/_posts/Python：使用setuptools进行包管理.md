---
title: Python：使用setuptools进行包管理
date: 2019-12-13
tags: [Python]
---

作为Python的打包和分发工具，steuptools是十分易用的，而将程序打包之后，可以更方便的进行部署和分发，也可以上传到Pypi。这里结合一个例子，记录一下自己在使用打包过程的笔记和遇到的问题。

只需写一个简短的setup.py安装文件，就可以开始了你的Python应用打包。

## Setup.py

假设要打包的程序为`setup-demo`，则当前目录结构如下：

```
setup-demo/
  |- setup.py
  |- setup_demo/
    |- __init__.py
    |- ...
```

现在编写最基础的`setup.py`

```python
from setuptools import setup

setup(
    name='setup-demo',    # 应用名
    version='1.0',        # 版本号
    packages=['setup_demo']    # 包括在安装包内的Python包
)
```

### 打包

有了上面的 setup.py 文件，我们就可以打出各种安装包：

- 创建egg包

```
$ python setup.py bdist_egg
```

该命令会在当前目录下的`dist`目录内创建一个`egg`文件，名为`setup_demo-1.0-py3.6.egg`。文件名格式就是“应用名-版本号-Python版本.egg”，同时你会注意到，当前目录多了`build`和``setup_demo.egg-info`子目录来存放打包的中间结果。

- 创建tar.gz包

```
$ python setup.py sdist
```

同上例类似，只不过创建的文件类型是`tar.gz`，文件名为`setup-demo-1.0.linux-x86_64.tar.gz`。

- 创建wheel包

**官方推荐的打包方式使用 wheel 打包**，首先要安装 wheel：

```
$ pip install wheel
```

然后使用 bdist_wheel 打包：

```
$ python setup.py bdist_wheel
```

打包完成之后，可以使用 pip 安装到本地 Python 的 site-packages 目录。例如`pip install dist/setup_demo-1.0-py3-none-any.whl`，然后现在和其他使用 pip 安装的三方库一样使用了。

### 安装

- 安装应用

```
$ python setup.py install
```

该命令会将当前的Python应用安装到当前Python环境的`site-packages`目录下，这样其他程序就可以像导入标准库一样导入该应用的代码了。

- 开发方式安装

```
$ python setup.py develop 
```

或

```
$ pip install -e . 
```
<!--more-->
如果应用在开发过程中会频繁变更，每次安装还需要先将原来的版本卸掉，很麻烦。使用`develop`开发方式安装的话，应用代码不会真的被拷贝到本地Python环境的`site-packages`目录下，而是在`site-packages`目录里创建一个指向当前应用位置的链接。这样如果当前位置的源码被改动，就会马上反映到`site-packages`里。

### 上传

注册 PyPI 账号，登录 [pypi.python.org/pypi](https://pypi.python.org/pypi) Register 注册账号。虽然 setuptools 支持使用` python setup.py upload `上传包文件到 PyPI，但只支持 HTTP 而被新的 twine 取代。先安装 twine：

```
$ pip install twine
```

使用 twine 上传：

```
$ twine upload dist/*
```

输入 username 和 password 即上传至 PyPI。如果不想每次输入账号密码，可以在`~`目录下创建 `.pypirc`文件，内容如下：

```
[distutils]
index-servers =
    pypi

[pypi]
username: 
password: 
```

填上自己的账号密码即可，这里配置了官方的 pypi，若要配置其他仓库，按格式添加。回到 PyPI 主页即可看到上传的。

### 一些比较重要的参数

上面的 setup.py 文件内，只使用了name, version, packages，但是对于一个具有完成功能的包来说这是远远不够的，我的依赖、非源码文件等等怎么办？下面是一些同样重要的参数：

- packages: 列出项目内需要被打包的所有 package。一般使用` setuptools.find_packages() `自动发现。

  ```python
  packages=find_packages(exclude=['docs', 'tests*'])
  ```

- description：项目的简短描述，一般一句话就好，会显示在 PyPI 上名字下端。

- long_description: 对项目的完整描述。如果此字符串是 rst 格式的，PyPI 会自动渲染成 HTML 显示。也可指定使用 markdown。一般会加载README.md文件中的内容。

  ```
  long_description=long_description,
  long_description_content_type='text/x-rst
  # long_description_content_type='text/markdown',
  ```

- url: 通常为 GitHub上的链接或者 readthedocs 的链接。

- author:作者信息

  ```
  author='example',
  author_email='example@example.com'
  ```

- license:项目许可证。关于各种许可证的介绍和选择，参考：[choosealicense.com/](https://choosealicense.com/)

- classifiers:项目分类，完整可选项参考：[https://pypi.python.org/pypi?%3Aaction=list_classifiers](https://pypi.python.org/pypi?%3Aaction=list_classifiers)

  ```
  classifiers=[
      # How mature is this project? Common values are
      # 3 - Alpha
      # 4 - Beta
      # 5 - Production/Stable
      'Development Status :: 3 - Alpha',
  
      # Indicate who your project is intended for
      'Intended Audience :: Developers',
      'Topic :: Software Development :: Build Tools',
  
      # Pick your license as you wish (should match "license" above)
       'License :: OSI Approved :: MIT License',
  
      # Specify the Python versions you support here. In particular, ensure
      # that you indicate whether you support Python 2, Python 3 or both.
      'Programming Language :: Python :: 2',
      'Programming Language :: Python :: 2.6',
      'Programming Language :: Python :: 2.7',
      'Programming Language :: Python :: 3',
      'Programming Language :: Python :: 3.2',
      'Programming Language :: Python :: 3.3',
      'Programming Language :: Python :: 3.4',
  ]
  ```

  如果是私有项目，不希望开源，可以在classifiers中添加 `'Private :: Do Not Upload'`这样，万一有小伙伴手抖上传到Pypi，官方也不会收录。

- python_requires: 指定运行时需要的Python版本。

  ```
  python_requires='>=3.5'
  ```

  以上指定仅在3.5及以上版本使用。

- keywords:项目关键词列表

  ```
  keywords='sample setuptools development'
  ```

- project_urls:项目相关额外连接，如代码仓库，文档地址等

  ```
  project_urls={
      'Documentation': 'https://packaging.python.org/tutorials/distributing-packages/',
      'Funding': 'https://donate.pypi.org',
      'Say Thanks!': 'http://saythanks.io/to/example',
      'Source': 'https://github.com/pypa/sampleproject/',
      'Tracker': 'https://github.com/pypa/sampleproject/issues',
  }
  ```

- **install_requires**:项目依赖的 Python 库，使用 pip 安装本项目时会自动检查和安装依赖。

  ```
  install_requires=['pyyaml']
  ```

- `extras_require`:指定了可选的功能与依赖。某些特殊的、偏门的功能，可能绝大多数用户不会去使用。这些功能的依赖，不适合放在`install_requires`里。这时就可以用`extras_require`来指定。

  ```
  extras_require={
      'security': ['pyOpenSSL>=0.14', 'cryptography>=1.3.4', 'idna>=2.0.0'],
      'socks': ['PySocks>=1.5.6, !=1.5.7'],
  },
  ```

  以上以[requests](http://python-requests.org/)的设置为例。`extras_require`需要一个dict，其中按（自定义的）功能名称进行分组，每组一个列表，与`install_requires`规则相同。使用时，可以用类似`'requests[security, socks]'`的形式来指定。

- package_data:项目依赖数据文件，数据文件必须放在项目目录内且使用相对路径

  ```
  package_data={
      'setup_demo': ['data/*.yml'],
  }
  ```

  如果不指定作为目录的键为空串，则代表对所有模块操作（下例中将包含所有包内 data 目录下的 yaml 文件）：

  ```
  package_data={
      '': ['data/*.yml'],
  }
  ```

- data_files: 如果数据文件存在于项目外，则可以使用 data_files 参数或者 MANIFEST.in 文件进行管理。如果用于源码包，则使用 MANIFEST.in；如果用于 wheel，则使用 data_files。
  ```
  data_files=[('mydata', ['data/conf.yml'])]
  ```

- zip_safe: 决定应用是否作为一个zip压缩后的`egg`文件安装在当前Python环境中，还是作为一个以”.egg”结尾的目录安装在当前环境中。因为有些工具不支持zip压缩文件，而且压缩后的包也不方便调试，所以建议将其设为False

  ```
  zip_safe=False
  ```

- `entry_points` :用来支持自动生成脚本，其值应该为是一个字典，从 entry_point 组名映射到一个表示 entry_point 的字符串或字符串列表，如：

  ```
  entry_points={
      'console_scripts': [
          'foo=foo.entry:main',
          'bar=foo.entry:main',
      ],    
  }
  ```

### 示例

以一个简单工具包为例（全部代码在 [https://github.com/suAdminWen/translate-it](https://github.com/suAdminWen/translate-it)）

```python
import os

from codecs import open
from setuptools import setup, find_packages

here = os.path.abspath(os.path.dirname(__file__))


requires = [
    'requests>=2.22.0',
    'lxml>=4.4.1',
    'cachelib',
    'appdirs'
]


about = {}
with open(os.path.join(here, 'translate_it', '__version__.py'),
          'r', 'utf-8') as f:
    exec(f.read(), about)

with open('README.md', 'r', 'utf-8') as f:
    readme = f.read()


setup(

    name=about['__name__'],
    version=about['__version__'],
    description=about['__description__'],
    long_description=readme,
    long_description_content_type='text/markdown',
    python_requires='>=3.5',
    packages=find_packages(exclude=('tests', 'tests.*')),
    zip_safe=False,
    author=about['__author__'],
    author_email=about['__author_email__'],
    url=about['__url__'],
    include_package_data=True,
    classifiers=[
        "Programming Language :: Python :: 3.5",
        "Programming Language :: Python :: 3.6",
        "Programming Language :: Python :: 3.7",
        "Programming Language :: Python :: 3.8",
    ],
    entry_points={
        'console_scripts': [
            'translate_it = translate_it.translate_it:command_line_runner'
        ]
    },
    install_requires=requires,
)
```

### 注意的地方

刚开始的时候打完包之后发现部分模块并没有被包含进去，百思不得其解，后来发现缺少`__init__.py`文件，在Python3中，即使模块中不包含该文件，也可以当作一个模块，但是打包的时候，如果缺少该文件，则认为不是源码包含的模块，会被忽略掉。

## 其他文件

除了最基本核心的 setup.py 文件和主程序之外，还会看到其他一些文件。

- `setup.cfg`包含了构建时候的一些默认参数。例如：

  ```
  [bdist_wheel]
  universal=1
  ```

  用于在使用 bdist_wheel 的时候的默认设置 --universal 参数 。

  ```
  [build_sphinx]
  all-files = 1
  build-dir = docs/_build
  warning-is-error = 1
  ```

  使用sphinx生成文档是的一些配置。

- README.rst/README.md:项目说明文档，使用 reStrutruedText 可以在 PyPI 上很好的渲染，但 Markdown 则支持不够好。

- MANIFEST.in:此文件在打源码包的时候告诉 setuptools 还需要额外打包哪些文件。

  ```
  # Include the README
  include *.md
  
  # Include the license file
  include LICENSE.txt
  
  # Include the data files
  recursive-include data *
  ```

## 参考

- [setup.py里的几个require](https://note.qidong.name/2018/01/python-setup-requires/)
- [Python打包分发工具setuptools](https://juejin.im/post/5d46eb4bf265da03ef79f7e3)
- [Python打包分发工具setuptools简介](http://www.bjhee.com/setuptools.html)