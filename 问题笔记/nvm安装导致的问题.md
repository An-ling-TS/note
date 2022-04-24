因为在安装nvm之前便已经安装了node-v14.16.0，并且安装nvm的时候并没有卸载已有的node，导致安装nvm之后很多脚手架或者一些其它的命令无法识别

![image-20211213105150816](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105159.png)

但这些是我已经安装过了，那多半是环境变量出了问题



翻看环境变量，果然

![image-20211213105318929](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105319.png)

![image-20211213105339518](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105339.png)

SYMLINK ？ SysTemLink 应该是这处

![image-20211213105413075](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105413.png)

进入npm_global文件夹，之前装的所有全局模块都在这

![image-20211213105722558](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105722.png)

将%NVM_SYMLINK%\npm_global添加进环境变量，因为怕出问题，所以新建了一个

![image-20211213105621928](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213105621.png)

测试一下

![image-20211213110022019](https://pic-1255740060.cos.ap-shanghai.myqcloud.com/MarkDown/img/20211213110137.png)