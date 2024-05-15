---
title: Hello World!
published: 2024-05-15
description: '个人博客的建立：基于Fuwari模板和Vercel部署服务'
image: './cover.png'
tags: ["Blogging"]
category: 'Guides'
draft: false 
---

## 崭新的开始
长期以来，我一直错误地认为想要建立网站需要自己处理服务器运维之类的细节，因而对迈出第一步望而生畏。但今天的实操让我发现，只要**选择合适的模板与网站部署服务**，这一切都可以变得非常简单。作为一个崭新的开始，我的第一篇博客自然是记录我基于Fuwari模板和Vercel部署本站的过程。

:::note
前置技能：Github（GUI就行），Markdown，Terminal
:::

## 选择博客框架与模板
选择博客框架首先需要明白自己的需求。我的需求主要是：丰富的可选模板、足够主流（有较多社区资源）、支持Markdown、支持Jupyter Notebook（我的主力工作流平台）。[Hexo](https://hexo.io/zh-cn/)显然符合前三个要求，也可以通过下载插件实现第四个要求，因此我选择Hexo搭建个人博客。

通过浏览Hexo的[模板库](https://hexo.io/themes/)，我发现了[Fuwari](https://github.com/saicaca/fuwari)这个很符合我审美的模板。

## 使用模板与部署
在[Fuwari](https://github.com/saicaca/fuwari)的主页点击**Use this tamplate**，基于这一模板创建仓库。Clone到本地，根据模板教程安装pnpm、模板依赖，就可以开始修改模板属性以符合自己需求，并添加新post了。要查看自己的更改，需要在Terminal中运行`pnpm build`来重新构造网页，并通过`pnpm preview`在本地预览。

接下来是我踩的坑：我最开始一直想着用Github Pages来部署网站，结果在build阶段一直报错。查看报错信息也无法独立解决。尝试在Settings-Pages中使用Astro的Github Action来部署，也不能成功。最后我发现Fuwari模板的部署是基于Vercel的，将Vercel和Github绑定、导入自己的repo，立刻就部署成功了，效果如头图。

## 未来计划

我计划在自己的博客中展示：

1. 我参与完成的项目
    - 数据科学相关
    - 运筹论相关
    - 其他
2. 研究生阶段的学习收获
3. 记录日常生活