# lib

个人项目集合。包含两个独立子项目，各自有独立的 git 仓库。

## 项目

| 项目 | 说明 | 技术栈 |
|------|------|--------|
| [langchain-project](https://github.com/moonaax/langchain-project) | DeepSeek 桌面聊天客户端 | Python + LangChain + Electron |
| [my-knowledge-lib](https://github.com/moonaax/my-knowledge-lib) | 个人技术知识库 | Obsidian + Markdown |

## 快速开始

```bash
# 1. 克隆本仓库
git clone https://github.com/moonaax/lib.git ~/lib
cd ~/lib

# 2. 克隆子项目
git clone https://github.com/moonaax/langchain-project.git
git clone https://github.com/moonaax/my-knowledge-lib.git
```

## 目录结构

```
lib/
├── CLAUDE.md                      # 项目规范（Claude Code 使用）
├── .claude-memory/                # Claude Code 自动记忆（随仓库同步）
├── langchain-project/             # DeepSeek 聊天客户端
│   ├── server.py                  # FastAPI 后端
│   ├── chat.py                    # 对话逻辑
│   ├── graph_agent.py             # LangGraph Agent
│   ├── tools.py                   # Agent 工具集
│   ├── electron/                  # Vue + Vite 前端
│   └── ...
└── my-knowledge-lib/              # 技术知识库
    ├── Android知识/                # Android 高级开发（57 篇）
    ├── Agent AI学习/               # AI Agent 开发（17 篇）
    └── LeetCode刷题/               # 高频算法题（100 道）
```

## 换电脑后恢复 Claude Code 记忆

本仓库的 `.claude-memory/` 目录保存了 Claude Code 的自动记忆(feedback / user / project 类)。由于 `.claude/settings.local.json` 不进 git，换电脑 clone 后需要手动配置一次，让 Claude Code 从仓库里的路径读取记忆:

在 `./.claude/settings.local.json`(或 `~/.claude/settings.json`)中加入:

```json
{
  "autoMemoryDirectory": "/Users/<你的用户名>/lib/.claude-memory"
}
```

参考:[Claude Code Memory 文档](https://code.claude.com/docs/en/memory#storage-location)
