# 九九的个人博客

基于 Hexo 的个人技术博客，托管于 GitHub Pages。

**在线访问**：[https://zhenwusi.github.io](https://zhenwusi.github.io)

## 快速开始

```bash
# 安装依赖
npm install

# 本地预览
npx hexo server

# 新建文章
npx hexo new "文章标题"

# 部署到 GitHub Pages
npx hexo clean && npx hexo generate && npx hexo deploy
```

## 写文章

在 `source/_posts/` 下新建 `.md` 文件，格式如下：

```markdown
---
title: 文章标题
date: 2025-02-12
tags:
  - 标签1
  - 标签2
categories:
  - 分类名
cover: 封面图片链接（可选）
excerpt: 文章摘要（可选）
---

正文内容，支持标准 Markdown 语法。
```

## 项目结构

```
├── _config.yml          # Hexo 主配置
├── source/_posts/       # 博客文章（Markdown）
├── source/about/        # 关于页面
├── source/friends/      # 友链页面
├── source/categories/   # 分类页面
├── source/tags/         # 标签页面
├── themes/flavor/       # 自定义主题
│   ├── layout/          # EJS 模板
│   └── source/          # 主题样式和脚本
├── img/                 # 图片资源
└── scaffolds/           # 文章模板
```


