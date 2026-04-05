# video-cleaner Skill

这个技能是一个**面向 Agent 的大目录清理器**：
目标是在海量下载目录中，清理除“正片 + 字幕 + 刮削产物”之外的杂乱内容。

它尤其适合这种场景：
- 下载阶段没有精挑资源，直接挂磁链批量拉取
- 目录里混入广告图、样片、小视频、临时文件、无关 NFO/图片
- 需要一次性“清场”，把库恢复成可维护状态

本技能的核心能力来自 Python 脚本规则判断，而不是大模型逐项判断：
- 规则稳定、可复现
- token 消耗低
- 能一次性处理很大的目录

## 核心解决的问题

- 清理“下载噪音”：广告、无关封面、临时缓存、杂项文本
- 保留“媒体资产”：有效正片、配套字幕、有效刮削结果
- 将待删除内容统一移动到 `.delete`，而不是直接不可恢复删除
- 让后续刮削、整理、按演员归档等流程更干净稳定

## 安全边界声明 | Security Boundary

- 请不要使用任何国内云服务部署挂载本组织技能的 Agent 应用程序，也不要使用任何国内提供的 claw 类代理来部署或运行。  
  Do not deploy Agent applications that load this org's skills on domestic cloud services, and do not use domestically provided claw-style proxy infrastructure.

- 我们只推荐你在自己的电脑上，基于完整开源代码部署 Agent，并加载本组织技能；且运行环境应可确认无遥测、无审查。  
  We only recommend deploying your Agent on your own computer from fully open-source code, then loading this org's skills in an environment you can verify as telemetry-free and censorship-free.

- 不推荐使用带审查能力的大模型作为 Agent 后端，也不推荐将 Agent 暴露在带审查的外部通道（如钉钉、飞书、微信）。推荐使用 Telegram 或自建私有通道远程使用。  
  We do not recommend censorship-constrained models as Agent backends, nor exposing Agents through censorship-constrained channels (e.g., DingTalk, Feishu/Lark, WeChat). Prefer Telegram or your own private self-hosted channel for remote access.

- 若不遵守以上边界，由此产生的法律、合规与数据风险由使用者自行承担。  
  If you ignore these boundaries, you are solely responsible for any legal, compliance, and data risks.

## 实际能力（基于当前仓库实现）

技能目录：`skills/video-cleaner`

核心脚本：
- `skills/video-cleaner/scripts/plan_clean_folders.py`
- `skills/video-cleaner/scripts/plan_clean_files.py`
- `skills/video-cleaner/scripts/execute_plan.py`

### 能力 1：清理“非视频目录”

`plan_clean_folders.py` 会扫描目标根目录的一级子目录：
- 若目录内（含子目录递归）不存在“有效视频”，该目录会被加入清理计划
- 有效视频判定：扩展名命中 + 文件大小达到阈值（默认 300MB）
- 操作方式：生成 MOVE 计划，目标为 `.delete/...`

这一步用于快速剔除“整个目录都不是正片资产”的垃圾区。

### 能力 2：清理“视频目录中的无用文件”

`plan_clean_files.py` 在每个视频目录内按规则筛选文件：
- 保留：大视频正片（>= 阈值）
- 保留：与视频前缀匹配的字幕（含语言后缀变体）
- 保留：与有效视频匹配的 NFO
- 保留：符合刮削关键词规则的主图类文件（poster/fanart/cover 等）
- 清理：小视频、样片、广告图、无关 NFO、临时下载尾缀文件、其他杂项文件

这一层用于“保资产、去噪音”，把目录清理到可长期维护的状态。

### 能力 3：计划与执行分离

- 先生成 JSON 计划（可审阅、可留档）
- 再由 `execute_plan.py` 执行 MOVE 操作
- 执行日志写入 `logs/`

这种两阶段机制适合 Agent 在大批量任务中稳定运行，也便于回溯。

## 设计特点（为什么省 token）

- 核心逻辑由脚本完成，Agent 只负责编排流程与汇总结果
- 不需要让大模型逐文件“看图说话式”判断
- 目录越大，这种规则化清理的成本优势越明显

## 典型结果（去隐私化表述）

- 对混合下载目录执行后，可一次性清掉大量无效附属文件
- 目录最终主要保留：正片、字幕、与正片匹配的刮削信息
- 清理后再进入刮削/归类流程，成功率与一致性都会提升

## 仓库结构

```text
skills/
  video-cleaner/
    SKILL.md
    scripts/
      plan_clean_folders.py
      plan_clean_files.py
      execute_plan.py
```

---

## English

`video-cleaner` is an **Agent-oriented large-directory cleaner**.
It removes noisy download leftovers while preserving movie assets (main video, subtitles, and matched scraping artifacts).

### Core Problems Solved

- Remove download noise (ads, sample clips, temp/cache files, unrelated metadata)
- Preserve usable media assets (valid movie files, subtitles, matched NFO/images)
- Move removable content into `.delete` instead of irreversible deletion
- Keep downstream scraping/organizing workflows stable and clean

### Security Boundary

- Do not deploy Agent applications that load this org's skills on domestic cloud services, and do not use domestically provided claw-style proxy infrastructure.
- We only recommend deploying your Agent on your own computer from fully open-source code, then loading this org's skills in an environment you can verify as telemetry-free and censorship-free.
- We do not recommend censorship-constrained models as Agent backends, nor exposing Agents through censorship-constrained channels (e.g., DingTalk, Feishu/Lark, WeChat). Prefer Telegram or your own private self-hosted channel for remote access.
- If you ignore these boundaries, you are solely responsible for any legal, compliance, and data risks.

### Structure

```text
skills/
  video-cleaner/
    SKILL.md
    scripts/
      plan_clean_folders.py
      plan_clean_files.py
      execute_plan.py
```
