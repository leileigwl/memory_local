# Memory Recorder - AI 用户偏好自学习 Skill

> 让 Claude Code 自动学习你的偏好，越用越懂你。

## 这是什么？

Memory Recorder 是一个 Claude Code Skill，它能自动从你的对话中学习：
- 🧠 **思维模式**：你如何思考问题、做决策
- 🎯 **工作习惯**：你喜欢的工具、流程、风格
- 🔄 **纠正记录**：AI 犯错时你的反馈，避免再犯

## 为什么需要它？

你有没有发现：
- 每次新对话都要重新告诉 AI 你的偏好？
- AI 总是犯同样的错误？
- 你的工作风格、语言习惯需要反复说明？

Memory Recorder 解决这些问题。它像一个永不遗忘的助手，持续学习你的风格。

## 工作原理

```
用户说"记录"
    ↓
读取当前会话 + 历史 session 日志
    ↓
自动分类（商业/模型建构/项目开发/通用）
    ↓
写入偏好文件到 ~/.claude/preferences/
    ↓
下次对话自动加载 → AI 更懂你
```

## 触发方式

- 用户主动触发：说「记录」「请记录」「学习我的习惯」
- 自动触发：用户纠正 AI 时（说「不对」「应该是」）

## 安装

```bash
# 复制到你的 Claude Code skills 目录
mkdir -p ~/.claude/skills/memory-recorder
cp SKILL.md ~/.claude/skills/memory-recorder/
```

## 偏好分类

```
~/.claude/preferences/
├── rules.md           # 永久规则（跨分类）
├── 商业/
│   ├── corrections.md # 纠正记录
│   ├── thinking.md    # 思维模式
│   └── habits.md      # 工作习惯
├── 模型建构/
├── 项目开发/
└── 通用/
```

## 特色功能

### 1. 自动升级规则
当同一纠正出现 3 次以上，自动升级为永久规则，每次会话自动加载。

### 2. Session 日志深度分析
即使对话被 `/compact` 压缩过，也能从 JSONL 日志恢复完整历史，提取你的偏好。

### 3. 按需加载
不一次性加载所有偏好，根据当前对话话题按分类加载，保持上下文清爽。

## 安全设计

- 禁止修改核心配置文件（CLAUDE.md）
- 只允许读写 `~/.claude/preferences/` 目录
- 敏感信息不会被记录

## 适用场景

- 你是某个领域的专家，希望 AI 用你的方式思考
- 你有固定的工作流程，不想每次重新说明
- 你在用 Claude Code 做长期项目，需要 AI 记住上下文

## 作者

由 Lei (leileigwl) 创建，用于提升 AI 辅助工作效率。

## 许可证

MIT License