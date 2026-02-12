# ZhenWusi's Blog

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

### 嵌入视频

```markdown
<iframe src="//player.bilibili.com/player.html?bvid=BV1xx411c7mD" width="100%" height="400" frameborder="0" allowfullscreen></iframe>
```

### 嵌入音乐

```markdown
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="330" height="86" src="//music.163.com/outchain/player?type=2&id=歌曲ID&auto=0&height=66"></iframe>
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

## 主题配置

编辑 `themes/flavor/_config.yml` 可自定义：

- **导航菜单**、**个人信息**、**社交链接**
- **封面图片**、**默认文章封面**
- **评论系统**（Utterances，基于 GitHub Issues）
- **音乐播放器**（网易云音乐歌单）
- **友情链接**
- **深色模式**

## License

MIT
