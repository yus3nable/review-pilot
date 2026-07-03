# Git Hook 集成

review-pilot 可以安装本地 Git hook，把本地 review pipeline 接到提交和推送动作里。

## 安装

```bash
review-pilot hooks install --pre-commit --pre-push
```

这条命令会写入：

```bash
.git/hooks/pre-commit
.git/hooks/pre-push
```

生成的脚本包含 `# review-pilot managed hook` marker。review-pilot 只会自动覆盖和卸载带有这个 marker 的 hook。

## 两种 profile

`pre-commit` 使用轻量 profile：

```bash
review-pilot review --staged --no-ai --fail-on P1
```

它适合在提交前拦截高风险本地规则问题。

`pre-push` 使用更完整的本地 profile：

```bash
review-pilot review --staged --no-ai --with-tools --fail-on P2
```

它可以运行 Semgrep 等工具证据检查。外部工具缺失时，review-pilot 会在工具结果中记录状态；具体是否阻止 push 取决于 review findings 和 `--fail-on` 阈值。

## 查看状态

```bash
review-pilot hooks status
```

输出会显示每个 hook 是否存在、是否由 review-pilot 管理，以及对应 profile。

## 卸载

```bash
review-pilot hooks uninstall --pre-commit --pre-push
```

卸载只删除 review-pilot 管理的 hook。已有的普通用户 hook 不会被自动删除。

## 已有 hook

如果 `.git/hooks/pre-commit` 或 `.git/hooks/pre-push` 已经存在，而且不是 review-pilot 生成的脚本，安装命令会停止并提示：

```bash
review-pilot hooks install --pre-commit --force
```

只有确认旧 hook 可以替换时才使用 `--force`。

## 跳过 hook

Git 自身支持跳过部分本地 hook：

```bash
git commit --no-verify
git push --no-verify
```

这适合紧急场景。本地 hook 不是团队级强制验证，后续 GitHub Actions 仍然要跑完整 review pipeline。
