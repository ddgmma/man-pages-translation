---
name: Man 手册翻译
description: 按 TRANSLATION-TODO.md 清单逐项完成 man-pages-translation 仓库的 zh_CN man 手册翻译、po4a 生成、验证和提交。适用于翻译或补全 PO 文件、新增 raw 页面、更新 src/，以及推进翻译清单。
---

# Man 手册翻译

本 skill 用于本仓库的 zh_CN man page 翻译。目标是一次只完成 `TRANSLATION-TODO.md` 中的一个命令，保持改动范围清晰、可审查。

## 单页执行清单

- [ ] 从 `TRANSLATION-TODO.md` 选择第一个未勾选、优先级最高的命令
- [ ] 确认 `raw/`、`po/`、`templates/`、`src/` 中该页面的当前状态
- [ ] 只编辑 `po/<pkg>/man<N>/<page>.zh_CN.po` 中的 `msgstr`
- [ ] 用 `_po4a-worker` 只生成这一页，避免全树改动
- [ ] 用 `msgfmt --check` 和 `git diff` 验证
- [ ] 更新 `TRANSLATION-TODO.md` 的勾选状态和完成率
- [ ] 用中文提交信息提交

## 开始前

1. 确认当前在仓库根目录；所有脚本都使用相对路径 `raw/`、`po/`、`templates/`、`src/`。
2. 读取 `TRANSLATION-TODO.md`，从第一优先中选一个未完成项。
3. 检查页面是否存在：
   - `raw/<pkg>/man<N>/<page>` 是上游英文源文件。
   - `po/<pkg>/man<N>/<page>.zh_CN.po` 是翻译文件。
   - `templates/` 和 `src/` 是生成文件，不能作为翻译源。

如果 `raw/` 中没有目标页面，先按“新增页面”流程补页。

## 翻译

1. 只修改 PO 中的 `msgstr`；不要改 `msgid`、`#:` 位置引用、`#.` 类型注释或 groff 转义。
2. 保持 PO 头部的 `Language: zh_CN`、UTF-8 和 `Plural-Forms` 不变。
3. 不要手工编辑 `templates/` 或 `src/`；这两个目录由 po4a 生成，`src/` 文件带有“generated with po4a”头。
4. coreutils 页面必须与上游 Translation Project 的 zh_CN 翻译一致；需要改措辞时先更新 TP。
5. 保持术语和已有页面的风格一致；不要顺便重排无关段落。

## 生成与验证

优先只处理当前页面：

```sh
./_po4a-worker zh_CN "" raw/<pkg>/man<N>/<page>
```

只看/更新 POT 和 PO，不生成 `src/`：

```sh
./_po4a-worker zh_CN --no-translations raw/<pkg>/man<N>/<page>
```

只根据现有 PO 重建 `src/`：

```sh
./_po4a-worker zh_CN --no-update raw/<pkg>/man<N>/<page>
```

验证：

```sh
msgfmt --check -o /dev/null po/<pkg>/man<N>/<page>.zh_CN.po
git diff --check
git diff -- po/ templates/ src/
```

注意：

- 只有翻译率达到 po4a 默认 **80%** 的页面才会写入 `src/`。低覆盖率页面没有 `src/` 输出是正常的。
- 已有 `src/` 文件在覆盖率下降后可能保持旧内容，不能仅凭 `src/` 存在判断翻译已完成。
- 全树同步才使用 `./update-all`。它会刷新 `POT-Creation-Date`，并可能因 po4a 换行产生无关 diff；不要把它和一次单页翻译混在同一个提交中。

## 新增页面

1. 将上游 man page 放到严格的 `raw/<pkg>/man<N>/<file>` 路径。脚本只扫描这个深度（`find raw -mindepth 3 -maxdepth 3`）。
2. 运行 `./update-po` 生成 `templates/` 和未翻译的 `po/`。
3. 翻译 PO，再运行 `./_po4a-worker zh_CN "" raw/...` 生成 `src/`。
4. 符号链接页面会被跳过，例如 `raw/coreutils/man1/[.1 -> test.1`；请翻译目标页面 `test.1`。

## 完成一项

1. 更新 `TRANSLATION-TODO.md`：勾选该项，并按当前 `msgfmt` 结果更新完成率。
2. 检查 `git status`，只应包含该页面的 `po/`、`templates/`、`src/`（新增页面时）以及清单改动。
3. 提交信息使用中文，例如：

   ```text
   翻译：完成 grep(1) 中文翻译
   ```

4. 不要把无关的全树生成改动混入翻译提交。
5. 如果用户只要求翻译、没有要求提交，先报告改动和验证结果，等待确认。

## 参考

- 仓库根目录 `AGENTS.md`：完整仓库约定。
- 仓库根目录 `TRANSLATION-TODO.md`：按优先级排列的待翻译清单。
- 官方说明：https://github.com/man-pages-zh/man-pages-translation
- 官方 README 提到的 `find . -type f -exec sed -i '/^#$/d' {} \;` 会遍历整个仓库（包括 `.git`）；不要直接照抄。确需清理空注释时，只限定到 `templates/` 和 `po/`。
