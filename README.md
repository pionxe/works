# 夜航 · works

一个用 Hugo + PaperMod 搭的个人博客。线上：<https://works.yuisui.workers.dev/>

这个仓库既是博客源码，也是作品仓：文章放在 `content/posts/`，改一个 Markdown、推一次代码，线上就更新。

## 怎么加一篇新笔记

在 `content/posts/` 下新建一个 Markdown 文件，文件名沿用编号习惯（`01-`、`02-`…），开头带一段基本信息：

```markdown
---
title: "函数"
date: 2026-08-15
tags: ["Bash"]
description: "首页列表里显示的一句话摘要。"
---

（正文从这里开始写）
```

字段说明：

| 字段 | 作用 |
| --- | --- |
| `title` | 文章标题，显示在首页列表和文章页顶部 |
| `date` | 日期，首页按它排序，**新的在上面**（格式 `YYYY-MM-DD`） |
| `tags` | 标签，自动生成分类页 |
| `description` | 首页列表里的灰色摘要，写一句让人知道这篇讲什么 |

写好后想先在本地看看效果：

```bash
hugo server -D
```

打开 <http://localhost:1313>，边写边自动刷新。`-D` 表示连草稿一起显示——在开头标了 `draft: true` 的文章本地能看、线上不发布，适合没写完的半成品。看完按 `Ctrl+C` 停。

## 发布

```bash
git add -A
git commit -m "加了一篇 Bash 函数笔记"
git push origin main
```

推送后 Cloudflare 自动重新构建，约半分钟到一分钟线上更新，不需要手动部署。

## 其他日常操作

| 想做什么 | 怎么做 |
| --- | --- |
| 改旧笔记 | 直接编辑对应 `.md` 文件，然后提交推送 |
| 删一篇 | 删除文件，提交推送 |
| 改整体样式 | `assets/css/extended/custom.css` |
| 背景图 / 玻璃拟态 / 星空动效 | `layouts/_partials/extend_head.html` 和 `static/images`、`static/js` |
| 快速新建空文章 | `hugo new content/posts/04-xxx.md`（模板生成，默认草稿） |
| 看线上构建日志 | Cloudflare 控制台 → Workers & Pages → 本项目 → Deployments |

## 新机器上手

```bash
git clone https://github.com/pionxe/works.git
cd works
git submodule update --init --recursive   # 拉主题
hugo server -D                             # 预览（需先装好 Hugo）
```

Hugo 需要 0.146 及以上（Cloudflare 构建侧固定了 `HUGO_VERSION` 环境变量）。

## 目录结构

```
content/posts/    博客文章（Markdown，即笔记）
assets/css/       主题自定义样式
layouts/          页面自定义（背景、动效）
static/           图片、脚本等静态资源
themes/PaperMod   主题（git submodule，勿直接改）
```

## 技术栈

Hugo + PaperMod（纯静态，无数据库），Cloudflare Pages 部署，域名 `works.yuisui.workers.dev`。
