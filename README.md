# LangChain RAG 学习文档

这个目录是一组围绕 LangChain、提示词、Runnable、记忆、文档加载、向量存储和 RAG 检索问答的练习代码。

建议学习时不要急着一次性跑完所有脚本，而是按“模型调用 -> 提示词 -> 链式调用 -> 输出解析 -> 记忆 -> 文档加载 -> 向量存储 -> RAG”的顺序逐步推进。每学完一个阶段，先能说清楚核心概念，再运行对应代码，最后记录一个自己的小例子。

## 快速入口

- [学习路线](docs/学习路线.md)：按阶段整理当前 01-29 个脚本。
- [阶段笔记模板](docs/阶段笔记模板.md)：以后补充每节学习笔记时可以直接套用。

## 阶段详解文档

- [阶段 1：模型调用入门](docs/阶段01-模型调用入门.md)
- [阶段 2：Embedding 与相似度](docs/阶段02-Embedding与相似度.md)
- [阶段 3：提示词模板](docs/阶段03-提示词模板.md)
- [阶段 4：Chain 与 Runnable](docs/阶段04-Chain与Runnable.md)
- [阶段 5：会话记忆](docs/阶段05-会话记忆.md)
- [阶段 6：文档加载与分割](docs/阶段06-文档加载与分割.md)
- [阶段 7：向量存储与检索](docs/阶段07-向量存储与检索.md)
- [阶段 8：RAG 检索问答](docs/阶段08-RAG检索问答.md)

## 当前文件分组

### 0. 基础补充

- `01[扩展]余弦相似度.py`：理解向量相似度，为后续向量检索做铺垫。
- `15[扩展]Python的或运算符的重写.py`：理解 `|` 运算符重载，为 LangChain 表达式语言做铺垫。
- `16Runnable接口源码查看.py`：认识 Runnable 接口。

### 1. 模型调用

- `02LangChain访问阿里云通义千问大模型.py`
- `03LangChain访问Ollama本地模型.py`
- `04LangChain的流式输出.py`
- `05LangChain调用聊天模型.py`
- `06LangChain调用Ollama的聊天模型.py`
- `07LangChain消息的简写形式.py`
- `08LangChain访问阿里云嵌入模型.py`
- `09LangChain访问Ollama的本地嵌入模型.py`

### 2. 提示词模板

- `10通用提示词模板.py`
- `11FewShot提示词模板.py`
- `12模板类的format和invoke方法.py`
- `13ChatPromptTemplate的使用.py`

### 3. Chain 与 Runnable

- `14Chain的基础使用.py`
- `17StrOutputParser解析器.py`
- `18JsonOutputParser解析器.py`
- `19RunnableLambda的基础使用.py`
- `29RunnablePassthrough的使用.py`

### 4. 会话记忆

- `20临时会话记忆.py`
- `21长期会话记忆.py`
- `chat_history/`：长期记忆示例产生或使用的会话记录。

### 5. 文档加载与分割

- `22CSVLoader的使用.py`
- `23JSONLoader的使用.py`
- `24PyPDFLoader的使用.py`
- `25TextLoader和文档分割器.py`
- `data/`：CSV、JSON、PDF、TXT 等示例数据。

### 6. 向量存储与 RAG

- `26内存向量存储.py`
- `27外部向量持久化存储.py`
- `28向量检索构建提示词.py`
- `chroma_db/`：Chroma 向量库持久化目录。

## 推荐学习节奏

1. 先读 `docs/学习路线.md`，明确每一阶段要解决的问题。
2. 每次只跑 2-4 个脚本，避免概念混在一起。
3. 跑完脚本后，用 `docs/阶段笔记模板.md` 记录：本节目标、关键 API、输入输出、踩坑点。
4. 学到 RAG 阶段后，再回头复习 Embedding、Loader、Retriever、Prompt 和 Chain 的关系。

## 后续整理方向

- 第二阶段：为每个阶段补一份“概念说明 + 代码讲解 + 练习任务”。
- 第三阶段：把脚本中可能存在的编码显示问题、环境变量、依赖安装说明统一整理。
- 第四阶段：根据学习路线整理成完整的课程笔记或复习手册。
