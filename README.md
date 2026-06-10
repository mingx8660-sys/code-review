# Code Review — AI代码审查助手 🦐

一个 OpenClaw 技能，提供专业的多维度代码审查。

## 审查能力

| 维度 | 覆盖范围 |
|------|----------|
| 🎯 五维度核心 | 正确性、安全性、性能、可读性、可维护性 |
| 🐍 Python专项 | 40+检查项、常见反模式 |
| 🐳 Docker/容器化 | 镜像构建安全、多阶段构建、运行时配置 |
| 🔄 CI/CD | GitHub Actions 安全、缓存策略、流程规范 |
| 🗄️ 数据库/SQL | N+1查询、索引设计、Schema规范、ORM优化 |
| 🌐 API设计 | RESTful规范、认证授权、错误处理、分页 |

## 支持语言

Python、JavaScript、TypeScript、Go、Rust、Java

## 安装

作为 OpenClaw 技能安装：

```bash
# 1. 克隆到 skills 目录
cd ~/.openclaw/workspace/skills/
git clone https://github.com/mingx8660-sys/code-review

# 2. 重启 OpenClaw
openclaw gateway restart
```

## 用法

```
帮我review一下这段代码
code review这个文件
看看这个函数有没有问题
审查这个项目的Dockerfile
```

## 文件结构

```
references/
├── python-checklist.md     # Python 审查清单
├── docker-checklist.md     # Docker/容器化
├── cicd-checklist.md       # CI/CD 流水线
├── database-checklist.md   # 数据库/SQL
└── api-checklist.md        # API 设计
```

## 许可

MIT
