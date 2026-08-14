# 实验 3：安全命令执行器

## 目标

实现一个教学用的受控命令工具，验证 Filesystem、Subprocess、Shell、Terminal、Approval、Policy 和 Sandbox 的关系。

## API 设计

只接受结构化参数：

```text
executable
args[]
cwd
allowedEnv[]
timeoutMs
maxOutputBytes
interactive
```

不要把完整命令拼成字符串交给 Shell，除非实验明确研究 Shell 模式，并且单独标注风险。

## 功能要求

- executable allowlist 或明确审批
- cwd 限制在 Workspace
- 环境变量白名单
- stdout/stderr 分离
- 超时和用户取消
- 进程树终止
- 输出上限
- PTY 可选模式
- Sandbox Provider 可用时执行包装
- 不支持隔离时给出明确状态
- Session Event 记录结果摘要

## 威胁测试

使用无害标记文件验证：

- shell metacharacter 注入
- 参数中的空格、引号、换行和 Unicode
- cwd 路径穿越
- symlink 指向 Workspace 外
- 环境变量泄漏
- 父进程派生子进程后取消
- 进程忽略终止信号
- 无限输出
- 审批后参数被修改
- Sandbox 不可用时是否 fail-open

## 跨平台矩阵

记录以下能力在 Linux、macOS、Windows 的验证状态：

- 普通 subprocess
- process tree cancellation
- PTY/ConPTY
- shell quoting
- environment handling
- filesystem isolation
- native package installation

无法在当前平台测试的项目必须标记“未验证”，不能标记“通过”。

## 完成标准

- 无测试残留进程。
- 无测试访问真实敏感数据。
- 审批展示参数与实际执行参数一致。
- 日志和事件中不存在假密钥标记。
- 取消、超时和非零退出有不同状态。
- 安全文档明确列出非目标和平台限制。
