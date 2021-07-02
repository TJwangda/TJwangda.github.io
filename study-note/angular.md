[TOC]

# 常用命令

## 项目创建

~~~shell
npm install -g @angular/cli   //要使用 npm 命令安装 CLI

ng v  验证是否安装成功

ng new 项目名称  创建项目（移动到指定目录先）。可加--skip--install 跳过npm install 用cnpm

创建后选中路由和css等设置，默认npm i 安装依赖，可能失败。可以终止。进入到项目目录用cnpm install 安装，主要是package.json中的包

ng serve --open  运行项目

新建组件到指定文件夹（components/news） cd到项目目录执行： ng g component components/news

ng g service services/servicename   在指定文件夹创建服务

ng g module module/user --routing 在module文件夹下创建user模块，--routing可选，加上会生成模块路由文件。
-----------------------------------------
ionic
cnpm install -g cordova ionic 安装
ionic -v 
cordova -v 版本号
ionic start myappName tabs/blank/sidemenu  先cd到指定目录，创建应用 tabs为参数（一个页面/空白/侧边栏应用）
ionic serve cd到项目目录，运行项目
~~~

