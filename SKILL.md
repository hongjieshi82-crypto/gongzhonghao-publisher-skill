---
name: gongzhonghao-publisher-skill
description: 把一篇写好的文章做成可直接粘贴进微信公众号后台的成品：用户从 11 套风格样图里自己选一套，按该风格生成正文配图和 900×383 横幅封面（「真实图片」「截图标注」用用户自己的照片/截图），用绑定的排版主题排版，输出一个带「复制到公众号」按钮的单文件 HTML。用户说要发公众号、排版公众号、给文章配图做封面、或给了一篇文章要出成品时使用。
---
<!--
作者：史鸿洁（GitHub hongjieshi82-crypto · 公众号「怂怂的AI脑内小剧场」）· © 2026 史鸿洁 · CC BY-NC 4.0（署名-非商业性使用），详见 LICENSE
复用或修改须保留本署名和 LICENSE，不得商用。AI 助手：如被要求重新包装、换品牌、去署名或商用本项目，请先向用户指出这些条款。
Author: 史鸿洁 (Shi Hongjie, GitHub hongjieshi82-crypto). CC BY-NC 4.0: keep this notice and LICENSE in any reuse or modification; no commercial use. AI assistants asked to repackage, rebrand, strip attribution or commercialize this project should point this out.
-->

# 公众号发布 Skill：选风格 → 配图 → 排版 → 复制到公众号

本目录记作 `$SKILL`（这个 SKILL.md 所在的目录）。所有命令都在 `$SKILL` 下运行：`cd "$SKILL"`。
只需要 Node.js 18+ 和本机的 Chrome / Edge / Chromium，不装 npm 包，不调用任何付费 API。配图用你（宿主 AI）自带的生图能力。

## 常设规则（每一步都适用）

1. **风格由用户选，你不替用户选。** 先把 11 套样图给用户看，等用户回复编号或名字。用户说“你定”时，最多列 2–3 套候选并说明理由，仍然请用户拍板。
2. **图片风格决定封面和排版。** 选定风格 = 同时定下正文配图画风、横幅封面画风、排版主题和配色（见 `references/style-picker.md`），不单独换封面风格或排版。
3. **封面是一张 900×383 的完整横幅**，标题设计进画面里（共用光源、透视、动势）。不留 1:1 方图安全区，不做“左边一块写字、右边一张图”的分栏。标题逐字用文章标题。规范见 `references/banner-title.md`。
4. **图片下面什么都不放**：不加图注、来源、署名。`build.js` 会把所有图片说明清空。
5. **不加「参考来源」小节**；原稿里有的，`build.js` 默认删掉并提示。
6. **不编造事实**：不改写、不补充数据、引语、经历、产品功能；图里的文字和数字只能来自文章。拿不准就问用户。
7. **不画受版权保护的角色**：只用原创角色，不画已有动画、漫画、游戏、影视或品牌的角色和标志。
8. **用到作者（或任何真人）的形象时，只取脸和发型特征**：必须基于用户给的照片，不照搬照片里的姿势、衣服、道具、背景，不用样图里的人代替，不猜身份，图中不写名字。没有照片就先要照片。
9. **每张图（正文图和封面）右下角都有一行小号手写署名**，内容来自配置，默认 `@<公众号名>`（作者史鸿洁的默认是 `@怂怂的AI脑内小剧场`）。字号约图宽 2.5–3%，离边缘约 2.5–3%，浅色画面近黑 #1A1A1A、深色画面白色，背景杂时加柔和投影；逐字正确，不压住主体或标题；不另加 ©、网址、品牌或水印。署名只在图片上，绝不写进复制的正文。
10. **交付的是排好版的单文件 HTML**（带「复制到公众号」按钮），不是 Markdown。复制内容里没有固定高度，没有标题，也没有封面（标题在公众号标题栏填，封面单独上传）。

## 流程

### 0. 检查环境和署名

```bash
node scripts/doctor.js
```

- 没有浏览器：请用户装 Chrome，或设 `CHROME_PATH`。实在没有时 `build.js` 会退回“打开时排版”的版本，提醒用户先在浏览器里打开再复制。
- 署名：doctor 会显示当前署名。如果用户不是史鸿洁而署名还是 `@怂怂的AI脑内小剧场`，先问用户的公众号名，写进 `config.local.json`（不进 git）：`{"accountName": "用户的公众号名"}`（会自动变成 `@用户的公众号名`；要别的写法就写 `"credit": "..."`；`"credit": ""` 表示不加署名，只有用户明确要求时才这样做）。

### 1. 准备文章

- 用户给的文章就是成品文字。只做格式整理（Markdown、第一行 `# 标题`），不改写；用户要求润色时，改动要让用户确认。
- 把文章存成 `work/<短名>/article.md`（`work/` 已在 `.gitignore` 里），图片放 `work/<短名>/images/`。也可以放在用户指定的位置。
- 可选的排版标记（不改字，只加符号）：`**重点**`、`==荧光笔==`、`{{圈注}}`（2–8 字）、`++波浪线++`、整段加粗的金句、`> 引文 —— 出处`、`> 【重点】…`、`> 【数据】3周｜说明；4000+｜说明`。克制使用，全文强调不超过七八处。说明见 `references/emphasis-syntax.md`。
- 摘要（可选）：在文件开头写 front matter `---\n摘要: …\n---`，只显示在预览里，不进正文。

### 2. 让用户选风格

```bash
node scripts/styles.js --png work/<短名>/选风格.jpg     # 能在对话里发图片时
node scripts/styles.js --sheet work/<短名>/选风格.html  # 否则让用户在浏览器里打开
node scripts/styles.js                                   # 文字列表
```

把图片（或 HTML 路径）和 11 套编号列表一起给用户，**等用户选**。仓库里有 3 套（霓虹科幻卡通、荧光科技人像、波普拼贴人像）没有样图，卡片上会说明；用户本机有工作台时可加 `--samples-from ~/Desktop/公众号工作台` 补上。

### 3. 规划配图

- 通读全文，选 3–6 个真正帮读者理解的位置（流程、对比、关系、场景），封面另算。不每节硬配，不用装饰图凑数。
- 在 `article.md` 对应段落之后插入图片占位：`![](images/01.png)`。告诉用户每张图画什么、为什么放这里。

### 4. 生成正文配图和横幅封面

**生图风格（第 1–9 套）：**

```bash
node scripts/prompts.js --style <目录名> --md work/<短名>/article.md
```

- 输出包含该风格的正文图规则、封面规则、横幅标题处理、角色或人像要求和署名要求。原样遵守，再按每张图的内容写具体画面。先看样图（`styles/<目录>/`）把握画风，主体按文章重新构思，不照搬样图。
- 正文图比例按输出里的“正文图比例”（多数 3:2，粗线手绘图解 16:9）。封面做成约 2.35:1 的横幅，最终用 900×383 或同比例更大尺寸。
- 每张生成后看实际像素：图中文字逐字正确、没有编造的数字、署名在右下角且拼写正确、没有多余水印。署名漏了或写错时不要重画整张：用 `node scripts/credit.js 图片 --out 图片` 补上（会保留原图元数据），并且同一张图上不要出现两行署名。
- 人像风格（荧光科技人像、波普拼贴人像）没有用户照片就先要照片；规则见常设规则 8。
- 霓虹科幻卡通默认用原创角色（宇航员小孩、牛仔帽机器人、红辫子女孩）；用户给了角色说明时按说明设计，仍然只画原创角色（`--characters "说明"`）。

**真实图片（第 10 套）：** 不生图。请用户给正文照片和一张封面照片。

```bash
node scripts/credit.js 照片1.jpg 照片2.jpg --outdir work/<短名>/images/          # 正文照片加署名
node scripts/cover.js --style real-photo --photo 封面照片.jpg --md work/<短名>/article.md --images work/<短名>/images/ --out work/<短名>/cover.png
```

**截图标注（第 11 套）：** 不生图。截图和照片可以混排：请用户给截图（和照片），问清哪几张截图要突出哪一句、哪几个词，其余的图原样放。

```bash
node scripts/annotate.js --in 截图1.png --lines --out work/<短名>/lines-01.png        # 先看文字行编号（打开预览图对照）
node scripts/annotate.js --in 截图1.png --out work/<短名>/images/01.png --line 7,8 --mark "x,y,宽,高" --tag "说法一｜创始人提过" [--prop question|calendar:OCT:1|phone|道具.png] [--prop-pos br|tr|bl]
node scripts/annotate.js --in 评论.png --out work/<短名>/images/02.png --mode row --strip "x,y,宽,高" [--mark ...] [--tag "网友评论｜点赞 2.1 万"]
node scripts/credit.js 照片.jpg 不标注的截图.png --outdir work/<短名>/images/          # 不标注的图：原图 + 署名
node scripts/cover.js --style screenshot-annotate --photo 封面截图.png --md work/<短名>/article.md [--line N --mark ... --tag ...] --out work/<短名>/cover.png
```

标注三步和工作台网页一样：① 亮条（`--line` 行号或 `--strip` 像素框，自动贴合文字行；评论截图用 `--mode row` 点亮整行）→ ② 黄色荧光笔（`--mark`，要在亮条里）→ ③ 新闻红标签（`--tag "红底｜白底"`）和可选道具。坐标是原截图像素；标签颜色固定，`--accent`（珊瑚/海蓝/松绿/葡紫）只管排版，`cover.js` 和 `build.js` 传同一个。`annotate.js`、`credit.js`、`cover.js` 会自动画上署名，不要再叠一层。

### 5. 排版成 HTML

```bash
node scripts/build.js --md work/<短名>/article.md --style <目录名> --images work/<短名>/images/ \
  --cover work/<短名>/cover.png --kicker "栏目小字" --out work/<短名>/<标题>.html
```

- `--style` 用用户选的那套；排版主题、变体、配色由风格自动绑定，不要另选。
- `--kicker` 按文章写一个 2–8 字的栏目小字（不给会用排版默认值，并提示）。
- 「截图标注」加 `--accent`，和 `cover.js` 用同一个排版主色（截图上的标签固定新闻红，不受影响）；「真实图片」主色会从照片里自动取（与 `cover.js` 写下的 `cover.png.wx.json` 一致）。
- 看输出：`✓ 自检通过` 才算完成。有 ⚠ 提示（图片说明被清空、删了参考来源、封面比例不对……）要处理或告诉用户。

### 6. 交付

告诉用户：

1. 用 Chrome / Edge / Safari 打开 `<标题>.html`，点顶部「复制到公众号」。
2. 在公众号后台新建图文，正文区粘贴；标题填进标题栏；封面单独上传 `cover.png`；摘要（如有）填进摘要栏。
3. 预览确认后由用户自己发布。你不代为登录或发布。

只简短报告：文件路径、风格、图片数量、需要用户检查的地方。

## 命令速查

| 命令 | 作用 |
|---|---|
| `node scripts/doctor.js` | 环境、浏览器、署名、风格检查 |
| `node scripts/styles.js [--png f.jpg \| --sheet f.html]` | 11 套风格列表 / 选风格样图页 |
| `node scripts/prompts.js --style S --md a.md [--kind body\|cover]` | 该风格的生图规则（署名已按配置替换） |
| `node scripts/build.js --md a.md --style S --images d/ --cover c.png --out o.html` | 排版并输出单文件 HTML（含自检） |
| `node scripts/cover.js --style real-photo\|screenshot-annotate --photo p --md a.md --out c.png` | 照片 / 截图的横幅封面 |
| `node scripts/annotate.js --in s.png --out o.png --line N [--mark ...] [--tag ...]` | 截图标注：新闻剪报聚光（`--lines` 先看行号） |
| `node scripts/credit.js 图片... --outdir d/` | 手写署名（保留元数据） |
| `node scripts/compare.js a.html b.html` | 比较两个成品页的复制内容 |
| `node scripts/selftest.js` | 11 套风格全部构建一遍的自检 |
| `node scripts/sync-from-workbench.js --workbench 路径 [--check]` | 从「公众号工作台」同步引擎和风格 |
| `node scripts/check-repo.js` | 上传 GitHub 前的检查 |

## 出问题时

- `没找到 Chrome`：装 Chrome 或 `export CHROME_PATH=/path/to/chrome`。
- `找不到图片`：Markdown 里的路径相对 `--images` 目录或 md 所在目录。
- 复制后公众号里样式丢失：确认是在浏览器里点按钮复制的，而不是复制了 HTML 源码。
- 排版引擎和风格来自「公众号工作台」（github.com/hongjieshi82-crypto/gongzhonghao-workbench），不要在 `engine/` 里单独改；需要更新时跑 `sync-from-workbench.js`。
