# 实验 4：全链路毕业项目

## 推荐主题

实现一个 `code-review` Bundle：对 Workspace 中选定变更进行结构化审查，可委派子 Agent，并在 Web、Headless 和 Python 中使用。

也可以替换成远程执行 Provider、测试诊断 Agent 或仓库知识索引，但必须覆盖相同层级。

## 必须覆盖的架构层

### Cordis 与配置

- 新插件或 Provider
- 配置 Schema
- Bundle
- 示例 Profile/Patch
- Agent Scope
- 安装、卸载和替换

### Agent

- System Prompt Section
- 至少两个 Tool
- Goal 或 Subagent
- 取消和失败路径
- 明确的 Turn/Step 行为

### Session

- 新 Session Event
- 持久化
- Replay
- Query
- Fork
- Transcript 策略

### 执行与安全

- Workspace 边界
- Approval/Policy
- 超时和取消
- 结果上限
- 凭据脱敏
- 必要时使用 Sandbox

### Web

- Conversation Node Definition
- keyed renderer
- 实时进度
- 历史重放
- 错误与取消状态
- 未知版本降级

### 外部接口

- CLI 或 Headless
- ACP 或 Python SDK
- 明确退出码/错误类型

### 质量

- unit test
- plugin integration test
- Session Replay/Fork test
- Web component/E2E test
- cancellation/error test
- performance or stress evidence
- package installation verification
- 文档和示例

## 推荐功能流程

```text
用户选择审查范围
→ 创建 review goal
→ 工具收集差异和相关文件
→ 可选子 Agent 分析不同关注点
→ 聚合结构化发现
→ 写入 review Session Event
→ Web Node 显示进度与结果
→ Replay/Fork 保持一致
```

## 审查发现结构

建议包含：

- 文件与行号
- 严重程度
- 标题
- 证据
- 影响
- 修复建议
- 置信度

不要把模型临时思考过程作为持久化输出；只保存完成产品功能所需的结构化事实。

## 验收场景

1. 无代码变更。
2. 单文件简单问题。
3. 多文件并由子 Agent 分析。
4. 工具运行中取消。
5. 一个子 Agent 失败。
6. 进程重启后 Replay。
7. 在发现产生前后分别 Fork。
8. Web 未加载 Renderer。
9. 大型差异触发结果上限。
10. 干净环境安装打包产物。

## 最终演示

演示控制在 20 分钟：

1. 架构和扩展点，3 分钟。
2. Web 正常流程，4 分钟。
3. Headless/Python 流程，3 分钟。
4. 取消、恢复和 Fork，4 分钟。
5. 安全与测试证据，4 分钟。
6. 已知限制，2 分钟。

只有同时展示成功路径、失败路径和恢复路径，毕业项目才算完成。
