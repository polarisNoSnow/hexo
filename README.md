# hexo搭建
[hexo官方文档](https://hexo.io/zh-cn/)<br/>
[nexT主题官方文档](http://theme-next.iissnan.com/getting-started.html) <br/>
[nexT主题个人参考](https://master--janking.netlify.com/post/hexonote.html) <br/>
[基础搭建参考文档 ](https://www.jianshu.com/p/21c94eb7bcd1) <br/>
[live2d（右下角人物）](https://github.com/EYHN/hexo-helper-live2d/blob/HEAD/README.zh-CN.md) <br/>
[markdown语法参考](http://www.markdown.cn/)

## 1.基础环境准备
git + node + npm <br/>

## 2.初步搭建
新建文件blog 然后进入  <br/>
``` bash
$ cd blog
```

安装hexo
``` bash
$ npm install -g hexo 
```

初始化（生成hexo的目录结构） 
``` bash
$ hexo init
```

写文章（文章在./source/_posts目录下）
``` bash
$ hexo new post "文章名称" 
```

生成静态页面，在public目录里面 
``` bash
$ hexo generate 
```

启动本地服务，可以本地预览  <br/>
``` bash
$ hexo server 
```
 
## 3.上传到GitHub
a.上传静态页面  <br/>
b.上传blog原文件（hexo产生的文件，方便配置样式等迁移）<br/>

## 4.进阶
初始化分类
``` bash
$ hexo new page categories
``` 

初始化标签
``` bash
$ hexo new page tags 
```

修改source里面对应功能下的index.md，如
```
---
title: 文章分类
date: 2019-04-26 15:02:54
type: "categories"
---
```
```
---
title: 标签
date: 2019-04-26 15:03:18
type: "tags"
---
```

#### 更换主题
下载next主题
``` bash
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```

编辑根目录_config.yml文件， <br/>
```
theme: next
language: zh-CN
```

切换后，用命令清除下缓存
``` bash
$ hexo clean
```

重新启动
``` bash
$ hexo server
```

可以编辑/themes/next/下的_config.yml文件


## 5.其他插件
#### live2d：
1.安装live2d的包 
``` bash
$ npm install --save hexo-helper-live2d <br/>
```

2.下载动画包
``` bash
$ npm install --save live2d-widget-model-haruto <br/>
```

3.修改根目录下_config.yml加入live2d配置 <br/>
```
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  tagMode: false
  debug: false
  model:
    use: live2d-widget-model-haru/01
  display:
    position: right
    width: 150
    height: 300
  mobile:
    show: true
  react:
    opacity: 1.0 
```

#### APlaye.js 音乐播放插件（暂无）
