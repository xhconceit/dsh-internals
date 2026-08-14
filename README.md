# DeepSeek Harness Internals

一个面向已有 TypeScript/Node.js 基础开发者的 DeepSeek Harness 源码精通项目。

课程周期为 30 天，每天 6 小时，总计约 180 小时。目标不是记住所有文件，而是建立完整的运行时模型，能够独立追踪、修改、测试和扩展 DeepSeek Harness 的主要子系统。

## 完成后的能力

完成全部课程和验收后，你应该能够：

- 从 `dsh` CLI 入口追踪 Web、Headless 两种启动链路。
- 解释 Cordis 的 Context、Service、Plugin、Event、Effect 与隔离作用域。
- 解释 Profile、Bundle、Patch 的组合和覆盖顺序。
- 追踪 Agent 从 Inbox 到 Turn、Step、LLM、Tool，再到 Session Log 的完整路径。
- 新增 LLM Provider、System Prompt Section 和模型配置。
- 开发带参数校验、审批、超时、取消及测试的工具插件。
- 新增可持久化、可重放、可查询、可分叉的 Session Event。
- 理解 Filesystem、Subprocess、Shell、PTY、ConPTY 和 Sandbox 的能力边界。
- 修改 React Web 客户端并正确处理流式更新和历史重放。
- 使用 Headless、ACP 和 Python SDK 驱动 Agent。
- 运行并维护单元、集成、快照、E2E、压力及性能测试。
- 构建、打包并验证 npm 与 Python 运行时产物。

## 目录

- [准备与学习方法](./00-setup.md)
- [架构地图](./01-architecture-map.md)
- [学习成果存放规范](./02-study-output-organization.md)
- [第 1 周：启动、Cordis 与 Agent 主循环](./weeks/week-01.md)
- [第 2 周：会话、工具、多 Agent 与恢复](./weeks/week-02.md)
- [第 3 周：文件、进程、终端、沙箱与安全](./weeks/week-03.md)
- [第 4 周：Web、SDK、测试、发布与毕业项目](./weeks/week-04.md)
- [实验 1：Cordis 插件](./labs/lab-01-cordis-plugin.md)
- [实验 2：生产级工具](./labs/lab-02-production-tool.md)
- [实验 3：安全执行环境](./labs/lab-03-execution-security.md)
- [实验 4：全链路功能](./labs/lab-04-capstone.md)
- [最终掌握度清单](./MASTERY-CHECKLIST.md)
- [每日笔记模板](./templates/daily-note.md)

## 30 天总表

| 天 | 主题 | 当日必须完成的结果 |
|---:|---|---|
| 1 | 固定源码、构建与运行 | Web、Headless、测试均有基线记录 |
| 2 | Monorepo 与包图 | 画出入口、Bundle 和核心包依赖图 |
| 3 | Cordis Context/Service | 写出 Provider 与 Consumer |
| 4 | Cordis Event/Effect | 验证插件卸载后副作用被撤销 |
| 5 | Profile/Bundle/Patch | 能解释 dump-config 中配置来源 |
| 6 | Agent/Turn/Step | 画出一次完整 Turn 时序图 |
| 7 | 周项目：最小 Agent 插件 | 插件能运行、卸载并通过测试 |
| 8 | Session Event 模型 | 从事件手工重建消息历史 |
| 9 | 持久化与恢复 | 验证中断后的 Replay/Resume |
| 10 | Fork/Query/Transcript | 在指定事件边界分叉会话 |
| 11 | Prompt 与 LLM Adapter | 编写 Mock Provider |
| 12 | Tool Pipeline | 实现带审批和取消的工具 |
| 13 | Goal/Job/Subagent/Scope | 跟踪父子 Agent 和作用域 |
| 14 | 周项目：会话型工具 | 工具、事件、恢复、测试全部打通 |
| 15 | Filesystem Seam | 验证工作区边界和路径规范化 |
| 16 | Subprocess | 正确处理输出、超时和进程树 |
| 17 | Shell、PTY、ConPTY | 完成交互终端和清理实验 |
| 18 | Approval 与 Sandbox | 分清审批、策略与系统隔离 |
| 19 | Landlock 与跨平台 | 形成 Linux/macOS/Windows 差异表 |
| 20 | Credentials/Settings/Telemetry | 追踪敏感数据和观测链路 |
| 21 | 周项目：安全执行器 | 通过路径、注入、残留进程测试 |
| 22 | Web Client 架构 | 从 Session Event 追踪到 React 渲染 |
| 23 | Conversation Node/Slots | 新增一种可重放 UI 节点 |
| 24 | CLI/Headless/ACP | 用无界面模式执行并取消任务 |
| 25 | Python SDK | 用 Python 完成创建、监听、恢复 |
| 26 | 测试体系 | 为一个功能补齐四层测试 |
| 27 | 构建、发布、许可证 | 打包并在干净环境安装验证 |
| 28 | 毕业项目实现 | 完成后端、事件、UI 与 SDK 链路 |
| 29 | 审计与修复 | 完成正确性、安全性和性能审计 |
| 30 | 闭卷验收 | 通过最终清单并形成后续缺口表 |

## 每天固定节奏

| 时间 | 内容 |
|---:|---|
| 45 分钟 | 阅读当天指定文档，写出术语和假设 |
| 90 分钟 | 用搜索和调试器追踪源码主路径 |
| 150 分钟 | 完成实验或功能修改 |
| 45 分钟 | 编写并运行测试，覆盖失败和取消路径 |
| 30 分钟 | 画图、复盘、更新证据索引 |

不要用“读完了”作为完成标准。当天任务只有同时具备以下证据才算完成：

1. 一张调用链、状态图或包关系图。
2. 一次可以重现的运行或调试记录。
3. 一项代码实验及相应测试。
4. 对当日验收问题的闭卷回答。

## 源码阅读规则

1. 固定一个 commit，30 天内不要跟随 `master` 漂移。
2. 第一遍只看接口、注册点、消费者和事件。
3. 第二遍用真实输入跟踪运行路径。
4. 第三遍修改行为，并用测试证明理解正确。
5. 搜索优先使用符号和事件名，例如 `turn/start`、`ctx.tools`、`SessionEventMap`。
6. 不从目录名字猜架构；所有结论必须有源码、测试或运行日志支持。

## 通过标准

- 每周实验必须完成，不允许只阅读。
- 每周至少制造并定位两个失败场景。
- 毕业项目必须同时涉及 Cordis、Agent、Session、Tool、UI 和测试。
- 最终清单中所有 P0 项必须独立完成，不能照抄教程。

## 推荐工作区结构

```text
dsh-internals/
├── README.md
├── 00-setup.md
├── 01-architecture-map.md
├── source/                 # 固定 commit 的 DeepSeek Harness 源码
├── notes/                  # 每日笔记、图和调试记录
├── experiments/            # 隔离的小实验
├── weeks/                  # 30 天课程正文
├── labs/                   # 四个综合实验规范
└── templates/              # 笔记模板
```

`source/`、`notes/` 和 `experiments/` 在实际学习时创建；课程文件不会替你生成实验答案。学习产物的命名和归档规则见[学习成果存放规范](./02-study-output-organization.md)。
