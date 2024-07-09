---
title: Jupyter matplotlib画图中文字体缺失
date: 2022-09-26 22:48:08
tags: 
  - Bug
---
> 系统：MacOS
> 2022年09月26日22:48:08

## 事出有因

- 在jupyterlab中使用matplotlib画图，图的坐标轴中文字体不显示，且jupyter cell 抛出 缺失当前字体
## Bug现场
![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230212/image.3dbaeh35agk0.webp)


![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230212/image.2fyf6uk38fb4.webp)

## 迎刃而解
### 根因分析

- 根据错误堆栈信息可看出，matplotlib默认不自带中文字体，
- 需要先下载中文字体，放到对应环境包的路径下
### 实践求证

- 在虚拟环境中进入matplotlib包下查看
```bash
(jupyter_py38env) ➜  fonts cd ttf
(jupyter_py38env) ➜  ttf pwd
/usr/local/Caskroom/miniforge/base/envs/jupyter_py38env/lib/python3.8/site-packages/matplotlib/mpl-data/fonts/ttf
(jupyter_py38env) ➜  ttf ls
DejaVuSans-Bold.ttf            LICENSE_DEJAVU                 STIXSizOneSymReg.ttf
DejaVuSans-BoldOblique.ttf     LICENSE_STIX                   STIXSizThreeSymBol.ttf
DejaVuSans-Oblique.ttf         STIXGeneral.ttf                STIXSizThreeSymReg.ttf
DejaVuSans.ttf                 STIXGeneralBol.ttf             STIXSizTwoSymBol.ttf
DejaVuSansDisplay.ttf          STIXGeneralBolIta.ttf          STIXSizTwoSymReg.ttf
DejaVuSansMono-Bold.ttf        STIXGeneralItalic.ttf          cmb10.ttf
DejaVuSansMono-BoldOblique.ttf STIXNonUni.ttf                 cmex10.ttf
DejaVuSansMono-Oblique.ttf     STIXNonUniBol.ttf              cmmi10.ttf
DejaVuSansMono.ttf             STIXNonUniBolIta.ttf           cmr10.ttf
DejaVuSerif-Bold.ttf           STIXNonUniIta.ttf              cmss10.ttf
DejaVuSerif-BoldItalic.ttf     STIXSizFiveSymReg.ttf          cmsy10.ttf
DejaVuSerif-Italic.ttf         STIXSizFourSymBol.ttf          cmtt10.ttf
DejaVuSerif.ttf                STIXSizFourSymReg.ttf
DejaVuSerifDisplay.ttf         STIXSizOneSymBol.ttf
(jupyter_py38env) ➜  ttf
```

- 确实没有找到中文字体
### 解决方式

1. 找到Mac中Python使用的字体库的位置
```python
In [1]: import matplotlib

# 查找字体路径
In [2]: print(matplotlib.matplotlib_fname())
/usr/local/Caskroom/miniforge/base/envs/jupyter_py38env/lib/python3.8/site-packages/matplotlib/mpl-data/matplotlibrc

# 查看字体缓存路径
In [3]: print(matplotlib.get_cachedir())
/Users/roninsswang/.matplotlib
```

2. 下载字体
- 常见中英文字体对照名称

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230212/image.64ni21o79zc0.webp)

- 下载需要的字体，此处下载SimHei
   - 下载链接：[http://d.xiazaiziti.com/en_fonts/fonts/s/SimHei.ttf](http://d.xiazaiziti.com/en_fonts/fonts/s/SimHei.ttf)
3. 将下载的字体移动到matplotlib的字体库的位置
```bash
(jupyter_py38env) ➜  ttf pwd
/usr/local/Caskroom/miniforge/base/envs/jupyter_py38env/lib/python3.8/site-packages/matplotlib/mpl-data/fonts/ttf
(jupyter_py38env) ➜  ttf ls
DejaVuSans-Bold.ttf            LICENSE_DEJAVU                 STIXSizOneSymReg.ttf
DejaVuSans-BoldOblique.ttf     LICENSE_STIX                   STIXSizThreeSymBol.ttf
DejaVuSans-Oblique.ttf         STIXGeneral.ttf                STIXSizThreeSymReg.ttf
DejaVuSans.ttf                 STIXGeneralBol.ttf             STIXSizTwoSymBol.ttf
DejaVuSansDisplay.ttf          STIXGeneralBolIta.ttf          STIXSizTwoSymReg.ttf
DejaVuSansMono-Bold.ttf        STIXGeneralItalic.ttf          SimHei.ttf
DejaVuSansMono-BoldOblique.ttf STIXNonUni.ttf                 cmb10.ttf
DejaVuSansMono-Oblique.ttf     STIXNonUniBol.ttf              cmex10.ttf
DejaVuSansMono.ttf             STIXNonUniBolIta.ttf           cmmi10.ttf
DejaVuSerif-Bold.ttf           STIXNonUniIta.ttf              cmr10.ttf
DejaVuSerif-BoldItalic.ttf     STIXSizFiveSymReg.ttf          cmss10.ttf
DejaVuSerif-Italic.ttf         STIXSizFourSymBol.ttf          cmsy10.ttf
DejaVuSerif.ttf                STIXSizFourSymReg.ttf          cmtt10.ttf
DejaVuSerifDisplay.ttf         STIXSizOneSymBol.ttf
(jupyter_py38env) ➜  ttf
```

4. 修改配置文件matplotlibrc
- 进入下列路径下：
```bash
(jupyter_py38env) ➜  mpl-data pwd
/usr/local/Caskroom/miniforge/base/envs/jupyter_py38env/lib/python3.8/site-packages/matplotlib/mpl-data
```

- 编辑、保存退出

![image](https://cdn.staticaly.com/gh/sswfive/blog-pic@main/20230212/image.1au7nhy8v9kw.webp)

5. 清除缓存
```bash
rm -rf /Users/roninsswang/.matplotlib
```

6. 重启jupyterlab ，再此运行，Bug解除。
