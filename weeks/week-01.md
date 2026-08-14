# 第 1 周：启动、Cordis 与 Agent 主循环

目标：从产品装配一路追踪到一次完整 Agent Turn，并能写出可卸载的 Cordis 插件。

## Day 1：固定源码、构建与运行

### 六小时安排

1. 记录仓库 commit、Node、pnpm、OS 和 CPU 架构。
2. 阅读根 README、开发指南和根 `package.json`。
3. 安装依赖并完成基础构建。
4. 分别启动 Web 和 Headless。
5. 运行最小单元测试和 Web 测试。
6. 将结果写入 `notes/day-01.md`。

### 源码入口

- 根 `package.json` 的 workspace、build、test、release 脚本
- `apps/cli` 的命令入口
- `apps/web` 的 Vite 入口
- `packages/bundle/base`
- `packages/bundle/web-app`
- `packages/bundle/headless`

路径可能随版本变化。若入口不存在，按包名和 `dsh` 字段搜索，不要猜。

### 实验

选择一个只包含少量文本文件的临时工作区，让 Web Agent 完成“列出文件并总结”。保存：

- Session ID
- 产生的 Session Event 类型
- 模型调用次数
- 工具调用次数
- 是否出现审批
- 最终文件变化

### 通过标准

- Web 与 Headless 至少一种成功完成任务，另一种能解释启动失败原因。
- 能指出 CLI 的参数解析、Profile 选择和应用启动位置。
- 基线结果可由另一位开发者复现。

## Day 2：Monorepo 与包依赖图

### 核心内容

- workspace 包发现
- host/client 两种构建面
- 包 `exports`
- Bundle 与普通库包的区别
- vendored 包、native 包、Python runtime 的职责
- 构建顺序和运行时闭包

### 任务

生成或手工整理三张图：

1. `dsh` CLI → boot → profile → bundles。
2. Agent 核心包之间的依赖关系。
3. Web Client 与 Host 之间的边界。

选取五个核心包，为每个包记录：

- 包名与目录
- `exports`
- 直接依赖
- 对外服务或类型
- 最重要的测试

### 追踪问题

- 为什么仓库分别构建 host 和 client？
- 哪些包能进入浏览器，哪些只能运行在 Node.js？
- 一个 Bundle 如何声明它挂载的配置？
- Python runtime 为什么出现在 pnpm workspace 中？

### 通过标准

随机给出一个核心包时，10 分钟内能找到 Provider、Consumer、配置和测试入口。

## Day 3：Cordis Context、Service 与 Plugin

### 核心内容

- Context 是能力容器，而不只是全局对象
- Service Definition 与 Provider 分离
- Consumer 通过 `ctx` 依赖服务
- Plugin 接收配置并安装能力
- 插件树决定生命周期和可见性

### 源码阅读

阅读 Cordis primer 和 tutorial，然后选择一个简单 Harness 服务，跟踪：

```text
service type declaration
→ provider registration
→ ctx key
→ consumer access
→ configuration row
```

### 实验

完成 [实验 1](../labs/lab-01-cordis-plugin.md) 的第一部分：

- 定义 `ctx.studyClock` 服务
- Provider 返回可控时间
- Consumer 注册一个读取时间的命令或工具
- 测试 Provider 可以被替换为 Fake

### 通过标准

- Consumer 不直接导入 Provider 实现。
- Fake Provider 能替换默认实现而不改 Consumer。
- 能解释服务不可用时是启动错误、运行错误还是可选能力。

## Day 4：Event、Effect 与生命周期

### 核心内容

- typed event
- waterfall/serial 等事件语义
- listener 顺序和委托
- Effect 的注册与撤销
- 插件启动失败时的回滚
- Plugin unload 和资源清理
- Realm、Fork 与隔离作用域

### 实验

扩展 `studyClock`：

- 注册一个类型化事件
- 增加两个监听器并记录顺序
- 注册计时器或资源句柄
- 卸载插件
- 证明监听器、计时器、命令和服务全部消失
- 让第二个监听器抛错，观察清理结果

### 必答问题

- 普通 Node `EventEmitter` 为什么不足以表达这里的插件生命周期？
- Waterfall listener 为什么需要显式继续委托？
- 插件卸载时，哪些资源不会被框架自动发现？
- Agent 级 Context 与根 Context 的注册可见性有什么差异？

### 通过标准

测试必须验证卸载前后状态，而不是只验证插件能启动。

## Day 5：Profile、Bundle、Patch 与启动装配

### 核心内容

- Profile 是具名组合
- Bundle 是配置与代码的分发单元
- Patch 按 ID 替换整行配置或插入配置
- Base、Web、Headless 的组合关系
- Profile/Home/CLI overlay 的覆盖顺序

### 实验

1. 保存 Web Profile 的 dump-config。
2. 为五行关键配置标注来源 Bundle。
3. 用 Profile Patch 替换其中一行配置。
4. 再用命令行 Patch 覆盖同一行。
5. 比较最终配置并解释优先级。
6. 制造一个错误插件名并追踪失败位置。

### 产出

在笔记中建立：

| Row ID | 原始 Bundle | Profile Patch | Home Patch | CLI Patch | 最终配置 |
|---|---|---|---|---|---|

### 通过标准

看到任意 dump-config 行，能解释它如何映射到插件包和配置 Schema。

## Day 6：Agent、Inbox、Turn 与 Step

### 核心内容

- Agent interface 与 live registry
- Agent Inbox 和输入领取
- Turn 是完整工作单元
- Step 是一次模型请求及其工具调用
- 运行中输入、注入上下文与唤醒语义
- `agent/pre-step`
- `agent/turn-stopping`
- 空输入、拒绝和无 Step Turn

### 追踪实验

为以下四种场景分别记录事件序列：

1. 单次模型回答，无工具。
2. 模型调用一个工具后回答。
3. 工具结果触发下一 Step。
4. Turn 运行中收到新消息。

再构造：

- `agent/pre-step` 拒绝输入
- 第一次输入被改写为空
- 仅注入不唤醒的上下文

### 产出

画出包含时间、Session Event 和 Live Event 三条泳道的时序图。

### 通过标准

- 能准确说出一个 Turn 为什么继续或结束。
- 能区分持久事件和仅运行时事件。
- 能解释为什么被拒绝的首次输入仍可能形成一个持久 Turn。

## Day 7：周项目——最小 Agent 插件

完成 [实验 1](../labs/lab-01-cordis-plugin.md)。插件需要同时包含：

- Service Definition
- 默认 Provider
- Fake Provider
- typed event
- 一个模型可调用工具
- Agent 级作用域示例
- 配置 Schema
- 卸载测试
- 最小文档与配置示例

### 闭卷验收

不看笔记回答：

1. `dsh web` 如何变成 Cordis 插件树？
2. Profile、Bundle 和 Patch 各解决什么问题？
3. Service Definition、Provider、Consumer 如何解耦？
4. Effect 如何保证卸载安全？
5. Turn 与 Step 如何划分？
6. 哪些事件必须持久化，哪些只能是 live event？

有两题答不完整，先补齐再进入第 2 周。
