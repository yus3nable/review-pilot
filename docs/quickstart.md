# Quickstart

这份 quickstart 用来跑通 review-pilot 的最终形态：本地 review、GitHub Actions artifact、PR Summary Comment 和飞书摘要通知。

## 安装

```bash
python -m pip install -e ".[dev]"
review-pilot doctor
```

## 本地 review

在任意 Git 仓库里制造一处 staged change，然后运行：

```bash
review-pilot review --staged --no-ai --format markdown --output review-report.md
```

如果要验证模型链路，可以使用 fake provider：

```bash
review-pilot review --staged --provider fake --format json --output review-report.json
```

## GitHub Actions artifact

本地用 fixture 模拟 GitHub Actions：

```bash
review-pilot github-action --event-path tests/fixtures/github/pull_request_event.json --output-dir review-pilot-artifacts --dry-run
```

命令会生成：

```bash
review-pilot-artifacts/review-report.md
review-pilot-artifacts/review-report.json
```

## PR Summary Comment

预览将要发布到 PR 下的 summary comment：

```bash
review-pilot github-action --event-path tests/fixtures/github/pull_request_event.json --output-dir review-pilot-artifacts --dry-run --post-summary-comment
```

真实 GitHub Actions 中发布评论时，需要 workflow 具备写评论权限，并保证 `GITHUB_TOKEN` 可用。summary comment 带有 `<!-- review-pilot-summary -->` marker，重复运行会更新旧评论。

## 飞书通知

本地预览飞书卡片 payload：

```bash
review-pilot notify feishu --report review-pilot-artifacts/review-report.json --dry-run
```

真实发送时，把 webhook 放到环境变量：

```bash
export FEISHU_WEBHOOK_URL='https://open.feishu.cn/open-apis/bot/v2/hook/xxxx'
review-pilot notify feishu --report review-pilot-artifacts/review-report.json
```

webhook 不应该写进仓库、截图、日志或教程正文。

## 全量测试

```bash
python -m pytest
```
