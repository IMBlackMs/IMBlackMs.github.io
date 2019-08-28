## 0x00 背景
-----------------------------------------------------------------------------------------------------------------------------------
除了.htaccess构建PHP后门外，.user.ini使用的更为广泛，只要是以fastcgi运行的php都可以用这个方法,不像.htaccess那么局限。
## 0x01 .user.ini
-----------------------------------------------------------------------------------------------------------------------------------
### .user.ini到底是什么东东？？？

[这是官方给出的解释](https://www.php.net/manual/zh/configuration.file.per-user.php)
```
自 PHP 5.3.0 起，PHP 支持基于每个目录的 .htaccess 风格的 INI 文件。此类文件仅被 CGI／FastCGI SAPI 处理。此功能使得 PECL 的 htscanner 扩展作废。如果使用 Apache，则用 .htaccess 文件有同样效果。
```
不是很清楚，但是我们知道当执行PHP的时候会先调用php.ini配置文件，官方文档中还提到
```
除了主 php.ini 之外，PHP 还会在每个目录下扫描 INI 文件，从被执行的 PHP 文件所在目录开始一直上升到 web 根目录（$_SERVER['DOCUMENT_ROOT'] 所指定的）。如果被执行的 PHP 文件在 web 根目录之外，则只扫描该目录。
```
意思就是除了php.ini以外，在PHP启动的时候还会扫描其他INI文件，包括.user.ini，那么.user.ini就可以重新修改配置了。但是官方文档中还提到
```
在 .user.ini 风格的 INI 文件中只有具有 PHP_INI_PERDIR 和 PHP_INI_USER 模式的 INI 设置可被识别。 
```
.user.ini修改配置是有条件的，只有在**PHP_INI_PERDIR** 和 **PHP_INI_USER** 模式下才可以自定义配置。其中配置模式有以下四种
![](inimode.png)

**结论：在fastcgi运行的PHP中，如果配置模式为PHP_INI_PERDIR或HP_INI_USER，那么.user.ini会添加或者修改原PHP中的配置选项**

### .user.ini用途
- 临时修改原有配置:文件大小、用户数量、文件目录、支持扩展等
- 添加配置选项

### 为什么会出现.user.ini漏洞
- 代码不规范，在上传文件时没有进行筛选，用户可以任意上传.user.ini
- 文件中已存在.user.ini
**通过修改.user.ini中的auto_prepend_file,自动执行预先设置的文件**

### .user.ini测试
那么，我们可以猥琐地想一下，在哪些情况下可以用到这个姿势？ 比如，某网站限制不允许上传.php文件，你便可以上传一个.user.ini，再上传一个图片马，包含起来进行getshell。不过前提是含有.user.ini的文件夹下需要有正常的php文件，否则也不能包含了。 再比如，你只是想隐藏个后门，这个方式是最方便的。
- 创建主页面echo.php和.user.ini和预先执行文件1.gif
![](files.png)
![](filescontent.png)
- 等待300秒(.user.ini)或者重启Apache服务器，访问主页面echo.php测试结果
![](result.png)

### .user.ini漏洞的防御
- 爬取指定文件中的文件，查看是否有.user.ini文件，必要时删除。
```python3
import os
import os.path
rootdir="/var"
for parent,dirnames,filenames in os.walk(rootdir):
    for filename in filenames:
        if filename== ".user.ini":
            print(parent,filename)
```
![](userlist.png)
