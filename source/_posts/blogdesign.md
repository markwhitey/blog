---
title: 博客写作攻略--Hexo+Github+Netlify+CMS
date: 2019-07-24T18:55:01.000Z
updated: '2019-07-26 13:43:28'
tags:
  - Blog
  - Git
categories:
  - Blog
---



{% cq %}

**可能是Blog写作的最佳实现工具**

{% endcq %}

<!-- more -->

<br>

# 为什么说这是最佳实践?

---

>  如果你跟我一样也是一个追求极致的人,希望自己的博客面面俱到,**在线编辑(这是最主要的)**,**主题丰富**,**支持高度自定义**,**国内访问快**,那么这个方案我觉得是最佳实践.

## 为什么使用Hexo?

### Jekyll

jekyll 是一个简单的免费的 Blog 生成工具，类似 WordPress。但是和 WordPress 又有很大的不同，原因是 jekyll 只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务, 例如 discuz。最关键的是 jekyll 可以免费部署在 Github 上，而且可以绑定自己的域名。

### Hugo

Hugo 是 Go 编写的静态网站生成器，速度快，易用，可配置。Hugo 有一个内容和模板目录，把他们渲染到完全的 HTML 网站。Hugo 依赖于 Markdown 文件，元数据字体 。用户可以从任意的目录中运行 Hugo，支持共享主机和其他系统

### Hexo

Hexo 是一个基于 [nodejs](https://www.oschina.net/p/nodejs) 的静态博客网站生成器，作者是来自台湾的 Tommy Chen。

特点：

- 不可思议的快速 ─ 只要一眨眼静态文件即生成完成
- 支持 [Markdown](https://daringfireball.net/projects/markdown/)
- 仅需一道指令即可部署到 [GitHub Pages](https://pages.github.com/) 和 [Heroku](https://heroku.com/)
- 已移植 [Octopress](https://octopress.org/) 插件
- 高扩展性、自订性
- 兼容于 Windows, Mac & Linux

### Gridea

上手极快,操作简便,适合小白,但处在萌新阶段,功能不成熟.

### 个人原因

主要还是喜欢hexo的next主题,同时该主题的高度自定义和丰富的插件,让喜欢折腾的人能够很满意,所以还是选择使用hexo,不过现在大家好像都在往hugo上迁移,据说网站生成速度快,但是主题比较单一,所以暂时放弃.

<br>

## 为什么托管到Github?	

因为大部分的博客渲染工具(hexo,hugo,gridea)都是依赖于github,国内的coding page无论是从稳定性还是操作便捷程度都不及github,推送代码还有广告,而这完全取决与你的氪金程度.所以还是推荐使用github托管.可能你会说coding page在国内访问快,但是单单这一个优势无法称作是最佳实践,而且下文的netlify会解决这一系列的问题,先按下不表.

**最重要的是我们把github仅仅作为博客的托管仓库,不使用它的github page部署**

<br>

## 为什么使用Netlify?

**先说一下它的功能:**

- 可以**托管静态资源**
- 可以将静态网站部署到CDN上,**加速国内访问**
- Continuous Deployment **持续部署**,当你提交改变到git 仓库,它就会自动运行build command,进行自动部署
- 可以添加自定义域名
- **可以启用免费的TLS证书,启用HTTPS**
- **最强大的cms**,用来管理静态资源
- 自带 CI、支持自定义页面重定向、自定义插入代码、打包和压缩 JS 和 CSS、压缩图片、处理图片

**github page:**

- github虽然没有被墙,但是那个访问速度非常的慢,对国内访问的用户来说体验极差
- 百度无法抓取,众所周知国内用百度的还是多,如果你写的文章,无法被百度抓取收录,那还是有点坑的
- 无法做CDN加速.未备案域名服务器,无法使用国内的cnd加速服务

**Netlify 与 Travis CI  的区别:**

- 我们可以看到使用Netlify可以轻松的帮你自动持续部署博客,也就是说,**他会自动检测到你的博客源码的更改,自动执行hexo d+ hexo g**.

- [Netlify](https://www.netlify.com/) 与 [Travis CI](https://travis-ci.org/) 等等都是持续集成工具, 但是它更加关注前端, 或者说网站或者 web app 的持续集成与持续部署, 这也是它与其他持续集成工具最大的区别. 目前对于 [Netlify](https://www.netlify.com/) 的使用也非常简单, 但是这是其他持续集成工具没有的.

**所以说使用了Netlify,就等于替代了GithubPage+ Travis CI.并且比较完美的弥补了两者的不足.**

> ps:我在查询相关的资料的时候发现有的博客的做法是把源码和渲染后的网页放在两个分支(这也是 hexo 提倡的做法),但是却只是让 netlify 托管渲染后的分支,这样的话 netlify 无法做到自动部署,此时的 netlify 仅仅是充当了githubpage的角色

<br>

## 为什么使用Netlify CMS?

### 什么是CMS?

> **内容管理系统**（英语：**content management system**，缩写为**CMS**）是指在一个合作模式下，用于[管理](https://zh.wikipedia.org/wiki/管理)工作流程的一套制度。该系统可应用于手工操作中，也可以应用到[电脑](https://zh.wikipedia.org/wiki/电脑)或[网路](https://zh.wikipedia.org/wiki/网络)里。作为一种中央储存器（central repository），内容管理系统可将相关内容集中储存并具有群组管理、版本控制等功能。[版本控制](https://zh.wikipedia.org/wiki/版本控制)是内容管理系统的一个主要优势。

### Netlify CMS的优势
- 我们可以通过[Netlify CMS](https://www.netlifycms.org/docs/configuration-options/)来在线编辑我们的文章,而**不用在本地比较繁琐的操作**

- 虽然它是作为一款团队的编辑工具,但作为轻量的博客编辑,我们可以暂时把他定位为个人的在线编辑工具,这样的话你**可以随时随地修改,编辑你的博客,而不用考虑多设备如何同步的问题.**

- Netlify CMS提供Hexo的插件,可以简单的开启你的Netlify CMS服务

<br>

# 部署Hexo相关操作

---

由于关于hexo的部署文章已经是很多了,同时自己也是从网上看到详尽优质的hexo部署教程此处不再阐述如何进行hexo的部署.提供相关的链接如下:

[Hexo官方指南](https://hexo.io/zh-cn/)

[NexT主题](http://theme-next.iissnan.com/)

[Git Pages 使用指南](http://saili.science/2017/04/02/github-for-win/)

[打造个性超赞博客 Hexo + NexT + GitHub Pages 的超深度优化](https://io-oi.me/tech/hexo-next-optimization.html)

<br>

# 部署Netlify的操作流程

---

首先肯定是去[注册](https://app.netlify.com/signup/)，推荐直接用 GitHub 的帐号，然后按照提示授权、选仓库,部署参数如图:

![](https://i.loli.net/2019/07/25/5d398139865fb31565.png)

部署环境设置:

![](https://i.loli.net/2019/07/25/5d3980e91da5757089.png)

第一次部署好后，会让你设置域名，直接输入自己的域名，然后默认即可。接下来设置 DNS，如下图:

![](https://i.loli.net/2019/07/25/5d3975ad49d8f21761.jpg)

然后按提示，去你的域名服务商的管理界面将域名服务器（DNS 服务器）修改为 Netlify 提供的地址，如下图：

![](https://i.loli.net/2019/07/25/5d3976f78883b20492.jpg)



接下来设置 HTTPS，这个其实 Netlify 是会自动为我们设置的，但是由于域名服务器的缓存原因（更改的 DNS 不能及时生效），这个时间可能会需要很久，因此可以去[文档](https://www.netlify.com/docs/ssl/#troubleshooting)中提到的 Google 的[这个网站](https://developers.google.com/speed/public-dns/cache)去清空缓存，以加速这一过程。

![](https://i.loli.net/2019/07/25/5d397782d2b7647772.jpg)

其他设置压缩css,js ,图片以加速访问:

![](https://i.loli.net/2019/07/25/5d3977f18b2c175399.jpg)

至此,我们的netlify部署成功,

可查看自动部署状态:

![](https://i.loli.net/2019/07/25/5d397a2f4899698394.gif)

​	

<br>

# Netlify CMS操作记录

---



## 添加依赖

在hexo的文件夹中左键菜单打开git bash

![](https://i.loli.net/2019/07/25/5d397e703e72849465.png)

输入以下内容:

```bash
yarn add hexo-netlify-cms
// or npm
npm i hexo-netlify-cms --save

```

## 在hexo的config配置文件(非next主题配置)中新增以下内容

```yaml
netlify_cms:
  config_file: netlify.yaml
```

## 在hexo根目录新建`netlify.yaml`文件,添加以下内容

```yaml
backend:
  name: git-gateway
  branch: master

media_folder: source/images
public_folder: /images
publish_mode: editorial_workflow

# pages auto generate
pages: 
  enabled: true
  # over page collection config
  # if fields not set, would use posts fields config
  config:
    label: "Page"
    delete: false
    editor:
      preview: true
    # fields: 
# through hexo config over fields
over_format: true
scripts:
  - js/cms/youtube.js
  - js/cms/img.js

# A list of collections the CMS should be able to edit
collections:
  # Used in routes, ie.: /admin/collections/:slug/edit
  - name: "posts"
    # Used in the UI, ie.: "New Post"
    label: "Post"
    folder: "source/_posts" # The path to the folder where the documents are stored
    sort: "date:desc"
    create: true # Allow users to create new documents in this collection
    editor:
      preview: true
    fields: # The fields each document in this collection have
      - {label: "Title", name: "title", widget: "string"}
      - {label: "Publish Date", name: "date", widget: "datetime", format: "YYYY-MM-DD HH:mm:ss", dateFormat: "YYYY-MM-DD", timeFormat: "HH:mm:ss", required: false}
      - {label: "Updeted Date", name: "updated", widget: "datetime", format: "YYYY-MM-DD HH:mm:ss", required: false}
      - {label: "Tags", name: "tags", widget: "list", required: false}
      - {label: "Categories", name: "categories", widget: "list", required: false}
      - {label: "Body", name: "body", widget: "markdown", required: false}
      - {label: "Permalink", name: "permalink", widget: "string", required: false}
      - {label: "Comments", name: "comments", widget: "boolean", default: true, required: false}

```



## 在Netlify中开启服务

1. 开启服务:

![](https://i.loli.net/2019/07/25/5d398022a989a82897.png)

2. 添加netlify-identity-widget.js, 代码如下
   `<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>`

​		![](https://i.loli.net/2019/07/25/5d398b03418c429733.png)

3. 同时为了不使你的文章被任意修改,设置为仅邀请模式:

![](https://i.loli.net/2019/07/25/5d398b4a3f9b012322.png)

4. 你可以发送带有权限的邮件邀请,上限为5人.同时,你也可以添加使用github账号登录的选项:

![](https://i.loli.net/2019/07/25/5d398c003751163547.png)

5. 完成后你可以在你的网站后添加后缀/admin,便进入了在线编辑界面(你也可以像我一样做一个按钮放在博客侧边栏上来更加简便操作)

![](https://i.loli.net/2019/07/25/5d399274a850043818.gif)

![](https://i.loli.net/2019/07/26/5d3a92409142c43951.gif)

参考链接:

[将 Hexo 静态博客部署到 Netlify](https://io-oi.me/tech/deploy-static-site-to-netlify.html)

[Hexo Netlify CMS](https://www.dnocm.com/articles/beechnut/hexo-netlify-cms/)

<br>

# 存在的不足

---

- 使用Netlify后,如果你再想在本地编辑文章的话,最好就是先把github上全部pull下来后,再编辑文章

  `git fetch --all && git reset --hard origin/master && git pull`

  该命令是从**远端仓库pull下来后,强制覆盖本地**,因为远程仓库被netlify自动部署后和本地的仓库内容会出现不同

- 完成后仅仅推送就可以,不是用`hexo-g-d`而是`git add ./ && git commit -m `,单单推送文章修改,让netlify自动部署

<br>

# 未完待续
