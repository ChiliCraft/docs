# ChiliCraft · 项目总文档（docs）

ChiliChill 2026 巡演「混入人类计划 II：方街」主题 Minecraft 服务器插件组（Paper 1.21.1）的共享文档仓。原 monorepo 拆分后，各模块独立建仓，本仓收纳全部跨模块资料。

## 仓库结构

| 仓库 | 职责 |
|---|---|
| `ChiliCraft/docs` | 本仓：文档、架构、事件协议、开发规范、调研报告；挂入聚合仓时位于 `docs/` |
| `ChiliCraft/cc-core` | 核心：档案、全局模式、参数服务、事件总线、灵魂货币、家园区、DB、`/menu` 与 `/cc` 统一入口 |
| `ChiliCraft/cc-survival` | 生存压力：饥饿加速、体温、负重、耐久损耗 |
| `ChiliCraft/cc-demon` | 饿魔：夜晚刷怪与索敌、饿意事件 |
| `ChiliCraft/cc-soul` | 灵魂：死亡灵魂损失与遗物、葬礼、收容播报 |
| `ChiliCraft/cc-martial` | 武学：竞技场对战、境界、技能与演出 |
| `ChiliCraft/cc-adventure` | 冒险：地城、远征与世界 Boss |
| `ChiliCraft/cc-season` | 季节：日历、天气、生态与指南 |
| `ChiliCraft/cc-quest` | 任务：章节状态、事件推进、灵魂奖励、旅途手册 |
| `ChiliCraft/cc-street` | 方街（规划中，尚未建立仓库） |

聚合仓：[`ChiliCraft/chilicraft`](https://github.com/ChiliCraft/chilicraft)，统一管理 Gradle 多工程、CI 与所有子仓的固定提交；八个插件仓和本仓均作为 Git 子模块挂载。

## 必读顺序

1. [architecture.md](architecture.md) —— 分层与装配模式
2. [events-protocol.md](events-protocol.md) —— 事件契约与全量清单
3. [performance.md](performance.md) —— 性能红线
4. 改模块前对照 [module-dev-guide.md](module-dev-guide.md)
5. [AGENTS.md](AGENTS.md) —— AI agent 操作指引（多仓结构版）

## 模块间协作

模块间零编译期依赖，只靠 cc-core 事件总线通信（事件名先在 events-protocol.md 登记再编码）。各附属以 `compileOnly("com.chilicraft:cc-core:1.0.0")` 引核心 API（GitHub Packages / mavenLocal 解析），第三方联动全部 `compileOnly` 正规 Maven 坐标。

## 各仓构建

每个模块仓独立 CI（`.github/workflows/build.yml`）：JDK 21 上 `gradlew build` 并上传 `build/libs/*.jar`。聚合仓在根目录执行完整构建并汇总产物；本地命令见[聚合仓 README](https://github.com/ChiliCraft/chilicraft/blob/main/README.md)。本仓 CI 仅校验关键文档在位。

## 本仓内容

- 本仓根目录 —— 架构、事件协议、性能、模块开发指南、开发计划、文案需求（挂入聚合仓后对应 `docs/`）
- `AGENTS.md` —— agent 操作指引
- 两篇根目录大文档 —— 插件版技术文档（实现规格 v1.1）、开源插件缝合候选调研报告
