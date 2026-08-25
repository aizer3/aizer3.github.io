# 博客部署说明（Hexo + GitHub Pages + Actions 自动部署）

本博客使用 Hexo 构建，通过 GitHub Actions 在 push 源码后自动构建并发布到 GitHub Pages。

- 源码分支：`source`
- 网站产物分支：`master`（GitHub Pages 自动发布）
- 线上地址：https://aizer3.github.io
- 主题：volantis（作为 npm 依赖安装，构建时自动从 node_modules 加载）

---

## 目录说明

| 路径 | 作用 |
|---|---|
| `hexo-source/` | 博客源码（本仓库内容），在此编辑文章和配置 |
| `hexo-source/source/_posts/` | 文章存放目录，写 `.md` 文件即可 |
| `hexo-source/themes/` | 本地主题目录（当前为空，主题走 npm 依赖加载） |
| `d:\acode\blog\aizer3.github.io\` | ⚠️ 旧版静态产物拷贝，已废弃，不要再手动维护，可删除 |

---

## 首次初始化（只需做一次）

> 前置：本机已装 Node.js（建议 18+）、已配置 GitHub SSH key。

```powershell
# 1. 初始化 git 并推送到 source 分支
cd d:\acode\blog\hexo-source
git init
git add .
git commit -m "init hexo source with github actions deploy"
git branch -M source
git remote add origin git@github.com:aizer3/aizer3.github.io.git
git push -u origin source

# 2. 生成专用 Deploy Key（不要覆盖已有 key）
ssh-keygen -t ed25519 -C "github-actions-deploy" -f $env:USERPROFILE\.ssh\id_ed25519_deploy -N ""

# 3. 复制公钥，到 GitHub 仓库 Settings → Deploy keys → Add deploy key
#    Title: ACTIONS_DEPLOY，勾选 Allow write access
Get-Content $env:USERPROFILE\.ssh\id_ed25519_deploy.pub

# 4. 复制私钥，到 GitHub 仓库 Settings → Secrets → Actions → New repository secret
#    Name: DEPLOY_KEY，内容粘贴下面输出的整段
Get-Content $env:USERPROFILE\.ssh\id_ed25519_deploy

# 5. 开启 GitHub Pages
#    仓库 Settings → Pages → Source 选 "Deploy from a branch"
#    Branch 选 master，目录 / (root)，Save
```

---

## 日常更新（以后每次都这样）

```powershell
cd d:\acode\blog\hexo-source

# 1) 在 source/_posts/ 下新建或编辑 .md 文章
# 2) 本地预览（可选）
npx hexo server        # 打开 http://localhost:4000

# 3) 提交并推送，Actions 自动构建部署
git add .
git commit -m "update posts"
git push origin source
```

push 后去 GitHub 仓库 **Actions** 标签页查看构建进度，跑完约 1–2 分钟网站即更新。

---

## 文章 Front-matter 参数（写在 `.md` 文件最顶部）

文章头部用 `---` 包裹的 YAML 区域，叫 Front-matter。示例：

```markdown
---
title: 操作
date: 2027-12-24 15:46:12
---
文章正文写在这里……
```

### 常用参数

| 参数 | 说明 | 示例 |
|---|---|---|
| `title` | 文章标题（必填） | `title: 操作` |
| `date` | 发布时间（必填，Hexo 自动生成） | `date: 2027-12-24 15:46:12` |
| `updated` | 更新时间 | `updated: 2027-12-25 10:00:00` |
| `tags` | 标签，多个用 YAML 列表 | `tags: [前端, React]` 或换行 `- 前端` |
| `categories` | 分类（注意：分类有层级，顺序即父子关系） | `categories: [技术, 前端]` |
| `permalink` | 自定义该文章 URL | `permalink: my-post` |
| `comments` | 是否开启评论（主题支持时） | `comments: true` |
| `layout` | 布局类型，默认 `post` | `layout: post` |
| `cover` | 封面图（部分主题如 volantis 支持） | `cover: https://xxx.png` |
| `description` | 文章摘要/描述 | `description: 这是一篇关于……` |
| `keywords` | 关键词（SEO） | `keywords: hexo, 教程` |
| `top` / `sticky` | 置顶（volantis 等主题支持） | `top: 1`（数字越大越靠前） |
| `hidden` | 隐藏文章（不出现在列表，仍可访问） | `hidden: true` |
| `password` | 文章密码（主题支持时） | `password: 1234` |
| `toc` | 是否显示目录 | `toc: true` |
| `math` / `mathjax` | 开启数学公式渲染（volantis 支持） | `math: true` |
| `comments` | 关闭评论 | `comments: false` |

### 多标签 / 多分类写法

```markdown
---
title: 操作
date: 2027-12-24 15:46:12
tags:
  - 前端
  - React
categories:
  - 技术
  - 前端
---
```

> 注意：`categories` 是有层级关系的（上面表示 "技术 > 前端"）。`tags` 是平级无层级。

### 参考

官方文档：https://hexo.io/docs/front-matter

---

## 常用命令

| 命令 | 作用 |
|---|---|
| `npx hexo new "文章标题"` | 新建一篇文章到 `source/_posts/` |
| `npx hexo server` | 本地预览（默认 4000 端口） |
| `npx hexo clean` | 清理缓存和 `public/` |
| `npx hexo generate`（或 `npx hexo g`） | 生成静态文件到 `public/` |
| `npm install` | 安装/恢复依赖（含主题） |

> 部署已由 GitHub Actions 接管，本地不需要再执行 `hexo deploy`。

---

## 注意事项

1. `themes/volantis` 目录为空是正常的——主题以 npm 依赖 `hexo-theme-volantis` 形式存在，`npm install` 后 Hexo 会自动从 `node_modules` 加载。
2. 改了 `_config.yml` 或 `_config.volantis.yml` 后，同样 `git push source` 即生效。
3. 若想用传统 `themes/volantis` 结构，可执行：
   `git clone https://github.com/volantis-x/hexo-theme-volantis.git themes/volantis`
4. 线上网站以 GitHub `master` 分支为准，本地 `aizer3.github.io` 旧目录已废弃，请勿手动修改。
