[toc]

# linux常用命令

## jar包启动关闭

~~~java
//启动  打印启动日志到configlog文件。
nohup java -jar cac-ticket-0.0.10-SNAPSHOT.jar >/dev/null 2>&1 &
//关闭
ps -ef | grep 项目名 //查pid
kill -s 9 pid    
    
~~~

## find 查找

~~~shell
find / -name 文件名 //从 '/' 开始进入根文件系统搜索文件和目录 
whereis name  文件在哪    
~~~

## 端口号

~~~java
jsp -l //-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
~~~

## windows文件上传linux

~~~java
rz：运行该命令会弹出一个文件选择窗口，从本地选择文件上传到服务器(receive)，即windows上传到linux服务器

sz+文件名：将选定的文件发送（send）到本地机器，即从linux服务型下载到windows
~~~

## 文件操作

~~~java
=======移动===============================================
mv b.txt c.bak   //将文件 b.txt 重命名为 c.bak
mv -i 456.txt /home/hk/cpdir/copy/abc   // 将456.txt 移动到 /home/hk/cpdir/copy/ 并取名为 abc 若已存在文件 abc则会询问是否覆盖。
mv -f 456.txt /home/hk/cpdir/copy/abc    //将  456.txt 移动到 /home/hk/cpdir/copy/ 并取名为 abc 若已存在文件 abc 覆盖时不会有任何提示。
========删除==============================================
 rm -rf 目录名字 //命令即可
-r 就是向下递归，不管有多少级目录，一并删除
-f 就是直接强行删除，不作任何提示的意思
eg
删除文件夹：rm -rf /var/log/httpd/access
将会删除/var/log/httpd/access目录以及其下所有文件、文件夹

删除文件：rm -f /var/log/httpd/access.log
将会强制删除/var/log/httpd/access.log这个文件   
~~~

## vim命令

~~~java
正常模式：可以使用快捷键命令，或按:输入命令行。
插入模式：可以输入文本，在正常模式下，按i、a、o等都可以进入插入模式。
可视模式：正常模式下按v可以进入可视模式， 在可视模式下，移动光标可以选择文本。按V进入可视行模式， 总是整行整行的选中。ctrl+v进入可视块模式。
替换模式：正常模式下，按R进入。
vim + 文件名 //从文件的末尾开始；
vim -R file: 以只读的方式打开文件，但可以强制保存；
vim -M file: 以只读的方式打开文件，不可以强制保存； 
    //保存修改
:q! -不保存退出    
:w – 保存修改。
:wq – 保存并退出。
:x – 保存并退出。
    //移动
ctrl+f: 下翻一屏。
ctrl+b: 上翻一屏。
ctrl+d: 下翻半屏。
ctrl+u: 上翻半屏。
    //查找
/something: 在后面的文本中查找something。
?something: 在前面的文本中查找something。
/pattern/+number: 将光标停在包含pattern的行后面第number行上。
/pattern/-number: 将光标停在包含pattern的行前面第number行上。
n: 向后查找下一个。
N: 向前查找下一个。
    

~~~

## 软件操作

1. 软件装在哪：whereis mysql

## 系统环境

1. `df`命令作用是列出文件系统的整体磁盘空间使用情况。可以用来查看磁盘已被使用多少空间和还剩余多少空间。

   #df -h  通过它可以产生可读的格式df命令的输出

2. netstat -tunlp|grep 端口号   查询端口号是否被占用