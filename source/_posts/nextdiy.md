---
title: 博客写作攻略--修改css以及增加标签云
date: 2019-08-03T20:07:20.000Z
updated: '2019-08-05 08:48:06'
tags:
  - Blog
categories:
  - Blog
---



{% cq %}

简单记录一下自己的博客搭建,方便复盘

{% endcq %}

<!-- more -->

<br>

# 修改NexT主题颜色

---

本博客使用的是NexT主题的默认子主题,主题默认颜色搭配是黑白,本人比较喜欢蓝绿色,于是乎决定小小修改一下主题,过大的主题颜色修改可能会不太和谐(主要还是太懒了....),所以就仅仅修改了博客名背景,菜单栏,阅读更多按键,侧边栏目录,修改的位置为`\hexo\themes\next\source\css\_variables\base.styl`

**以readmore按键修改为例**

```diff
// Buttons
// --------------------------------------------------
$btn-font-weight                = normal

$btn-default-radius             = 0
+$btn-default-bg                 = #0E6655  // $black-deep
$btn-default-color              = white
$btn-default-font-size          = 14px
$btn-default-border-width       = 2px
+$btn-default-border-color       = #0E6655  // $black-deep
$btn-default-hover-bg           = white
+$btn-default-hover-color        = #0E6655  // $black-deep
+$btn-default-hover-border-color = #0E6655  // $black-deep

```

以上代码块中的**绿色部分**为修改部分(无视+号),`$black-deep`为全局变量,这个可以在配置文件的顶部找到

```diff
// Colors
// colors for use across theme.
// --------------------------------------------------
$whitesmoke   = #f5f5f5
$gainsboro    = #eee
$gray-lighter = #ddd
$grey-light   = #ccc
$grey         = #bbb
$grey-dark    = #999
$grey-dim     = #666
$black-light  = #555
$black-dim    = #333
$black-deep   = #222
$red          = #ff2a2a
+$blue-bright  = #45B39D //#87daff
$blue         = #0684bd
$blue-deep    = #262a30
$orange       = #0E6655
```

如果你想修改所有的想改的指定颜色,可以在这里改,其他的修改部分同此操作.

<br>

# Tags页添加3D标签云

---

参考链接:https://github.com/MikeCoder/hexo-tag-cloud

## 如何安装

- 进入到 hexo 的根目录，然后在 `package.json` 中添加依赖: `"hexo-tag-cloud": "2.1.*"`
- 然后执行 `npm install` 命令
- 然后需要你去修改主题的 tagcloud 的模板，这个依据你的主题而定。

## 修改模板文件

> ps:这里主要以NexT主题默认子主题修改为例,其他的主题修改可以参考插件文档修改

**找到`\hexo\themes\next\layout\page.swig`文件,添加绿色部分(无视+号)**

```diff
                {% endif %}
              {% endfor %}
              {{ _p('counter.tag_cloud', visibleTags) }}
            </div>
			
+			<!-- Tag Cloud Code -->
+			{% if site.tags.length > 1 %}
+			<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcloud.js') }}"></script>
+			<script type="text/javascript" charset="utf-8" src="{{ url_for('/js/tagcanvas.js') }}"></script>
+			<div class="widget-wrap">
+				<!-- <h3 class="widget-title">Tag Cloud</h3> -->
+				<div id="myCanvasContainer" class="widget tagcloud">
+					<canvas width="800" height="800" id="resCanvas" style="width=100%">
+						{{ list_tags() }}
+					</canvas>
+				</div>
+			</div>
+			{% endif %}
+			<!-- Tag Cloud Code -->
			
            <div class="tag-cloud-tags">
              {% if not theme.tagcloud %}
                {{ tagcloud({min_font: 12, ma
```

## 最后一步

- 完成安装和显示，可以通过 `hexo clean && hexo g && hexo s` 来进行本地预览, hexo clean 为必须选项。
- **PS: 不要使用 `hexo g -d 或者 hexo d -g` 这类组合命令。

## 自定义标签云

> 可以改变你的标签云的字体和颜色，还有突出高亮

**在你的博客根目录，找到 *_config.yml* 文件然后添加如下的配置项:**

```yaml
# hexo-tag-cloud
tag_cloud:
    textFont: Trebuchet MS, Helvetica
    textColor: '#333'
    textHeight: 25
    outlineColor: '#E2E1D1'
    maxSpeed: 0.5
```

