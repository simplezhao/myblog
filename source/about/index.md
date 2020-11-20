---
title: about
date: 2020-01-05 09:50:59
---
## Hexo
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页
## 安装
具体参考官网（[https://hexo.io/zh-cn/docs/][1]）
### 安装前提
准备好如下工具
* Node.js （建议10.0以上版本）
* Git
### 安装Hexo
```bash
npm install -g hexo-cli
```
### 主题配置
下面介绍本站所使用的的主题以及相关修改
#### hexo
##### 固定文章连接
```yaml
#/_config.yml
# npm install hexo-abbrlink --save
# 依赖 https://github.com/Rozbo/hexo-abbrlink
permalink: posts/:abbrlink/
# abbrlink config
abbrlink:
  alg: crc16  #support crc16(default) and crc32
  rep: hex    #support dec(default) and hex
```
##### 自动摘录
```yaml
# npm install hexo-excerpt --save
# 依赖 https://github.com/chekun/hexo-excerpt
# excerpt
excerpt:
  depth: 5
  excerpt_excludes: []
  more_excludes: []
  hideWholePostExcerpts: true
```
#### [hexo-theme-next][2]
##### 增加自定义style
```yaml
# /next/_config.yml
custom_file_path:
  style: source/_data/styles.styl
```
##### styles.styl
```css
//styles.styl
// 添加背景图片
body {
      background: #3d3d3d;  //自己喜欢的图片地址
      // background-size: cover;
      // background-repeat: no-repeat;
      // background-attachment: fixed;
      // background-position: 50% 50%;
}

// 修改主体透明度
.main-inner {
      //background: #fff;
      opacity: 0.9;
}

// 修改菜单栏透明度
.header-inner {
      opacity: 0.9;
}
```
##### footer增加since
```yaml
footer:
  # Specify the date when the site was setup. If not defined, current year will be used.
  since: 2015
```
##### Schemes 选择Gemini
```yaml
# Schemes
# scheme: Muse
# scheme: Mist
# scheme: Pisces
scheme: Gemini
```
##### 修改menu
```yaml
menu:
  home: / || home
  archives: /archives/ || archive
  tags: /tags/ || tags
  categories: /categories/ || th
  读书: /categories/book || coffee
  about: /about/ || user
```
##### 提示badges数量
```yaml
menu_settings:
  icons: true
  badges: true
```
##### 修改头像，并设置圆形
```yaml
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: https://oss.smart-lifestyle.cn/blog/2019-11-10-smart-lifestyle-logo.jpeg #/images/avatar.gif
  # If true, the avatar will be dispalyed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```
##### 修改社交连接
```yaml
social:
  GitHub: https://github.com/simplezhao|| github
  E-Mail: mailto:zhaogang@smart-lifestyle.cn || envelope
  #Weibo: https://weibo.com/yourname || weibo
  #Google: https://plus.google.com/yourname || google
  #Twitter: https://twitter.com/yourname || twitter
  #FB Page: https://www.facebook.com/yourname || facebook
  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
  #YouTube: https://youtube.com/yourname || youtube
  #Instagram: https://instagram.com/yourname || instagram
  #Skype: skype:yourname?call|chat || skype
  RSS: /atom.xml || rss
```
##### 字数、阅读时间统计
```yaml
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
symbols_count_time:
  separated_meta: true
  item_text_post: true
  item_text_total: false
  total_time: true
  awl: 2
  wpm: 275
```
##### 文字底部标签显示图标、不用‘#’
```yaml
# Use icon instead of the symbol # to indicate the tag at the bottom of the post
tag_icon: true
```
##### 顶部阅读进度条
```yaml
# Reading progress bar
reading_progress:
  enable: true
  # Available values: top | bottom
  position: top
  color: "#37c6c0"
  height: 3px
```
##### 博客访问量统计
```yaml
# Show Views / Visitors of the website / page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```
##### 本地搜索
```yaml
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false

```
##### 文章加载进度条
```yaml
# Dependencies: https://github.com/theme-next/theme-next-pace
# For more information: https://github.com/HubSpot/pace
pace:
  enable: true
  # Themes list:
  # big-counter | bounce | barber-shop | center-atom | center-circle | center-radar | center-simple
  # corner-indicator | fill-left | flat-top | flash | loading-bar | mac-osx | material | minimal
  theme: minimal
```
##### 彩虹背景
```yaml
# Dependencies: https://github.com/theme-next/theme-next-canvas-ribbon
# For more information: https://github.com/zproo/canvas-ribbon
canvas_ribbon:
  enable: true
  size: 300 # The width of the ribbon
  alpha: 0.6 # The transparency of the ribbon
  zIndex: -1 # The display level of the ribbon
```

[1]:	https://hexo.io/zh-cn/docs/
[2]:	https://github.com/theme-next/hexo-theme-next