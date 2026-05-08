# 阶段 8：RAG 检索问答

## 本阶段要解决什么

前面所有内容最终都会汇到这里：让模型基于外部资料回答问题，而不是完全依赖模型自己的记忆。

对应脚本：

- `28向量检索构建提示词.py`
- `29RunnablePassthrough的使用.py`

## RAG 是什么

RAG 是 Retrieval-Augmented Generation，意思是“检索增强生成”。

它的核心流程：

```text
用户问题
-> 检索相关资料
-> 把资料和问题一起放进提示词
-> 模型基于资料生成答案
```

RAG 解决的问题：

- 模型不知道你的私有资料。
- 模型知识可能过时。
- 直接问模型容易胡编。
- 需要答案尽量贴近指定资料来源。

## 脚本 28：手动构建 RAG

脚本 `28` 先创建 prompt：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "以我提供的已知参考资料为主，简洁和专业的回答用户问题。参考资料:{context}。"),
        ("user", "用户提问：{input}")
    ]
)
```

这里有两个变量：

- `{context}`：检索到的参考资料。
- `{input}`：用户问题。

接着创建内存向量库并写入资料：

```python
vector_store.add_texts(
    ["减肥就是要少吃多练", "在减脂期间吃东西很重要,清淡少油控制卡路里摄入并运动起来", "跑步是很好的运动哦"]
)
```

然后手动检索：

```python
result = vector_store.similarity_search(input_text, 2)
```

再把检索结果拼成字符串：

```python
reference_text = "["
for doc in result:
    reference_text += doc.page_content
reference_text += "]"
```

最后调用链：

```python
chain = prompt | print_prompt | model | StrOutputParser()
res = chain.invoke({"input": input_text, "context": reference_text})
```

这就是最直观的 RAG：先检索，再手动把资料放入 prompt。

## 脚本 29：把检索也放进 Chain

脚本 `29` 进一步自动化：

```python
retriever = vector_store.as_retriever(search_kwargs={"k": 2})
```

`as_retriever()` 会把向量库变成一个可以放进链里的 Retriever。

然后定义格式化函数：

```python
def format_func(docs: list[Document]):
    if not docs:
        return "无相关参考资料"

    formatted_str = "["
    for doc in docs:
        formatted_str += doc.page_content
    formatted_str += "]"

    return formatted_str
```

最后构建完整链：

```python
chain = (
    {"input": RunnablePassthrough(), "context": retriever | format_func}
    | prompt
    | print_prompt
    | model
    | StrOutputParser()
)
```

这条链的输入只是一个字符串：

```python
res = chain.invoke(input_text)
```

但链内部会自动拆成两路：

```text
input_text -> RunnablePassthrough() -> input
input_text -> retriever -> format_func -> context
```

然后合并成：

```python
{
    "input": "怎么减肥？",
    "context": "[减肥就是要少吃多练...]"
}
```

## RAG 中每个组件的职责

| 组件 | 职责 |
| --- | --- |
| Embedding | 把文本转成向量 |
| VectorStore | 保存文档向量并支持相似度搜索 |
| Retriever | 根据问题取回相关文档 |
| Prompt | 把问题和参考资料组织成模型输入 |
| Model | 基于提示词生成答案 |
| OutputParser | 把模型输出转换成需要的格式 |

## 资料为空怎么办

脚本 `29` 中有这个判断：

```python
if not docs:
    return "无相关参考资料"
```

真实项目中还可以进一步要求模型：

```text
如果参考资料不足以回答，请明确说“不知道”，不要编造。
```

这是减少幻觉的重要手段。

## RAG 答案质量取决于什么

主要取决于四件事：

- 文档质量：原始资料是否准确、完整。
- 切分质量：chunk 是否保留足够上下文。
- 检索质量：Embedding 和检索参数是否合适。
- 提示词质量：是否明确要求模型基于资料回答。

模型本身也重要，但 RAG 项目里不能只盯模型。很多问题其实出在文档、切分和检索。

## 脚本 28 和 29 的区别

| 脚本 | 特点 | 适合阶段 |
| --- | --- | --- |
| `28` | 手动检索、手动拼接 context | 理解 RAG 原理 |
| `29` | Retriever 和 RunnablePassthrough 自动组链 | 构建可复用 RAG 链 |

建议先完全理解 `28`，再看 `29`。`29` 更像工程化写法。

## 易错点

- 只检索不把资料放入 prompt，模型不会知道检索结果。
- context 太长会增加成本，也可能淹没重点。
- context 太短可能缺少回答所需信息。
- 提示词没有约束“基于参考资料”，模型可能自由发挥。
- Retriever 返回的是 `list[Document]`，通常要格式化成字符串再进 prompt。

## 本阶段检查题

- RAG 的三步核心流程是什么？
- Retriever 的输入和输出分别是什么？
- 为什么需要 `RunnablePassthrough()`？
- 脚本 `28` 和 `29` 哪个更适合理解原理？哪个更适合工程化？

## 小练习

把向量库资料改成你自己的学习笔记，例如：

```python
vector_store.add_texts([
    "LangChain 的 PromptTemplate 用于构建可复用提示词。",
    "RunnablePassthrough 可以在链中保留原始输入。",
    "RAG 的核心是先检索资料，再让模型基于资料回答。"
])
```

然后提问：

```text
RunnablePassthrough 有什么作用？
```

观察回答是否主要来自参考资料。

## 进阶练习

把 `25TextLoader和文档分割器.py`、`26内存向量存储.py`、`29RunnablePassthrough的使用.py` 串起来，做一个基于 `data/Python基础语法.txt` 的最小 RAG 问答。

目标流程：

```text
加载 TXT
-> 切分 Document
-> 写入向量库
-> as_retriever()
-> prompt + model + parser
-> 回答 Python 基础问题
```
