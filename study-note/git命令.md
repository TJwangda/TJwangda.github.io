1. git项目远程双仓库

   ~~~shell
   1.1#添加多个远程仓库
   先添加第一个仓库：
   git remote add origin https://gitee.com/fsoooo/test.git
   再添加第二个仓库：
   git remote set-url --add origin https://github.com/fsooo/test.git
   如果还有其他，则可以像添加第二个一样继续添加其他仓库。
   
   然后使用下面命令提交：
   git push origin --all
   
   打开.git/config，可以看到这样的配置：
   [core]
       repositoryformatversion = 0
       filemode = false
       bare = false
       logallrefupdates = true
       symlinks = false
       ignorecase = true
   [remote "origin"]
       url = https://gitee.com/wangslei/test.git
       fetch = +refs/heads/*:refs/remotes/origin/*
       url = https://github.com/fsoooo/test.git
   [branch "master"]
       remote = origin
       merge = refs/heads/master
       
   刚才的命令其实就是添加了这些配置。如果不想用命令行，可以直接编辑该文件，添加对应的url即可。
   
   1.2# 查看远程仓库的情况
   git remote
   origin
   
   git remote
   origin  https://gitee.com/fsoooo/test.git (fetch)
   origin  https://gitee.com/fsoooo/test.git (push)
   
   1.3# 查看远程仓库情况。可以看到 github 远程仓库有两个 push 地址。
   git remote -v
   gitee  https://gitee.com/fsoooo/test.git (fetch)
   gitee  https://gitee.com/fsoooo/test.git (push)
   gitee  https://github.com/fsoooo/test.git (push)
   
   1.4 # 推送远程分支
   
   这种方法的好处是每次只需要 push 一次就行了（账号密码不同的话，可能会输两次账号密码）
   ~~~

2. sd