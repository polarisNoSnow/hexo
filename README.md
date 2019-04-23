# hexo搭建
官方文档  https://hexo.io/zh-cn/ <br/>
next官方  http://theme-next.iissnan.com/getting-started.html <br/>
参考文档  https://www.jianshu.com/p/21c94eb7bcd1 <br/>
## 1.基础环境准备
git + node + npm <br/>
<br/>

## 2.初步搭建
新建文件blog 然后cd 进入  <br/>
npm install -g hexo #安装hexo  <br/>
hexo init #初始化（生成hexo的目录结构）  <br/>

hexo new post "文章名称" #写文章（文章在./source/_posts目录下）  <br/>
hexo generate #生成静态页面，在public目录里面  <br/>
hexo server #启动本地服务，可以本地预览  <br/>
<br/>
 
## 3.上传到GitHub
a.上传静态页面  <br/>
b.上传blog原文件（hexo产生的文件，方便配置样式等迁移）<br/>
<br/>

## 4.进阶
#### 下载模板
git clone https://github.com/theme-next/hexo-theme-next themes/next

编辑根目录_config.yml文件，
theme: next
language: zh-CN

切换后，用命令清除下缓存
#hexo clean
重新启动
hexo server

编辑/themes/next/下的_config.yml文件



## 其他插件
APlaye.js 音乐播放插件
