# 阶段 4：Chain 与 Runnable

## 本阶段要解决什么

前面你已经会单独调用模型、写提示词模板。现在要学习如何把多个组件串起来，形成可复用的处理流程。

对应脚本：

- `14Chain的基础使用.py`
- `15[扩展]Python的或运算符的重写.py`
- `16Runnable接口源码查看.py`
- `17StrOutputParser解析器.py`
- `18JsonOutputParser解析器.py`
- `19RunnableLambda的基础使用.py`
- `29RunnablePassthrough的使用.py`

## Runnable 是什么

Runnable 可以理解为 LangChain 中“可以被运行的组件”。很多对象都实现了 Runnable 接口，例如：

- PromptTemplate
- ChatPromptTemplate
- LLM
- Chat Model
- OutputParser
- Retriever

只要组件是 Runnable，就可以用 `|` 串起来。

## 管道符 `|`

脚本 `15` 用纯 Python 解释了 `|` 运算符重载：

```python
def __or__(self, other):
    return MySequence(self, other)
```

当你写：

```python
d = a | b | c
```

Python 实际上会调用对象的 `__or__` 方法。LangChain 正是利用这个机制，让你可以写：

```python
chain = prompt | model | parser
```

这不是普通的“或运算”，而是在构建一条执行链。

## 最基础的 Chain

脚本 `14`：

```python
chain = chat_prompt_template | model
```

执行流程：

```text
输入 dict
-> ChatPromptTemplate 组装消息
-> ChatTongyi 生成回复
-> AIMessage
```

如果使用：

```python
for chunk in chain.stream({"history": history_data}):
    print(chunk.content, end="", flush=True)
```

说明整条链也支持流式输出。

## 输出解析器

### StrOutputParser

脚本 `17`：

```python
chain = prompt | model | parser
```

聊天模型通常返回 `AIMessage`，而 `StrOutputParser()` 会把它转换成普通字符串。

这很重要，因为下一步如果还要把模型输出交给其他模板，普通字符串往往更容易处理。

脚本里还有一个更长的链：

```python
chain = prompt | model | parser | model | parser
```

它表示第一次模型输出被解析成字符串后，又继续送给第二次模型调用。

### JsonOutputParser

脚本 `18`：

```python
chain = first_prompt | model | json_parser | second_prompt | model | str_parser
```

流程是：

```text
姓氏和性别
-> 第一个提示词：要求模型返回 JSON
-> 模型生成 JSON 文本
-> JsonOutputParser 转成 dict
-> 第二个提示词读取 dict 中的 name
-> 模型解释名字含义
-> StrOutputParser 转成字符串
```

这里的关键是：第一个模型输出必须尽量严格符合 JSON 格式，否则解析器可能报错。

## RunnableLambda

脚本 `19` 中间用了一个 lambda：

```python
chain = first_prompt | model | (lambda ai_msg: {"name": ai_msg.content}) | second_prompt | model | str_parser
```

这个函数的作用是把上一步的 `AIMessage` 转成第二个模板需要的字典：

```text
AIMessage(content="曹若曦") -> {"name": "曹若曦"}
```

在 LangChain 中，普通函数可以被包装成 Runnable，用来做中间数据转换。。

## RunnablePassthrough

脚本 `29` 中有这一段：

```python
chain = (
    {"input": RunnablePassthrough(), "context": retriever | format_func}
    | prompt
    | print_prompt
    | model
    | StrOutputParser()
)
```

这是 RAG 链里非常关键的写法。

输入是一个字符串：

```text
怎么减肥？
```

字典左侧会生成两个字段：

- `input`: 原样保留用户问题。
- `context`: 把用户问题交给 retriever 检索，再用 `format_func` 格式化。

结果变成：

```python
{
    "input": "怎么减肥？",
    "context": "[减肥就是要少吃多练...]"
}
```

然后这个字典再进入 prompt。

## 学习 Chain 时的核心思维

不要只看代码长什么样，要盯住每一步的输入输出类型：

```text
dict -> PromptValue -> AIMessage -> str
```

或者：

```text
str -> {"input": str, "context": str} -> PromptValue -> AIMessage -> str
```

只要你能说清楚每一步的输入输出，复杂链就不会乱。

## 易错点

- 上一步输出类型必须能被下一步接收。
- `JsonOutputParser` 依赖模型输出格式，提示词要写严格。
- lambda 函数里要清楚当前拿到的是 `AIMessage`、`str` 还是 `dict`。
- `RunnablePassthrough()` 常用于保留原始输入，不是做检索的组件。

## 本阶段检查题

- `prompt | model | parser` 三步分别输入输出什么？
- 为什么 `StrOutputParser` 经常放在模型后面？
- `JsonOutputParser` 失败时通常是什么原因？
- `RunnablePassthrough` 在 RAG 链中解决了什么问题？

## 小练习

修改 `19RunnableLambda的基础使用.py`，让第一步模型不仅返回名字，还返回一句祝福语。然后写一个 lambda，只取名字传给第二个提示词。

目标是练习：在链中间进行数据清洗和结构转换。
