# GitHub Action

`review-pilot github-action` 用在 GitHub Actions 的 `pull_request` 事件里。它读取事件文件，识别当前 PR，复用 GitHub provider 和 workspace plan，生成 `review-report.md` 与 `review-report.json` 两个 artifact。

## 本地模拟

```bash
review-pilot github-action --event-path tests/fixtures/github/pull_request_event.json --dry-run
```

预览 PR summary comment：

```bash
review-pilot github-action --event-path tests/fixtures/github/pull_request_event.json --dry-run --post-summary-comment
```

## Workflow 示例

```yaml
name: review-pilot

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: read
  issues: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: python -m pip install -e .
      - run: review-pilot github-action --event-path "$GITHUB_EVENT_PATH" --output-dir review-pilot-artifacts --dry-run
      - uses: actions/upload-artifact@v4
        with:
          name: review-pilot-report
          path: review-pilot-artifacts/
```

`issues: write` 只用于 PR summary comment。GitHub 的 PR 评论接口走 Issues Comments API，所以权限名不是 `pull-requests: write`。如果只生成 artifact，不发布评论，可以保留 `contents: read` 和 `pull-requests: read`。

## 输出

命令会写出：

```bash
review-pilot-artifacts/review-report.md
review-pilot-artifacts/review-report.json
```

## Summary Comment

M24 开始，`--post-summary-comment` 会把报告摘要渲染成一条 PR 总览评论。评论正文带有稳定 marker：

```html
<!-- review-pilot-summary -->
```

重复运行时，review-pilot 先查找带 marker 的旧评论，再更新旧评论。这样 CI 反复运行不会在 PR 下刷屏。

dry-run 模式会把将要发布的 comment body 打印在 JSON 输出里，不调用 GitHub 评论 API。真实发送时需要 `GITHUB_TOKEN` 可用，并给 workflow 配置写评论权限。

## 和飞书通知的关系

GitHub artifact 保存完整报告，PR summary comment 展示协作摘要，飞书通知负责把摘要推到团队 IM。三者都从 `review-report.json` 读取数据，不反向解析 Markdown。
