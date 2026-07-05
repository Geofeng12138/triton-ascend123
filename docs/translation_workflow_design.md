# 自动文档翻译工作流设计说明

## 概述

本文档详细说明了 `schedule_doc_translate.yaml` 工作流及 `translate_md.py` 翻译脚本的设计思路、实现原理和操作指南。

该工作流的目的是：**自动将 `docs/zh/` 目录下的中文 Markdown 文档翻译成英文，并输出到对应的 `docs/en/` 目录**。通过 GitHub Actions 定时触发（每 3 天一次），或手动触发。

---

## 一、原始工作流存在的问题

原始文件 `.github/workflows/schedule_doc_translate.yaml` 存在以下 **6 个关键问题**：

### 1.1 翻译方向错误

原始脚本的 Prompt 写的是 `"from English to Chinese"`（英译中），但需求是**中译英**。

### 1.2 工作流形式与实际需求不匹配

原始流程基于 **Sphinx gettext + PO 文件** 的翻译流程：

```
zh .rst 文件 → sphinx-build -b gettext → .pot 文件 → sphinx-intl → .po 文件 → 翻译 .po → 生成英文文档
```

但实际需求只是：**翻译 Markdown 文件**（`.md` → `.md`）。

### 1.3 脚本只支持 PO 文件格式

`po_translate.py` 第 81 行硬编码检查 `path.suffix != ".po"`，无法处理 `.md` 文件。

### 1.4 权限不足

`permissions: contents: read` 只有只读权限，但工作流需要创建分支、推送代码、创建 PR。

### 1.5 PR 目标仓库写死

第 181 行 PR 创建到 `triton-project/triton-ascend` 仓库，与当前仓库 `Geofeng12138/triton-ascend123` 不一致。

### 1.6 分支引用可能出错

`ref: master` 可能不存在（默认分支可能是 `main`）。

---

## 二、修改方案

### 2.1 新建翻译脚本 `docs/scripts/translate_md.py`

**为什么新建而不是修改 `po_translate.py`？**

因为 `po_translate.py` 是面向 PO 文件格式的，其核心逻辑（按 PO entry 拆分、处理 msgid/msgstr、移除 `#, fuzzy` 标记等）与 Markdown 翻译完全无关。新建一个清洁的脚本更容易维护。

**设计要点：**

| 设计 | 说明 |
|------|------|
| 路径映射 | `docs/zh/foo/bar.md` 自动映射到 `docs/en/foo/bar.md` |
| API 调用 | 使用 DeepSeek API（`deepseek-chat` 模型），每文件一次调用 |
| 并行控制 | `asyncio.Semaphore` 控制并发数（默认 5），避免 API 限流 |
| 容错处理 | 翻译失败不会中断其他文件，失败原因会打印到日志 |
| 输出 JSON | 任何情况下都会写入结果文件，即使翻译数为 0 |

**两次修复：**

1. **第一次**（解决 `No translation results found` 错误）：
   - 发现工作流中 detect 步骤的 `git diff` 输出路径是 `docs/zh/quick_start.md`，但脚本期望的是相对路径 `quick_start.md`。在 workflow 中添加了 `sed 's|^docs/zh/||'` 剥离前缀。
   - 发现脚本在某些退出路径（无 API Key、无有效文件）中 `sys.exit()` 跳过了 JSON 写入。新增 `write_empty_json()` 函数确保所有路径都写入结果文件。

2. **第二次**（路径格式兼容）：
   - 修改脚本使其同时支持 `quick_start.md`（相对）和 `docs/zh/quick_start.md`（完整）两种路径格式。

### 2.2 重写工作流

**从 Sphinx PO 流程改为 Markdown 直译流程：**

```
触发（定时/手动）
  → checkout 仓库目标分支
  → 检测 docs/zh/ 下变更/新增的 .md 文件
  → translate_md.py 逐文件中译英
  → 写入 docs/en/ 对应路径
  → 创建新分支 + commit + push
  → gh CLI 创建 PR
```

**权限修复：**

`permissions` 从 `contents: read` 改为：
```yaml
permissions:
  contents: write
  pull-requests: write
```

**分支修复：**

`ref: master` 改为 `ref: ${{ env.TARGET_BRANCH }}`，默认值为 `main`，支持通过 input 指定。

**PR 目标修复：**

写死 `triton-project/triton-ascend` 改为使用 `context.repo` 动态获取当前仓库。

### 2.3 PR 创建 403 错误的修复

**问题：** 使用 `actions/github-script@v8` 创建 PR 时收到 `403 - GitHub Actions is not permitted to create or approve pull requests`。

**根因：** GitHub 的安全策略限制——当工作流由 `schedule` 事件触发时，默认的 `GITHUB_TOKEN` **不允许创建 PR**（因为 `schedule` 事件没有用户的身份认证上下文）。

**解决方案（两步）：**

1. **将 PR 创建方式从 `actions/github-script` 改为 `gh` CLI**
   - `gh` 是 GitHub Actions runner 中预装的 GitHub CLI 工具
   - 使用 `gh pr create` 比 Octokit API 更简洁、更易维护

2. **引入 Personal Access Token (PAT)**
   - 新建 `GH_TOKEN` 环境变量，优先使用 `secrets.PAT_TOKEN`（用户手动配置的 PAT），回退到 `secrets.GITHUB_TOKEN`
   - checkout、git push、gh pr create 全部使用同一个 token，保证一致性

---

## 三、最终工作流文件结构

```
# 文件变更
.github/workflows/schedule_doc_translate.yaml   # 完全重写
docs/scripts/translate_md.py                     # 新建

# 旧的未使用文件（保留不动）
.github/workflows/scripts/po_translate.py         # 不再使用
```

## 四、配置要求

### GitHub Secrets（需在仓库 Settings → Secrets and variables → Actions 中设置）

| Secret 名称 | 必填 | 说明 |
|------------|------|------|
| `DEEPSEEK_API_KEY` | ✅ | DeepSeek API 密钥，用于文档翻译 |
| `PAT_TOKEN` | ✅ | GitHub Personal Access Token，用于创建 PR |

### PAT_TOKEN 创建步骤

1. 打开 https://github.com/settings/tokens?type=beta
2. 点击 **"Generate new token"** → **"Fine-grained token"**
3. 设置：
   - **Repository access**: 选择 `Geofeng12138/triton-ascend123`
   - **Permissions → Contents**: `Read and write`
   - **Permissions → Pull requests**: `Read and write`
4. 生成后复制 token，添加到仓库 secrets → `PAT_TOKEN`

---

## 五、工作流触发方式

### 自动触发（定时）

每 3 天执行一次（cron 表达式：`0 0 */3 * *`，即 UTC 时间 00:00，北京时间 08:00）。

### 手动触发

1. 打开 GitHub 仓库 → Actions 标签
2. 选择 **"Auto Doc Translate (zh → en)"**
3. 点击 **"Run workflow"**
4. 可选参数：`target_branch`（默认为 `main`）

---

## 六、翻译脚本使用说明

```bash
# 安装依赖
pip install openai

# 翻译特定文件（路径相对于 docs/zh/）
python docs/scripts/translate_md.py --files "quick_start.md,FAQ.md"

# 翻译所有变更的文件（git diff 检测或对比 en/zh 差异）
python docs/scripts/translate_md.py --all

# 指定 API Key
python docs/scripts/translate_md.py --all --api-key "your-key"
```

---

## 七、工作流执行流程（完整路径）

```
┌─────────────────────────────────────────────────────┐
│  1. 触发: schedule (每3天) 或 workflow_dispatch     │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  2. Checkout 仓库（使用 PAT_TOKEN）                  │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  3. 安装 Python + openai                            │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  4. Detect: 找出需要翻译的 .md 文件                  │
│     - git diff HEAD -- docs/zh/                     │
│     - 或检查是否存在无 en 对应版的 zh 文件            │
└─────────────────────────┬───────────────────────────┘
                          ▼ (无变更则跳过)
┌─────────────────────────────────────────────────────┐
│  5. translate_md.py 逐文件中译英                     │
│     - 调用 DeepSeek API                             │
│     - 写入 docs/en/ 对应路径                        │
│     - 输出结果到 /tmp/translation_results.json      │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  6. git add 翻译后的文件                             │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  7. 创建新分支 + commit + git push                  │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│  8. gh pr create 创建 Pull Request                  │
│     - 标题: [Doc] Auto-translate... 日期             │
│     - 内容: 翻译文件列表 + 工作流链接                 │
└─────────────────────────────────────────────────────┘
```

---

## 八、常见问题

### Q1: 翻译质量如何控制？

翻译 Prompt 中包含了详细规则：保留 Markdown 格式、不翻译代码块、保留专业术语等。DeepSeek API 的 `temperature=0.3` 保证翻译结果的一致性。如果遇到翻译质量不佳的段落，规则 9 允许保留原文。

### Q2: 如何处理超大文件？

`max_tokens=16384` 对于大多数文档文件是足够的。如果文件超大，翻译会失败并跳过，不影响其他文件。未来可在此基础上添加文件分块功能。

### Q3: 为什么不用 ChatGPT 或其他 API？

DeepSeek 在中文→英文翻译场景下性价比更高（价格约为 GPT-4 的 1/20），且支持 OpenAI 兼容的 API 接口，集成成本低。

### Q4: PR 重复创建会怎样？

`gh pr create` 后加了 `2>&1 || true`，即使 PR 已存在也不会报错退出，工作流继续正常完成。