## Why

用户当前缺少信用卡积分兑换商品/服务的能力。需要在 app 内提供积分余额查询、商品浏览与选择、积分兑换扣减的完整流程，提升信用卡用户活跃度和积分使用率。

## What Changes

**业务变更：**
- 新增 `points-redemption` capability：积分余额查询、商品兑换、积分扣减
- 提供积分兑换 API 接口

**脚手架变更：**
- openspec change 结构（proposal/specs/.meta.json/.openspec.yaml）

## Capabilities

### New Capabilities
- `points-redemption`: 信用卡积分兑换功能——用户可查询积分余额、选择商品进行兑换、兑换成功后自动扣减积分

### Modified Capabilities
<!-- 本次不涉及已有 capability 的修改 -->

## Impact

- 新增 API：积分查询接口、商品列表接口、兑换提交接口
- 数据存储：需新增积分兑换记录表
- 依赖：依赖现有信用卡积分系统获取积分余额
