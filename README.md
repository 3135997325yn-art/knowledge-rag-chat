# 双系统智能 RAG 问答助手（Dual-Mode Intelligent RAG Assistant）

## 项目简介

本项目基于 FastAPI、LangChain 和向量数据库构建，实现了一个融合「通用 AI 对话」与「知识库问答」能力的智能问答助手。

系统支持用户上传本地文档构建专属知识库，并通过检索增强生成（RAG）技术实现基于知识库内容的精准问答。同时保留大模型通用对话能力，实现双模式智能问答体验。

---

## 功能特性

### AI 智能对话

- 支持自然语言聊天
- 支持多轮上下文对话
- 调用大语言模型生成回答

### RAG 知识库问答

- 支持上传 TXT、PDF、DOCX 等文档
- 自动进行文档切分与向量化
- 基于语义检索召回相关内容
- 利用检索结果增强模型回答准确性

### 知识库管理

- 文档上传
- 文档解析
- 向量库构建
- 知识库清空与重建

### Web 可视化界面

- 聊天界面
- 文件上传界面
- 实时问答交互

---

## 系统架构

```text
用户提问
    │
    ▼
问题分类
    │
 ┌──┴─────────┐
 │            │
 ▼            ▼
通用聊天     RAG检索
 │            │
 ▼            ▼
大模型      向量数据库
 │            │
 └─────┬──────┘
       ▼
    生成回答
       ▼
    返回用户
```

---

## 技术栈

### 后端

- Python
- FastAPI

### AI框架

- LangChain

### 向量数据库

- FAISS
- Chroma

### 文档处理

- LangChain Document Loader
- RecursiveCharacterTextSplitter

### 模型能力

- Embedding Model
- Large Language Model（LLM）

### 前端

- HTML
- CSS
- JavaScript

---

## 项目结构

```text
project/
│
├── backend/
│   ├── api/
│   ├── core/
│   ├── services/
│   └── main.py
│
├── fronted/
│   ├── templates/
│   ├── static/
│   └── index.html
│
├── knowledge_base/
│
└── README.md
```

---

## 安装与运行

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 启动项目

```bash
uvicorn backend.main:app --reload
```

### 3. 访问系统

```text
http://127.0.0.1:8000
```

---

## 项目亮点

- 实现通用聊天与知识库问答双模式融合
- 支持私有知识库构建
- 支持文档上传与动态更新
- 基于向量检索提升回答准确率
- 前后端分离架构设计
- 具备 Agent 化问答能力

---

## 后续优化方向

- 多知识库管理
- Agent 工具调用
- MCP 协议接入
- 多模型切换
- Docker 部署支持

---

## 作者

叶娜

南昌交通学院｜智能科学与技术

持续学习 AI Agent、RAG 与大模型应用开发。
