# 实验 2：生产级项目统计工具

## 目标

实现 `project_stats` 工具，同时掌握 Tool Pipeline、Filesystem、Session Event、Replay、Query、Fork 和 Agent Scope。

## 输入

- `path`：Workspace 内相对目录
- `ignore`：忽略模式
- `maxFiles`：扫描上限
- `includeHidden`：是否包含隐藏文件

## 输出

- 文件总数
- 总行数
- 按扩展名分类
- 被忽略或截断的数量
- 扫描耗时
- 是否完成或取消

## 强制约束

- 不接受 Workspace 外路径。
- 明确处理符号链接。
- 支持 AbortSignal。
- 设置文件数、结果大小和耗时上限。
- 结果使用结构化类型。
- 工具失败不能破坏整个 Session。
- 不把完整文件内容写入 Session Event。

## Session 扩展

新增项目统计事件，并为它实现：

- 写入
- 序列化
- Replay
- Query
- Fork
- Transcript 策略
- 未知版本降级

## 测试矩阵

| 场景 | 预期 |
|---|---|
| 空目录 | 返回零统计 |
| 普通项目 | 分类与行数正确 |
| 不存在目录 | 结构化错误 |
| `../` 越界 | 执行前拒绝 |
| symlink 越界 | 按明确策略拒绝或跳过 |
| 扫描中取消 | 保存取消事实且释放资源 |
| 超过文件上限 | 返回截断信息 |
| 插件卸载 | 工具和事件处理器不可用 |
| Replay | 与实时结果一致 |
| Fork | 边界前后上下文正确 |

## 完成标准

- 模型可以正确发现并调用工具。
- 安全性不依赖 Prompt。
- Session 恢复后 UI/Query 结果一致。
- 测试不调用真实模型 API。
- 插件文档包含限制和安全边界。
