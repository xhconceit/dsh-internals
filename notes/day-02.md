# Day 02：DeepSeek Harness 全局架构地图

## 今日目标

- 建立 DeepSeek Harness 的完整概念地图。
- 理解 Profile、Bundle、Plugin 和 Cordis Context。
- 理解 Session、Turn、Step 与 Agent Loop。
- 理解 LLM、Tool、Session Event 和 Web UI 的数据流。
- 理解执行环境、安全、取消、错误、测试和发布边界。

## 1. Host、Client 与 Web

```text
Host（Node.js）
├── 读取和修改文件
├── 启动进程与终端
├── 调用模型
├── 执行工具
└── 管理 Session

Client（浏览器可用库）
├── 会话投影
├── 实时状态
├── UI 扩展接口
└── 调用 Host 的远程接口

Web（最终应用）
└── 使用 Client、React 和 Vite 组装页面
```

浏览器不能直接使用 `node:fs` 或 `node:child_process`。Client 必须通过受控接口请求 Host 执行本地操作。

Host 先完成类型检查和远程契约生成，Client 再使用这些契约进行类型检查。Host 与 Client 也是两个独立 TypeScript Program，避免双方在相同 Cordis `Context` 键上声明不同服务时发生类型合并冲突。

## 2. Profile、Bundle、Plugin 与 Patch

### Plugin

真正提供功能的运行单元，例如 Agent Loop、Session、LLM Adapter、Tool、Filesystem 和 Web Server。Plugin 接收配置、注册 Service/Event/Tool，并在卸载时清理资源。

### Bundle

一组插件和配置的分发单元。例如 Base Bundle 组合 Agent、Session、LLM、Tools、Filesystem、Approval 和 Persistence。

### Profile

具名的完整产品组装方案：

```text
web Profile
├── base Bundle
└── web-app Bundle

headless Profile
├── base Bundle
└── headless Bundle
```

### Patch

通过稳定 Row ID 替换整行配置或插入新配置。配置应用顺序：

```text
Profile 中的 Bundles
→ Profile cordis.patch.yml
→ Harness Home cordis.patch.yml
→ 命令行 --patch
```

后层优先级更高。最终配置使用 `--dump-config` 检查，不能只看某个 Bundle 文件。

## 3. Cordis Context 与 Capability Seam

Cordis Context 是插件化能力容器：

```ts
ctx.sessions
ctx.tools
ctx.llm
ctx.fs
ctx.agents
```

一条 Capability Seam 包含：

```text
Service Definition：能力接口
Provider：具体实现
Consumer：接口使用方
```

消费者通过 `ctx` 使用 Service，而不直接依赖 Provider，因此可以替换本地、远程或测试实现。

Filesystem、Subprocess、Shell、Terminal 和 LSP 等相关 Provider 必须属于同一执行世界，否则 Agent 读取的文件与命令执行环境可能不一致。

## 4. Cordis Effect 与生命周期

插件加载会产生服务、工具、监听器、Timer、Job、进程和连接等副作用。这些资源必须作为可撤销 Effect 登记：

```text
Plugin 加载
→ Effects 生效
→ Plugin 卸载或启动失败
→ Effects 回滚/清理
```

框架只能清理已经登记的资源；未纳入生命周期的 Timer、Listener、进程和连接仍会泄漏。

## 5. Session、Turn 与 Step

```text
Session
└── 多个 Turn
    └── 每个 Turn 有零个或多个 Step
```

- Session：整个长期会话，可持久化、恢复和分叉。
- Turn：处理一次用户请求的完整工作轮次。
- Step：一次模型请求以及该响应产生的工具调用。

一个 Turn 可以包含多个 Step：

```text
Step 1：模型请求 read_file → Tool Result
Step 2：模型看到结果 → 最终回答
```

输入在 `agent/pre-step` 被拒绝时，Turn 也可能包含零个 Step。

## 6. Agent Loop

```text
用户消息
→ Agent Inbox
→ turn/start
→ agent/pre-step
→ step/start
→ 保存输入并派生模型历史
→ 组装 Prompt 与 Tool Schema
→ agent/request
→ llm/stream
→ assistant/chunk*
→ assistant/message
→ tool/call*
→ tools/pre-execute
→ tools/execute
→ tools/post-execute
→ tool/result*
→ step/end
→ 有后续工作则进入下一 Step
→ agent/turn-stopping
→ turn/end
```

Inbox 让外部输入在安全边界被 Agent 领取，而不是任意破坏正在运行的内部状态。

## 7. 三类事件

| 类型 | 回答的问题 | 持久化 |
|---|---|---|
| Session Event | 已经发生了什么？ | 是 |
| Agent Event | Agent 当前如何运行？ | 通常否 |
| Capability Event | 某项能力如何被控制？ | 通常否 |

Session Event 示例：`turn/start`、`user/message`、`assistant/chunk`、`assistant/message`、`tool/call`、`tool/result` 和 `turn/end`。

Agent Event 示例：`agent/pre-step`、`agent/request`、`agent/turn-stopping`。

Capability Event 示例：`tools/pre-execute`、`tools/execute`、`tools/post-execute`、`fs/*`、`telemetry/*`。

## 8. Session Event Log 与 Event Sourcing

Harness 用只追加的事件日志保存完整过程：旧事件不原地修改，新事实追加到末尾。

事件可用于重建模型历史、恢复 Web UI、Resume、Fork、Transcript、Session Query 和遥测派生。当前状态是事件的投影，不是唯一事实来源。

## 9. Model-visible means logged

任何进入模型上下文的信息都必须拥有可持久化、可重建的来源：

```text
Session Events
→ deriveMessages()
→ Model Messages
```

不是所有 Session Event 都需要发送给模型，例如 `turn/start` 和 `step/end` 主要用于系统状态。但所有模型可见消息都必须能从日志重建，否则 Resume、Fork 和调试会失真。

## 10. Replay、Resume 与 Fork

| 功能 | 是否创建新 Session | 作用 |
|---|---:|---|
| Replay | 否 | 从事件重建历史状态 |
| Resume | 否 | 在原 Session 后追加新事件 |
| Fork | 是 | 从事件边界创建新会话分支 |

Replay 不应重新执行历史工具，只重建 `tool/call` 与 `tool/result` 表示的事实。

Session Fork 不会自动分叉或回滚文件系统。执行环境隔离仍需要 Git Worktree、独立 Workspace 或远程 Sandbox。

## 11. Persistence 与副作用

Persistence Provider 将 Session Events 保存到磁盘或其他持久存储。

危险窗口：

```text
外部操作已经完成
→ Tool Result 尚未持久化
→ 进程崩溃
```

恢复后无法确定是否应该重试。发送邮件、支付、删除文件等非幂等工具必须有额外的幂等键、事务、状态查询或人工确认机制。

Session 恢复不等于文件系统或外部系统自动恢复。

## 12. LLM Adapter 与 Prompt

```text
Agent Loop
→ ctx.llm
→ 当前 LLM Provider
```

每个 Step 的模型请求由 System Prompt Sections、从 Session 派生的消息历史、当前输入、Tool Schemas 和模型配置组装。

Adapter 在 Harness 统一消息/Chunk 格式与具体模型 API 之间转换。Mock Provider 用于稳定测试文本、Tool Call、错误、取消和流式中断，不需要真实 API Key。

## 13. Streaming

```text
agent/request
→ llm/stream
→ assistant/chunk*
→ assistant/message
```

文本和 Tool Call 都可能分片到达。只有 Tool Call 参数完整后才能执行工具。保存 Chunk 可以支持实时显示、中断诊断和准确 Replay；完整 Message 表示正常聚合结果。

## 14. Tool Calling

模型不会亲自执行工具，只生成结构化请求：

```json
{
  "name": "read_file",
  "arguments": { "path": "README.md" }
}
```

Harness 负责：

```text
Schema 校验
→ 业务与路径校验
→ Policy
→ Approval
→ tools/execute
→ 结果处理与截断
→ tool/result
→ 下一 Step
```

Tool 面向模型；Service 定义内部能力；Provider 提供具体实现。例如 `read_file Tool → ctx.fs Service → Local/Remote Provider`。

JSON Schema 只能验证类型，不能保证路径安全或授权。

## 15. Approval、Policy 与 Sandbox

- Approval：用户是否同意这次操作？
- Policy：系统规则是否允许？
- Sandbox：操作系统能否强制限制？

三者不能互相替代。用户批准 `npm test` 不代表其所有后代进程可以不受限制地访问整台电脑。

`native/landlock-run` 是 Linux 文件系统隔离机制；当前 macOS 环境不能标记为已经实际验证 Landlock。

## 16. Filesystem、Subprocess、Shell、Terminal

- Filesystem：读取、写入、遍历文件。
- Subprocess：使用结构化 executable/args 启动进程。
- Shell：解释管道、重定向、变量、引号等命令语言。
- Terminal：通过 PTY/ConPTY 支持交互式程序。

Shell 字符串比结构化 argv 风险更高。取消进程时需要处理完整进程树，而不是只停止最外层 Shell。

## 17. Goal、Job、Subagent 与 Scope

- Goal：同一 Session 中持续推进的长期目标。
- Job：可查询、收集或停止的后台任务。
- Subagent：被委派推理任务的另一个 Agent。
- Agent Scope：为不同 Agent 限定 Tool、Prompt 和 Provider 可见性。

创建资源的组件负责清理资源。主任务取消时需要明确传播到 Job、Subagent、Subprocess 和 PTY。

## 18. Cancellation 与 Error Recovery

取消链路：

```text
Client
→ Agent Handle
→ Turn/Step
→ LLM Stream
→ Tool
→ Subprocess/Job/Subagent
```

每层都必须主动响应 AbortSignal 等取消机制并释放资源。

终态要区分：

```text
success
error
cancelled
timeout
```

Tool Error 通常可以转换为结果交给模型；Permission Denied 应在执行前阻止；LLM Error 按类型决定是否重试；Persistence Error 可能破坏可恢复性；取消不能撤销已经发生的外部副作用。

## 19. Web 数据流

```text
Session Event Log
→ Client Projection
→ Conversation Node
→ Keyed React Renderer
```

实时路径负责当前 Chunk、工具进度和审批；Replay 路径负责刷新与恢复。两者必须产生一致 UI。

Renderer 缺失时应安全降级，不能因为旧插件事件或插件卸载导致整个会话页面崩溃。

## 20. Web、Headless、ACP 与 Python SDK

- Web：长期服务器和 React UI，面向人类用户。
- Headless：通常执行一次任务后退出，面向 Shell/CI。
- ACP：通过 JSON-RPC/stdio 等协议供编辑器和程序控制 Agent。
- Python SDK：提供 Python 风格接口，底层仍驱动同一 Harness Runtime。

Human Command 可以直接由命令分发器执行；User Message 会进入 Agent Inbox 并触发模型 Turn。

## 21. Settings、Credentials、Session、Telemetry

| 数据 | 作用 |
|---|---|
| Settings | 普通运行配置 |
| Credentials | API Key 等秘密 |
| Session State | 具体会话事实 |
| Telemetry | 可选运行观测 |

凭据不能进入普通配置、Session Event、日志、错误堆栈、Telemetry 或测试快照。Telemetry 可被关闭或采样，不能作为会话恢复的唯一事实来源。

## 22. 测试体系

- Unit：局部逻辑。
- Plugin Integration：Cordis 服务、事件与生命周期。
- Replay：实时投影与历史重放一致。
- Snapshot：复杂结构与生成产物。
- E2E/Playwright：真实产品链路。
- Property-based：大量生成输入验证不变量。
- Performance/Stress：性能和稳定性。

常规测试使用 Mock LLM，不依赖真实模型、网络、费用和秘密凭据。

掌握一个功能至少需要验证正常、失败、取消、卸载或恢复路径。

## 23. 构建与发布

```text
Host tsc
→ Host tsdown
→ Client tsc
→ Client tsdown
→ Vite Web build
```

- `tsc`：类型 Program、Project References 和类型产物。
- `tsdown`：生成可发布库产物。
- Vite：构建浏览器应用。

源码仓库中能运行不等于发布包可用。还需要验证 package exports、运行时依赖闭包、干净消费者安装、native dependencies、npm/Python 打包、跨平台兼容、许可证和质量门禁。

## 当前理解总图

```text
Profile
└── Bundles
    └── Cordis Plugins
        ├── Services / Providers / Effects
        └── Agent Loop
            ├── Session Event Log
            ├── Prompt + LLM Adapter
            ├── Tool Pipeline
            ├── Filesystem / Process / Sandbox
            └── Web / Headless / ACP / Python
```

## 下一步

高层概念学习完成，下一阶段必须进入源码验证。第一条源码路径：

```text
package.json scripts.dsh
→ apps/cli/src/bin.ts
→ CLI 主函数
→ web 命令处理
→ Profile 启动
```

待完成：

- 阅读 `apps/cli/src/bin.ts`。
- 标出入口、参数、主函数、错误处理和退出码。
- 找到 `web` 命令进入的下一层模块。
- 用源码证据修正本笔记中的高层推断。
