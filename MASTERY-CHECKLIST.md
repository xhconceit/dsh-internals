# DeepSeek Harness 最终掌握度清单

标记规则：

- `[P]`：可以独立完成，并有代码、测试或运行证据。
- `[I]`：只能解释，尚未亲手验证。
- `[N]`：尚未掌握。

只有 P0 全部达到 `[P]`，才能认为掌握了完整主干。平台限定项允许 `[I]`，但必须写明验证计划。

## P0：必须独立完成

- [ ] 从 `dsh` CLI 入口追踪到 Web 与 Headless Profile 启动。
- [ ] 根据 dump-config 解释 Bundle、Profile Patch、Home Patch 和 CLI overlay。
- [ ] 编写包含 Provider、Consumer、事件和卸载逻辑的 Cordis 插件。
- [ ] 用 Fake Provider 替换默认 Provider，而不修改 Consumer。
- [ ] 解释 Agent、Inbox、Turn、Step 和继续条件。
- [ ] 写出一次文本、Tool Call 和多 Step Turn 的完整事件序列。
- [ ] 修改 Agent 生命周期行为并补测试。
- [ ] 从 Session Events 手工重建模型消息。
- [ ] 新增 Session Event 并支持持久化、Replay、Query 和 Fork。
- [ ] 编写 LLM Mock Provider，覆盖文本、工具、错误和取消。
- [ ] 新增 System Prompt Section，并满足模型可见信息可重建约束。
- [ ] 编写带校验、审批、超时、取消和大小限制的 Tool。
- [ ] 证明 Agent Scope 能限制工具或 Provider 的可见性。
- [ ] 运行 Goal、Job 和 Subagent，并解释资源所有权。
- [ ] 验证 Workspace 路径、符号链接和越界策略。
- [ ] 取消子进程并证明没有残留进程树。
- [ ] 解释 pipes、Shell、PTY 和 ConPTY 的差异。
- [ ] 区分 Approval、Policy 和 Sandbox。
- [ ] 证明秘密不会进入日志、Session 或测试快照。
- [ ] 从 Session Event 追踪到 React Conversation Node。
- [ ] 新增支持实时和 Replay 的 Web Renderer。
- [ ] 通过 Headless 执行、取消并处理错误。
- [ ] 通过 Python SDK 创建、监听、取消和恢复任务。
- [ ] 为一个功能编写 unit、integration、Replay 和 E2E 测试。
- [ ] 构建并在干净环境安装 npm/Python 产物。
- [ ] 完成全链路毕业项目。

## P1：核心深度

- [ ] 解释 Cordis waterfall/serial 事件语义和委托规则。
- [ ] 证明插件启动失败不会留下部分副作用。
- [ ] 解释实时 Agent Event 与 durable Session Event 的边界。
- [ ] 分析流式输出中断时的恢复行为。
- [ ] 解释非幂等工具为什么不能简单 Replay。
- [ ] 比较 Fork 在不同事件边界的模型历史。
- [ ] 解释增量 Tool Call 的流式解析。
- [ ] 处理大 Tool Result、Chunk 数量和 UI 性能。
- [ ] 解释 Filesystem 与 Subprocess 为什么必须属于同一执行世界。
- [ ] 分析进程组、信号和终端资源清理。
- [ ] 分析 Landlock 的 fail-open/fail-closed 行为。
- [ ] 解释 Credential、Setting、Session 和 Telemetry 的存储边界。
- [ ] 处理 Web 断线、刷新和未知 Node 类型。
- [ ] 解释 Python 包如何携带或定位 Node runtime。
- [ ] 解释 runtime closure、package exports 和 native dependency。

## 平台验证

| 能力 | Linux | macOS | Windows | 证据/计划 |
|---|---|---|---|---|
| 普通 Subprocess | | | | |
| 进程树取消 | | | | |
| PTY/ConPTY | | | | |
| Shell quoting | | | | |
| Filesystem policy | | | | |
| Landlock/系统隔离 | | | | |
| native 依赖安装 | | | | |
| npm 打包运行 | | | | |
| Python runtime | | | | |

## 最终评分

- P0 `[P]` 数量：____ / 26
- P1 `[P]` 数量：____ / 15
- 已验证平台：
- 尚未验证平台：
- 最大知识缺口：
- 下一个真实 Issue：

## 判断

- P0 少于 20：尚未掌握完整主干。
- P0 20～25：能够开发，但仍有关键缺口。
- P0 全部完成：完整主干已掌握。
- P0 全部完成且大多数 P1 完成：接近核心开发水平。
- 再处理真实跨平台 Issue 和维护者 Review：开始形成维护者级熟练度。
