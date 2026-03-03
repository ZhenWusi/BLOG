# Hexo 主题使用说明（jiujiu / Blog 分支）

## 前置
- Node.js ≥ 16（含 npm）
- 全局 CLI：`npm i -g hexo-cli`
- 依赖安装：在项目根目录执行 `npm i`

推荐额外依赖（可选）：
- 更好的 Markdown/公式渲染：`npm i hexo-renderer-markdown-it-plus --save`
- Git 部署：`npm i hexo-deployer-git --save`

## 常用命令
- 本地预览：`hexo clean && hexo g && hexo s`
- 构建输出：`hexo clean && hexo g`
- 部署（如需）：在 `_config.yml` 配置 deploy 后 `hexo d`

## 公式渲染（MathJax）
主题已启用 MathJax（见 `themes/NINE/_config.yml`）：
```yaml
mathjax:
  enable: true
  per_page: false
  cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```
写公式时请：
- 给下划线转义：`$\pi_\theta$` → `$\pi\_\theta$`
- 或用代码块包裹 LaTeX，避免 Markdown 把 `_` 当作斜体：
  ```latex
  $$\max_{\pi_\theta} \mathbb{E}_{x \sim \mathcal{D}}[r(x)]$$
  ```

若使用 `hexo-renderer-markdown-it-plus`，它对公式与下划线更友好，可减少转义问题。

## 音乐播放器（网易云）
在 `themes/NINE/_config.yml`：
```yaml
music:
  enable: true
  netease_playlist_id: 2993057618  # 替换为你的歌单 ID
  autoplay: true
  play_mode: random  # order | random | single
```

## 目录说明（关键路径）
- 文章源文件：`source/_posts/`
- 主题配置：`themes/NINE/_config.yml`
- 站点配置：`_config.yml`
- 生成输出：`public/`（已在 `.gitignore`，不必提交）

## 开发流程
1. 安装依赖：`npm i`
2. 写作/改主题：编辑 `source/_posts` 或 `themes/NINE`
3. 预览：`hexo clean && hexo g && hexo s`，浏览 `http://localhost:4000`
4. 构建：`hexo g`
5. 部署（可选）：配置 `_config.yml` 的 `deploy` 后 `hexo d`

## 部署到 GitHub Pages（可选）
安装部署插件：`npm i hexo-deployer-git --save`
在 `_config.yml` 配置：
```yaml
deploy:
  type: git
  repo: https://github.com/<your>/<repo>.git
  branch: gh-pages
```
然后执行 `hexo clean && hexo g && hexo d`