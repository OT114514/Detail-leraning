# Detail-leraning
## 5.17 
学习了labelme的基础使用和更改vscode的python环境路径
1.labelme的基础使用
使用labelme官网提供的“STATER GUIDE”               （https://wkentaro.notion.site/STARTER-GUIDE-199f17e59a114086b8f40f3fa0c0576a）
![屏幕截图 2024-05-16 234524](https://github.com/OT114514/Detail-leraning/assets/169598472/c16a52b8-bac9-406a-af1f-7f40e99243da)
2.python环境路径的更改
按f1或者 ctrl+shift+p 选择“python 选择解释器”
![屏幕截图 2024-05-17 010619](https://github.com/OT114514/Detail-leraning/assets/169598472/2a60a595-d072-4691-8eec-68e790d220b2)
点进去后选择自己需要的python环境

## 5.18
了解了不同终端之间的独立性，我现在有两个python终端，一个是自带的终端，一个是anaconda终端
两个终端所下载的内容不同，anaconda终端里有labelme相关的exe文件
想要运行相应的文件，要在环境路径中添加exe文件所在的文件夹到系统路径中“path”
![128b8ee8e762f35dd1ade213f03c6f2](https://github.com/OT114514/Detail-leraning/assets/169598472/0b571027-c701-414b-b59c-84bdd72310d1)
今天发现的一个错误，labelme不小心下在原来的环境里面了，导致anaconda终端一直运行不了 labelme相关指令，此时可以输入pip install labelme来查看该终端是否下载了labelme相关exe文件

## 5.20~5.23
清除终端中先前的语句，让界面看起来更加清爽 输入“cls”
我pytorch版本选择2.1.2 cuda选择11.8，python版本在11以上才可以添加opencv环境
opencv环境直接用pip install opencv——python 语句就可以下载
