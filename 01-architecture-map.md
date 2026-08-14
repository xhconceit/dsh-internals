# DeepSeek Harness 架构地图

本文件给出学习导航，不替代源码。每个箭头都需要在课程中用实际代码和运行证据验证。

## 1. 产品装配

```text
dsh CLI
  └─ boot selected profile
       ├─ base bundle
       │    ├─ models and credentials
       │    ├─ session and persistence
       │    ├─ agent and agent loop
       │    ├─ tools
       │    ├─ filesystem/subprocess/sandbox
       │    └─ settings/telemetry
       ├─ web-app bundle OR headless bundle
       ├─ profile cordis.patch.yml
       ├─ home cordis.patch.yml
       └─ command-line patch overlay
```

Profile 决定使用哪些 Bundle；Bundle 提供有序配置；Patch 按层覆盖或插入配置行。最终结果是一棵 Cordis 插件树。

## 2. Cordis 运行时

```text
Context
├─ Services: 可替换能力
├─ Events: 观察与拦截点
├─ Effects: 可撤销注册和副作用
├─ Plugins: 安装能力及生命周期
└─ Realms/Scopes: 隔离不同 Agent 或配置实例
```

阅读任何插件时都标记：输入配置、注册服务、监听事件、产生副作用、卸载行为和失败路径。

## 3. Agent 主流程

```text
message/injection
      ↓
Agent Inbox
      ↓
turn/start
      ↓
claim input ── agent/pre-step
      ↓
step/start
      ↓
append user/session events
      ↓
derive model history + assemble prompt/tools
      ↓
agent/request → llm/stream
      ↓
assistant/chunk* → assistant/message
      ↓
tool/call*
      ↓
pre-execute → execute → post-execute
      ↓
tool/result*
      ↓
step/end ── more work? ── yes → next step
      ↓ no
agent/turn-stopping
      ↓
turn/end
```

核心不变量：任何进入模型请求的可见信息，都必须能从持久化会话事实中重建。

## 4. 三类事件

### Session Event

持久事实，例如用户消息、模型 Chunk、工具调用和工具结果。用于重放、恢复、分叉、转录和 UI 渲染。

### Agent Event

运行中控制点，携带活跃 Agent，例如预处理输入、模型请求、状态变化、继续与停止。

### Capability Event

能力边界上的策略和适配点，例如文件系统、工具执行和遥测。消费者不需要直接依赖 Agent Loop。

## 5. Capability Seam

每条能力缝隙都有三个角色：

```text
Service Definition ← Provider
        ↑
     Consumer
```

需要重点建立下表：

| 能力 | Definition | 默认 Provider | 主要 Consumer | 策略/事件 |
|---|---|---|---|---|
| LLM | 待源码确认 | 待确认 | Agent Loop | LLM/Agent 事件 |
| Tools | `ctx.tools` | 工具注册实现 | Agent Loop/Prompt | `tools/*` |
| Session | `ctx.sessions` | 内存与持久化实现 | Agent/UI | `session/event` |
| Filesystem | `ctx.fs` | 本地或远程实现 | 文件工具 | `fs/*` |
| Subprocess | `ctx.subprocess` | 本地/远程实现 | Shell/Terminal | 执行策略 |
| Sandbox | `ctx.sandbox` | 平台实现 | 进程消费者 | 审批/包装 |
| Jobs | `ctx.jobs` | Job 实现 | 工具/Agent | Job 事件 |
| Goals | `ctx.goals` | Goal 实现 | Agent/工具 | `agent/*` |
| Subagent | 待源码确认 | 子 Agent/外部委派 | Agent 工具 | Scope/Session |

“待确认”必须在对应课程日用固定 commit 的源码填完，避免课程文档随着上游变化而给出错误包名。

## 6. 会话作为事实来源

```text
SessionEvent stream
├─ deriveMessages() → model history
├─ replay → restored runtime/UI
├─ fork(boundary) → child session
├─ transcript → human-readable record
├─ telemetry → observations
└─ persistence → durable storage
```

新增模型可见内容时，不能只修改 Prompt；还需要设计 Session Event、序列化、重放、派生和 UI 表示。

## 7. 执行世界

Filesystem 和 Subprocess 应指向一致的执行世界：

```text
Agent tools
├─ Filesystem Provider ─┐
├─ Shell Backend       ├─ local machine OR remote sandbox
├─ Terminal Backend    │
└─ LSP                 ┘
```

如果文件工具看到本地目录，而 Shell 在远程容器中运行，就会产生状态不一致。课程第 3 周需要专门验证这个边界。

## 8. Web 数据方向

```text
Agent runtime
  ↓ session/event + live status
client transport/state
  ↓
Conversation Node definition
  ↓ keyed renderer
React UI
```

UI 不能只处理实时事件；刷新页面后必须从持久化 Session Event 得到一致显示。

## 9. 研究完成的判断方式

对每个子系统至少能够回答：

- 服务接口在哪里？
- 谁注册默认实现？
- 谁消费它？
- 配置如何选择实现？
- 哪些事件允许拦截？
- 取消、失败和卸载如何清理？
- 状态是否持久化？如何重放？
- 有哪些跨平台差异？
- 哪些测试定义了契约？

不能回答其中两项以上，就不能把该子系统标记为已掌握。
