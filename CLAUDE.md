# CLAUDE.md

## 项目概述

个人 AI Agent 学习项目，目标是搭建一个**基于 LangGraph 的技术知识库智能问答系统**，同时作为学习笔记和简历项目。

```
lib/
├── langchain-project/   # DeepSeek Chat — Electron 桌面客户端（有独立 .git）
└── my-knowledge-lib/    # 个人知识库（Obsidian，有独立 .git）
```

两个子项目的关系：`my-knowledge-lib` 中的文档通过 FAISS 索引构建后，被 `langchain-project` 的 `knowledge_search` 工具检索使用（RAG）。

---

## langchain-project

基于 LangChain + DeepSeek 的桌面聊天客户端，支持多轮对话、工具调用、RAG 知识库检索、LangGraph 编排、自纠错、Plan-and-Execute、人机协作。

**架构**：Electron (React) ←→ FastAPI (Python) ←→ LangChain + DeepSeek + FAISS

**环境要求**：Python 3.12 / Node.js / PyTorch 仅支持到 2.2.x

**关键依赖版本约束**：

| 包 | 版本 |
|---|---|
| langchain-classic | 1.0.x |
| langgraph-checkpoint-sqlite | >=0.1 |
| torch | 2.2.x |
| numpy | <2 |
| transformers | <4.51 |
| sentence-transformers | <4 |
| rank_bm25 | >=0.2 |
| jieba | >=0.42 |

**安装**：

```bash
# Python 依赖
pip3 install -r requirements.txt
pip3 install "numpy<2" "transformers<4.51" "sentence-transformers<4" langchain-classic

# Electron 依赖
cd electron && npm install && cd ..

# 配置 API Key
cp .env.example .env  # 编辑 .env 填入 DeepSeek API Key
```

**运行**：`./start.sh` 一键启动，或分开启动 `uvicorn server:app --port 8000` + `cd electron && npx electron .`

**功能**：接入 DeepSeek 大模型、多轮对话记忆、流式输出（逐字显示）、清空对话历史、暗色主题

**前端技术栈**：React 19 + Ant Design X + Vite + TypeScript

**关键文件速查**：

| 文件 | 职责 |
|------|------|
| `server.py` | FastAPI 后端，五个端点：`/chat` / `/graph_chat` / `/plan_chat` / `/human_chat` / `/human_confirm` |
| `graph_agent.py` | LangGraph Agent 终端版，支持 ReAct + 自纠错 + Plan-and-Execute + 人机协作 |
| `chat.py` | 终端版入口，支持四种模式：AgentExecutor / LangGraph+自纠错 / Plan-and-Execute / 人机协作 |
| `tools.py` | 工具定义模块，4 个工具：calculator / get_current_time / search_weather / knowledge_search（混合检索） |
| `build_index.py` | 读取 my-knowledge-lib 文档，构建 FAISS 向量索引 + BM25 语料 |
| `eval_rag.py` | RAG 评测脚本，25 个 QA 对，输出关键词命中率和来源准确率 |
| `electron/src/src/App.tsx` | 前端主组件，SSE 流式 + JSON 响应、ToolCard / RetryCard / PlanCard / ToolConfirmCard |
| `electron/main.js` | Electron 主进程 |
| `start.sh` | 一键启动脚本（后端 + 前端） |

**SSE 协议**（后端 → 前端，agent/graph/plan 模式）：

```
event: tool_start    → {"tool": "xxx", "input": {...}}
event: tool_end      → {"tool": "xxx", "output": "..."}
event: tool_retry    → {"message": "..."}    ← 自纠错重试
event: plan_start    → {"plan": ["步骤1", "步骤2", ...]}  ← Plan-and-Execute
event: plan_step     → {"step": 1, "description": "...", "result": "..."}
data: xxx            → LLM 逐字输出
data: [DONE]         → 流结束
```

**JSON 响应**（human 模式，非 SSE）：

```
POST /human_chat  → {"status": "confirm_required", "tool_calls": [...]} 或 {"status": "done", "content": "..."}
POST /human_confirm → {"status": "done", "content": "..."}
```

**开发规范**：

1. **全链路改动**：修改后端（server.py / graph_agent.py / chat.py）时，必须同步考虑前端（electron/src/src/App.tsx）是否需要适配（如新增 SSE 事件类型、UI 组件、状态栏文案等）
2. **代码注释**：新增或修改的函数、类、关键逻辑块必须添加 `【知识点】` 注释块，格式：
   ```python
   # ============================================================
   # 【知识点】名称 — 简要说明
   # ============================================================
   ```
   前端使用 `// ── 名称 ──` 格式的分隔注释

**当前开发阶段**（截至 2026-04-24）：

```
已完成：
  ✅ 第一阶段：多轮对话客户端（Electron + FastAPI + DeepSeek）
  ✅ 第二阶段：工具调用 Agent（@tool + AgentExecutor）
  ✅ 第三阶段：知识库 RAG 问答（FAISS + bge-zh + knowledge_search 工具）
  ✅ 第四阶段：LangGraph ReAct（StateGraph + ToolNode + 条件路由 + MemorySaver）
  ✅ 第四阶段进阶：自纠错循环（AgentState + corrector 节点 + 前端 RetryCard）
  ✅ 第四阶段进阶：Plan-and-Execute（planner → executor → replanner + 前端 PlanCard）
  ✅ 第四阶段进阶：人机协作（interrupt_before + /human_chat + /human_confirm + 前端 ToolConfirmCard）
  ✅ 第五阶段：记忆持久化（AsyncSqliteSaver）+ RAG 混合检索（向量 + BM25 + RRF）+ 评测集

下一步（优先级从高到低）：
  1. 第六阶段：Docker 容器化 + LangSmith 链路追踪
```

详细进度见：`my-knowledge-lib/Agent AI学习/LangChain实战项目/0-进度总览.md`

---

## my-knowledge-lib

技术学习知识库，使用 Obsidian 管理。包含三大模块：

- **Android 知识**：10 个模块、57 篇文档（基础/进阶/性能优化/自定义View/组件化/跨进程通信/Gradle/开源框架/架构设计/SDK设计）
- **Agent AI 学习**：8 个模块、16 篇文档（基础概念/框架工具/RAG/Agent设计模式/Function Calling/记忆管理/应用实战/部署优化）
- **LeetCode 刷题**：100 道高频算法题，Java 实现

| 专题 | 题数 | 核心技巧 |
|------|------|---------|
| 数组与哈希 | 15 | HashMap、前缀和、原地哈希、桶排序 |
| 双指针与滑动窗口 | 12 | 左右指针、快慢指针、窗口模板 |
| 链表 | 10 | dummy 节点、快慢指针、递归反转 |
| 二叉树 | 15 | DFS/BFS、递归三要素、前中后序 |
| 回溯 | 8 | 选择-递归-撤销、剪枝、去重 |
| 动态规划 | 20 | 状态转移、背包、区间 DP、空间优化 |
| 栈与队列 | 8 | 单调栈、括号匹配、滑动窗口最大值 |
| 图与BFS/DFS | 7 | 拓扑排序、多源 BFS、连通分量 |
| 贪心 | 5 | 区间调度、跳跃游戏、局部最优 |

### 文档规范

新生成或修改文档时必须遵守：

1. **标题**：`# 主题` 一级标题
2. **引言**：开头用 `>` 引用块做简介
3. **双链**：引言中包含 `相关主题：[[xxx]] | [[yyy]]`
4. **分隔**：引言后用 `---` 分隔
5. **章节**：正文用 `## 一、` `## 二、` 编号，子节用 `### 1.1` 格式
6. **深度**：包含原理分析、源码分析、代码示例、实战场景
7. **代码块**：必须标注语言（java/python 等）
8. **重点提示**：适当使用 `> 🔑` 标记
9. **字数**：每篇不少于 3000 字
10. **双链校验**：`[[]]` 链接指向的文档必须存在
11. **面试要点**：文档末尾必须包含 `## 面试题精选` 章节（5-10 题含简要解答）
12. **不重复**：生成前先检查是否已存在同名文件，避免内容重复

### 语言约定

- Android 模块：代码示例优先使用 Java，能用 Java 就不用 Kotlin
- Agent AI 模块：使用 Python

### 生成文档标准结构

```
# {topic}
> 一句话简介
相关主题：[[相关1]] | [[相关2]]
---
## 一、核心概念
## 二、原理分析（含源码/流程图）
## 三、实战应用（含代码示例）
## 四、常见问题与面试要点
## 五、面试题精选（5-10 题，含简要解答）
## 六、总结与参考
```

### 索引更新

- 新增文档后，需在对应模块 `README.md` 中追加 `[[文档名]]` 链接
- 如果文档标题与文件名不同，使用 `[[文件名|标题]]` 别名语法
- README 格式：`# {模块名}` 开头，每行 `- [[文档名]]`
- 保留现有 README 中手动添加的别名说明（`[[xx|描述]]` 格式）
- 保留原 README 的自定义排列顺序，不强制按文件名排序
- 如果模块数量或文档数量变化，同步更新根目录 README.md 中的统计数字

### 审查要点

审查文档时检查：标题结构、引言、双链、分隔线、内容深度、代码标注、关键提示、字数、双链有效性。

审查报告格式：
```
## 📋 审查报告：{文件名}
- ✅ 标题结构：正常
- ⚠️ 缺少相关主题链接
- ❌ 代码块未标注语言（第 45 行）
```

约束：
- 仅修复格式和结构问题，不擅自修改内容观点
- 修复前必须向用户展示审查报告并获得确认
- 检查双链时只验证文件是否存在，不验证内容
