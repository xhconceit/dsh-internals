# 实验 1：可替换、可卸载的 Cordis 插件

## 目标

实现一个 `study-clock` 插件，用最小功能验证 Cordis 服务、Provider、Consumer、事件、配置和生命周期。

## 功能要求

- 定义 `StudyClock` 服务接口。
- 默认 Provider 返回系统时间。
- Fake Provider 返回可控时间，用于测试。
- Consumer 提供模型工具或人工命令读取当前时间。
- 每次读取发出类型化事件。
- 支持时区或格式配置。
- 可挂载到根 Context，也可限制到单个 Agent。
- 卸载时撤销服务、工具、监听器和资源句柄。

## 建议步骤

1. 选择与现有简单服务一致的包布局。
2. 声明接口和 `ctx` 类型扩展。
3. 实现默认 Provider。
4. 注册 Consumer。
5. 注册 typed event。
6. 创建 Fake Provider 测试。
7. 创建 Agent 级隔离测试。
8. 创建 unload 测试。
9. 编写配置示例和 README。

## 必须测试

- 默认 Provider 返回有效值。
- Fake Provider 完全可替换。
- 事件监听顺序符合预期。
- 无 Provider 时错误清晰。
- 两个隔离 Agent 使用不同 Provider。
- 卸载后服务和工具不可见。
- 插件启动失败不会留下部分注册。

## 验收问题

- Consumer 为什么不应导入默认 Provider？
- 哪些副作用由 Cordis 自动撤销，哪些需要手动注册清理？
- 根 Context 和 Agent Context 的工具可见性如何决定？
- 配置错误在哪个阶段被发现？

## 交付物

- 插件源码
- 单元和集成测试
- 配置 Schema
- 示例 Profile/Patch
- 生命周期图
- 10 分钟演示脚本
