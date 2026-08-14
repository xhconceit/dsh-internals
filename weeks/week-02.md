# 第 2 周：会话、模型、工具、多 Agent 与恢复

目标：掌握 Agent 的事实来源、模型边界、工具执行管线和跨 Agent 协作。

## Day 8：Session Event 数据模型

### 核心内容

- append-only Session Event log
- `SessionEventMap`
- Turn、Step、User、Assistant、Tool 事件
- raw assistant chunks
- Session 内存表示
- `session/event` 广播
- 事件顺序与标识符

### 实验

运行三个任务：纯文本、一次工具调用、多次工具调用。导出事件序列并标注：

- durable fact
- UI-only projection
- model-visible projection
- live-only state

手工实现一个小型投影器，只根据事件生成：

- 用户/助手消息列表
- 工具调用列表
- Turn/Step 统计

### 通过标准

不能读取运行时 Agent 对象，只允许使用 Session Event，也能生成正确结果。

## Day 9：持久化、Replay 与 Resume

### 核心内容

- Session Store 与 Persistence Provider
- JSONL 或当前 commit 使用的持久化格式
- 写入原子性
- raw chunk 与完整消息
- 重放和 UI 恢复
- Agent Resume
- 崩溃或截断记录
- 数据迁移边界

### 实验

1. 在模型流式输出期间中断进程。
2. 重启并观察事件和 UI 如何恢复。
3. 在工具执行前、执行中、执行后分别中断。
4. 比较哪些状态可以恢复，哪些需要标记失败。
5. 制造一个无效尾行，仅在测试副本上研究恢复行为。

### 通过标准

- 能说明持久化成功不等于操作副作用可重放。
- 能解释如何避免恢复时重复执行非幂等工具。
- 能指出当前实现对损坏或旧版本数据的处理策略。

## Day 10：deriveMessages、Fork、Query 与 Transcript

### 核心内容

- `deriveMessages()` 投影规则
- 模型历史与 UI 历史的差异
- Session Fork 的边界
- Session Query
- Transcript
- 标题生成
- 遥测如何从事件派生

### 实验

1. 从真实事件手工推导一次模型请求历史。
2. 在第一次 Tool Result 前后各 Fork 一次。
3. 比较两个子 Session 的模型历史。
4. 写一个 Session Query，统计工具失败率。
5. 生成 Transcript 并指出它省略了哪些内部事实。

### 通过标准

- 能解释为什么 Fork 应基于事件边界而非任意内存状态。
- 能证明模型看到的上下文可由日志重建。

## Day 11：System Prompt 与 LLM Adapter

### 核心内容

- Prompt Sections
- Tool Schema assembly
- model message vocabulary
- LLM service seam
- Provider 注册和模型路由
- streaming chunk vocabulary
- 增量 Tool Call
- usage、错误、重试和取消
- credentials 与自定义 endpoint

### 实验

实现确定性的 Mock Provider：

- 能输出文本 Chunk
- 能分片输出一次 Tool Call
- 能返回 Usage
- 能响应 AbortSignal
- 能注入预设错误

新增一个 Prompt Section，并验证：

- 它只在目标 Agent 生效
- 它进入模型请求
- 相关信息能从 Session 事实重建或明确标记为非模型可见

### 通过标准

测试覆盖文本、工具、取消和错误四条路径。

## Day 12：Tool Registry 与受保护执行管线

### 核心内容

- scoped tool registry
- Tool Schema
- 参数校验
- Prompt 中的工具定义
- `tools/pre-execute`
- approval/policy
- `tools/execute`
- `tools/post-execute`
- Tool Result 持久化
- 长输出、二进制输出、敏感输出
- timeout 和 cancellation

### 实验

开始 [实验 2](../labs/lab-02-production-tool.md)，实现 `project_stats`：

- 支持指定相对目录
- 统计文件、扩展名和代码行数
- 忽略配置化目录
- 路径必须限制在 Workspace
- 支持取消
- 大结果必须限制大小
- 错误返回结构化信息

### 失败测试

- 参数缺失或类型错误
- 目录不存在
- `../` 越界
- 符号链接越界
- 运行中取消
- 输出超过上限
- 插件卸载后仍尝试调用

### 通过标准

模型能选择工具，但安全策略和参数校验不能依赖模型自觉。

## Day 13：Goal、Job、Subagent 与 Agent Scope

### 核心内容

- same-session objective
- Goal 的生命周期与继续执行
- 后台 Job 的启动、收集和停止
- Subagent Provider seam
- 新子 Agent 与外部产品委派
- 父子会话和消息关系
- 每 Agent Context
- scoped registrations
- 取消传播和资源所有权

### 实验

1. 创建一个目标并跨多个 Turn 推进。
2. 启动后台 Job，轮询结果并停止。
3. 让父 Agent 委派一个只读子任务。
4. 为子 Agent 注册专用工具，证明父 Agent 看不到。
5. 取消父任务，记录对子 Agent 和 Job 的影响。

### 通过标准

- 能区分 Goal、Job 和 Subagent 的职责。
- 能指出子 Agent 能力集合在哪里构造。
- 能解释取消和清理的所有权。

## Day 14：周项目——会话型生产工具

完成 [实验 2](../labs/lab-02-production-tool.md)。除工具本身外，新增一种用于记录统计摘要的 Session Event，并完成：

- 类型定义
- 事件写入
- 持久化
- Replay
- Query
- Transcript 策略
- Fork 行为
- 单元和集成测试

### 闭卷验收

1. Session Log 如何成为模型上下文的事实来源？
2. raw Chunk 为什么值得持久化？
3. Tool Call 执行前后经过哪些拦截点？
4. 非幂等工具中断后如何避免重复副作用？
5. Goal、Job 和 Subagent 有何区别？
6. Agent Scope 如何改变工具可见性？

毕业条件：从全新配置安装插件，完成任务，中断并恢复，然后在 Fork 中得到正确上下文。
