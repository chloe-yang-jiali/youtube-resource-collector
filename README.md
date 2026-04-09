# YouTube Resource Collector

> Claude Code Skill — 一键搜集产品/关键词在 YouTube 上的曝光视频数据，自动写入飞书多维表格并生成分析报告。

## 它解决什么问题？

做出海产品的增长同学经常需要回答这些问题：

- 竞品在 YouTube 上被哪些博主报道过？
- 哪些 KOL/KOC 的内容与我们产品相关？
- 近期有哪些视频在讨论我们所在赛道？

手动一页页翻 YouTube 搜索结果、逐个记录视频信息，耗时且容易遗漏。**YouTube Resource Collector** 让你用一句话完成整条调研链路。

## 核心功能

- **关键词搜索** — 支持任意产品名或关键词，自动构建 YouTube 搜索 URL
- **时间范围筛选** — 支持 `after:YYYY-MM-DD before:YYYY-MM-DD` 精确限定时间窗口
- **完整数据提取** — 视频URL、标题（原文+中文翻译）、发布日期、观看量、评论数、频道名称、频道粉丝数、所属国家
- **飞书多维表格自动写入** — 数据结构化存储到飞书 Base，方便团队协作
- **Markdown 曝光分析报告** — 自动生成包含曝光概览、主题规律、推荐频道和原始数据链接的分析文档
- **KOL 资源发现** — 按粉丝数和内容质量筛选值得关注的频道

## 使用前提

| 依赖 | 说明 |
|------|------|
| **Claude Code** | Anthropic 官方 CLI 工具 ([安装指南](https://docs.anthropic.com/en/docs/claude-code)) |
| **CDP 浏览器** | 需要通过 Chrome DevTools Protocol 连接本地浏览器（通常由 Claude Code 的 web-access skill 管理） |
| **lark-cli** | 飞书多维表格读写依赖 lark-cli ([安装指南](https://github.com/nicepkg/lark-cli)) |
| **飞书账号** | 需要配置好飞书应用权限（多维表格读写） |

## 安装

### 方式一：直接安装到 Claude Code Skills 目录

```bash
# 克隆到 Claude Code skills 目录
git clone https://github.com/chloe-yang-jiali/youtube-resource-collector.git ~/.claude/skills/youtube-resource-collector
```

### 方式二：手动安装

将 `SKILL.md` 文件放入以下任一路径即可被 Claude Code 识别：

```
~/.claude/skills/youtube-resource-collector/SKILL.md
```

## 使用方法

在 Claude Code 中直接用自然语言触发，无需记忆命令：

**基础用法（仅搜集数据）：**

```
帮我搜集 YouTube 上 higgsfield 的曝光视频，时间范围 2025年8月
```

**指定飞书表格（写入已有表格）：**

```
搜集 Pollo AI 在 YouTube 上的曝光视频，2026年3月，飞书表格: https://my.feishu.cn/base/xxx?table=yyy
```

**触发关键词：**

以下表述都可以触发本 skill：

- `搜集 YouTube 曝光`
- `YouTube 资源调研`
- `油管产品曝光`
- `youtube exposure research`
- `油管 KOL 调研`

## 输出示例

### 1. 飞书多维表格

| 视频URL | 标题原文 | 中文标题 | 发布日期 | 观看量 | 评论数 | 频道名称 | 频道粉丝数 | 所属国家 |
|---------|---------|---------|---------|--------|--------|---------|-----------|---------|
| https://youtube.com/watch?v=xxx | Higgsfield AI Review | Higgsfield AI 评测 | 2025-08-15 | 13661 | 0 | TechReviewer | 250K | US |
| https://youtube.com/watch?v=yyy | Best AI Video Tools 2025 | 2025最佳AI视频工具 | 2025-08-22 | 227000 | 0 | AI Tools Hub | 700K | US |

### 2. Markdown 分析报告

自动生成 `[关键词]-youtube-exposure-analysis.md`，包含：

- **曝光概览** — 时间范围、关键词、视频总数
- **曝光主题规律分析** — 从标题中提炼内容趋势
- **推荐关注频道** — 按粉丝数和内容质量筛选的 KOL 列表
- **数据来源** — 所有视频原始链接汇总

## 执行流程

```
用户输入关键词和时间范围
        │
        ▼
  构建搜索 URL
        │
        ▼
  CDP 打开 YouTube 搜索页
        │
        ▼
  滚动加载 + 提取视频列表
        │
        ▼
  日期转换 / 观看量标准化
        │
        ▼
  采样获取频道粉丝数（TOP 5-10）
        │
        ▼
  推断频道所属国家
        │
        ▼
  翻译标题为中文
        │
        ▼
  ┌─────────────┐
  │ 写入飞书表格  │
  │ 生成 MD 报告  │
  └─────────────┘
```

## 已知限制

| 限制 | 说明 |
|------|------|
| 评论数 | YouTube 搜索结果页不显示评论数，默认填 0。如需准确数据，仅对 TOP 3 视频补充获取 |
| 联系方式 | YouTube 频道不公开联系方式，填 "N/A"。需通过频道简介或第三方平台获取 |
| 搜索结果数量 | 每次搜索最多获取约 20-30 个视频 |
| 日期精度 | YouTube 的 `after/before` 参数偶有不稳定，以页面显示的相对时间做二次校准 |
| 反爬限制 | 自动控制请求节奏（2-3秒间隔），触发风控时自动等待重试 |

## 适用场景

- **竞品调研** — 了解竞品在 YouTube 上的曝光分布和内容策略
- **KOL 发现** — 找到正在报道你所在赛道的博主，评估合作价值
- **内容选题** — 分析热门视频的主题规律，指导自己的内容策略
- **市场感知** — 追踪特定时间窗口内的产品讨论热度变化

## 文件结构

```
youtube-resource-collector/
├── SKILL.md       # Skill 定义文件（Claude Code 读取）
└── README.md      # 本文件
```

## License

MIT
