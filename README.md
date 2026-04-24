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
