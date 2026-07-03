## Constitution Check
*GATE: Must pass before design phase. Re-check after design complete.*

**Triggered HARD_RULEs**:
- 提交规范 (CLAUDE.md:58)：所有 commit 必须 `[ARD222222] <type>: <subject>` 格式

**Triggered SOFT_RULEs**:
- 推荐架构实践 §公共 API 文档 (CLAUDE.md:73)：新增的积分查询/商品列表/兑换接口必须有文档注释
- 推荐架构实践 §异步非阻塞 (CLAUDE.md:68)：兑换接口涉及外部积分系统调用，使用异步非阻塞方式
- 推荐架构实践 §连接池 (CLAUDE.md:69)：HTTP 和数据库连接需使用连接池管理

**Compliance Plan**:
- 提交规范：所有 task commit 统一用 `[ARD222222] <type>: <task-id> - <subject>`
- 公共 API 文档：三个 API 接口在代码中写 JSDoc/OpenAPI 注释
- 异步非阻塞：兑换核心链路使用 async/await，外部调用不阻塞主线程
- 连接池：数据库连接使用连接池管理

Re-check at design complete: no new rules triggered

---

## 技术选型

| 维度 | 选择 | 理由 |
|------|------|------|
| 运行时 | Node.js + TypeScript | 项目统一技术栈 |
| Web 框架 | Express | 轻量，社区成熟 |
| 数据库 | SQLite (better-sqlite3) | 零配置、适合 demo，同步 API 简化事务处理 |
| 积分管理 | 本地积分管理 | 独立闭环，不依赖外部系统 |
| 幂等机制 | 独立幂等 Key (UUID) | 客户端生成 `X-Idempotency-Key`，服务端按 key 去重 |

## 架构设计

### 模块划分

```
src/
  points-redemption/
    routes.ts          # Express 路由定义
    service.ts         # 业务逻辑层（积分查询、兑换校验、扣减）
    db.ts              # SQLite 初始化 + 数据访问层
    types.ts           # TypeScript 类型定义
```

### API 设计

**1. GET /api/points/:userId** — 积分余额查询

```
Response 200:
{
  "userId": "u001",
  "pointsBalance": 15000
}

Response 404:
{
  "error": "POINTS_ACCOUNT_NOT_FOUND",
  "message": "积分账户不存在"
}
```

**2. GET /api/points/products** — 可兑换商品列表

```
Response 200:
{
  "products": [
    { "id": "p001", "name": "星巴克咖啡券", "pointsRequired": 500, "stock": 100 },
    { "id": "p002", "name": "电影票",   "pointsRequired": 800, "stock": 0 }
  ]
}
```

**3. POST /api/points/redeem** — 提交兑换

```
Request:
{
  "userId": "u001",
  "productId": "p001"
}
Headers: X-Idempotency-Key: <UUID>

Response 200 (成功):
{
  "redemptionId": "R20260703-001",
  "userId": "u001",
  "productId": "p001",
  "pointsUsed": 500,
  "pointsRemaining": 14500
}

Response 409 (积分不足):
{ "error": "INSUFFICIENT_POINTS", "message": "积分不足" }

Response 409 (库存不足):
{ "error": "OUT_OF_STOCK", "message": "商品已兑完" }
```

### 数据库设计

**points_accounts** — 积分账户表

| 列 | 类型 | 说明 |
|----|------|------|
| user_id | TEXT PRIMARY KEY | 用户 ID |
| points_balance | INTEGER NOT NULL | 可用积分余额 |
| created_at | TEXT | 创建时间 |
| updated_at | TEXT | 更新时间 |

**products** — 商品表（预置种子数据）

| 列 | 类型 | 说明 |
|----|------|------|
| id | TEXT PRIMARY KEY | 商品 ID |
| name | TEXT NOT NULL | 商品名称 |
| points_required | INTEGER NOT NULL | 所需积分 |
| stock | INTEGER NOT NULL | 库存数量 |

**redemption_records** — 兑换记录表

| 列 | 类型 | 说明 |
|----|------|------|
| id | TEXT PRIMARY KEY | 兑换编号 |
| user_id | TEXT NOT NULL | 用户 ID |
| product_id | TEXT NOT NULL | 商品 ID |
| points_used | INTEGER NOT NULL | 消耗积分 |
| idempotency_key | TEXT UNIQUE | 幂等 Key |
| created_at | TEXT | 兑换时间 |

### 兑换核心流程

```
POST /api/points/redeem
  │
  ├─ 1. 幂等检查：查 redemption_records WHERE idempotency_key = ?
  │     └─ 已存在 → 直接返回已有结果（200）
  │
  ├─ 2. 参数校验：userId/productId 非空
  │
  ├─ 3. 查用户积分：SELECT points_balance FROM points_accounts WHERE user_id = ?
  │     └─ 不存在 → 404 POINTS_ACCOUNT_NOT_FOUND
  │
  ├─ 4. 查商品：SELECT points_required, stock FROM products WHERE id = ?
  │     └─ 库存=0 → 409 OUT_OF_STOCK
  │
  ├─ 5. 积分校验：balance >= points_required ?
  │     └─ 不足 → 409 INSUFFICIENT_POINTS
  │
  └─ 6. 事务执行：
        BEGIN TRANSACTION
          UPDATE points_accounts SET points_balance = balance - points_required
          UPDATE products SET stock = stock - 1
          INSERT INTO redemption_records (id, user_id, product_id, points_used, idempotency_key)
        COMMIT
        → 返回 200 + 兑换结果
```

### 关键设计决策

- **SQLite WAL 模式**：better-sqlite3 默认同步 API，但 WAL 模式下读写不互斥，适合并发场景
- **事务保证一致性**：积分扣减 + 库存扣减 + 记录插入在同一事务中，任一失败则回滚
- **建表脚本 idempotent**：`CREATE TABLE IF NOT EXISTS`，重复执行安全
