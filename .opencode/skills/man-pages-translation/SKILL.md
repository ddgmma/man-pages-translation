---
name: Man 手册翻译
description: 按 TRANSLATION-TODO.md 清单逐项完成 man-pages-translation 仓库的 zh_CN man 手册翻译，包括功能分支、po4a 生成、验证、提交和上游 PR。适用于翻译或补全 PO 文件、新增 raw 页面、更新 src/，以及推进翻译清单。
---

# Man 手册翻译

本 skill 用于本仓库的 zh_CN man page 翻译。目标是一次只完成 `TRANSLATION-TODO.md` 中的一个命令，保持改动范围清晰、可审查。

## 单页执行清单

- [ ] 先读取 `TRANSLATION-TODO.md` 选定页面
- [ ] 确认当前不在 `master`，从 `upstream/master` 创建功能分支
- [ ] 确认 `raw/`、`po/`、`templates/`、`src/` 中该页面的当前状态
- [ ] 只编辑 `po/<pkg>/man<N>/<page>.zh_CN.po` 中的 `msgstr`
- [ ] 用 `_po4a-worker` 只生成这一页，避免全树改动
- [ ] 用 `msgfmt --check`、`git diff --check` 和 `groff` 验证
- [ ] 提交翻译分支，按需要推送并向上游发 PR
- [ ] 单独更新 `TRANSLATION-TODO.md`（不要混入翻译分支或上游 PR）

## 开始前

1. 确认当前在仓库根目录；所有脚本都使用相对路径 `raw/`、`po/`、`templates/`、`src/`。
2. 读取 `TRANSLATION-TODO.md`，从第一优先中选一个未完成项。
   如果当前不在 `master`，可用 `git show master:TRANSLATION-TODO.md`。
3. 不要在 `master` 上修改。先从 `upstream/master` 创建功能分支：

   ```sh
   git fetch upstream
   git switch -c <page>-translation upstream/master
   ```

   翻译分支只放本页面的 `raw/`、`po/`、`templates/`、`src/`；
   不要修改 `TRANSLATION-TODO.md`、`AGENTS.md` 或 `.opencode/`，上游没有这些文件。
4. 检查页面是否存在：
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
LC_ALL=C msgfmt --statistics -o /dev/null po/<pkg>/man<N>/<page>.zh_CN.po
git diff --check
groff -Kutf8 -man -Tutf8 src/<pkg>/man<N>/<page> >/dev/null
git diff -- po/ templates/ src/
```

注意：

- 只有翻译率达到 po4a 默认 **80%** 的页面才会写入 `src/`。低覆盖率页面没有 `src/` 输出是正常的；`cron(8)`、`grep(1)` 目前就没有 `src/`。
- 已有 `src/` 文件在覆盖率下降后可能保持旧内容，不能仅凭 `src/` 存在判断翻译已完成。
- 全树同步才使用 `./update-all`。它在干净检出上也会产生约 47 个无关 diff（`POT-Creation-Date` 和 po4a 换行）；不要把它和一次单页翻译混在同一个提交中。

## 已知问题

- `test(1)` 的 raw 中含 `.B "[\& ]\&"`，po4a 0.69 会错误生成 `\fB[\& ]"\fP`。不要重新生成它的 `src/`，保留仓库中已有的正确版本。
- `grep(1)` 的 raw 是 GNU grep 2.27，当前上游为 3.11；翻译前确认是否需要先同步新版页面。
- 符号链接形式的 master 页面会被跳过（如 `raw/coreutils/man1/[.1 -> test.1`），请翻译链接指向的目标页面。
- `po/` 下并非所有文件都是有效页面：遗留的单体文件 `po/coreutils/coreutils-9.1-pre1.zh_CN.po` 和 Lokalize 工程文件没有对应的 `raw/` 页面，修改它们不会更新 `src/`。

## 新增页面

1. 将上游 man page 放到严格的 `raw/<pkg>/man<N>/<file>` 路径。脚本只扫描这个深度（`find raw -mindepth 3 -maxdepth 3`）。
2. 只处理这一页，生成 `templates/` 和未翻译的 `po/`：

   ```sh
   ./_po4a-worker zh_CN --no-translations raw/<pkg>/man<N>/<file>
   ```

3. 翻译 PO，再生成 `src/`：

   ```sh
   ./_po4a-worker zh_CN --no-update raw/<pkg>/man<N>/<file>
   ```

4. 新增页面时，`raw/`、`templates/`、`po/`、`src/` 的对应文件要一起提交。

## 提交与 PR

1. 在翻译分支上提交 `po/`、`templates/`、`src/`（新增页面时再加 `raw/`）。提交信息用中文，例如：

   ```text
   翻译：完成 grep(1) 中文翻译
   ```

2. 推送功能分支：

   ```sh
   git push -u origin <page>-translation
   ```

3. 需要上游收录时创建 PR：

   ```sh
   gh pr create -R man-pages-zh/man-pages-translation \
     --base master --head <fork-owner>:<page>-translation
   ```

   确认 PR 只包含本页面的 `raw/`、`po/`、`templates/`、`src/`；不要把 `TRANSLATION-TODO.md` 或 `AGENTS.md` 带进去。

4. 清单更新单独处理：不要在翻译分支上修改 `TRANSLATION-TODO.md`。
   如果用户允许更新 fork 的 `master`，切回 `master` 单独提交：

   ```sh
   git switch master
   # 勾选并更新 TRANSLATION-TODO.md
   git commit -m "文档：更新翻译清单"
   ```

   否则先询问用户应在哪个分支更新；清单提交不要混入翻译 PR。

5. 如果用户只要求翻译、没有要求提交，先报告改动和验证结果，等待确认。

## 参考

- 仓库根目录 `AGENTS.md`：完整仓库约定。
- 仓库根目录 `TRANSLATION-TODO.md`：按优先级排列的待翻译清单。
- 官方说明：https://github.com/man-pages-zh/man-pages-translation
- README 中清理空注释的命令应限定到 `templates/` 和 `po/` 下的 `*.po`/`*.pot`，不要使用遍历整个仓库（包括 `.git`）的 `find .` 版本；执行后重新运行 `msgfmt --check`。
