# 准备与学习方法

## 1. 固定研究对象

DeepSeek Harness 处于开发者预览阶段。开始学习时记录仓库 URL、分支、commit、Node.js 版本和 pnpm 版本。整个 30 天都基于同一个 commit，避免上游重构导致路径和行为变化。

建议将源码放在：

```text
dsh-internals/source
```

第一天在 `notes/baseline.md` 记录：

```text
Repository:
Branch:
Commit:
Node:
pnpm:
OS:
Architecture:
Baseline build:
Baseline tests:
```

## 2. 建立基线

按照官方 README 安装依赖并构建，然后分别运行：

- Web Profile
- Headless Profile
- 一项最小单元测试
- 一项 Web 测试

记录命令、耗时、输出和失败原因。后续每次核心修改都与该基线比较。

不要在第一天追求解决所有平台相关测试；先区分：

- 环境缺失
- 当前平台不适用
- 依赖安装问题
- 真正的代码失败

## 3. 建立证据索引

为每个结论记录四项信息：

| 字段 | 内容 |
|---|---|
| Claim | 你认为系统如何工作 |
| Source | 支持结论的源码或文档路径 |
| Runtime evidence | 日志、调试器或测试证据 |
| Counterexample | 哪种情况会推翻或限制该结论 |

示例：

```text
Claim: 模型可见输入必须能从 Session Log 重建。
Source: docs/architecture.md；Session 派生消息实现。
Runtime evidence: 重启后得到相同模型历史。
Counterexample: 仅运行时注入但未形成 Session Event 的数据。
```

## 4. 源码追踪方法

面对任何功能都执行同一套六步法：

1. 找 Service Definition：接口是什么？
2. 找 Provider：谁实现该接口？
3. 找 Consumer：谁使用它？
4. 找 Event：哪里可以观察或拦截？
5. 找 Configuration：如何装配和替换？
6. 找 Tests：作者认为哪些行为必须保持？

建议搜索模式：

```text
ctx.<service>
<domain>/*
SessionEventMap
plugin
provide
effect
dispose
abort
cancel
```

## 5. 调试策略

不要一上来逐行单步整个项目。先选一个最小任务，例如“读取一个文件并总结”，在以下边界记录结构化信息：

- 输入进入 Agent Inbox
- `turn/start`
- `step/start`
- Prompt 组装完成
- LLM 请求和流式 Chunk
- Tool Call
- Tool 执行前后
- `step/end`
- `turn/end`
- Session 持久化

日志只记录结构和标识符，避免输出 API Key、完整环境变量或敏感文件内容。

## 6. 实验分支

建议建立独立分支：

```text
study/day-01-baseline
study/week-01-cordis
study/week-02-session-tool
study/week-03-execution-security
study/week-04-capstone
```

每个实验保持小提交，并在提交说明中写清：

- 假设
- 改动
- 验证方法
- 结果
- 未解决问题

## 7. 每日完成条件

当天结束前必须回答：

1. 今天研究的能力由哪个服务定义？
2. 默认 Provider 是什么？
3. 哪些消费者依赖它？
4. 发生失败或取消时状态如何变化？
5. 如何从配置替换该能力？
6. 哪些测试保护了该行为？

有一题无法回答，就把它写入第二天的首要问题，不要用模糊理解掩盖缺口。

## 8. 安全规则

- 学习 Shell、Subprocess、Sandbox 时只使用专门的临时工作区。
- 不对个人主目录或真实项目执行破坏性测试。
- 不把真实 API Key 写进仓库、日志、截图或测试夹具。
- 路径穿越、符号链接和命令注入实验只能以无害标记文件作为目标。
- 测试进程清理时记录 PID，并确认子进程全部退出。
- 修改持久化前备份测试会话，不使用真实重要会话作为夹具。

## 9. 一个月的现实目标

180 小时足够建立全面、可开发的掌握程度，但不应声称已经经历所有操作系统和生产环境问题。最终成果应包含：

- 完整架构图
- 四个综合实验
- 一个贯穿全栈的毕业功能
- 一套自动化测试
- 一份尚未验证的平台与边界清单

真正的维护者级熟练度来自后续处理真实 Issue 和代码审查。
