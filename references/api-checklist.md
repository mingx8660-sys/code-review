# API 设计审查清单

## RESTful 规范
- [ ] URL 使用名词复数而非动词（`/api/users` 而非 `/api/getUsers`）
- [ ] HTTP 方法语义正确：GET(查) POST(增) PUT(全量改) PATCH(部分改) DELETE(删)
- [ ] 嵌套资源路径合理（`/api/users/123/orders`）
- [ ] 查询参数用于过滤/排序/分页（`?status=active&page=1&sort=-created_at`）
- [ ] 响应格式统一（统一包一层 `{data, meta, error}`）
- [ ] HTTP 状态码使用正确（200/201/204/400/401/403/404/409/422/500）
- [ ] 版本化策略（URL path: `/api/v1/` 或 Header: `Accept: application/vnd.api+json;version=1`）

## 安全性
- [ ] 认证方式明确（JWT / OAuth2 / API Key / Session）
- [ ] JWT 实现规范：过期时间、签名算法、不包含敏感信息
- [ ] 授权校验：用户能否访问该资源
- [ ] 速率限制（Rate Limiting）实现
- [ ] 输入校验：请求体 Schema 验证（Pydantic / Zod / Joi）
- [ ] CORS 配置：来源白名单而非 `*`
- [ ] HTTPS 强制
- [ ] 敏感字段过滤：不在响应中返回 password/token/secret
- [ ] CSRF 防护（Cookie-based 认证时）

## 错误处理
- [ ] 错误响应格式统一（`{error: {code, message, details}}`）
- [ ] 不暴露内部错误详情（500 时返回通用消息）
- [ ] 验证错误返回具体字段信息
- [ ] 幂等性（Idempotency）支持（POST 创建）
- [ ] 批量操作的事务性和错误回滚

## 性能与可用性
- [ ] 分页：`page`/`per_page` + `total`/`has_more`
- [ ] 字段选择：支持 `?fields=id,name` 按需返回字段
- [ ] 缓存头：`ETag` / `Last-Modified` / `Cache-Control`
- [ ] 压缩：响应体 Gzip/Brotli 压缩
- [ ] 批量查询：支持 `?ids=1,2,3` 减少请求次数
- [ ] 异步接口：长时间任务返回 202 + 轮询/Webhook

## API 文档（OpenAPI/Swagger）
- [ ] 所有端点有对应的 OpenAPI 描述
- [ ] 请求/响应 Schema 定义完整
- [ ] 认证方式在文档中声明
- [ ] 错误响应有示例
- [ ] 弃用端点标记 `deprecated: true`

## 常见反模式

```python
# ❌ 反模式：动词在 URL 中
@router.get("/getUserOrders")
def get_user_orders(user_id: int):
    ...

# ✅ 正确写法：名词 + HTTP 方法
@router.get("/users/{user_id}/orders")
def list_user_orders(user_id: int):
    ...
```

```python
# ❌ 反模式：错误信息暴露内部细节
return {"error": "Database connection failed: MySQL server has gone away"}

# ✅ 正确写法：统一错误格式 + 日志记录
return {"error": {"code": "INTERNAL_ERROR", "message": "服务内部错误"}}
# 同时在日志中记录完整错误
logger.error(f"数据库连接失败: {e}")
```

```python
# ❌ 反模式：没有分页
GET /api/orders?status=pending  # 返回所有数据

# ✅ 正确写法：游标/偏移分页
GET /api/orders?status=pending&page=1&per_page=20
# 响应: {data: [...], meta: {page: 1, per_page: 20, total: 150, has_more: true}}
```
