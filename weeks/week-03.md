# 第 3 周：文件、进程、终端、沙箱与安全

目标：理解 Agent 执行外部操作时的真实安全边界，并能验证跨平台资源清理。

本周所有攻击性测试必须在专用临时目录中进行，不要把个人目录或真实项目作为目标。

## Day 15：Filesystem Capability Seam

### 核心内容

- Filesystem Service Definition
- local/remote Provider
- Workspace root
- path normalization
- real path 与 lexical path
- symlink
- 权限策略事件
- 文件工具 Consumer
- 文件系统和执行世界一致性

### 实验

建立：

```text
workspace/
├── allowed/a.txt
├── nested/b.txt
└── link-outside -> outside/
outside/
└── secret-marker.txt
```

验证读取、写入、创建、重命名、遍历和符号链接行为。覆盖：

- `.`、`..`
- 绝对路径
- Unicode 文件名
- 不存在路径
- 指向 Workspace 外的符号链接
- 检查后再替换符号链接的竞态风险

### 通过标准

- 能区分字符串前缀检查、规范化路径检查和真实路径检查。
- 能指出策略检查与实际文件操作之间的竞态窗口。
- 能追踪一个文件工具到底层 Provider。

## Day 16：Subprocess 与进程生命周期

### 核心内容

- Subprocess Service Definition
- argv 与 shell string
- cwd、env
- stdin/stdout/stderr
- streaming 与 buffering
- exit code、signal
- AbortSignal
- timeout
- process group/tree
- orphan/zombie
- local 与 remote Provider

### 实验

编写无害测试程序并覆盖：

1. 正常退出并分别输出 stdout/stderr。
2. 非零退出。
3. 永不退出，使用 timeout 终止。
4. 父进程再创建子进程，取消后检查进程树。
5. 输出超过限制。
6. 忽略或延迟处理终止信号。
7. cwd 不存在或无权限。

### 通过标准

- 每次测试都记录 PID、退出原因和资源清理证据。
- 能解释为什么只杀父 PID 可能留下子进程。
- 能区分执行错误、非零退出和取消。

## Day 17：Shell、PTY 与 ConPTY

### 核心内容

- Shell Backend 与 Subprocess 的关系
- argv 执行与 shell 解析
- quoting 和 escaping
- PTY 与 pipes 的差异
- TTY 检测、ANSI、行缓冲
- terminal resize
- persistent terminal
- Windows ConPTY
- Bash、zsh、PowerShell 语义差异
- `node-pty` native boundary

### 实验

1. 同一命令分别用普通子进程和 PTY 运行，比较输出。
2. 运行等待输入的交互程序并写入 stdin。
3. 改变终端尺寸，观察布局变化。
4. 运行彩色输出并分析 ANSI 序列。
5. 启动长驻终端、断开客户端、重新连接。
6. 取消终端并确认 shell 与子进程全部退出。
7. 为 POSIX 与 PowerShell 分别写包含空格和特殊字符的参数测试。

### 通过标准

- 不把字符串拼接当作安全 argv 构造。
- 能解释 PTY 为什么改变程序行为。
- 能指出哪些测试只能在 Windows 真机或 CI 验证。

## Day 18：Approval、Policy 与 Sandbox

### 核心内容

- 模型工具调用不是授权
- Tool Policy
- 人工 Approval
- Filesystem Policy
- Sandbox Provider
- 命令包装
- allow/deny 的决策对象
- 审批缓存和作用域
- UI 请求与运行时执行之间的绑定

### 威胁模型

分别考虑：

- 模型误操作
- 恶意仓库提示注入
- 工具参数注入
- 已批准命令被替换
- 路径或符号链接发生变化
- 子进程创建未批准的后代
- 环境变量和凭据泄漏

### 实验

记录一次需要审批的操作，从 Tool Call 到 UI、用户决定、运行时执行和 Session Event 的全过程。尝试改变参数，确认旧审批是否会错误复用。

### 通过标准

能够清楚说明：

- Approval 回答“用户是否同意”。
- Policy 回答“系统规则是否允许”。
- Sandbox 回答“操作系统最终能否限制”。

三者不能互相替代。

## Day 19：Landlock 与跨平台隔离

### 核心内容

- Linux Landlock 基本模型
- Ruleset 与受限文件系统访问
- 继承和子进程
- native launcher
- kernel/ABI 支持检查
- fail-open 与 fail-closed
- macOS/Windows 替代边界
- 平台不支持时的降级策略

### 源码追踪

- `native/landlock-run`
- Sandbox Consumer 如何包装命令
- native 包如何构建和分发
- 平台检测
- 失败和错误消息
- 相应测试及 CI 平台

### 实验

在支持的 Linux 环境验证允许和拒绝路径；若当前不是 Linux：

- 阅读 native 实现与测试
- 建立容器或 CI 验证方案
- 不伪造本地成功结果
- 明确列出仍需真机验证的行为

### 通过标准

形成 Linux、macOS、Windows 能力矩阵，包括执行、文件隔离、PTY、信号和分发差异。

## Day 20：Credentials、Settings 与 Telemetry

### 核心内容

- Model Settings
- Credential Store
- API Key 的输入、保存和读取
- 配置与秘密分离
- 日志和错误脱敏
- Telemetry 事件来源
- Session 信息与遥测信息的边界
- Opt-in/Opt-out 和隐私

### 实验

用假 Key 追踪：

```text
Web settings
→ transport
→ credential service
→ storage
→ model request
```

检查假 Key 是否出现在：

- dump-config
- 日志
- Session Event
- 错误堆栈
- 测试快照
- 前端状态持久化

增加一个测试，防止敏感字符串进入日志或事件。

### 通过标准

- 能说明 Credential、普通 Setting 和 Session State 为什么需要不同存储策略。
- 能指出 Telemetry 的 Provider、事件源和关闭方式。

## Day 21：周项目——安全执行器

完成 [实验 3](../labs/lab-03-execution-security.md)。实现一个受控命令执行工具：

- 只接受结构化 argv
- cwd 限制在 Workspace
- 环境变量白名单
- 超时和取消
- 输出上限
- 进程树清理
- 审批摘要与实际参数绑定
- 可选 Sandbox 包装
- Session Event 记录结果但不泄漏秘密

### 必须通过的攻击测试

- Shell metacharacter 不被意外解析
- `../` 和绝对路径越界失败
- symlink 越界失败或按文档策略处理
- 取消后没有残留子进程
- 假 API Key 不出现在日志
- 巨量输出被安全截断
- 不支持 Sandbox 的平台明确提示，而非假装隔离

### 闭卷验收

解释 Filesystem、Subprocess、Shell、Terminal、Approval、Policy 和 Sandbox 各自负责什么，以及它们如何指向同一执行世界。
