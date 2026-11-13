# RobotWorld × Harbor: CLI agent results

RobotWorld 的 84 个官方任务(20 个 benchmark),在 Harbor 中用三种「模型 + CLI agent」组合评测,
**pass@1**(每任务一个回合),**每回合 3 小时**。本仓库包含结果与每个槽位决定性尝试的完整轨迹。

最后更新:2026-11-13。DeepSeek 一组的结果**无效**,放在文末单独说明。

## 结果

| 组合 | 成功 | 失败 | 拒答 | infra/API error | **严格 pass@1** | 已完成任务上的 pass@1 |
|---|---|---|---|---|---|---|
| Antigravity CLI + Gemini 4 Argon | 17 | 67 | 0 | 0 | **20.2%** (17/84) | 20.2% (17/84) |
| Claude Code + Opus 5.5 | 15 | 54 | 15 | 0 | **17.9%** (15/84) | **21.7%** (15/69) |
| Claude Code + GLM-5.3-flash | 1 | 83 | 0 | 0 | **1.2%** (1/84) | 1.2% (1/84) |

每个(组合, 任务)只有四种结局:

- **成功 / 失败**:打分器给出 1 / 0;回合跑满 3 小时或上下文溢出也记失败(模型原因)。
- **拒答**:模型真正应答的每一次尝试都是上游拒答。
- **infra/API error**:没有一次尝试能排除平台故障——回合在环境自己结束之前,就被 429 限流、
  403 `Spend cap breached`(提供方项目的花费上限)、Google 组织策略的 400 或 5xx 打断。这类槽位
  一律重跑,直到出现能排除平台故障的尝试。三组目前都没有剩余的 infra/API error 槽位:Gemini 的
  10 个 robodojo 槽位在 2026-11-11 至 11-13 补跑完成,全部为失败。

**严格 pass@1** = 成功 / 84。**已完成任务上的 pass@1** = 成功 / (成功 + 失败),排除拒答与
infra/API error。

Gemini 与 Opus 只差 2 个任务,在 n=84、单次尝试下**不具统计显著性**。这两组与 GLM 之间是量级差异。

### 剔除 behavior_1k 后(74 个任务)

behavior_1k 的 10 个任务在 3 小时预算下**对任何模型都不可打分**:官方打分器只在回合正常结束时
写出 `results.json`,而没有任何一组在 3 小时内跑完(三组都记为失败)。剔除后的口径:

| 组合 | 严格 pass@1(74 任务) | 已完成任务上的 pass@1(74 任务) |
|---|---|---|
| Antigravity + Gemini 4 Argon | **23.0%** (17/74) | 23.0% (17/74) |
| Claude Code + Opus 5.5 | **20.3%** (15/74) | 25.4% (15/59) |
| Claude Code + GLM-5.3-flash | **1.4%** (1/74) | 1.4% (1/74) |

### 按 benchmark(成功数;R = 拒答,I = infra/API error)

| benchmark | 任务 | Opus 5.5 | Gemini 4 Argon | GLM-5.3-flash |
|---|---|---|---|---|
| robodojo | 15 | 0 (R15) | 1 | 0 |
| wheeledlab | 11 | **5** | 4 | 0 |
| behavior_1k | 10 | 0 | 0 | 0 |
| robocasa | 10 | 1 | **5** | 1 |
| robolab | 10 | 4 | 4 | 0 |
| bench2dex | 9 | 0 | 0 | 0 |
| ai_cps | 3 | **1** | 0 | 0 |
| omnidrones | 2 | **2** | 1 | 0 |
| volleybots | 2 | 1 | 1 | 0 |
| omniisaacgymenvs | 1 | 0 | **1** | 0 |
| reflexbench | 1 | **1** | 0 | 0 |
| 其余 9 个 benchmark | 各 1–2 | 0 | 0 | 0 |

其余 9 个 benchmark:wheel_legged(2)、digit、flamingo、go2_push、humanoid_soccer、robot_lab、
steadytray、ttrl、wheeled_quadruped。

**Opus 的 15 个拒答就是 robodojo 全部 15 个任务**:上游拒答在每次尝试中稳定复现。其中 3 个槽位
补跑到第 6 次,中间混有环境未能加载场景的故障,模型真正应答的尝试全部是拒答。

**判定 infra/API error 的依据**:只看平台错误是否早于环境结束回合。Gemini 另有 13 个回合最后一轮
也报了 429,但发生在环境已经结束回合(机器人倒地、任务终止)之后,0 分是模型自己的结果,记为失败。

每个槽位的结局、决定性尝试编号、总尝试次数见 [`slots.csv`](slots.csv)。

## 评测设置

| 参数 | Opus 5.5 | Gemini 4 Argon | GLM-5.3-flash |
|---|---|---|---|
| 模型 ID | `claude-opus-5-5[1m]` | `gemini-4-argon` | `glm-5.3-flash` |
| CLI agent | Claude Code 2.1.312 | Antigravity CLI 1.4.8 | Claude Code 2.1.312 |
| reasoning effort | `xhigh` | `high`(Argon 最高档) | `max`(GLM 最高档) |
| 单次响应输出上限 | 128,000 | 65,536 | 128,000 |
| 上下文窗口 | 1,000,000 | 1,048,576 | 1,000,000(显式设置) |

三组共同的设置:

| 参数 | 值 |
|---|---|
| 回合时长 | 10,800 s(由环境自身执行) |
| 轮数上限 | 不设;环境步数上限与官方 `step_limit` 一致 |
| pass@k | pass@1 |
| 工具 | 各 benchmark 自己的机器人工具,经 MCP 提供;CLI 自带工具全部关闭 |
| 系统提示 | 用 RobotWorld 的 `baseInstructions` 替换各 CLI 自带的提示 |
| 图像记忆 | 只保留最新观察包的图像,与官方 Codex 的 `IMAGE_WINDOW=1` 同一规则,由模型代理统一执行 |
| 重试 | 平台故障(429、5xx、无可用部署)最多重试 3,600 s/调用;模型的回答与拒答不重试 |
| 终局 | 打分、跑满 3 h、上下文溢出为终局;平台故障或整次尝试模型未应答则换下一次尝试(最多 3 次) |
| 硬件 | 74 个任务在 RTX 3080 10 GB;behavior_1k 的 10 个在 Tesla T4 15 GB(三组相同) |

reasoning effort 与输出上限各组不同,是因为各模型可用的档位与上限不同,每组都取了该模型的最高档。

## 评测过程中修复、并影响结果的问题

**Claude Code 截断了环境返回的观察结果(已修复,受影响槽位已重跑)。**
Claude Code 默认把单个 MCP 工具结果截断在 25,000 token。bench2dex 与 behavior_1k 一次观察带
15 张图,最新一帧排在最后,截断后模型看不到当前画面;同时缺了当前帧的标签,图像窗口停止裁剪,
请求越积越大,最终被上游以请求过大拒绝(Opus 有 17 个槽位因此提前失败)。

修复方式是设置 `MAX_MCP_OUTPUT_TOKENS=200000`。Claude Code 两组的 bench2dex(9)、robocasa(10)、
robodojo(15)共 34 个任务用修复后的配置重跑,**替换**了原槽位。重跑中截断 0 次、请求过大 0 次。
修复前的原始结果:Opus 16 个成功,严格 19.0%;GLM 0 个成功,0.0%。

修复后 Opus 在 bench2dex 上仍然 0/9(全部跑满 3 小时),在 robocasa 上两个任务由成功变失败、
一个由失败变成功。所以**修复没有提高 Opus 的分数**,变化在单次 pass@1 的方差范围内。
Antigravity 没有这个上限。`slots.csv` 的 `run` 列标明每个槽位来自原始运行还是修复后重跑。

## 已知局限

1. **单次 pass@1,方差大。** 修复截断前后,Opus 在 robocasa 上有两个任务由成功变失败、
   一个由失败变成功。
2. **补跑跨越了平台变化。** Gemini 的 10 个 robodojo 槽位补跑时,原 API key 已停用,改用同一模型网关的
   另一个 key(同一上游、同一模型与参数);其间多次遇到限流,决定结局的尝试均未被打断。
3. **behavior_1k 不可打分**(见上)。
4. **各组 effort 与输出上限不同**,每组都取该模型的最高档。
5. **部分回合缺少环境侧日志**(`episode/env-service.jsonl`),集中在 behavior_1k
   (因归档截断丢失)。pass@1 与结局判定不受影响:奖励由环境内的打分器直接产出,
   与日志是否保留无关。

## 轨迹

```
trajectories/<group>/<task>/
  result.json, config.json, trial.log      Harbor 的 trial 记录
  verifier/                                打分输出(reward.txt、episode-summary.json、test-stdout.txt)
  episode/env-service.jsonl.gz             环境侧:每次工具调用、结果、回合起止
  agent/rw/agent-runtime-summary.json      用量、轮数、结束状态、耗时
  agent/rw/runtime.log
  agent/rw/cli-agent/
    agent.stream.jsonl.gz                  CLI 的完整事件流:思考、工具调用、工具结果、重试
    requests.jsonl.gz                      每次模型请求:状态、延迟、token 用量、裁图数、错误
    request-NNNN.skeleton.json             抽样请求的完整结构(第 1、2、3、50、100… 次)
    bridge.jsonl.gz, agent.stderr.log
  video/                                   回合的完整录像(mp4),路径同环境输出目录
```

- 每个(组合, 任务)只收录**决定该槽位结局的那次尝试**,不收录被平台故障作废的尝试。
- **回合录像**:435 个 mp4,共 2.7 GB、3.1 小时,覆盖 252 个槽位中的 217 个(Gemini 79、Opus 64、GLM 74),
  是环境自己录制的整段回合画面(视角与路径因 benchmark 而异,如 `video/out/video/camera.mp4`、
  robolab 的 `video/out/official/<任务>/…`、behavior_1k 的 `video/videos/<任务>_….mp4`)。逐字节保留,
  已逐个解码校验;文件内嵌标签只有编码器名。缺录像的 35 个槽位:Opus 的 15 个 robodojo 拒答(模型
  没有执行回合)、Opus 5 个与 GLM 10 个 behavior_1k 槽位(环境未写出录像,或回合归档在拉取时被截断)、
  Gemini 5 个 robodojo 槽位(环境未写出录像)。
- **相机图像已替换为占位符** `<image sha256=… bytes=…>`,保留原图的摘要与大小,其余内容逐字节保留。
  替换前 20.3 GB,替换后 4.9 GB(共 138,803 张)。这里指的是发给模型的观察图像;回合录像见上一条。
- 内部文件系统路径、内网主机名、内部平台名、上游路由前缀、网关请求 ID、沙盒实例名都已替换为
  中性占位符(如 `<repo>/`、`<internal-host>`、`AgentBox`、`GPUVM`、`SandboxSDK`、`<redacted>`、`<pod>`)。
  一次 Bash 调用打印出的内部 SDK 文档已删除。
- 所有 `.jsonl` 都经过 gzip 压缩,存放在 Git LFS 中。
- 上传前已扫描所有文件(包括解压后的内容),未发现任何凭据。

---

## 附:DeepSeek Harness + DeepSeek-V4.1-Flash(结果无效)

| 组合 | 成功 | 失败 | 拒答 | 未出分(未按新口径复查) | 严格 pass@1 |
|---|---|---|---|---|---|
| DeepSeek Harness 0.2.1-rc.5 + `deepseek-v4.1-flash`(effort `max`) | 4 | 72 | 0 | 8 | ~~4.8%~~ **无效** |

**DeepSeek Harness 在整个评测中没有把任何相机图像交给模型。** dsh 在把 MCP 工具结果交给模型
之前,先把图像写入自己的附件存储,这一步对所有图像都失败了,每张图被替换为一行
`[image unavailable: ...]`。失败原因有两种:

- `Image batch exceeds the configured image-count limit.`:dsh 默认单个工具结果最多 20 张图,
  而 bench2dex 一次观察有 25 张;
- `durable image storage rejected the result`:附件存储写入失败,根因尚未查明。

抽样请求中,DeepSeek 在每个 benchmark 上收到的图像都是 0 张。它的 4 个成功
(`reflexbench-cup_ball_catching`、`robocasa-CloseDrawer`、`wheeledlab-rw-courtyard`、
`robolab-PutTwoMugsOnShelfTask`)都是只靠文本状态完成的。因此 4.8% 衡量的是「无视觉的
DeepSeek」,不能与上面三组比较。修复 dsh 的图像通道后需要重测。

使用 V4.1-Flash 而不是 V4.1-Pro,是因为 dsh 0.2.1-rc.5 的模型目录中只有 Flash 标注了图像能力。
DeepSeek 的轨迹不在本仓库中。
