# Git 子仓工作区设计

## 目标

将 ChiliCraft 组织的八个插件仓库和文档仓库作为 `ChiliCraft/chilicraft` 的 Git 子模块，保留根仓库现有聚合工程，同时提供统一、可重复的子仓初始化入口。

## 仓库与目录

- 父仓：`ChiliCraft/chilicraft`，继续拥有根级 Gradle 聚合构建、CI、README 和项目说明。
- 插件子仓：`cc-core`、`cc-survival`、`cc-demon`、`cc-soul`、`cc-martial`、`cc-adventure`、`cc-season`、`cc-quest`，分别挂载在父仓现有同名路径。
- 文档子仓：将 GitHub 仓库 `ChiliCraft/chilicraft-meta` 重命名为 `ChiliCraft/docs`，并挂载在父仓 `docs/`。
- 父仓不把自己再次嵌套；`cc-street` 尚无独立仓库，不在本次范围。

## 内容合并与构建

各插件子仓包含父仓相应模块目录中的现有源文件，以及拆仓后增加的独立仓库文件。对照 GitHub `main` 树，八个模块均无父仓独有文件；现有源文件 blob 一致，差异只有子工程 `build.gradle.kts`，每个独立仓另有 9 个根级/构建文件。使用完整子仓作为子模块可保留两边内容，而不把拆仓文件丢掉。

独立模块的构建文件以 Maven 坐标 `com.chilicraft:cc-core:1.0.0` 引用核心；父仓聚合构建仍需从本地 `:cc-core` 工程解析。父仓根构建加入 Gradle dependency substitution，将该坐标替换为 `project(":cc-core")`；独立模块仓的构建文件不改，继续支持独立构建。

文档仓现有 `docs/` 下的 11 个文件与父仓 `docs/` 的 11 个文件逐项 blob 一致。为使文档仓可直接挂载为父仓 `docs/` 且不形成 `docs/docs/`，将文档仓 `docs/*` 展平至文档仓根，保留其 README、AGENTS、规格与调研资料；同步修正文档仓 README、AGENTS 与 docs CI 中的路径。

## 初始化

- 增加 `scripts/init-repos.sh` 与 `scripts/init-repos.ps1`，分别供 macOS/Linux 与 Windows PowerShell 使用。
- 两个入口统一调用 `git submodule update --init --recursive`，不跟踪子仓的最新分支，也不覆盖子模块中的本地修改。
- 新克隆父仓时记录 `git clone --recurse-submodules`；已有克隆执行对应平台初始化脚本即可。
- `.gitmodules` 记录每个子仓 URL，父仓 gitlink 固定其所需提交，保证不同机器初始化得到相同内容。

## 验收

1. GitHub `ChiliCraft/docs` 可访问，文档仓展平后的 README/AGENTS/CI 路径正确。
2. 父仓 `.gitmodules` 包含八个插件仓与 `docs`，路径与本设计一致。
3. 两个平台的初始化入口可重复执行，子模块状态无缺失提交。
4. 根工程 `gradlew build`（网络受限时 `gradlew build --offline`）成功；不启动 Minecraft 服务端。
5. 父仓变更保留在工作区，不提交、不推送；文档仓重命名与布局提交推送仅限本设计范围。

## 风险与约束

- 将现有同名模块目录改为子模块会改变父仓的 Git 跟踪形态；子仓代码改动需在对应子仓提交，父仓只更新 gitlink。
- 文档仓重命名会改变其规范 clone URL；旧 GitHub URL 通常提供重定向，但 `.gitmodules` 使用新 URL。
- 本次不改插件 Java 源码、不新增依赖、不启动服务端、不提交/推送父仓。
