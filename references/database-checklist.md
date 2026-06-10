# 数据库/SQL 审查清单

## 查询性能
- [ ] EXPLAIN 分析是否执行（全表扫描 vs 索引扫描）
- [ ] 是否存在 N+1 查询（循环中查询数据库）
- [ ] SELECT * 是否必要（应只查需要的列）
- [ ] WHERE 条件列是否建立了索引
- [ ] JOIN 是否可以通过冗余字段或反范式化避免
- [ ] 子查询 vs JOIN vs CTE 的性能比较
- [ ] OR 条件是否导致索引失效（改用 UNION）
- [ ] LIKE '%xxx' 是否必要（前缀模糊搜索不走索引）
- [ ] IN + 子查询的优化（改用 JOIN 或 EXISTS）
- [ ] LIMIT + OFFSET 深分页优化（改用游标分页）

## 索引设计
- [ ] 复合索引的列顺序（高选择性列在前）
- [ ] 冗余索引清理（前缀相同的重复索引）
- [ ] 覆盖索引减少回表查询
- [ ] 索引列避免函数操作（`WHERE DATE(created_at)` → 不走索引）
- [ ] 索引列避免隐式类型转换
- [ ] 外键约束是否有对应索引（避免锁表）
- [ ] 唯一约束 vs 唯一索引的选择

## 安全性
- [ ] 参数化查询防止 SQL 注入（$1 或 ? 占位符）
- [ ] ORM 的 Raw SQL 是否经过检查
- [ ] 最小权限原则：数据库用户只授权需要操作的表
- [ ] 敏感字段加密存储（密码使用 bcrypt/argon2）
- [ ] 不直接存储原始 Token/API Key（存储哈希）
- [ ] 日志中不打印 SQL 参数值（尤其是包含敏感数据时）
- [ ] 备份数据加密

## Schema 设计
- [ ] 字段类型选择合理（INT vs BIGINT、VARCHAR vs TEXT）
- [ ] NOT NULL + DEFAULT 减少 NULL 处理
- [ ] 适当的约束（CHECK、UNIQUE、FOREIGN KEY）
- [ ] 表命名规范：单数/复数统一、小写+下划线
- [ ] 主键设计：自增 PK vs UUID vs 雪花ID
- [ ] 预留扩展字段方案（不要存 JSON 万能字段）
- [ ] 迁移脚本可回滚
- [ ] 软删除 vs 硬删除 vs 归档策略

## ORM 常见问题 (SQLAlchemy / Prisma / TypeORM)
- [ ] N+1: ORM 懒加载导致的多次查询（使用 `joinedload` / `include`）
- [ ] 批量操作使用批量 API（`bulk_insert` / `bulk_create`）
- [ ] 事务范围控制：过长事务导致锁等待
- [ ] 查询返回所有字段（ORM 默认行为）
- [ ] 连接池配置：`pool_size` / `max_overflow` 是否合理

## 常见反模式

```sql
-- ❌ 反模式：SELECT * + 大表全表扫描
SELECT * FROM orders WHERE status = 'pending';

-- ✅ 正确写法：只查需要的列 + 索引
SELECT id, user_id, amount, created_at
FROM orders
WHERE status = 'pending'
  AND created_at >= '2024-01-01';
```

```sql
-- ❌ 反模式：N+1（在代码中循环查询）
for user in users:
    orders = db.query(Order).filter_by(user_id=user.id).all()

-- ✅ 正确写法：一次查询
user_ids = [u.id for u in users]
orders = db.query(Order).filter(Order.user_id.in_(user_ids)).all()
```

```sql
-- ❌ 反模式：函数套在索引列上
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-01';

-- ✅ 正确写法：范围查询，走索引
SELECT * FROM orders
WHERE created_at >= '2024-01-01 00:00:00'
  AND created_at < '2024-01-02 00:00:00';
```
