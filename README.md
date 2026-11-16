# 本科毕设协作启动包

一个自包含的多 Agent 协作基础设施：让 Codex、TRAE、WorkBuddy、DeepSeek Harness 在同一项目目录内共享任务状态、接续上下文，并留下可复核的决策与运行证据。

## 它解决什么

多个 AI 工具轮流做同一个毕设项目时，常见问题是：上下文丢失、结论被重复推翻、实验结果无法复现、交接靠人肉总结。本包用一套轻量协议和本地程序替代这些口头约定：

- 每轮开工先同步当前任务，只读增量，不遍历全部历史
- 重要进展、否决理由、用户裁决当轮发布，写入按会话隔离的原始事件流
- 公共视图（日志、交接卡）由程序生成，禁止多会话并发改写
- 实验启动前自动采集命令、代码与输入快照，退出码不当作结论
- 无进展时提交 noop 收据，不制造空事件

## 结构

| 路径 | 作用 |
|---|---|
| `AGENTS.md` | Agent 日常入口 |
| `START_HERE.md` | 人类入口与使用步骤 |
| `shared/protocol.md` | 协作协议（同步/确认/发布/交接） |
| `shared/QUICKSTART.md` | 命令行速查 |
| `shared/BASELINE.md` | 初始基线与未确认项 |
| `tools/coop/` | 核心程序（纯标准库，约 70 KB） |
| `tests/` | 51 项测试：核心 28 + hook 15 + 集成 8 |
| `repro/` | 一次完整验收的命令、代码快照与原始日志 |
| `outcomes/candidates/` | 待用户裁决的成果候选 |
| `handoff/` | 程序生成的交接卡 |
| `.codex` `.trae` `.codebuddy` `.dsh` | 四个工具的接入配置 |

## 快速开始

    python tools/coop/coop.py status
    python tools/coop/coop.py session --tool codex --task T-START
    python tools/coop/coop.py sync --tool codex --session <SESSION> --task T-START --mode auto
    python tools/coop/coop.py doctor

在任一工具中打开项目目录并新建会话，发送：

    读取 AGENTS.md，接上毕设当前任务，并按我的下一条要求开始工作。

## 常用命令

    # 查看状态与自检
    python tools/coop/coop.py status
    python tools/coop/coop.py doctor

    # 建立或复用会话身份
    python tools/coop/coop.py session --tool codex --task T-START

    # 同步当前任务（普通增量 / 上下文恢复）
    python tools/coop/coop.py sync --tool codex --session SESSION --task TASK --mode auto
    python tools/coop/coop.py sync --tool codex --session SESSION --task TASK --mode recover

    # 确认已收到的同步包
    python tools/coop/coop.py ack --tool codex --session SESSION --packet PACKET

    # 发布本轮进展 / 无进展收据
    python tools/coop/coop.py publish --tool codex --session SESSION --task TASK --turn TURN --input .coop/publish.json
    python tools/coop/coop.py receipt --tool codex --session SESSION --task TASK --turn TURN --noop

    # 运行留证、成果候选、用户裁决、备份
    python tools/coop/coop.py run --help
    python tools/coop/coop.py candidate --help
    python tools/coop/coop.py decide --help
    python tools/coop/coop.py backup --help

## 设计要点

- **会话隔离**：不同工具、不同会话使用独立游标，不共享读写位置
- **增量优先**：默认只给新增事件与变化的卡片，长历史给出计数与索引
- **预算可控**：恢复约 2000、普通增量约 500 tokens 的保守估算上限
- **证据先行**：`run` 在执行前保存真实命令、工作目录、代码与显式输入
- **裁决留痕**：决策状态（提议/采纳/否决/替代）与证据状态（未验证/支持/失败/不确定）分离
- **零依赖**：仅使用 Python 标准库，Windows 下可直接运行

## 当前状态

- 51 项本地机械测试通过（52.329 秒，退出码 0）
- Codex / TRAE 已部署项目 hook；WorkBuddy / DSH 采用规则入口
- 四个客户端的真实会话加载与跨工具接续尚未验收
- 仅包含协作设施，未包含论文、研究数据或实验结论

## 边界与限制

- 恢复水位表示当前视图已交付，不等于逐条读过全部历史，重要动作前仍需沿指针核实
- `run` 只采集显式声明文件，不隔离并发修改，也不保存全部外部依赖
- 程序无法独立验证“用户采纳”是否来自真实授权，参数不可代替裁决
- 代码编辑仍受用户指令约束，未实现拦截所有外部编辑器的强制锁
- 原始事件流出现不完整末行时会显式报错并保留文件，修复前应先备份

## 许可

如无特别说明，本仓库仅用于个人本科毕业设计协作实践。
