# 豆摇峰周复盘 · 转录 → 滚动叙事网页（公开脱敏版）

> 这是豆摇峰每期回顾页面的生成流程文档（脱敏版）。
> 输入一份会议转录 `.txt`，输出一个滚动叙事网页并自动部署上线。
> 适合给协作者了解流程，或给其他 AI Agent 作为操作手册。

## 流程总览

```
.txt 转录
  │
  ▼
[读] 提取日期、嘉宾、8–12 个章节、金句
  │
  ▼
[脱敏] 真名/公司/IP/金额 → 跑 grep 自检
  │
  ▼
[图] 文生图 API × 5 并行 → ep/XX/images/01–05.jpg
  │
  ▼
[HTML] 复制 skill/templates/episode.html → ep/XX/index.html，按本期内容填槽
  │
  ▼
[列表] 主站 index.html 顶部插入 <a class="ep fi" …>
  │
  ▼
[验证] grep 脱敏 + 图片路径一一对应
  │
  ▼
[部署] git push → GitHub Actions 自动部署到 Cloudflare Pages
  │
  ▼
[确认] 线上 grep 本期 EPXX → 回报地址
```

## 第 1 步 · 读取转录

读 `.txt`（通常 5000–20000 字，可分段读）。识别：

- 当期日期（文件名前缀 `YYYYMMDD`）
- 是否有做客嘉宾
- 8–12 个核心章节主题
- 每章的「金句」（适合做引用块）
- 语音转写常见错别字（人名、专有名词按上下文修正）

## 第 2 步 · 隐私脱敏（最重要）

**保留**：三位成员的公开昵称（白豆 / 摇摇 / 李墨玩）、做客嘉宾名、已是公开知识的公众人物。

**必须替换**：

| 类别 | 处理 |
|------|------|
| 成员真名 / 旧 ID（转写常见错字也算） | → 对应公开昵称 |
| 节目名的错写变体 | → `豆摇峰` |
| 其他真人名字 | → `一位朋友` / `某老师` / `T 同学` |
| 客户 / 合作方公司名 | → `某大厂` / `某合作方` / `某律所客户` |
| 自家 IP / 产品名 | → `公司 IP 形象` |
| 地名 + 品牌组合 | → `一场线下活动` / `某地区品牌活动` |
| 具体金额（报价、提成、流水） | 删除或写成「已签合同推进中」 |
| 具体用户量、收入数字 | 删除或写成「超预期」 |
| 个人履历细节（学校、学历组合） | 删除 |
| 灰色 / 争议性商业操作细节 | 整段删除，不保留暗示 |

**脱敏自检**（HTML 写完后必跑）：

```bash
# 维护一份内部敏感词表（不入库），对生成的页面跑：
grep -nE "<真名1>|<旧ID>|<公司名>|<IP名>|<地点品牌>" ep/XX/index.html
# 输出必须为空
```

## 第 3 步 · 配图（5 张并行）

用任意文生图 API（本项目用即梦 Seedream，模型名见 `~/.env` 配置）生成 5 张 2K 横图：

- prompt 写**意象**：「夜色咖啡馆透窗光晕」「巫师独自走过山路，云缝透光」
- 禁止出现真人面孔、品牌、文字
- 关闭水印
- 返回的图片 URL 是临时的，**立即下载**到 `ep/XX/images/01.jpg … 05.jpg`
- 配图与章节主题呼应，不必硬塞每章一张

```bash
mkdir -p ep/XX/images
# 每张：调 API → 取返回 url → curl -o ep/XX/images/0N.jpg "<url>"
```

## 第 4 步 · 生成 HTML

复制 `skill/templates/episode.html` 到 `ep/XX/index.html`，按本期改：

1. `<title>` → `豆摇峰 · 第 N 期 — 标题`
2. `.hero .episode` → `豆摇峰 · 第 N 期`
3. `.hero h1` → 本期标题
4. `.hero .subtitle` → 一句话副标题
5. 参与人员（三人 + 嘉宾）
6. 章节区块（`.section`）：标题 + 正文 + `.quote-block` 金句 + `<img>`
7. `.insight-grid` 启示卡片（10–12 张）
8. Footer 期数和日期

**约定**：

- 模板的 CSS / 字体 / 滚动动画脚本不要动
- 图片用相对路径 `images/01.jpg`
- 引语块只留真正有含义的金句；过场对话放进正文或 `.dialogue`

## 第 5 步 · 加入主站列表

编辑根目录 `index.html`，episodes 列表**顶部**（最新在前）插入：

```html
<a class="ep fi" href="/ep/XX/">
  <span class="ep-n">EPXX</span>
  <span class="ep-t">本期标题</span>
  <span class="ep-d">MM.DD</span>
</a>
```

## 第 6 步 · 验证（push 前必做）

```bash
# 1) 脱敏自检（用内部敏感词表）
grep -rnE "<敏感词表>" ep/XX/ index.html   # 必须为空

# 2) 图片路径与文件一一对应
diff <(grep -oE 'images/[^"]+' ep/XX/index.html | sort -u) \
     <(ls ep/XX/images/ | sed 's|^|images/|')
```

## 第 7 步 · 部署（自动）

```bash
git add ep/XX/ index.html
git commit -m "Add EPXX: <英文或拼音标题>"   # commit message 用 ASCII
git push origin main
```

push 到 `main` 后，GitHub Actions（`.github/workflows/deploy.yml`）会自动把整个仓库部署到 Cloudflare Pages，约 1–2 分钟生效。

```bash
gh run watch   # 可选：盯着部署跑完
```

> 后备方案：Actions 不可用时，本地 `npx wrangler pages deploy . --project-name=douyaofeng --branch=main --commit-message="Add EPXX"`（需要 `CLOUDFLARE_API_TOKEN` 环境变量）。

## 第 8 步 · 线上确认（必做）

```bash
# 主页应包含新一期
curl -sL https://douyaofeng.com/ | grep -oE "EP[0-9]+" | sort -u

# 本期页面应有本期标题
curl -sL https://douyaofeng.com/ep/XX/ | grep -c "本期标题关键词"   # > 0

# 5 张图可访问
for n in 01 02 03 04 05; do
  curl -sI "https://douyaofeng.com/ep/XX/images/$n.jpg" | head -1
done
```

两条都过 → 回报两个地址：

- 主站：`https://douyaofeng.com/`
- 本期：`https://douyaofeng.com/ep/XX/`

## 写作风格备忘

- 滚动叙事：hero 大标题 → 章节渐入 → 启示卡片 → Epilogue 收束
- 每期找一条「暗线」把所有章节串起来，在 Epilogue 点题
- 金句用 `.quote-block`，三人对话用 `.dialogue`，步骤用 `.steps`，并列概念用 `.platform-grid`
- 启示卡片标题要能独立成立（隔一年再看也懂）
- 上一期的 flag / 话头，本期要回收呼应
