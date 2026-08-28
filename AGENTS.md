# OpenCode 通用任务闭环规范 (DSL)
> 适用所有项目；强制结构化执行，防止 Agent 自动放飞。

## 核心约束（强制遵循）
所有任务必须输出 `#TASK_CYCLE` YAML‑DSL 闭环结构，严格执行：
`定目标 → 定方案 → 方案论证 → 人工闸门审批 → 分步执行 → 交付物 → 核验证据 → 问题复盘 → 修正迭代 → 终止判断`

- 禁止自由流式输出命令直接执行。
- 高危操作标记 `RISK_LEVEL: HIGH`，触发人工阻塞。
- 核验环节必须给出可运行只读校验命令，不可口头宣告任务成功。
- 迭代轮次不得超过 `meta.max_cycle` 上限（默认5轮）。
- 用户随时拥有最高叫停权限。

## DSL 顶层结构（精简）
```yaml
#TASK_CYCLE:
  meta: { task_id, cycle_index, max_cycle:5, risk_overall, status }
  goal:          # 定目标（可衡量验收标准）
  plan:          # 定方案（分步任务清单，含 risk_level）
  plan_argument: # 方案论证 + 风险分析
  gate_review:   # 人工审批闸门（HITL 阻塞点）
  execution:     # 追过程（分步执行，进度上报）
  deliverable:   # 交付物清单
  verify:        # 核验证据（只读校验命令）
  retrospect:    # 问题复盘
  revise_plan:   # 修正迭代方案
  terminate:     # 终止条件判定
```

## 安全前置约定（全局）
- 所有 `rmdir /s /q` / `rd /s /q` / `del /s /q` / `Remove-Item -Recurse -Force` 标记为 `RISK_LEVEL: HIGH`
- HIGH 风险指令**禁止自动下发 Executor**；仅输出脚本文本交由用户手动执行
- 文件操作必须携带解析后绝对路径，禁止纯相对路径；SafetyGuard 校验路径逃逸

## 使用方式
1. Agent 收到任务时，必须先输出 `#TASK_CYCLE` YAML 结构，等待 `gate_review` 审批。
2. 用户审批 `ACCEPT` 后，Agent 按 `plan.steps` 分步执行，每步写入 `execution.run_log`。
3. 完成后执行 `verify.check_commands`，提供核验证据。
4. 如核验失败，进入 `revise_plan`，循环回 `gate_review`。

## 终止条件（三重）
1. **成功终止**：所有验收标准核验通过（`verify_result: PASS`）
2. **迭代上限**：`cycle_index >= max_cycle`（默认5）
3. **用户终止**：用户发送 `stop` 指令（最高优先级）

---
*本规范为 OpenCode 全局约束，适用于所有项目。项目可在 `.opencode/AGENTS.md` 中补充特定规则。*
