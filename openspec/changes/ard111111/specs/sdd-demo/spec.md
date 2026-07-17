## ADDED Requirements

### Requirement: 用户环境测试——多环境配置可切换

系统 SHALL 支持通过环境变量 `SDD_ENV` 切换运行环境（dev/staging/prod），不同环境加载不同配置，SDD 流程在各环境下均能正常运行。

#### Scenario: 默认环境为 dev

- **WHEN** 未设置 `SDD_ENV` 环境变量时执行 SDD 流程
- **THEN** 系统默认使用 dev 环境配置，流程正常运行

#### Scenario: 切换至 staging 环境

- **WHEN** 设置 `SDD_ENV=staging` 后执行 `/sdd-apply`
- **THEN** 系统加载 staging 环境配置（如 staging API 端点），`openspec validate` 通过

#### Scenario: 切换至 production 环境

- **WHEN** 设置 `SDD_ENV=production` 后执行 `/sdd-apply`
- **THEN** 系统加载 production 环境配置，输出日志标记 `[PRODUCTION]`，流程正常运行

#### Scenario: 环境变量值非法

- **WHEN** `SDD_ENV` 设置为 unrecognized 值（如 `SDD_ENV=unknown`）
- **THEN** 系统输出错误信息"不支持的环境: unknown，可选值: dev/staging/production"，流程停止

### Requirement: 用户环境测试——环境间数据隔离

不同环境的 SDD 流程数据（change 状态、产物路径、日志）SHALL 相互隔离，互不干扰。

#### Scenario: 同一 change 在不同环境各自独立

- **WHEN** 先后在 dev 和 staging 环境下对同一 change 执行 `/sdd-apply`
- **THEN** 两环境的产物（tasks.md 执行状态、日志输出）存储在独立路径，不相互覆盖

#### Scenario: 环境切换后不丢失已有数据

- **WHEN** 在 dev 环境执行 `/sdd-apply` 完成部分 task，切换至 staging 执行后，再切回 dev
- **THEN** dev 环境的执行进度（已完成 task 标记）完整保留

### Requirement: 用户环境测试——配置校验

系统 SHALL 在流程启动时校验当前环境配置完整性，配置缺失时给出明确错误信息和修复指引。

#### Scenario: 环境配置文件缺失

- **WHEN** 切换至某环境但对应配置文件（如 `config/staging.yaml`）不存在
- **THEN** 系统输出错误"环境配置缺失: staging，请创建 config/staging.yaml"，流程停止

#### Scenario: 环境配置修复后可继续

- **WHEN** 因配置缺失导致流程停止后，用户补全配置文件并重新执行
- **THEN** 系统读取新配置，校验通过后正常运行
