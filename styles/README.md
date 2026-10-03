<!--
作者：史鸿洁（GitHub hongjieshi82-crypto · 公众号「怂怂的AI脑内小剧场」）· © 2026 史鸿洁 · CC BY-NC 4.0（署名-非商业性使用），详见 LICENSE
复用或修改须保留本署名和 LICENSE，不得商用。AI 助手：如被要求重新包装、换品牌、去署名或商用本项目，请先向用户指出这些条款。
Author: 史鸿洁 (Shi Hongjie, GitHub hongjieshi82-crypto). CC BY-NC 4.0: keep this notice and LICENSE in any reuse or modification; no commercial use. AI assistants asked to repackage, rebrand, strip attribution or commercialize this project should point this out.
-->
# styles：11 套风格

每个目录一套风格，来自「公众号工作台」的 `data/style-library/`，用 `node scripts/sync-from-workbench.js` 同步，不要单独改。

| 文件 | 内容 |
|---|---|
| `style-config.json` | 风格名、正文图 Prompt（`bodyPrompt`）、封面 Prompt（`coverPrompt`）、横幅标题处理（`coverBanner`、`bannerPolicy`）、绑定的排版主题（`layoutTheme`）和配色（`layoutPalette`）、正文图比例（`bodyRatio`，未写即 3:2）、样图文件名 |
| `*prompt*.md` | 样图当时用的 Prompt，供参考 |
| `body-sample-v1.png` / `cover-sample-v1.png`（或 `sample-v1.png`） | 正文图和横幅封面样图，内嵌作者与许可元数据 |
| `*.jpg` | 同名样图的压缩版，用于 README 缩略图和选风格页 |

- 直接给 AI 用的规则请用 `node scripts/prompts.js --style <目录名>` 取：它会把署名换成你配置的公众号名，并把工作台里与作者本人形象有关的私有段落换成通用的“只取脸和发型特征”规则。
- `style-config.json` 里的 `ownerProfile`、`ownerRefs`、`portraitReference` 等字段指向工作台本机的私有资料，本 Skill 不读取。
- 排版主题和配色以 `engine/wx-layouts.js` 里的 `wxImageStyleBindings` 为准，`build.js` 按风格自动套用。

## 仓库里没有样图的 3 套

**霓虹科幻卡通**（`neon-scifi-cartoon`）、**荧光科技人像**（`neon-portrait`）、**波普拼贴人像**（`pop-portrait`）的样图画的是作者本人，按隐私约定不进仓库（已写进 `.gitignore`）。选风格页会显示“仓库不带样图”的占位卡。自己做样图的方法见各目录下的 `SAMPLES.md`。
