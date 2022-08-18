---
title: 使用Hexo搭建静态blog的心得体会
date: 2020-10-07 17:57:27
categories: Hexo
tags: Hexo
---
## 实现方式
Hexo + git pages + Godaddy域名映射配置

## Hexo构建
可以follow这篇知乎专栏[文章](https://zhuanlan.zhihu.com/p/60578464)。已经很详细了

## Git pages 托管
在上面教程里面有步骤，需要注意的是，在github.io这个repository里面的setting配置，将个人的域名填写在输入框，并且打开enable HTTPS。

另外在自己的托管项目source目录下面，新建一个CNAME配置文件，并将自己的完整网站URL填写进去

## Godaddy域名配置
接上面，创建了CNAME配置前，在godaddy应该已经购买了个人站点域名了。Domain配置可以参考这篇[教程](https://medium.com/@supriyakankure/how-to-add-a-custom-domain-to-your-github-page-with-godaddy-84495781143e)。配置一个个性化的domain template，然后在domain setting使用这个模板。

配置完等上几分钟后，访问域名就可以看到页面了。如果还是没有，可以使用dig命令测试下：
``` bash
dig www.yoursite.com
```

## 博客主题选择
本主题是选用3-hexo的主题。具体使用方式不再赘述了。更多主题可以看[这里](https://hexo.io/themes/)

## 常用命令
hexo new "name"       # 新建文章
hexo new page "name"  # 新建页面
hexo g                # 生成页面
hexo d                # 部署
hexo g -d             # 生成页面并部署
hexo s                # 本地预览
hexo clean            # 清除缓存和已生成的静态文件
hexo help             # 帮助

## FAQ
1. CNAME被重置问题。
在gitpages的settings里，如果CNAME在每次deploy时候被重置，需要在模板工程的/source文件夹里面，新建文件CNAME，将你的域名www.example.com 写入后保存，即可规避这个问题。
2. themes问题
在hexo中，themes是可以替换为自定义主题的。本blog采用3-hexo的模板，并且folk出一个分支进行维护。所以在themes里面，可以引入目标folk模板工程，并且在_config.yml中将配置项改为本地的theme，这样就能完成替换了。
```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: 3-hexo // 在这里替换
```
## 总结
技术上并没有复杂的东西，就是常规的配置与托管。不过能有个码字的地方，构建自己的知识管理，还是挺开心的一件事；也算是了却了多年来的一个心愿吧！