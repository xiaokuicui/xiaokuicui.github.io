日常工作/学习笔记.欢迎 Star 和 Fork.

访问地址：<http://www.xiaokui.org>

博客主题使用: [Yummy-Jekyll](https://github.com/DONGChuan/Yummy-Jekyll)

# Fork 指南

Fork 之后需要修改以下几处:

- 设置仓库名称为:username.github.com.(username是您的用户名)

- 如果需要绑定自己的域名,那么修改 CNAME 文件的内容;如果不需要绑定自己的域名,那么删掉 CNAME 文件。

- 修改配置：

  - 网站的配置都集中在 `_config.yml` 文件中,将其中与个人信息相关的部分替换成你自己的,比如网站的`url、title、subtitle` 和第三方评论模块的配置等.

  - 评论模块:使用 gitalk,在 `_config.yml` 文件的 Comments 一节里把 gitalk 的信息换成自己的.[Gitalk 安装指南](https://github.com/gitalk/gitalk#install)

  - 支持 Google Analytics,在 `_config.yml` 文件的 Google Analytics 一节里把 `google.analytics_id`换成自己的.

- 删除我的文章与图片。

  - `_posts` 文件夹中是我已发布的博客文章。
  - `__waitupload` 文件夹中是我尚未发布的博客文章。
  - `_wiki` 文件夹中是我已发布的 wiki 页面。
  - `images` 文件夹中是我的文章和页面里使用的图片。

- 修改关于页面: `pages/about.md` 文件内容对应网站的关于页面,里面的内容多为个人相关,将它们替换成你自己的信息.

# 关于排版

笔记内容按照 [简体中文文案排版](https://github.com/mzlogin/chinese-copywriting-guidelines) 进行排版,以保证内容的可读性.

# 致谢

本博客基于 [码志](http://mazhuang.org/) 修改,感谢！
