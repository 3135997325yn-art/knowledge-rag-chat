# RAG 知识库 + AI聊天助手（已修复版）

## 怎么跑起来

```bash
# 1. 进入项目根目录（也就是这个README所在的目录）
cd Final_Project

# 2. 安装依赖
pip install -r fronted/local_rag/requirements.txt

# 3. 启动服务
python run.py
```

浏览器打开 http://127.0.0.1:8000 即可使用网页界面：
- 上传文档（.txt / .md / .pdf / .docx / .csv）入库
- 在对话框提问，AI会先检索知识库相关内容，再结合大模型生成回答
- **支持多轮对话记忆**：比如先说"我是叶娜"，再问"我是谁"，AI能记住上下文正确回答
- 没有上传任何文档时，也能像普通AI聊天助手一样正常对话（不会强行说"知识库无相关信息"）
- 点"清空对话记忆"只清空聊天历史，知识库文档不受影响
- 点"清空知识库"会删除所有已上传文档的向量索引，对话记忆不受影响

接口文档：http://127.0.0.1:8000/docs

## API Key 配置

`fronted/local_rag/.env` 里已经配置了你的阿里云百炼 `DASHSCOPE_API_KEY`。
如果要换模型，把 `CHAT_MODEL` 改成阿里云百炼支持的其他模型名即可，比如：
- `qwen-turbo`（更快更省）
- `qwen-plus`（默认，平衡性价比，当前配置）
- `qwen-max`（效果更好但更贵）

## 这次新增：多轮对话记忆

之前的版本每次提问都是"失忆"的——问完"我是叶娜"再问"我是谁"，AI完全不记得上一句。
现在已修复：

- `KnowledgeService` 按 `session_id` 维护每个会话最近10轮的问答历史
- 每次提问时，历史对话会和当前问题一起发给大模型，让它有上下文记忆
- 同时调整了提示词逻辑：检索不到知识库相关内容时，AI会正常聊天回答，
  不会再生硬地说"知识库中没有相关信息"
- 前端在浏览器本地生成一个 `session_id`（存在 localStorage），同一个浏览器
  标签内的对话历史会保持连续；新增"清空对话记忆"按钮可以随时重新开始

## 这次改了什么（原项目跑不起来的原因）

原代码是一个写到一半、几处互相矛盾的半成品：

1. **`KnowledgeService()` 漏传参数** —— `routes.py` 里调用时没传 `settings`，但
   `__init__` 强制要求这个参数，直接报 `TypeError`。已修复：改成依赖注入，统一在
   `app.py` 里创建一次 `KnowledgeService`，再传给路由用。

2. **`RAGChain` 参数数量不匹配** —— 调用方传2个参数，但类定义只收1个参数。
   已修复并重写：现在 `RAGChain` 真正接收 `settings` 和 `vector_store_manager`。

3. **AI聊天功能其实没接上** —— 原来的 `rag_chain.py` 根本没调用任何大模型，
   命中知识库就把原文拼接返回，没命中就返回"这是AI回答：{question}"这种占位字符串。
   已修复：现在用 `langchain-openai` 的 `ChatOpenAI`，通过阿里云百炼的
   OpenAI兼容接口（`https://dashscope.aliyuncs.com/compatible-mode/v1`）
   真正调用 `qwen-plus` 模型生成回答。

4. **Embedding 是假的随机向量** —— 原来用的是 `FakeEmbeddings`（LangChain自带的
   随机向量生成器），检索结果和问题毫无语义关联。已修复：换成本地免费的
   `sentence-transformers`（`all-MiniLM-L6-v2`模型），真正的语义检索，
   完全离线、无需API Key，首次运行会自动从HuggingFace下载模型（几十MB）。

5. **前端调的接口后端没实现** —— `index.html` 调用了 `/api/upload`、
   `/api/reset`，但后端只写了 `/api/ask`。已修复：补全 `upload` / `reset` /
   `status` 三个接口。

6. **import 路径混乱，启动方式不统一** —— 所有模块用 `from fronted.local_rag.xxx
   import ...` 这种绝对包路径，但 `fronted/` 目录下原来没有 `__init__.py`，
   且数据存储路径用的是相对路径（依赖你从哪个目录启动）。已修复：补上
   `__init__.py`，把数据路径改成基于代码文件位置的绝对路径，并提供统一的
   `run.py` 作为唯一启动入口。

## 目录结构

```
Final_Project/
├── run.py                  # 启动入口，在这一层目录下运行
└── fronted/
    ├── __init__.py
    └── local_rag/
        ├── .env             # 配置文件（含你的阿里云Key）
        ├── app.py           # FastAPI应用组装
        ├── api/routes.py    # 接口：/upload /ask /status /reset
        ├── config/settings.py
        ├── core/
        │   ├── document_loader.py     # 加载 txt/md/pdf/docx/csv
        │   ├── text_splitter.py       # 文本切分
        │   ├── embedding_service.py   # 本地语义向量（sentence-transformers）
        │   ├── vector_store.py        # FAISS向量库
        │   └── rag_chain.py           # 检索 + 调用阿里云大模型生成回答
        ├── services/knowledge_service.py  # 业务逻辑整合
        ├── index.html        # 前端页面
        └── data/              # 上传文件 & 向量索引存储位置
```

## 安全提醒

`.env` 里包含你的真实 API Key，**不要把这个文件提交到公开的 GitHub 仓库**。
`.gitignore` 里已经排除了它，但如果你之前有 commit 记录包含过这个 key，
建议去阿里云百炼控制台重新生成一个新的 Key。
