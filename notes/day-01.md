# Day 01：环境、仓库结构与构建基线

## 今日目标

- 使用 Git Submodule 管理 DeepSeek Harness 官方源码。
- 确认 Node.js、pnpm、Git 和操作系统环境。
- 理解 Monorepo 一级目录的职责。
- 理解 Host、Client、Web 三阶段构建。
- 安装依赖并建立 TypeScript 检查基线。

## 固定环境

- 源码目录：`/Users/orange/code/dsh-internals/source`
- 源码管理：Git Submodule
- Node.js：`v25.8.1`
- 当前全局 pnpm：`10.32.1`
- 仓库指定 pnpm：`11.7.0`
- Git：`2.50.1`
- OS：macOS arm64
- Branch：待补充
- Commit：待补充

## 仓库入口

### CLI

CLI 位于：

```text
apps/cli
```

它提供 `dsh` 命令，并负责根据命令行参数选择和启动 Profile。

### Web

Web 应用位于：

```text
apps/web
```

它是使用 React 和 Vite 组装出的最终浏览器应用。

## `packages/*/*` 两层结构

```text
packages/<领域>/<具体包>
```

- 第一层是职责领域或包组，例如 `core`、`llm`、`bundle`。
- 第二层才是具体的 Workspace Package，通常包含自己的 `package.json`。

使用 `packages/*/*` 可以让 pnpm 把具体包纳入 Workspace，而不把领域分类目录本身当成包。

## `native/landlock-run`

`native/landlock-run` 是 Linux 平台的原生沙箱启动器。它使用 Landlock 限制 Agent 启动的进程能够访问哪些文件和目录，从而让 Shell 或工具命令在受限的文件系统环境中执行。

边界：

- Landlock 是 Linux 内核能力，不是 macOS 或 Windows 的通用方案。
- 它主要限制文件系统访问，并不自动限制网络、CPU 和内存等所有资源。
- 用户审批表示用户是否同意；Landlock 才是操作系统层面的强制限制。

## `python/sdk-runtime`

Python SDK 提供符合 Python 使用习惯的调用接口，但 DeepSeek Harness 的核心运行时仍主要由 TypeScript/Node.js 实现。

`python/sdk-runtime` 是 Python SDK 所需 Node 运行部分的依赖清单和打包根节点。它确定需要随 Python SDK 分发的 Harness 包及其完整依赖闭包，让 Python 用户能够启动真正的 Harness 运行时，而不需要用 Python 重写 Agent、Tool 和 Session 等核心功能。

```text
Python 程序
  → Python SDK
  → Python/Node 通信边界
  → Harness Runtime
  → Agent Loop、Tools、Session、LLM
```

## `vendor/` 与 `packages/`

### `vendor/`

保存 Harness 使用的第三方或上游基础库源码。这些库被纳入 Monorepo，并固定为仓库内的受控版本。

### `packages/`

保存 DeepSeek Harness 自身的 Agent 产品功能，例如 Agent、Session、Tool、LLM、Filesystem 和 Bundle。

`pnpm-workspace.yaml` 中的 `overrides` 会强制部分依赖使用 `vendor/` 内的本地版本，从而避免不同包解析到不同版本、上游更新影响构建，或本地修改没有真正被使用。

## Host、Client 与 Web 构建

```text
build
├── build:lib
│   ├── build:lib:host
│   │   ├── tsc：Host 类型检查和发射
│   │   └── tsdown：Host 包构建
│   └── build:lib:client
│       ├── tsc：Client 类型检查和发射
│       └── tsdown：Client 包构建
└── build:web
    └── Vite：组装最终网页
```

- Host 是 Node.js 后端，可以访问文件、进程、终端和本地系统。
- Client 是浏览器端可复用库，不能意外依赖 Node.js API。
- Web 使用 Client、React 和 Vite 组装最终网页。

分开构建既隔离运行环境和依赖，也避免 Host 与 Client 在相同 Cordis `Context` 键上声明不同服务时出现 TypeScript 声明合并冲突。

## Git Submodule 安装问题

### 现象

执行依赖安装时失败：

```text
[install-lefthook] cannot enable extensions.worktreeConfig
while core.worktree is in the common config
```

### 原因

`source` 是父学习仓库中的 Git Submodule。它的 `.git` 指向：

```text
/Users/orange/code/dsh-internals/.git/modules/source
```

子模块的公共 Git 配置使用：

```ini
core.worktree = ../../../source
```

DeepSeek Harness 的 `postinstall` 脚本需要启用：

```ini
extensions.worktreeConfig = true
```

以便为每个 Worktree 配置独立的 Lefthook Hooks 和翻译配对合并驱动。脚本检测到公共配置中的 `core.worktree` 后，为避免擅自迁移或破坏 Git 子模块元数据而主动拒绝继续。

这不是 pnpm 依赖解析错误，也不是 Node.js 版本错误。

### 当前处理

学习项目将 Harness 作为只固定源码版本的子模块，不需要它修改父仓库的 Git Hooks，因此使用：

```bash
CI=true npx pnpm@11.7.0 install --frozen-lockfile
```

在 `CI=true` 环境下，`scripts/install-lefthook.mjs` 会在发现和修改 Git 配置前直接返回。其他依赖安装和构建流程仍正常执行。

暂时不手工修改：

```text
/Users/orange/code/dsh-internals/.git/modules/source/config
```

如果以后要向 DeepSeek Harness 正式贡献代码，优先使用独立 Clone 或按照项目诊断完整迁移 Worktree 配置，而不是长期跳过贡献者 Hooks。

## 下一步基线

```bash
cd /Users/orange/code/dsh-internals/source
CI=true npx pnpm@11.7.0 install --frozen-lockfile
npx pnpm@11.7.0 run typecheck
```

结果待补充：

- install 是否成功：
- install 耗时：
- typecheck 是否成功：
- typecheck 耗时：
- 第一处错误：
- `git status --short`：

## 尚未解决

- 补充当前源码 Branch 和 Commit。
- 完成依赖安装与 TypeScript 基线检查。
- 确认子模块模式下 Web、Headless 和测试是否都能正常运行。
