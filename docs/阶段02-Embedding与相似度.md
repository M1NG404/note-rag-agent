# 阶段 2：Embedding 与相似度

## 本阶段要解决什么

RAG 的核心不是“把文档原封不动塞给模型”，而是先根据用户问题找到最相关的文档片段。要做到这一点，就需要 Embedding 和相似度计算。

对应脚本：

- `01[扩展]余弦相似度.py`
- `08LangChain访问阿里云嵌入模型.py`
- `09LangChain访问Ollama的本地嵌入模型.py`

## 什么是 Embedding

Embedding 是把文本转换成数字向量的过程。

例如：

```text
我喜欢你 -> [0.12, -0.08, 0.31, ...]
我稀饭你 -> [0.11, -0.07, 0.29, ...]
晚上吃啥 -> [-0.33, 0.41, 0.02, ...]
```

语义越接近的文本，向量方向通常越接近。这样计算机就可以通过数学方式判断文本相似度。

## 余弦相似度

脚本 `01` 手写了余弦相似度：

```python
result = get_dot(vec_a, vec_b) / (get_norm(vec_a) * get_norm(vec_b))
```

公式是：

```text
cos_sim = (A · B) / (||A|| * ||B||)
```

你可以把它理解为：只比较两个向量的方向像不像，不太关心它们的长度。

常见结果含义：

- 越接近 `1`：越相似。
- 接近 `0`：关系不明显。
- 越接近 `-1`：方向相反。

脚本中的例子：

```python
vec_a = [0.5, 0.5]
vec_b = [0.7, 0.7]
vec_c = [0.7, 0.5]
vec_d = [-0.6, -0.5]
```

`vec_a` 和 `vec_b` 方向几乎一样，所以相似度很高；`vec_d` 方向相反，所以相似度会是负数。

## embed_query 和 embed_documents

脚本 `08` 使用阿里云 Embedding：

```python
from langchain_community.embeddings import DashScopeEmbeddings

model = DashScopeEmbeddings()

print(model.embed_query("我喜欢你"))
print(model.embed_documents(["我喜欢你", "我稀饭你", "晚上吃啥"]))
```

这里有两个核心方法：

- `embed_query(text)`：把一个查询问题转成向量。
- `embed_documents(list[str])`：把一批文档转成向量。

为什么分成两种方法？因为有些 Embedding 模型会针对“查询”和“文档”使用不同优化方式。LangChain 保留这两个方法，是为了让检索流程更清晰。

## 本地 Embedding

脚本 `09` 使用 Ollama 本地 Embedding：

```python
from langchain_ollama import OllamaEmbeddings

model = OllamaEmbeddings(model="qwen3-embedding:4b")
```

本地 Embedding 的优点是数据不用发到云端，适合处理隐私数据或离线学习。缺点是依赖本机资源，并且模型效果和速度取决于本地环境。

## RAG 中 Embedding 的位置

RAG 中通常有两次 Embedding：

1. 建库时：把文档切块后转成向量，存进向量库。
2. 查询时：把用户问题转成向量，用它去向量库里找相似文档。

流程可以理解为：

```text
文档 -> 切分 -> Embedding -> 向量库
用户问题 -> Embedding -> 相似度搜索 -> 相关文档
```

## 易错点

- Embedding 模型不是聊天模型，不能用它直接回答问题。
- 向量维度通常很长，不需要人工理解每一个数字。
- 建库和查询时最好使用同一个 Embedding 模型，否则相似度空间可能不一致。
- 中文语义检索要选对支持中文效果较好的 Embedding 模型。

## 本阶段检查题

- 为什么语义检索不能只靠关键词匹配？
- `embed_query()` 和 `embed_documents()` 分别适合什么场景？
- 余弦相似度为什么更关注方向而不是长度？
- 如果换了 Embedding 模型，原来的向量库通常要不要重建？

## 小练习

把 `08LangChain访问阿里云嵌入模型.py` 中的文本改成三组更接近业务的句子，例如：

```python
[
    "如何学习 LangChain",
    "LangChain 应该怎么入门",
    "今天晚上吃什么"
]
```

观察前两个句子的向量是否无法直接肉眼理解，但你应该能推断它们在向量空间里更相近。
