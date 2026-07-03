# 飞书通知

`review-pilot notify feishu` 从 `review-report.json` 读取结构化报告，渲染一张飞书摘要卡片。卡片只放协作入口需要的信息：项目、PR、风险等级、finding 数量、最高风险、Top Findings 和完整报告链接。

## 本地预览

```bash
review-pilot notify feishu --report review-pilot-artifacts/review-report.json --dry-run
```

dry-run 只输出卡片 payload，不请求飞书 webhook，也不会把 webhook 地址打印出来。

## 真实发送

配置飞书群机器人 webhook：

```bash
export FEISHU_WEBHOOK_URL='https://open.feishu.cn/open-apis/bot/v2/hook/xxxx'
```

然后关闭 dry-run 发送。真实发送适合放在 CI 的受控环境变量里，不应该把 webhook 写进仓库、截图或课程文档。

## 卡片内容

飞书卡片字段来自同一份 `review-report.json`：

- 项目：`repo_info.repository`
- PR：`repo_info.pull_request`
- 风险等级：根据最高 severity 和 finding 数量计算
- Finding 数量：`summary.total_findings`
- 最高风险：`summary.highest_severity`
- Top Findings：按 severity 取前 3 条
- 完整报告链接：由 `--report-url` 传入

飞书只做摘要通知和跳转入口，完整报告仍然放在 GitHub artifact 或后续 Dashboard。
