
以下是步骤：

1、下载中文字体（黑体，看准系统版本）[http://www.fontpalace.com/font-details/SimHei/](http://www.fontpalace.com/font-details/SimHei/)
2、安装字体

```
sudo mv SimHei.ttf /usr/share/fonts/
```

3、在`~/.config/matplotlib/`下创建 `matplotlibrc`添加以下三行：

```
font.family         : sans-serif        
font.sans-serif     : SimHei, Bitstream Vera Sans, Lucida Grande, Verdana, Geneva, Lucid, Arial, Helvetica, Avant Garde, sans-serif   
axes.unicode_minus  : False
```

或者 修改配置文件 `matplotlibrc` 同样在`matplotlib/mpl-data/fonts`目录下面添加三行(原配置文件这是三个配置项应该是被注释掉的)

**如果还不行**

删除matplotlib的缓存:
```
sudo rm ~/.cache/matplotlib
```
