# 学习成果存放规范

所有 DeepSeek Harness 学习成果统一放在：

```text
/Users/orange/code/dsh-internals
```

## 目录结构

```text
dsh-internals/
├── README.md
├── 00-setup.md
├── 01-architecture-map.md
├── 02-study-output-organization.md
├── MASTERY-CHECKLIST.md
│
├── source/                  # 固定 commit 的官方源码
├── notes/                   # 每日学习笔记
│   ├── day-01.md
│   ├── day-02.md
│   └── ...
├── diagrams/                # 架构图、时序图和依赖图
├── experiments/             # 单一知识点的小型验证代码
│   ├── cordis-basics/
│   ├── agent-loop/
│   ├── session-replay/
│   ├── tool-pipeline/
│   └── sandbox-tests/
├── projects/                # 四个完整综合项目
│   ├── study-clock/
│   ├── project-stats/
│   ├── secure-executor/
│   └── code-review-bundle/
├── evidence/                # 测试输出、性能结果和运行证据
├── issues/                  # 尚未解决的源码问题和知识缺口
├── weeks/                   # 每周课程正文
├── labs/                    # 综合实验要求
└── templates/               # 笔记模板
```

## 各目录用途

### `source/`

只存放固定 commit 的 DeepSeek Harness 官方源码。尽量保持官方源码原样；研究性修改使用独立 Git 分支，不把个人笔记混入源码目录。

### `notes/`

每天建立一个文件：

```text
notes/day-01.md
notes/day-02.md
...
notes/day-30.md
```

每日笔记从 `templates/daily-note.md` 复制，记录概念、源码证据、调用链、实验、失败场景和未解决问题。

### `diagrams/`

存放：

- 包依赖图
- Cordis 插件树
- Agent Turn 时序图
- Session Event 状态图
- Tool Pipeline
- Filesystem/Subprocess/Sandbox 能力图
- Web 实时与 Replay 数据流

推荐同时保存可编辑源文件和导出的图片。

### `experiments/`

用于验证单一假设。每个实验应该足够小，包含自己的 README、运行方法和结论。临时实验不能直接变成毕业项目代码。

### `projects/`

存放四个综合项目：

1. `study-clock`：Cordis 服务、Provider、Consumer 和生命周期。
2. `project-stats`：Tool Pipeline、Filesystem 和 Session Event。
3. `secure-executor`：Subprocess、PTY、Approval 和 Sandbox。
4. `code-review-bundle`：毕业全链路项目。

### `evidence/`

保存用于证明掌握程度的材料：

- 测试摘要
- 性能基线
- 事件序列
- 调试记录
- 打包验证结果
- 跨平台验证矩阵

不要保存 API Key、完整环境变量或敏感文件内容。

### `issues/`

每个未解决问题单独记录，至少包含：

- 问题描述
- 当前假设
- 已查阅的源码位置
- 已完成的实验
- 仍缺少的证据
- 下一步验证方式

问题解决后保留原文件，并增加结论和解决日期，不要直接删除。

## 命名规则

- 每日笔记：`day-XX.md`
- 架构图：`day-XX-topic.mmd` 或 `day-XX-topic.png`
- 实验证据：`day-XX-topic.md`
- 未解决问题：`issue-XXX-short-title.md`
- 性能记录：`YYYY-MM-DD-topic.md`

## 每日归档流程

每天学习结束前完成：

1. 更新 `notes/day-XX.md`。
2. 将图放入 `diagrams/`。
3. 将实验代码提交到 `experiments/` 或对应 `projects/`。
4. 将测试与运行证据放入 `evidence/`。
5. 将未解决问题写入 `issues/`。
6. 更新 `MASTERY-CHECKLIST.md`，只把有证据的项目标记为完成。

## 核心原则

- 官方源码与个人成果分离。
- 笔记中的结论必须附源码或运行证据。
- 实验与完整项目分离。
- 敏感数据永不进入学习仓库。
- “看过”不等于“掌握”；只有通过测试或实验验证后才更新验收清单。
