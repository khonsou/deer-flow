# AM Agent Fleet 定制分支（am-fleet/base）

> **用途声明**：本 fork 仅为 [AMAgentFleet](https://github.com/khonsou/AMAgentFleet)
> 提供**部署基线镜像来源**（compose 按 tag 构建网关镜像）。我们不向
> bytedance/deer-flow 提交任何 PR / issue / 代码贡献；本 fork 的自定义
> 内容全部在 `am-fleet/base` 分支，跟随上游演进只做 rebase，不回流。

本仓库是 [bytedance/deer-flow](https://github.com/bytedance/deer-flow) 的 fork，作为
[AMAgentFleet](https://github.com/khonsou/AMAgentFleet) 平台的执行底座（harness）。
上游代码零修改——所有平台定制以**配置**形式落在本分支，不触碰 DeerFlow 源码
（ADR-0002 spike 门验证结论，见 AMAgentFleet `docs/spikes/0001`）。

## 基线锁定

- 基线 commit：`89c9f0871432e169a67b7db21984c8d2c3486c21`（上游 main，2026-10-08）
- 验证方式：CHANGELOG.md / config.example.yaml / gateway services.py 等 6 文件
  blob 哈希与部署快照逐一比对一致
- 升级上游：rebase 本分支到新基线后重过 AMAgentFleet spike 0001/0002 回归

## 定制内容（相对上游仅此一项）

`config.fleet.yaml` —— 平台网关配置模板，部署时复制为 `config.yaml` 并经
`DEER_FLOW_HOME` / `DEER_FLOW_PROJECT_ROOT` 环境变量启动。相对
`config.example.yaml` 的改动面：

| 配置段 | 改动 | 用途 |
|--------|------|------|
| `models:` | 锁定 DeepSeek 官方端点（`api_key: $DEEPSEEK_API_KEY` 环境变量注入） | ADR-0003 模型锁定 |
| `tools:` | 新增 `agent_spawn` / `agent_message`，`group: fleet`，`use: fleet_tools:*` | ADR-0010 舰队协作 runner 工具（fleet_tools.py 由 AMAgentFleet 仓库经 PYTHONPATH 提供，不是本仓库文件） |
| `tool_groups:` | 声明 `- name: fleet` | 同上，工具组挂载 |
| `agents_api:` | `enabled: true` | ADR-0008 per-agent 配置层 |
| `scheduler:` | `enabled: true` | YAML cron ↔ scheduled-tasks 对账 |
| `run_events:` | `backend: db` | 消息持久化（默认 memory，重启即丢） |
| `skill_evolution:` | `enabled: true`（上游默认 false） | agent 经 `skill_manage` 工具在对话中沉淀持久化技能（skills/custom/，静态扫描 + moderation 模型双闸） |

## 部署

不要直接用本仓库部署—— compose 与运维手顺见 AMAgentFleet 仓库
[`deploy/`](https://github.com/khonsou/AMAgentFleet/tree/main/deploy) 目录
（`DEPLOY.md` / `docker-compose.yml` / `.env.example`）。

## License

跟随上游 MIT License（Copyright Bytedance Ltd.）。
