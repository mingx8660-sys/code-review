# Docker/容器化审查清单

## 镜像构建
- [ ] 使用官方基础镜像而非 `latest` 标签（如 `python:3.12-slim`）
- [ ] 多阶段构建分离构建环境和运行环境
- [ ] `.dockerignore` 排除不必要的文件（node_modules、.git、__pycache__）
- [ ] 按变更频率排序 RUN 指令（不变更的先执行，利用缓存）
- [ ] 合并连续的 `RUN apt-get install` 减少层数
- [ ] `apt-get` 后清理缓存（`rm -rf /var/lib/apt/lists/*`）
- [ ] 区分 `RUN pip install` 和源代码 COPY（先装依赖再 COPY 源码）
- [ ] 使用 `--no-cache-dir` 避免 pip 缓存层
- [ ] 固定基础镜像和依赖版本（`python:3.12.2-slim@sha256:xxx`）
- [ ] 避免安装不必要的包（如 gcc、build-essential）

## 安全性
- [ ] 不以 `root` 用户运行（`USER nobody` 或创建专用用户）
- [ ] 不通过 ARG/ENV 传递敏感信息（改用运行时环境变量）
- [ ] 基础镜像定期扫描 CVE（使用 `docker scout` / `trivy`）
- [ ] 不安装 `curl`/`wget` 在最终镜像（除非必要）
- [ ] `HEALTHCHECK` 指令是否配置
- [ ] 非必要的 setuid/setgid 二进制移除
- [ ] 使用 `--chmod` 或 `--chown` 在 COPY 时设置权限

## 运行时
- [ ] `CMD` vs `ENTRYPOINT` 使用正确（CMD 做默认参数，ENTRYPOINT 做主程序）
- [ ] 环境变量有默认值和说明
- [ ] 挂载点（`VOLUME`）声明
- [ ] 容器端口暴露使用 `EXPOSE`
- [ ] OOM 保护：`--memory` 和 `--memory-reservation` 限制
- [ ] 只读根文件系统（`readOnlyRootFilesystem: true`）
- [ ] `privileged` 模式是否必要
- [ ] 资源限制（CPU/Memory 配额）

## Docker Compose
- [ ] 服务间依赖使用 `depends_on` 控制启动顺序
- [ ] 敏感信息使用 `secrets` 而非环境变量
- [ ] 持久化数据使用命名卷（`volumes:`）而非绑定挂载
- [ ] 网络隔离：创建专用网络而非使用默认网络
- [ ] `restart` 策略合理配置（always/unless-stopped/on-failure）
- [ ] 日志大小限制（`logging.driver` 和 `max-size`）
- [ ] `healthcheck` 在服务级别配置

## 常见反模式

```dockerfile
# ❌ 反模式：使用 :latest
FROM python:latest

# ✅ 正确写法：固定版本 + 精简镜像
FROM python:3.12-slim
```

```dockerfile
# ❌ 反模式：先 COPY 源码再装依赖（每次源码改动都重装依赖）
COPY . /app
RUN pip install -r requirements.txt

# ✅ 正确写法：先 COPY requirements 再 COPY 源码
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt
COPY . /app
```

```dockerfile
# ❌ 反模式：root 用户运行
FROM python:3.12-slim
COPY . /app
CMD ["python", "app.py"]

# ✅ 正确写法：创建非 root 用户
FROM python:3.12-slim
RUN useradd --create-home appuser
USER appuser
COPY --chown=appuser:appuser . /app
CMD ["python", "app.py"]
```
