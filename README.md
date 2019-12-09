# hexo搭建
[hexo官方文档](https://hexo.io/zh-cn/)<br/>
[nexT主题官方文档](http://theme-next.iissnan.com/getting-started.html) <br/>
[nexT主题个人参考](https://master--janking.netlify.com/post/hexonote.html) <br/>
[基础搭建参考文档 ](https://www.jianshu.com/p/21c94eb7bcd1) <br/>
[live2d（右下角人物）](https://github.com/EYHN/hexo-helper-live2d/blob/HEAD/README.zh-CN.md) <br/>
[markdown官网语法参考](http://www.markdown.cn/)、[Markdown图表语法](https://www.jianshu.com/p/3cf83d22dd3d)



## 1. 基础环境准备
git + node + npm 

## 2. 初步搭建
新建文件blog 然后进入
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

启动本地服务，可以本地预览 
``` bash
$ hexo server 
```

## 3. 上传到GitHub

- 上传blog原文件（hexo产生的文件）到github，方便后期文件、配置等迁移

然后生成静态页面
``` bash
$ hexo generate 
```

- 上传静态页面（在public目录下的文件）到yourgithub.github.io项目下，可参考Github Page搭建

**可以在public下面初始化一个git项目，关联yourgithub.github.io，每次提交改动的文件（blog的原文件可以忽略掉此文件夹）**

在hexo目录，提交改动信息并生成静态页面。
``` bash
git add . #将工作区的所有改动添加到缓冲区
git commit -m "提交的信息" #将缓冲区的内容提交到版本库
git push #推送至远程仓库，此处是我的github
hexo generate
```

切换到public目录，提交刚刚生成的静态页面。
``` bash
git add .
git commit -m "提交的信息"
git push
```

## 4. 进阶
### 分类&标签管理
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

### 更换主题
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

---
## 5. 第三方插件管理
### live2d动画
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
### gitalk评论
参考：https://mrluyc.github.io/2019/07/30/HexoNexT%E9%9B%86%E6%88%90Gitalk/ <br/>
1.github创建一个OAuth Apps获取对应的ID及Secret<br/>
2.修改根目录下_config.yml加入gitalk配置
```
gitalk:
  # 是否启动gitalk
  enable: true
  # 这里填写你的GitHub 的ID，即用户名
  github_id: polarisnosnow
  # 你博客的仓库地址
  repo: polarisnosnow.github.io
  # 填写之前拷贝的client id
  client_id: 
  # 填写之前拷贝的client secret
  client_secret: 
  # 这里填写你的GitHub 的ID，即用户名
  admin_user: polarisnosnow
  distraction_free_mode: true
  # 设置为中文，当然你用外文也可以，配置文件里有
  language: zh-CN    
```


### APlaye.js 音乐播放插件（暂无）
---
## 6. 其他问题
### markdown图片问题

存在到md文件中。首先将图片压缩，再转成base64编码，复制到md文件中，如下

```
1. ![avatar](picPath)
```

```
2. 
![avatar][picID]
你的博客内容等
[picID]:data:image/jpeg;base64.....
```

picPath可以直接用网图地址或图片的base64编码，picID此处是将图片内容放到最后，以便编写主要的博客内容（可参考Apollo配置中心篇）。

