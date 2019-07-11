---
title: HEXO项目中文章图片上传的几种方式
tags: ["markdown中使用图片"]
date: 2019-07-11 10:06:08
permalink:
categories: 其他
description:
image:
---
<p class="description"></p>

#### 写在前面

近期对博客进行了一些小小的改动，包括git提交记录的规范化、分支管理、文章图片上传方式调整等等。其中文章图片上传的方式改动了多次，主要目的在于提高图片分类、替换、管理的效率，以及能够在文章中更方便自由地引用。

#### 文章中使用图片的几种方式

**1、在themes/next/source路径下创建图片文件**

这是最开始使用的图片上传方式，主要利用的是hexo generate生成静态文件时会将source路径下的新建立的图片文件原样打包到根目录下的public文件中，而public文件中的内容实际上就是push到github pages上的静态博客的内容，所以只需在在文章中使用文章到图片文件的相对路径即可引用图片。步骤如下：

<!-- more -->

* 在themes/next/source路径下创建uploads文件夹（文件名任意），添加需要的图片文件

* 在文章中引入图片前先使用hexo generate编译静态文件（这样做的目的是方便根据目录结构得到相对路径，熟悉静态文件结构以后可以跳过这一步）

  > 使用hexo generate生成博客静态文件时，生成的目录结构按照"年\月\日\文章文件名\index.html"形式逐级深入，如：public\2019\07\08\ShouldComponentUpdate\index.html
  >
  > 

* 在文章中使用相对路径引入图片

  > 自定义的图片文件uploads位于public目录下，因此相对路径就为："../../../../../uploads/xxx.png"

这种使用方式类似于webpack项目中将静态文件放入public文件，可以不经过编译。缺点在于本地编辑时markdown文档中无法预览，并且书写相对路径时容易出错。只建议在Hexo项目中只有少量图片时使用。

**2、使用Hexo文章资源文件**

如果想要实现文章的模块化管理，Hexo官方提供了一种组织化的方式。开启相关配置后通过 `hexo new [layout] <title>` 命令创建新文章时会自动创建一个与这个 markdown 文件一样的名字资源文件夹。经过编译以后，资源文件夹中的图片会与index.html处于同一目录下，因此在使用相对路径引入时更加方便。使用步骤如下：

* 配置Hexo静态资源文件选项，设置post_asset_folder为true

  ~~~JavaScript
  // _config.yml
  post_asset_folder: true
  ~~~

* 生成新文章的同时生成文章资源文件夹，在文件夹内添加图片
* 在文章中使用相对路径引入图片（因为与index.html在同一目录下，相对路径更好书写，如：./xxx.png）

对比第一种方式，使用文章资源文件的好处在于文章及其资源实现了就近原则，方便管理的同时也减少了引用时相对路径的书写难度。

**3、使用github作为图床**

图床，是一个专门储存图片的仓库，同时允许为图片仓库地址创建连接作为URL。

这里使用的图床是以raw形式上传图片至github，操作步骤如下：

* 创建图床仓库

* 本地克隆仓库，添加所需图片然后push到仓库中

* 在github仓库中打开图片，复制链接

  > 例如：`https://github.com/tanyioO/image-lib/blob/master/blog/img/xxx.png`

* 将链接中的"blob"改成"raw"后，在markdown文档中使用

除了使用github作为图床，还可以使用自建服务器，七牛云等云储存对象等，各有优缺点。

<hr />
