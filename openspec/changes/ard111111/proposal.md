## Why

当前 `sdd-demo` capability 仅验证了 SDD 五环流程的规格定义完整性，缺少对**多用户环境**（开发/测试/预发/生产）场景的覆盖。需要在 demo 中增加用户环境维度的测试案例，确保 SDD 流程在不同环境中可复现、可验证。

## What Changes

**业务变更：**
- 修改 `sdd-demo` capability：新增"用户环境测试"相关 requirement 与 scenario，覆盖多环境配置、环境隔离、环境变量注入等场景

**脚手架变更：**
- openspec change 结构（proposal/specs/.meta.json）

## Capabilities

### New Capabilities
<!-- 本次不涉及新增 capability -->

### Modified Capabilities
- `sdd-demo`: 新增用户环境测试 requirement，包含多环境配置验证、环境间隔离、环境变量驱动行为等 scenario

## Impact

- 仅影响 `openspec/specs/sdd-demo/spec.md`（archive merge 后更新）
- 无代码/API/数据存储变更
- 无下游依赖
