# notebooklm-skill

[English](README.en.md) · [繁體中文](README.zh-TW.md)

> 让 NotebookLM 负责资料导入、联网研究和来源问答，再把带引用的证据交给 Claude 完成写作。

`notebooklm-skill` 是一个面向终端和 AI Agent 的 NotebookLM 自动化工具。它可以把网页、文本和本地文件整理进 NotebookLM，执行快速或深度研究，返回结构化答案与引用，并通过 CLI、Pipeline 或 MCP 接入 Claude Code、Cursor 等客户端。

适合深度文章、行业分析、研究综述、资料型内容、RSS 摘要，以及播客、幻灯片等多格式内容生产。

> [!IMPORTANT]
> 这是基于 NotebookLM 网页接口的非官方集成，不由 Google 提供或支持。接口、登录方式、配额、生成时间和产物格式都可能变化。

> [!NOTE]
> 内置的 `research-to-article` Pipeline 会让 **NotebookLM 直接生成文章草稿**，不会调用 Claude API。若要求“NotebookLM 查资料、Claude 写终稿”，请使用本文的“证据交接”流程。

## 它能做什么

- 导入网页、YouTube 链接、原始文本和本地文件；
- 创建、查询和管理 NotebookLM 笔记本；
- 进行带来源引用的问答与摘要；
- 执行 NotebookLM 快速或深度联网研究，并导入研究结果；
- 生成文章草稿、社媒草稿、RSS 摘要和问答；
- 生成并下载音频、视频、幻灯片、报告、测验等产物；
- 通过 5 个命令行入口、5 条高层 Pipeline 和 13 个 MCP 工具供人或 Agent 调用；
- 用 JSON 输出明确报告成功、部分成功和失败，不隐藏单个来源或任务的异常。

## 工作方式

```text
网页 / PDF / 报告 / 访谈记录
              ↓
    NotebookLM 导入与联网研究
              ↓
  证据摘要 + 引用 + 冲突 + 信息缺口
              ↓
      Claude 组织结构、分析和写作
              ↓
       人工核对事实、引用和时效性
```

项目提供四个使用层：

| 使用层 | 适合场景 |
|---|---|
| Claude Code Skill | 用自然语言让 Claude 编排 NotebookLM 操作 |
| Core CLI | 手动操作、脚本和可复现自动化 |
| Pipeline | 文章、社媒、RSS 摘要和批量产物等固定流程 |
| MCP Server | 供 Claude Code、Cursor 和其他 MCP 客户端调用 |

## 环境要求

- Python 3.10 或更高版本；
- 一个可访问 NotebookLM 的 Google 账号；
- Chromium，或本机安装的 Chrome/Edge，用于首次登录；
- `uv`/`uvx` 仅在选择免安装运行方式时需要；
- `ffmpeg`、`ffprobe` 和 Poppler 仅在使用 `scripts/make_video.sh` 时需要。

本压缩包声明的版本为 `1.3.0`，项目状态为 Beta，许可证为 MIT。

## 项目来源与归属

本仓库基于 [claude-world/notebooklm-skill](https://github.com/claude-world/notebooklm-skill) `1.3.0` 整理，新增了简体中文说明、实现边界澄清和安全使用提示。原始源码及其版权归上游作者所有，并继续遵循仓库内的 MIT License。

这不是 Google、NotebookLM 或 Anthropic 的官方项目，也不应被理解为原作者对本中文整理版本的背书。

本整理仓库不自动向 PyPI 发布同名软件包；文中的 `uvx`/PyPI 命令使用的是上游发布版本。

## 安装

### 推荐：独立虚拟环境

在解压后的项目目录执行：

```bash
python3 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install .
python -m playwright install chromium
```

Windows PowerShell 激活命令为：

```powershell
.venv\Scripts\Activate.ps1
```

### 自动安装脚本

```bash
./install.sh
```

脚本会：

1. 在用户数据目录创建独立 Python 环境；
2. 安装本项目及依赖；
3. 下载 Playwright Chromium；
4. 向 `~/.local/bin` 链接 5 个命令；
5. 安装用户级 Claude Code Skill，并检查登录状态。

如命令不可用，请确认 `~/.local/bin` 已加入 `PATH`：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

可用 `NOTEBOOKLM_SKIP_BROWSER=1`、`NOTEBOOKLM_SKIP_SKILL=1` 或 `NOTEBOOKLM_SKIP_AUTH_CHECK=1` 跳过相应步骤。

### 使用 uvx 临时运行

```bash
uvx --from 'notebooklm-skill==1.3.0' notebooklm-auth setup
uvx --from 'notebooklm-skill==1.3.0' notebooklm-skill list
```

固定版本有助于避免非官方上游接口变化导致行为漂移。

## 登录 NotebookLM

```bash
notebooklm-auth setup
notebooklm-auth verify
```

使用本机 Chrome 和全新 NotebookLM 浏览器配置：

```bash
notebooklm-auth setup --browser chrome --fresh
```

命名 profile 可隔离不同账号或用途：

```bash
notebooklm-auth --profile work setup
notebooklm-auth --profile work verify
notebooklm-skill --profile work list
```

登录状态通常保存在 `~/.notebooklm/profiles/<profile>/storage_state.json`。该文件包含 Google 会话信息，应当像密码一样保护。

## 五分钟快速开始

创建一个包含网页、文件和访谈摘要的笔记本：

```bash
notebooklm-skill create \
  --title "企业 AI 助手行业研究" \
  --sources \
    https://example.com/official-report \
    https://example.com/independent-study \
  --files ./industry-report.pdf \
  --text-sources "访谈纪要：目标客户最关心部署成本与数据安全。" \
  --strict > create-result.json
```

取得稳定的 notebook ID：

```bash
NOTEBOOK_ID=$(jq -r '.notebook.id' create-result.json)
```

执行深度研究并导入结果：

```bash
notebooklm-skill research \
  --notebook "$NOTEBOOK_ID" \
  --query "近两年企业 AI 助手的市场变化、采用障碍和独立评估" \
  --mode deep \
  --max-sources 10 > research-result.json
```

提取可交给写作者的证据包：

```bash
notebooklm-skill ask \
  --notebook "$NOTEBOOK_ID" \
  --query "请输出证据表：核心结论、支持来源、反对证据、数据日期和仍待验证的问题。" \
  > evidence.json

jq '{status, answer, references}' evidence.json
```

## NotebookLM 研究，Claude 写作

将 `evidence.json` 和必要的原始资料交给 Claude，并使用类似提示：

```text
请基于 evidence.json 写一篇面向企业技术决策者的行业分析。

要求：
1. 明确区分事实、来源观点和作者推断；
2. 每个关键数字保留日期、统计口径和对应引用；
3. 对相互冲突的来源分别陈述，不强行合并；
4. 没有足够证据的内容标记为“待验证”；
5. 先给出文章提纲和证据映射，再完成正文；
6. 不得补写 evidence.json 中不存在的事实。
```

如果已在 Claude Code 中安装本 Skill，也可以直接提出：

```text
请使用 notebooklm-research 建立来源库并做深度研究。
先返回带引用的证据表、冲突观点和信息缺口，不要让 NotebookLM 写全文；
然后由你基于证据表完成文章，并逐条保留引用。
```

这一步“不要让 NotebookLM 写全文、由你完成文章”是确保 Claude 真正接手写作的关键。

## 直接生成 NotebookLM 文章草稿

若只需要一条现成流水线：

```bash
notebooklm-pipeline research-to-article \
  --sources \
    https://example.com/official \
    https://example.com/independent \
  --files ./paper.pdf \
  --title "企业 AI 助手证据回顾" \
  --language zh-CN \
  --audience "企业技术决策者" \
  --tone "准确、克制，明确区分证据与推断" \
  > article-result.json
```

这条 Pipeline 会并行询问 5 个研究问题，再请求 NotebookLM 生成文章草稿。它不会直接写出 `.md` 文件，可从 JSON 中提取：

```bash
jq -r '.article.answer' article-result.json > article.md
```

如最终作者必须是 Claude，应把 Pipeline 输出作为研究材料交给 Claude 重写。

## 其他 Pipeline

```bash
# 来源 → 社媒草稿
notebooklm-pipeline research-to-social \
  --sources https://example.com/a \
  --platform linkedin --variants 3

# RSS / Atom → 摘要与问答
notebooklm-pipeline batch-digest \
  --rss https://example.com/feed.xml \
  --max-entries 20 --qa-count 5

# 来源 → 多种 NotebookLM 产物
notebooklm-pipeline generate-all \
  --files ./paper.pdf \
  --types audio slides report mind-map \
  --output-dir ./output \
  --artifact-concurrency 2

# 趋势 → 研究 → 草稿；需要另行安装 trend-pulse
notebooklm-pipeline trend-to-content \
  --geo TW --count 5 --platform threads \
  --research-mode deep
```

Pipeline 只生成草稿和本地产物，不会自动发布到社交平台、CMS 或其他远程目的地。

## 可生成的产物

| 类型 | 默认下载格式 | 说明 |
|---|---:|---|
| `audio` | M4A | 深度讨论、简报、评论或辩论 |
| `video` | MP4 | 视频概览 |
| `cinematic` | MP4 | 电影化视频流程 |
| `slides` | PDF | 也可下载为 PPTX |
| `report` | Markdown | 简报、博客、学习指南或自定义报告 |
| `study-guide` | Markdown | 学习指南快捷方式 |
| `quiz` | JSON | 也支持 Markdown 或 HTML |
| `flashcards` | JSON | 也支持 Markdown 或 HTML |
| `mind-map` | JSON | 思维导图数据 |
| `infographic` | PNG | 信息图 |
| `data-table` | CSV | 结构化数据表 |

示例：

```bash
notebooklm-skill generate \
  --notebook "$NOTEBOOK_ID" \
  --type slides \
  --lang zh-CN \
  --output ./output/deck.pptx \
  --output-format pptx
```

长任务可用 `--no-wait` 只取得任务 ID，稍后通过 `list-artifacts` 和精确的 `artifact-id` 下载。

## Claude Code Skill

安装到用户级目录：

```bash
notebooklm-install-skill
```

仅安装到当前项目：

```bash
notebooklm-install-skill --scope project
```

目标存在且内容不同时，安装器默认拒绝覆盖；显式使用 `--force` 时会先创建带时间戳的备份。符号链接目标会被拒绝。

## MCP Server

stdio 模式适合 Claude Code、Cursor 和其他 MCP 客户端：

```json
{
  "mcpServers": {
    "notebooklm": {
      "command": "uvx",
      "args": [
        "--from",
        "notebooklm-skill==1.3.0",
        "notebooklm-mcp"
      ]
    }
  }
}
```

重启 MCP 客户端后，先调用 `nlm_list` 验证连接。自动化任务建议显式选择 profile，并使用 notebook ID，而不是可能重名的标题。

本地 HTTP 模式：

```bash
notebooklm-mcp --http --host 127.0.0.1 --port 8765
```

服务器会拒绝非 loopback 地址。不要把 HTTP 模式直接暴露到局域网或公网。

## JSON 输出与验收

成功命令把一个 JSON 文档写到 stdout，进度和诊断写到 stderr。

| `status` | 含义 |
|---|---|
| `ok` | 请求的主要工作已完成 |
| `started` | 异步任务已创建，但调用方选择不等待 |
| `partial` | 至少一个独立来源、问题或产物失败，需要复核 |
| `failed` | 未能产生主要结果 |

常见退出码：

| 退出码 | 含义 |
|---:|---|
| `0` | 完成；结果仍可能是 `partial` |
| `1` | 上游、网络、生成或其他运行错误 |
| `2` | 参数无效或对象名称有歧义 |
| `4` | 需要重新登录 |
| `130` | 用户中断 |

自动化程序不应只检查文件是否存在，至少要验证：

- 进程退出码为 0；
- 顶层 `status` 为 `ok`；
- 没有未处理的嵌套 `failed` 或 `partial`；
- `source_summary.failed` 为 0，或失败来源已经人工处理；
- 关键结论可追溯到 `references`；
- 引用与原始来源的实际表达一致；
- 时效性内容已在发布前重新确认。

## 安全与隐私

这不是一个只读提示词包，而是一个可读取本地文件、写入本地路径并修改 Google NotebookLM 账号状态的高权限自动化工具。只应在可信环境中运行。

- 不要提交、打印、复制或分享 `storage_state.json` 和 `NOTEBOOKLM_AUTH_JSON`；
- 本地文件、文本和研究查询会发送到 NotebookLM，请勿上传无权处理的敏感资料；
- 建议为自动化使用独立 Google 账号和命名 profile；
- MCP 可读取并上传运行进程有权限访问的普通文件，也可向指定本地路径下载产物；项目没有应用级 MCP 认证，只授权可信客户端和操作系统账号；
- loopback 绑定只能减少网络暴露，不等于鉴权，也不能防止同一台机器上的不可信进程访问；
- 删除笔记本必须显式使用 `--yes` 或 `confirm=true`；
- 下载默认拒绝已有普通文件和最终路径的符号链接；`--force` 只能覆盖普通文件。该检查不是文件系统沙箱，仍应使用专用输出目录；
- 创建 Notebook、上传资料、导入研究结果和启动生成任务都会在 Google 侧产生持久状态并消耗配额；失败不会自动回滚或清理；
- `generate-all` 未显式传入 `--types` 时会尝试启动 10 类产物，运行前应确认范围和配额；
- RSS 抓取会跟随重定向，但当前代码不阻止 localhost、私网或云 metadata 地址；不要把不可信 RSS URL 交给自动化流程；
- `trend-pulse` 是会被实际执行的外部程序，并继承当前进程环境。仅配置可信的绝对路径，不要让它与 `NOTEBOOKLM_AUTH_JSON` 等凭据共享环境；
- JSON 输出、错误、源 URL、本地路径、查询、回答和引用都可能包含敏感信息，日志应按敏感数据处理；
- `./install.sh` 会联网安装 Python 依赖和 Chromium，并写入用户级命令与 Skill 目录，运行前应确认这些行为符合本机策略；
- 项目没有 lockfile、哈希锁定或 SBOM；生产使用应固定本项目和依赖版本，并在隔离环境中安装；
- 重要业务、法律、医疗或财务内容必须由合适的专业人员复核。

## 已知限制

- NotebookLM 使用非官方网页 API，Google 端变化可能导致认证或功能突然失效；
- `research-to-article` 的写稿者是 NotebookLM，不是 Claude；
- `research-to-article` 先取得的 5 组研究回答不会被显式注入最终文章提示词，而是与文章结果并列返回；
- `research-to-social` 中的摘要与社媒稿是并行请求，摘要并不是社媒稿的输入；
- MCP 的 `nlm_research_pipeline` 主要拼接多次问答结果，不等同于完整的编辑与重写流程；
- “基于来源”不代表答案一定正确，模型仍可能误读、遗漏反例或生成证据不足的表述；
- 代码会保留引用元数据，但不会自动验证每条引用是否真正支持相邻主张；
- 社媒字数限制主要依赖提示词，输出仍需检查；
- 深度研究和媒体生成受 Google 端配额、时延和服务状态影响，超时后任务可能仍在后台运行；
- `research --no-wait` 可以返回任务 ID，但当前 CLI 没有与产物任务同等完整的“按任务 ID 查询并导入”恢复命令；
- `trend-to-content` 依赖额外的 `trend-pulse` 可执行程序；
- 自动测试以 mock 单元测试为主，不能代替真实 Google 登录、上传、研究和下载的集成验证；
- 项目没有社交平台、CMS 或发布 API。

## 故障排查

登录失效：

```bash
notebooklm-auth verify
notebooklm-auth setup
```

生成任务超时：

```bash
notebooklm-skill list-artifacts --notebook "$NOTEBOOK_ID"
notebooklm-skill download \
  --notebook "$NOTEBOOK_ID" \
  --type slides \
  --artifact-id ARTIFACT_ID \
  --output ./output/deck.pdf
```

缺少 Chromium：

```bash
python -m playwright install chromium
```

查看本地真实命令契约：

```bash
notebooklm-skill --help
notebooklm-pipeline --help
notebooklm-mcp --help
```

## 项目结构

```text
SKILL.md                  Claude Code Skill 定义
scripts/                  CLI、Pipeline、认证与安装器
mcp_server/               FastMCP Server 与 13 个工具
references/               API、JSON 输出和 Pipeline 契约
examples/                 文章、社媒与趋势研究示例
docs/                     安装指南
tests/                    单元测试与契约测试
```

## 开发与验证

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e '.[dev]'

ruff check .
ruff format --check .
mypy scripts mcp_server
pytest --cov --cov-report=term-missing
python -m build
twine check dist/*
```

项目 CI 声明覆盖 Python 3.10、3.12 和 3.14，并要求测试覆盖率不低于 80%。

## 许可证

MIT
