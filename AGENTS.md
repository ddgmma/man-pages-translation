# AGENTS.md

本仓库是 zh_CN（简体中文）man 手册翻译工作区。每个页面在四个目录树中按 `<pkg>/man<N>/<page>` 镜像存放：

- `raw/` — 上游英文 man 手册（抽取输入；不要为翻译而修改）
- `templates/` — 生成的 `.pot` 文件（已纳入版本控制）
- `po/` — 人工编辑的 `*.zh_CN.po` 翻译
- `src/` — 生成的已翻译 man 手册（已纳入版本控制）

成果导出到 `man-pages-zh/manpages-zh`。

## 命令

所有脚本都要在仓库根目录运行；它们使用相对路径 `raw/`、`po/`、`templates/` 和 `src/`。需要统一的 `po4a` 命令行接口（已在 0.69 测试；不是已弃用的 `po4a-updatepo`/`po4a-translate`）。没有 CI、测试套件或 linter。

- `./update-all` — 完整流程：刷新 POT、合并 PO、重新生成 `src/`。
- `./update-po` — 只处理 POT/PO；绝不写入 `src/`。（`./update-pot` 是别名。）
- `./update-release` — 仅根据现有 PO 重新生成 `src/`；不改动 POT/PO。
- `PO4A_JOBS=4 ./update-all` — 覆盖默认的 `nproc` 并行度。
- 单页处理：`./_po4a-worker zh_CN "" raw/<pkg>/man<N>/<page>`（内部辅助脚本；流程相同，只处理一页）。在第二个参数传入 `--no-translations` 或 `--no-update`，可分别对应 `update-po` 或 `update-release`。
- 检查 PO 文件：`msgfmt --check -o /dev/null po/<...>.zh_CN.po`。

## 工作流注意事项

- 只编辑 `po/` 中的 `msgstr`。不要手工编辑 `templates/` 或 `src/`；两者都是生成文件（`src/` 文件带有 "generated with po4a" 头）。保持 `msgid`、`#:` 引用和 groff 转义不变。
- 生成的 `templates/` 和 `src/` 已纳入版本控制；它们的变化应与对应的 `po/` 修改放在同一个提交中。
- 新增页面时，将上游 man 手册复制到严格的 `raw/<pkg>/man<N>/<file>` 路径；脚本只发现该深度（`find raw -mindepth 3 -maxdepth 3`）。新 PO 文件会以未翻译状态创建。
- README 要求修改后运行 `./update-all`。但它是全树操作，并不保证无差异：在干净检出上会改动 47 个无关文件（`POT-Creation-Date` 和 po4a 换行）。迭代时优先使用 `_po4a-worker`，不要把这类改动混入翻译提交。
- 只有达到 po4a 默认 80% 翻译阈值的页面才会写入 `src/`。覆盖率更低的页面会被跳过；它们已有的 `src/` 输出会保持原样，可能已过期。`cron` 和 `grep` 当前没有 `src/` 输出。
- 作为符号链接的 master 文件会被跳过（`raw/coreutils/man1/[.1 -> test.1`）；请翻译目标页面。
- 并非 `po/` 下所有文件都是有效页面：`po/coreutils/coreutils-9.1-pre1.zh_CN.po`（遗留的单体 PO 文件）和 Lokalize 工程文件没有对应的 `raw/` 页面，更新脚本会忽略它们。编辑该单体文件不会更新 `src/`。
- coreutils 页面必须与上游 Translation Project 的 zh_CN 翻译一致；如需改变措辞，先更新 TP。

## Git

`origin` 是 fork：`ddgmma/man-pages-translation`；`upstream` 是 `man-pages-zh/man-pages-translation`。
