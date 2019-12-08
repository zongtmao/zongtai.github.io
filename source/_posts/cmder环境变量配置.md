### Cmder
    电脑系统都自带命令行输入面板，个人感觉样式太难看，而且无法自定义个人喜好使用，所以想搞搞cmder，点击鼠标右键就能快速定位到文件夹，进行操作

1.官网下载cmder: 官网地址：<http://cmder.net/>

2.安装：直接解压到某个目录就可以了，点击Cmder.exe运行。

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/1.png)

3.配置环境变量

- 变量名： CMDER_HOME
- 变量值： 安装绝对路径

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/2.png)

最后在Path添加一条斜体文字

%CMDER_HOME%

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/3.png)

4.添加cmder到右键菜单，方便使用

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/4.png)

然后输入命令：

cmder /register all 回车

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/5.png)

此时点击任何文件夹右键都有了cmder命令

使用问题

1.解决中文乱码问题

Settings->Startup->Environment 添加

set LANG=zh_CN.UTF-8

set LC_ALL=zh_CN.utf8

![img](https://raw.githubusercontent.com/zongtmao/markdownPicture/master/images/cmder/6.png)

最后重启cmder就能正常显示

2.常用命令

- 利用Tab，自动路径补全；
- 利用Ctrl+T建立新页签；利用Ctrl+W关闭页签;
- 利用Ctrl+Tab切换页签;
- Alt+F4：关闭所有页签
- Alt+Shift+1：开启cmd.exe
- Alt+Shift+2：开启powershell.exe
- Alt+Shift+3：开启powershell.exe (系统管理员权限)
- Ctrl+1：快速切换到第1个页签
- Ctrl+n：快速切换到第n个页签( n值无上限)
- Alt + enter： 切换到全屏状态；
- Ctr+r 历史命令搜索
- dir 可查看文件夹里有多个文件夹和文件
- e. 可打开资源管理 页面

3.设置aliases及分屏打开vscode

用文本编辑器打开安装路径下 -> config -> user-aliases.cmd

添加相应的命令， 使得可以自定义一些短命令来替代某些长命令：

gc = git commit -am $1 vs = "C:\Program Files\Microsoft VS Code\Code.exe" $1 -new_console:s80H

键入命令 vs 就可直接在窗口右边80%横向打开vscode，若是想纵向打开则更改参数(new_console:s50V)，当中的数字作为百分比。（注意cmder窗口要足够大小才能分栏显示）