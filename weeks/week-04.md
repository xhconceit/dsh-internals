# 第 4 周：Web、SDK、测试、发布与毕业项目

目标：掌握产品表面和工程交付链路，并完成贯穿所有核心子系统的功能。

## Day 22：Web Client 架构

### 核心内容

- Vite/React 应用入口
- client/host 包边界
- Client Modules
- UI Primitives
- Slots
- transport/API client
- Session store/projection
- live Agent status
- settings and approval UI
- reconnect 与历史恢复

### 追踪实验

从一次 `assistant/chunk` 开始，追踪到页面文本更新；刷新页面后，再追踪从持久化事件到同一 UI 的路径。

分别标记：

- Host 产生的数据
- 传输层格式
- Client 状态
- React 组件
- 实时路径
- Replay 路径

### 通过标准

- 能证明实时渲染和历史渲染结果一致。
- 能区分 Agent live status 与 durable Session state。
- 能指出 Web 包为什么依赖特定 client libraries。

## Day 23：Conversation Node 与 Slots

### 核心内容

- Conversation Node Definition
- keyed renderer
- event-to-node projection
- streaming update
- unknown node fallback
- Slots 和 UI 扩展
- 插件卸载后的 UI 行为
- 可访问性和错误状态

### 实验

为第 2 周的项目统计事件新增 UI Node：

- 实时显示扫描进度
- 完成后显示文件数、行数和分类
- 刷新页面后从 Session Event 重建
- 老版本事件仍可显示或安全降级
- 插件未加载时不导致整个会话崩溃

### 测试

- projection unit test
- React component test
- replay test
- Playwright interaction test

### 通过标准

实时、刷新、Fork 和未知事件四种场景都可预测。

## Day 24：CLI、Headless 与 ACP

### 核心内容

- CLI command routing
- human command 与 model turn
- Headless one-shot runner
- stdin/stdout 协议
- exit code
- cancellation
- Agent Client Protocol
- editor/client integration
- 无服务器运行和 Web Profile 的差异

### 实验

1. 通过 Headless 启动任务并捕获结构化输出。
2. 从外部发送取消。
3. 模拟模型错误和工具错误，检查退出码。
4. 通过 ACP 示例建立客户端连接。
5. 比较 CLI human command 和用户消息的事件差异。

### 通过标准

能为自动化调用者定义明确的成功、失败、取消和部分完成语义。

## Day 25：Python SDK 与 Runtime 分发

### 核心内容

- Python 包公开 API
- Node runtime/executable 的分发
- Python 与 Harness 进程通信
- 生命周期和资源清理
- async/sync API
- 事件迭代
- 错误映射
- cancellation
- package installation
- platform wheel/runtime concerns

### 实验

用 Python 完成：

1. 创建 Harness 客户端或运行实例。
2. 启动一个 Headless 任务。
3. 监听 Session/Agent 事件。
4. 在工具运行时取消。
5. 重启并恢复 Session。
6. 捕获并分类模型、工具和进程错误。

### 通过标准

- Python 退出后没有遗留 Node 进程。
- 取消和错误不会被错误地包装成成功结果。
- 在干净虚拟环境中可以复现安装和运行。

## Day 26：测试体系

### 测试层级

- unit
- integration
- snapshot
- Web snapshot
- E2E
- performance
- stress
- property-based
- consumer/package contract
- cross-platform gate

### 工具

- Vitest
- Testing Library
- Playwright
- Mock LLM
- fast-check
- coverage

### 实验

选择毕业项目的一个垂直功能，分别编写：

1. 纯函数单元测试。
2. Cordis 插件集成测试。
3. Session Event Replay 测试。
4. Web E2E 测试。
5. 取消或失败测试。
6. 一项属性测试或随机输入测试。

### 通过标准

- 测试失败时能定位到契约，而非依赖大快照猜问题。
- 不用真实模型 API 作为常规测试前提。
- 时间、随机数和外部进程均可控制。

## Day 27：构建、质量门禁与发布

### 核心内容

- host/client TypeScript build
- `tsdown`
- Vite Web build
- package exports
- runtime closure
- npm pack
- Python runtime packaging
- native dependency
- license/third-party notices
- OXLint、Knip、Publint、jscpd
- docs verification
- Node compatibility
- Windows/Linux CI gates

### 实验

1. 构建全部库和 Web。
2. 对一个修改运行相关质量门禁。
3. 生成 npm 包但不发布。
4. 在干净临时目录安装打包产物。
5. 启动 Web/Headless 验证运行时闭包。
6. 检查 package exports、source map 和许可证文件。
7. 验证 Python runtime 的干净安装。

### 通过标准

源码仓库中“能运行”不算完成；打包产物在干净环境中也必须工作。

## Day 28：毕业项目实现

开始 [实验 4](../labs/lab-04-capstone.md)。推荐功能：代码审查 Bundle。

当天完成：

- Service/Provider/Consumer 设计
- Profile/Bundle 配置
- 工具和权限策略
- Agent Scope 或 Subagent
- Session Event
- Web Conversation Node
- Headless/Python 调用路径
- 单元和集成测试

禁止直接修改所有调用方以接入功能；优先通过现有扩展点实现。

## Day 29：审计、故障注入与修复

对毕业项目进行四类审计：

### 正确性

- 空输入
- 重复输入
- 并发输入
- 中途取消
- 模型失败
- 工具失败
- 重启和 Replay
- Fork

### 安全性

- 路径越界
- 命令注入
- 凭据泄漏
- 审批参数变化
- 子进程残留
- 不支持平台的危险降级

### 性能

- 大 Session
- 大 Tool Result
- 大量流式 Chunk
- 并发 Job/Subagent
- UI 重渲染
- 内存和句柄泄漏

### 可维护性

- 包边界
- 配置 Schema
- 错误分类
- 文档
- 测试层次
- 插件卸载

修复发现的最高风险问题，并保留失败前后的测试证据。

## Day 30：闭卷验收与知识缺口

完成 [最终掌握度清单](../MASTERY-CHECKLIST.md)。

### 现场任务

不参考课程答案，在固定 commit 上完成：

1. 从 CLI 追踪 Web 启动。
2. 替换一个 Provider。
3. 新增 Agent 级工具。
4. 新增 Session Event 并 Replay/Fork。
5. 为事件增加 Web Renderer。
6. 取消一个带子进程的工具并确认清理。
7. 用 Python/Headless 调用该功能。
8. 打包并在干净环境验证。

### 最终报告

报告必须分成三类：

- 已用代码和测试证明掌握
- 仅用源码推导，尚未在当前平台验证
- 尚未掌握，需要后续实践

不要为了“全部学会”的表述隐藏第三类。准确知道自己的边界，也是维护复杂系统的核心能力。
