# ARD222222 任务清单

## 概述

- **Change ID**: ARD222222
- **Capability**: points-redemption（信用卡积分兑换）
- **技术栈**: Node.js + TypeScript + Express + SQLite (better-sqlite3)

---

### Task 1: 项目初始化与类型定义

- [ ] 安装依赖：`npm install express better-sqlite3 uuid`
- [ ] 安装类型：`npm install -D @types/express @types/better-sqlite3 @types/uuid typescript`
- [ ] 创建 `src/points-redemption/types.ts`，定义所有 TypeScript 类型：
  - `PointsAccount`（user_id, points_balance, created_at, updated_at）
  - `Product`（id, name, points_required, stock）
  - `RedemptionRecord`（id, user_id, product_id, points_used, idempotency_key, created_at）
  - `RedeemRequest`（userId, productId）
  - `RedeemResponse`（redemptionId, userId, productId, pointsUsed, pointsRemaining）
  - `ApiError`（error, message）
- [ ] 创建 `src/points-redemption/db.ts`，实现 SQLite 初始化：
  - `initDb()` 函数：创建 `points_accounts`、`products`、`redemption_records` 三张表（`CREATE TABLE IF NOT EXISTS`）
  - `seedData()` 函数：插入预置种子数据（2 个用户 + 3 个商品）
  - 使用 WAL 模式：`PRAGMA journal_mode=WAL;`

**Commit**: `[ARD222222] feat: T1 - 项目初始化与类型定义`

### Task 2: 积分查询接口

- [ ] 在 `src/points-redemption/service.ts` 中实现 `getPointsBalance(userId: string): Promise<PointsBalanceResponse>`
  - 查询 `points_accounts` 表
  - 不存在时抛出 `POINTS_ACCOUNT_NOT_FOUND` 错误
- [ ] 在 `src/points-redemption/routes.ts` 中注册 `GET /api/points/:userId`
  - 参数校验：userId 非空
  - 成功返回 `{ userId, pointsBalance }`
  - 失败返回 404 + 错误信息
- [ ] 写单元测试：`src/points-redemption/__tests__/get-balance.test.ts`
  - 正常查询返回积分余额
  - 不存在的用户返回 404

**Commit**: `[ARD222222] feat: T2 - 积分查询接口`

---

### Task 3: 商品列表接口

- [ ] 在 `src/points-redemption/service.ts` 中实现 `listProducts(): Promise<ProductListResponse>`
  - 查询 `products` 表所有记录
  - 返回 `{ products: [...] }` 格式
- [ ] 在 `src/points-redemption/routes.ts` 中注册 `GET /api/points/products`
  - 无参数，直接返回所有在架商品
  - 空列表场景正常返回 `{ products: [] }`
- [ ] 写单元测试：`src/points-redemption/__tests__/list-products.test.ts`
  - 正常返回商品列表
  - 无商品时返回空列表

**Commit**: `[ARD222222] feat: T3 - 商品列表接口`

---

### Task 4: 兑换核心逻辑

- [ ] 在 `src/points-redemption/service.ts` 中实现 `redeemPoints(request: RedeemRequest): Promise<RedeemResponse>`
  - 幂等检查：按 `idempotency_key` 查 `redemption_records`，已存在直接返回已有结果
  - 参数校验：userId/productId 非空
  - 查用户积分：不存在 → 404
  - 查商品：库存=0 → 409
  - 积分校验：balance < points_required → 409
  - 事务执行：扣减积分 + 扣减库存 + 插入兑换记录
  - 返回兑换结果
- [ ] 在 `src/points-redemption/routes.ts` 中注册 `POST /api/points/redeem`
  - 从 header 读取 `X-Idempotency-Key`
  - 请求体校验：userId/productId 必填
  - 成功返回 200 + 兑换结果
  - 幂等重复返回 200 + 已有结果
  - 积分不足返回 409
  - 库存不足返回 409
- [ ] 写单元测试：`src/points-redemption/__tests__/redeem.test.ts`
  - 正常兑换成功
  - 积分不足返回 409
  - 库存不足返回 409
  - 幂等重复提交返回已有结果
  - 事务回滚验证（积分不足时不扣减）

**Commit**: `[ARD222222] feat: T4 - 兑换核心逻辑`

---

### Task 5: 集成测试与端到端验证

- [ ] 写集成测试：`src/points-redemption/__tests__/integration.test.ts`
  - 完整流程：查询积分 → 浏览商品 → 提交兑换 → 验证积分扣减
  - 验证幂等：同一 key 重复提交，积分只扣一次
  - 验证事务：并发提交时数据一致性
- [ ] 启动服务并手动验证 3 个 API 端点
- [ ] 运行 `openspec validate --strict ard222222` 确认 spec 通过

**Commit**: `[ARD222222] test: T5 - 集成测试与验证`

---

## 依赖关系

```
T1 (初始化) → T2 (查询) → T3 (列表) → T4 (兑换) → T5 (集成测试)
                  ↘           ↙
                  可并行开发
```

## 验收标准

- [ ] 3 个 API 端点全部可调通
- [ ] 单元测试覆盖 ≥ 80%
- [ ] 幂等性验证通过（重复提交不重复扣减）
- [ ] 事务回滚验证通过（积分不足时不扣减）
- [ ] `openspec validate --strict ard222222` 通过