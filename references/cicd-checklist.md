# CI/CD 流水线审查清单

## GitHub Actions

### 安全性
- [ ] 使用 `actions/checkout@v4` 而非 `@v3` 或 `@master`
- [ ] 第三方 Action 固定到 commit SHA（如 `actions/checkout@abc123`）
- [ ] 不打印敏感 Secrets（`echo ${{ secrets.KEY }}`）
- [ ] Pull Request 触发使用 `pull_request_target`（有安全风险）
- [ ] `GITHUB_TOKEN` 权限设置为最小必要（`contents: read`）
- [ ] 避免在 `script` 或 `run` 中拼接 Secrets
- [ ] OIDC 代替长期密钥（`id-token: write` + `permissions`）
- [ ] `actions/upload-artifact` 的上传内容不含敏感信息

### 效率
- [ ] 缓存依赖（`actions/cache` 或 `setup-python` 内置缓存）
- [ ] 并行 job / matrix build 充分利用
- [ ] 条件触发避免不必要的运行（`paths-ignore` / `paths`）
- [ ] 构建产物复用（artifacts 缓存而非重新构建）
- [ ] 工作流拆分：lint/test/build/deploy 分步执行
- [ ] 超时设置（`timeout-minutes`）防止 runaway job
- [ ] 自托管 Runner 的资源限制

### 流程
- [ ] 代码质量门禁：lint → test → build → deploy
- [ ] 测试覆盖率阈值（低于阈值阻断）
- [ ] 分支保护规则：PR 必须通过 CI、必须 Review
- [ ] 环境隔离：dev/staging/production 分环境部署
- [ ] 部署回滚机制
- [ ] 通知：失败时通知相关人（Slack/Discord/邮件）

## 常见反模式

```yaml
# ❌ 反模式：所有分支都触发完整流水线
on: [push]

# ✅ 正确写法：按路径和分支过滤
on:
  push:
    branches: [main, develop]
    paths-ignore:
      - 'docs/**'
      - '*.md'
  pull_request:
    branches: [main]
```

```yaml
# ❌ 反模式：Secrets 泄露到日志
- run: echo "Deploying with key ${{ secrets.DEPLOY_KEY }}"

# ✅ 正确做法：永远不要打印 Secrets
- run: echo "Deploying to production"
```

```yaml
# ❌ 反模式：没有超时限制
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

# ✅ 正确写法：设置超时
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - run: npm test
```
