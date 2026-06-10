# 豆摇峰 — 不复盘，豆摇峰

三个朋友，每周开一次线上复盘会，聊这周干了什么、学了什么、想明白了什么。

自 2024 年 1 月 1 日起，持续进行中。

## 站点

- **线上地址**：https://douyaofeng.com
- **托管平台**：Cloudflare Pages
- **项目名称**：`douyaofeng`

## 目录结构

```
douyaofeng-site/
├── index.html              # 主站首页（含每期列表）
├── images/                 # 主站用图片（已弃用，历史遗留）
├── skill/                  # 每期页面的生成流程文档（脱敏公开版）+ 模板
│   ├── SKILL.md
│   └── templates/episode.html
├── .github/workflows/      # push 后自动部署到 Cloudflare Pages
└── ep/                     # 每期回顾（滚动叙事页面），每期一个目录
    ├── 04/ … 13/           # EPXX/index.html + EPXX/images/
    └── evelyn/             # 特别篇 · AI 工具启蒙实录
```

## 路由

| 路径 | 内容 |
|------|------|
| `/` | 主站：计数器、每期列表、关于、文章、成员 |
| `/ep/XX/` | 第 XX 期回顾（最新一期见主站列表顶部） |
| `/ep/evelyn/` | 特别篇：AI 工具启蒙实录 |

## 每期页面的生成流程

完整流程见 [skill/SKILL.md](skill/SKILL.md)（脱敏公开版）。概要：

1. **输入**：会议转录文本（.txt）
2. **阅读全文**，提炼核心主题、章节结构、金句
3. **隐私处理**：
   - 三位成员使用公开昵称（白豆、摇摇、李墨玩），转录中的真名/旧 ID 一律替换
   - 做客嘉宾保留名字
   - 其他人名模糊化（如「某企业高管」「一位朋友」）
   - 敏感金额、公司名、IP 名、项目报价等脱敏
   - 发布前对页面跑敏感词 grep 自检
4. **生成配图**：调用文生图 API 生成 5 张 2K 插图，下载到 `ep/XX/images/`
5. **生成 HTML**：复制 `skill/templates/episode.html`，滚动叙事风格填槽
6. **放入 `ep/XX/` 目录**，更新主站 `index.html` 的 episodes 列表
7. **推送即部署**：`git push` 后 GitHub Actions 自动部署到 Cloudflare Pages

## 协作与自动部署

- 仓库 `main` 分支每次 push 会触发 `.github/workflows/deploy.yml`，自动把整个仓库部署到 Cloudflare Pages（约 1–2 分钟生效）
- 协作者可直接 push，或走 Pull Request（merge 进 `main` 后同样自动部署）
- 部署凭证存放在仓库 Secrets（`CLOUDFLARE_API_TOKEN` / `CLOUDFLARE_ACCOUNT_ID`），不在代码中

## 技术栈

- 纯静态 HTML/CSS/JS，无构建工具
- 字体：Noto Serif SC + Noto Sans SC（Google Fonts）
- 图片：即梦 API（doubao-seedream-5-0-260128）生成
- 部署：Cloudflare Pages
- 仓库：GitHub

## 主站功能

- **打字机标题**：先打出「不复盘，都要疯？」，再退格变为「不复盘，豆摇峰」
- **实时计数器**：显示从 2024.01.01 至今的天数、周数、年数
- **每期回顾列表**：最新的排最前
- **三篇文章**：三位成员各自写的复盘感悟，支持展开/收起全文
- **做客邀请**：微信号 cupaibaidou

## 新增一期的步骤

1. 在 `ep/` 下创建新目录（如 `ep/14/`）
2. 将生成的 `index.html` 和 `images/` 放入
3. 编辑根目录 `index.html`，在 episodes 列表顶部添加新条目
4. `git push`（GitHub Actions 自动部署）

## 成员

- **白豆** — 公众号「醋泡白豆」
- **摇摇** — 公众号「自供能前站」
- **李墨玩** — 公众号「李墨玩」
