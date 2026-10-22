# man-pages-translation：中文 man 手册翻译工作仓库

本仓库是中文 man 手册的翻译工作区。任何修改之后，请用更新脚本重新生成受影响的文件；
单页工作流见下文“仓库工作流”。

达到里程碑后，成果应导出到
[manpages-zh](https://github.com/man-pages-zh/manpages-zh) 项目。

# 译者指南

进入 `po/` 目录，挑选你感兴趣的 PO 文件，认真翻译后把成果提交回来。

## coreutils 的特殊处理

coreutils 的翻译主要在 [TP 网站](https://translationproject.org/team/zh_CN.html)
上进行。翻译单页 man 手册时，请先确认上游 TP 翻译中已经有对应译文，并保持两者一致。

# 管理员指南

请先安装以下软件：

* po4a

本仓库使用 `po4a` 自动化翻译与生成流程。`./update-all` 会刷新整棵目录树；
日常处理单页时请改用 `./_po4a-worker`（见下文）。

如果要新增待翻译的 man 手册，按现有结构放进 `raw/` 目录即可，脚本会自动处理。

注意：可以考虑对生成的 POT/PO 文件运行
`find templates po -type f -exec sed -i '/^#$/d' {} \;`，删除空的译者注释行。

# 仓库工作流（个人 fork）

本仓库是 [man-pages-zh/man-pages-translation](https://github.com/man-pages-zh/man-pages-translation)
的 fork。日常翻译在 fork 中完成，达到里程碑后再把成果导出到上游。

## 远程与分支

| 名称 | 地址 | 用途 |
| --- | --- | --- |
| `origin` | `https://github.com/ddgmma/man-pages-translation.git` | 个人 fork，日常推送 |
| `upstream` | `https://github.com/man-pages-zh/man-pages-translation.git` | 上游，只用于同步 |

- `master` 保持可发布状态；每组页面单独开功能分支，完成后合入 `master`
  再推送 `origin`，需要上游收录时向上游发 PR。
- 历史分支示例：`add-ss-locate-time`、`coreutils-test-du-cut-tail`。
- 同步上游：

  ```sh
  git fetch upstream
  git switch master
  git merge upstream/master
  ```

  若分支已经推送且不想产生合并提交，可改用 `git rebase upstream/master`。

## 单页翻译流程

1. 从 `TRANSLATION-TODO.md` 挑一项，先确认 `raw/` 下是否已有对应页面。
2. 缺页时，把上游 man page 复制到 `raw/<pkg>/man<N>/<file>`。脚本只发现
   三层深度（`find raw -mindepth 3 -maxdepth 3`），路径不能多也不能少。
3. 生成模板与 PO（不写 `src/`）：

   ```sh
   ./update-po    # 全树
   # 或只处理一页
   ./_po4a-worker zh_CN --no-translations raw/coreutils/man1/nl.1
   ```

   新页面会得到一份全未翻译的 `po/<...>.zh_CN.po`，直接在上面翻译。
4. 只编辑 `po/` 中的 `msgstr`。`msgid`、`#:` 引用和 groff 转义保持原样；
   `templates/` 与 `src/` 都是生成文件，不要手改。
5. 重新生成译文：

   ```sh
   ./_po4a-worker zh_CN "" raw/coreutils/man1/nl.1           # 刷新 POT/PO 并重建 src
   ./_po4a-worker zh_CN --no-update raw/coreutils/man1/nl.1  # 只重建 src
   ```

6. 校验：

   ```sh
   msgfmt --check -o /dev/null po/coreutils/man1/nl.1.zh_CN.po
   LC_ALL=C msgfmt --statistics -o /dev/null po/coreutils/man1/nl.1.zh_CN.po
   ```

7. 提交：`po/`、`templates/`、`src/` 的对应变化放在同一个提交里，
   提交信息用中文，例如：

   ```
   翻译：完成 nl(1) 中文翻译
   ```

## 全树脚本

| 命令 | 作用 |
| --- | --- |
| `./update-all` | POT + PO + `src/` 一次性全量刷新 |
| `./update-po` | 只刷新 POT/PO，不写 `src/`（`./update-pot` 是别名） |
| `./update-release` | 只根据现有 PO 重建 `src/` |
| `PO4A_JOBS=4 ./update-all` | 覆盖默认并行度（默认 `nproc`） |

注意：`./update-all` 是全树操作，在干净检出上也会产生约 50 个仅由
`POT-Creation-Date` 和 po4a 换行引起的无关差异。迭代单页时请用
`_po4a-worker`，不要把这类噪音混进翻译提交。

## 约束与已知问题

- po4a 默认阈值是 **80%**：低于阈值的页面不写 `src/`，其既有 `src/` 会保持
  原样，可能已经过期；`cron(8)`、`grep(1)` 目前没有 `src/` 输出。
- 符号链接形式的 master 页面会被跳过（如 `raw/coreutils/man1/[.1 -> test.1`），
  请翻译链接指向的目标页面。
- `po/` 下并非所有文件都是有效页面：遗留的单体文件
  `po/coreutils/coreutils-9.1-pre1.zh_CN.po` 和 Lokalize 工程文件没有对应的
  `raw/` 页面，修改它们不会更新 `src/`。
- coreutils 的译文必须与上游 Translation Project 的 zh_CN 版本一致；
  需要改措辞时，先更新 TP。
- `test(1)` 的 raw 中含 `.B "[\& ]\&"`，po4a 0.69 会错误生成
  `\fB[\& ]"\fP`，因此暂时保留既有 `src/`，待升级 po4a 后重新验证。

## 发布

达到里程碑后，把 `src/` 的成果导出到
[manpages-zh](https://github.com/man-pages-zh/manpages-zh)。
每完成一项，勾选 `TRANSLATION-TODO.md` 中的条目并更新完成率。

# 当前活跃翻译

以下软件包已具备 `raw/` 页面和 `po/` 译文目录：

* autoconf
* coreutils
* cron
* diffutils
* findutils
* grep
* gzip
* kbd
* man-pages（`manpages`、`manpages-dev`）
* procps
* util-linux
* zstd

每页进度见 `TRANSLATION-TODO.md`。

# 其他项目

* [manpages-zh](https://github.com/man-pages-zh/manpages-zh)：中文 man 手册的存储与主发布仓库。
* [manpages-l10n](https://salsa.debian.org/manpages-l10n-team)：使用类似工具（po4a）的跨语言 man 手册翻译项目。

# 版权与许可

以下许可声明保留英文原文。

```
Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.2
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover
Texts.
```

致贡献者：向本项目提交补丁或 Pull Request，即表示你同意你的贡献按上述许可授权。
