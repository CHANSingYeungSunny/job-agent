# E2E 02 — Scratch → Fleet Member · 接手进度总结

> 给下一个 CodeBuddy 的上下文。读完即可继续，无需翻历史。
> 最后更新：2026-08-15

## 0. 一句话状态

`job-agent` 已通过 self-service 注册，平台侧的 `members.yaml` PR 已被打开（registrar 返回 `pr_opened`，PR #31 on `turingplanet/agent-registry`）。**当前唯一未完成项是平台 admin 合并那个 PR** —— 合并 = 正式入 fleet。其余 E2E 02 步骤（scaffold / 本地测试 / gate / register / 失败排查）全部已完成。

## 1. 仓库事实

| 项 | 值 |
|----|----|
| Agent 仓库 | `https://github.com/CHANSingYeungSunny/job-agent` |
| 可见性 | **public**（曾为 private，是 register 首次失败的根因） |
| 默认分支 | `main` |
| 本地路径 | `c:\Users\Asus\Desktop\job-agent\` |
| 平台 registry | `turingplanet/agent-registry`（私有，你无直接 gh 读取权限） |
| 平台 App | `fleet-migration-bot`（已安装到你的账户，Repository access 覆盖 job-agent） |
| SLUG / repo 名 | `job-agent` |

## 2. E2E 02 各阶段进度

| Stage | 内容 | 状态 | 备注 |
|-------|------|------|------|
| 1 Scaffold | copier 生成 agent + 本地测试 | ✅ | 模板 v0.0.19+ |
| 2 Declare intent | `fleet.register: true` 写入 manifest | ✅ | `agent.manifest.yaml` 的 `fleet:` 段 |
| 3 Publish | 首次 push main 自动触发 register.yml | ✅ | |
| 4 Verdict | 读 registrar 结果 | ✅ | 见下 §3 排错史 |
| 5 Admission | admin 合并 members.yaml PR | ⏳ **待 admin 合并（PR #31）** | 这是平台侧动作，本地无法做 |
| 6 Ship code | 开 PR，gate 自动跑 | ⬜ 未做 | 计划 step 8 |
| 7 AI review | PR 评论 `/review` 触发平台付费 review | ⬜ 未做 | 计划 step 9，quota 默认 2/周 |
| 8 Stay current | 模板更新以 PR 形式到达 | 架构已就绪 | 入 fleet 后 bot 会自动开 sync PR |
| 9 Hosting (optional) | `job-agent.agents.turingplanet.ai` | ⬜ 未做 | 需 admin 在 deployments.yaml 加条目 |
| 10 Teardown | `scripts/teardown.sh` | 可用 | v0.0.21+ scaffold 自带 |

## 3. 关键排错史（为什么现在卡在 admin 合并）

1. **第一次 register 失败**：`repository '.../job-agent/' not found` / `git exit 128`。
   - 原因：仓库是 **private**，平台 `fleet-migration-bot` App 没权限读。
   - 解决：把仓库设为 **public**（`gh repo edit ... --visibility public --accept-visibility-change-consequences`）。
2. **第二次 register 仍失败（逻辑上）**：registrar 返回 `app_not_installed`，install_url `https://github.com/apps/fleet-migration-bot`。
   - 原因：App 根本没装到账户。公开仓库只解决"读权限"，不解决"App 安装"。
   - 解决：用户在网页安装了 `fleet-migration-bot` App（Repository access = All repositories / 覆盖 job-agent）。
3. **第三次 register 成功**：空提交重新触发 → run `31858358678` → job `94947141381` 日志：
   ```
   registrar says: {"status":"pr_opened","pr":"https://github.com/turingplanet/agent-registry/pull/31"}
   ```
   - 注：`agent-registry` 是私有仓库，PR #31 公开网页 404，但 registrar 明确返回了链接 = PR 已创建。

## 4. 本地门禁（已全绿，入 fleet 前已验证）

- `poetry run pytest` → **4 passed**
- `ruff check` → 全部通过
- `bandit` → 无问题
- `poetry.lock` 已生成并提交（104KB，确保 builder 识别 Poetry 项目）

## 5. 文件布局（job-agent/）

```
agent.manifest.yaml   # fleet.register: true 已声明
pyproject.toml / poetry.lock
api/  mcp_server/  tests/  scripts/  config.py  railpack.json  README.md
docs/research/job-agent-research-report.md   # 2026-08-15 从 "work agent" 目录移入
```

> 注：`docs/research/` 是本次（接手总结时）从 `c:\Users\Asus\Desktop\work agent\docs\` 移过来的调研报告，与 E2E 流程无关，仅作背景资料。

## 6. 下一步该做什么（按 E2E 02 顺序）

**A. 等 / 催 admin 合并 PR #31**（stage 5）
- 你无法用命令合并——只有 admin merge 才能让 job-agent 成为 fleet member。
- 可 nudge：在 job-agent 任意 PR 评论 `/register`（registrar 会确认状态，若已合并则跳过）。
- 合并后 `job-agent` 写入 `agent-registry` 的 `members.yaml`。

**B. stage 6 — Ship code（入 fleet 后做）**
```bash
cd c:\Users\Asus\Desktop\job-agent
git checkout -b test-pr
echo "# t" >> README.md
git add README.md && git commit -m "test"
git push -u origin test-pr
gh pr create --title "e2e" --body "x" --head test-pr --base main
```
- gate 会自动跑（~1 min 变绿）。

**C. stage 7 — AI review**
```bash
gh pr comment 1 --repo CHANSingYeungSunny/job-agent --body "/review"
sleep 90
gh pr view 1 --repo CHANSingYeungSunny/job-agent --comments | tail -25
```
- 期望：真实安全 review + quota footer（默认 2/周）。

**D. （可选）stage 9 — 平台 hosting**
- 入 fleet 后，让 admin 在 `agent-registry/deployments.yaml` 加：
  ```yaml
  - slug: job-agent
    repo: CHANSingYeungSunny/job-agent
    host: platform
  ```
- 验证：`bash scripts/test_platform_mcp.sh`（模板 ≥ v0.0.22 才有）。

**E. （可选）stage 8 — 保持最新**
- 被动：bot 开 `chore/template-sync-vX.Y.Z` PR，你 merge。
- 主动：`git switch -c chore/manual-sync && copier update --defaults --trust --conflict inline` → PR。

## 7. 工具链备注

- `gh` 不在 PATH，完整路径：`C:\tools\gh\bin\gh.exe`
- 涉及 `turingplanet/agent-registry` 的命令（如 `gh pr view 31 --repo turingplanet/agent-registry`）会失败——该仓库私有且你无读取权限，只能靠 registrar 返回链接或 admin 侧确认。
- job-agent 是你自己的公开仓库，所有 `gh` 操作（run view / pr / review）都能正常访问。

## 8. 已知风险 / 待确认

- `register.yml` **没有 `workflow_dispatch` 触发器**——只能靠 push 触发，没法 `gh workflow run`。每次要重跑 register 就得做一次空提交 `git commit --allow-empty`。
- 当前是否已被 admin 合并 PR #31？本总结撰写时**未知**，下一个 CodeBuddy 第一步应先用 `/register` 或询问 admin 确认 admission 状态，再决定从 stage 6 继续还是等待。
