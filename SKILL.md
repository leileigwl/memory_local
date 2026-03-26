---
name: memory-recorder
description: "自动学习用户偏好。触发词：记录、请记录、记录上下文、记录这条指令、学习我的习惯。读取当前会话+历史session日志，按分类提取思维模式、纠正记录、工作偏好，写入 ~/.claude/preferences/ 持久化。"
---

# Memory Recorder — 用户偏好自学习

## 铁律（不可违反）

1. **禁止读取** `~/.claude/CLAUDE.md`（核心配置，由系统自动加载）
2. **禁止修改** `~/.claude/CLAUDE.md`
3. **禁止修改** 本文件 `~/.claude/skills/memory-recorder/SKILL.md`
4. **只允许读写** `~/.claude/preferences/` 目录

## 触发条件（两种，行为不同）

### 触发 A：用户主动"记录"
触发词："记录"、"请记录"、"记录上下文"、"记录这条指令"、"学习我的习惯"、"记住这个"、"记录一下"

**行为**：用户已经想好了，理解全部上下文，写入对应分类文件。
- 写入 `分类/thinking.md`、`分类/habits.md`、`分类/communication.md`
- **不写** rules.md（rules.md 只放最核心的规则）

### 触发 B：用户纠正 AI
触发词："不对"、"应该是"、"你错了"、"我不是这个意思"

**行为**：自动检测到纠正，记录下来。
- 写入对应分类的 `corrections.md`
- 如果同一纠正 ≥3 次，才升级到 rules.md

## 分类体系

```
~/.claude/preferences/
├── rules.md                  ← 永久规则（跨分类，每次会话自动加载）
├── 商业/
│   ├── corrections.md        ← 纠正记录
│   ├── thinking.md           ← 思维模式
│   └── habits.md             ← 工作习惯
├── 模型建构/
│   ├── corrections.md
│   ├── thinking.md
│   └── habits.md
├── 项目开发/
│   ├── corrections.md
│   ├── thinking.md
│   └── habits.md
└── 通用/
    ├── corrections.md
    ├── thinking.md
    ├── habits.md
    └── communication.md      ← 交流风格（只有通用分类有）
```

## 执行流程

### 第一步：判断分类

根据当前对话上下文自动判断属于哪个分类：
- 提到商业模式、赚钱、客户、定价、运营、选品、营销 → **商业**
- 提到 AI模型、Prompt、Agent、架构设计、数据处理、训练、推理 → **模型建构**
- 提到代码、开发、部署、Bug、前端、后端、数据库、API → **项目开发**
- 无法归类或跨分类 → **通用**

### 第二步：分析当前上下文

1. 回顾当前会话中最近的对话轮次
2. 识别用户的意图、决策逻辑、纠正内容
3. 提取关键模式

### 第三步：读取历史 session 日志

1. 列出 `~/.claude/projects/-Users-leileigwl/` 下最近的 `.jsonl` 文件（排除 `agent-*.jsonl`）
2. 按修改时间排序，读取最近 5-10 个 session
3. 从 JSONL 中提取 `role: "user"` 的消息内容
4. **只分析与当前对话同分类的内容**，识别跨 session 的重复模式

JSONL 格式：每行一个 JSON 对象，用户消息在 `message.content` 字段。

### 第四步：按触发类型写入

#### 如果是触发 A（用户主动"记录"）

理解用户全部上下文，提取以下内容写入对应分类目录：

**thinking.md — 思维模式**
```markdown
## [日期] 思维模式
- 场景：什么情况下
- 用户逻辑：用户怎么思考这个问题的
- 偏好：用户倾向什么方式
```

**habits.md — 工作习惯**
```markdown
## [日期] 工作习惯
- 习惯：具体行为
- 工具偏好：喜欢用什么
- 流程偏好：习惯怎样的工作流
```

**communication.md — 交流风格（仅通用分类）**
```markdown
## [日期] 交流风格
- 发现：用户喜欢/不喜欢的交流方式
- 语言偏好：中文/英文、简洁/详细
- 反馈模式：用户如何给反馈
```

**重要：主动"记录"不写 rules.md。** rules.md 只放最核心的规则。

#### 如果是触发 B（用户纠正 AI）

#### corrections.md — 纠正记录
```markdown
## [日期] 纠正内容
- 场景：当时在做什么
- 纠正：用户说了什么
- 正确做法：应该怎么做
- 提取规则：一句话总结
```

#### thinking.md — 思维模式
```markdown
## [日期] 思维模式
- 场景：什么情况下
- 用户逻辑：用户怎么思考这个问题的
- 偏好：用户倾向什么方式
```

#### habits.md — 工作习惯
```markdown
## [日期] 工作习惯
- 习惯：具体行为
- 工具偏好：喜欢用什么
- 流程偏好：习惯怎样的工作流
```

#### communication.md — 交流风格（仅通用分类）
```markdown
## [日期] 交流风格
- 发现：用户喜欢/不喜欢的交流方式
- 语言偏好：中文/英文、简洁/详细
- 反馈模式：用户如何给反馈
```

### 第五步：去重

1. 检查新记录是否与同分类下已有记录重复
2. 如果重复，更新已有条目而不是新增
3. 如果不重复，追加新条目

### rules.md 升级规则（严格控制）

rules.md 是最核心的规则集，必须保持精简。只有以下情况才能写入：

1. **用户纠正 ≥3 次**：同一错误被反复纠正，说明是 AI 的顽固坏习惯
2. **用户明确说"这是规则"**：用户主动指定要升级为规则

写入 rules.md 的格式：
```markdown
## 规则 [编号] — [分类]
- 触发条件：什么情况下适用
- 规则内容：必须做什么/不能做什么
- 首次发现：日期
- 来源：手动记录 / 纠正升级
```

### 第六步：反馈给用户

完成后报告：
```
已完成学习 [分类]，本次记录：
- 纠正记录：N 条
- 思维模式：N 条
- 工作习惯：N 条
- 新增/更新永久规则：N 条

当前永久规则总数：N 条
```

## 读取偏好（按需加载，不是全量加载）

### 会话开始时
- 只读取 `~/.claude/preferences/rules.md`（永久规则，跨分类，文件较小）

### 对话过程中（按需）
- 根据当前对话的分类，读取对应分类目录下的文件
- 例：聊商业话题 → 读取 `~/.claude/preferences/商业/` 下所有文件
- 例：用户纠正 → 先检查对应分类的 corrections.md

### 用户触发"记录"时
- 读取当前分类 + 最近 5-10 个 session 日志
- 全量分析当前分类的偏好文件

## Session 日志读取

```bash
# 列出最近 10 个 session（排除 agent 日志）
ls -t ~/.claude/projects/-Users-leileigwl/*.jsonl | grep -v agent | head -10

# 提取用户消息
cat session.jsonl | python3 -c "
import sys, json
for line in sys.stdin:
    try:
        obj = json.loads(line)
        if obj.get('type') == 'human' or (obj.get('message', {}).get('role') == 'user'):
            content = obj.get('message', {}).get('content', '')
            if isinstance(content, list):
                content = ' '.join(c.get('text', '') for c in content if c.get('type') == 'text')
            if content and len(content) > 10:
                print(content[:500])
                print('---')
    except: pass
"
```

## 升级机制总结

```
触发 A：用户说"记录"
       │
       ▼
  理解全部上下文（当前会话 + 历史 session 日志，包括被 compact 压缩的）
       │
       ▼
  判断分类（商业/模型建构/项目开发/通用）
       │
       ▼
  写入对应分类的 thinking.md / habits.md / communication.md
  （不写 rules.md）
       │
       ▼
  下次同分类对话时按需加载 → AI 更懂你

触发 B：用户纠正 AI
       │
       ▼
  写入对应分类的 corrections.md
       │
       ▼
  同一纠正 ≥3 次？
       │
    是 │     否
       ▼      └→ 继续积累在 corrections.md
  升级为永久规则 → rules.md
       │
       ▼
  rules.md 每次会话自动加载 → AI 不再犯同类错误
```

## 关于 session 日志

**核心原则：不能只依赖当前会话上下文。** 当前会话可能已经被 `/compact` 压缩过，
丢失了早期对话内容。因此必须读取 JSONL 日志文件来恢复完整历史。

- 所有 session 日志在 `~/.claude/projects/-Users-leileigwl/*.jsonl`
- JSONL 完整记录了每次对话的原始内容，不受 compact 影响
- 触发"记录"时，必须读取最近 5-10 个 session 的用户消息
- 从中提取与当前分类相关的偏好和模式
