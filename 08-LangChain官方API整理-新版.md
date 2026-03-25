# 《08-LangChain 官方 API 整理》新版速查版

## 1. 先说结论：新版 LangChain 应该怎么理解

如果你现在学习的是 `2025-2026` 这一代 LangChain Python 生态，那么最重要的认知不是死记很多类名，而是先建立下面这张图：

- `langchain`：主入口，偏“应用层”和“Agent 层”
- `langchain-core`：核心抽象层，偏接口、消息、提示词、Runnable、输出解析
- `langchain-text-splitters`：文本切分
- `langchain-classic`：老版本经典实现，很多旧教程还在用
- `langchain-xxx`：各类 provider/integration 包，比如 `langchain-openai`、`langchain-anthropic`

一句话理解：

`新版 LangChain = langchain 主包 + langchain-core 抽象层 + 各 provider 独立安装`

所以现在学习 API，最好分成两条线：

1. `应用主线 API`：`create_agent`、`init_chat_model`、`@tool`
2. `底层组合 API`：`PromptTemplate`、`ChatPromptTemplate`、`Runnable`、`OutputParser`

---

## 2. 安装层 API

### 2.1 基础安装

```bash
pip install -U langchain
```

### 2.2 安装 provider 集成

```bash
pip install -U langchain-openai
pip install -U langchain-anthropic
```

也可以按官方推荐写成 extras 形式：

```bash
pip install -U "langchain[openai]"
pip install -U "langchain[anthropic]"
```

### 2.3 这和旧版最大不同

旧教程常常会默认很多东西都在一个 `langchain` 包里。

新版更强调：

- 通用抽象在 `langchain` / `langchain-core`
- 具体模型实现放到独立 provider 包
- 你要用谁，就额外装谁

---

## 3. 当前最核心的官方 API 地图

按照官方新版文档，核心组件主要是：

- `Agents`
- `Models`
- `Messages`
- `Tools`
- `Short-term memory`
- `Streaming`
- `Structured output`
- `Retrieval`

如果再细分成“最常写代码的 API”，可以整理成下面这张表。

| 模块 | 常用 API | 作用 |
| --- | --- | --- |
| 模型初始化 | `init_chat_model()` | 快速初始化聊天模型 |
| Agent | `create_agent()` | 创建官方主线智能体 |
| 消息 | `SystemMessage` `HumanMessage` `AIMessage` `ToolMessage` | 标准化对话消息 |
| 工具 | `@tool` `ToolRuntime` | 定义工具、读状态、读写 store |
| 提示词 | `PromptTemplate` `ChatPromptTemplate` | 构造 prompt |
| Runnable | `Runnable` `RunnableLambda` `RunnableParallel` `RunnablePassthrough` | 组合执行流 |
| 输出解析 | `StrOutputParser` `JsonOutputParser` | 解析模型输出 |
| 结构化输出 | `response_format` `ProviderStrategy` `ToolStrategy` | 让输出变成结构化对象 |
| 检索 | `Document`、loader、splitter、embeddings、vector store、retriever | 搭建 RAG |

---

## 4. Models 相关 API

## 4.1 `init_chat_model()`

这是新版最推荐的模型初始化入口之一。

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-5.2")
```

也可以带 provider 前缀：

```python
model = init_chat_model("google_genai:gemini-2.5-flash-lite")
model = init_chat_model("azure_openai:gpt-5.2", azure_deployment="xxx")
```

### 4.2 模型对象最常见方法

- `invoke()`：单次调用，最常用
- `stream()`：流式输出
- `batch()`：批量调用
- `bind_tools()`：把工具绑定给模型，启用 tool calling

示例：

```python
response = model.invoke("为什么鹦鹉羽毛颜色鲜艳？")
```

### 4.3 `bind_tools()`

当你不想直接上 Agent，只是想让模型具备调用工具的能力时，可以先绑定工具：

```python
model_with_tools = model.bind_tools([get_weather])
resp = model_with_tools.invoke("北京和上海今天哪个更热？")
```

常见能力：

- 支持并行工具调用
- 可设置 `tool_choice="any"` 强制使用工具
- 某些模型支持 `parallel_tool_calls=False`

### 4.4 模型层学习重点

学习时优先掌握：

1. `init_chat_model`
2. `invoke / stream / batch`
3. `bind_tools`
4. 模型参数：`temperature`、`max_tokens`、`timeout`

---

## 5. Messages 相关 API

LangChain 用统一消息对象表示对话，而不是只传普通字符串。

常见消息类：

- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`

示例：

```python
from langchain.messages import SystemMessage, HumanMessage

messages = [
    SystemMessage("你是一个资深 Python 工程师"),
    HumanMessage("帮我写一个 REST API 示例"),
]

response = model.invoke(messages)
```

### 5.1 各消息类怎么理解

- `SystemMessage`：系统设定，定义角色和规则
- `HumanMessage`：用户输入
- `AIMessage`：模型输出
- `ToolMessage`：工具执行结果回传给模型

### 5.2 为什么这个 API 很重要

因为新版很多能力都建立在“消息流”上：

- Agent 运行时的状态核心就是 `messages`
- 多轮对话本质上是消息列表不断追加
- Tool calling 过程里会出现 `AIMessage.tool_calls` 和 `ToolMessage`

---

## 6. Tools 相关 API

## 6.1 `@tool`

这是定义工具最常见的写法：

```python
from langchain.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """Search the customer database for records matching the query."""
    return f"Found {limit} results for '{query}'"
```

学习这个 API 时要注意三点：

- 函数签名就是工具输入 schema 的基础
- 类型注解很重要
- docstring 很重要，因为模型靠它理解“什么时候该调用这个工具”

### 6.2 自定义工具名和描述

```python
@tool("web_search", description="搜索互联网信息")
def search(query: str) -> str:
    return f"Results for: {query}"
```

### 6.3 `ToolRuntime`

新版工具不只是“收参数然后返回结果”，还可以在工具内部访问运行时上下文。

常见能力：

- `runtime.state`：访问当前状态，如消息历史
- `runtime.store`：访问长期/共享存储
- `runtime.stream_writer`：输出工具执行中的流式进度
- `runtime.tool_call_id`：与当前 tool call 对应

示例思路：

```python
from langchain.tools import tool, ToolRuntime

@tool
def get_last_user_message(runtime: ToolRuntime) -> str:
    messages = runtime.state["messages"]
    return messages[-1].content
```

### 6.4 工具返回值

工具可以返回：

- `str`：给模型读的普通文本
- `dict`：结构化对象
- 某些场景下返回 `Command`：直接更新图状态

---

## 7. Agents 相关 API

## 7.1 `create_agent()`

这是新版 LangChain 官方主线智能体 API。

```python
from langchain.agents import create_agent

agent = create_agent(
    model="gpt-5",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)
```

运行：

```python
result = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

### 7.2 `create_agent()` 里最值得关注的参数

- `model`：字符串或模型实例
- `tools`：工具列表
- `system_prompt`：系统提示词
- `middleware`：中间件列表
- `response_format`：结构化输出配置
- `context_schema`：运行时上下文 schema
- `store`：给 agent/tool 持久化用
- `name`：Agent 名称

### 7.3 新版 Agent 的核心特点

相比旧版 `initialize_agent()` / `AgentExecutor` 风格，新版更强调：

- 统一入口是 `create_agent`
- 底层基于 `LangGraph`
- 支持 middleware
- 支持动态模型选择
- 支持动态工具过滤
- 支持 structured output

### 7.4 动态模型选择

可以通过中间件 `@wrap_model_call` 在运行时切模型。

适合场景：

- 短对话用便宜模型
- 长对话/复杂推理切换到更强模型

### 7.5 动态工具选择

可以基于：

- 当前状态 `state`
- 运行时上下文 `context`
- store 中的用户偏好/权限

这说明新版 Agent 已经不只是“LLM + tools”，而是一个可插拔、可控制的执行框架。

---

## 8. Prompt 相关 API

Prompt 主要在 `langchain-core` 里。

常见类：

- `PromptTemplate`
- `ChatPromptTemplate`

### 8.1 `PromptTemplate`

适合普通字符串模板：

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("请用一句话解释 {topic}")
text = prompt.format(topic="RAG")
```

### 8.2 `ChatPromptTemplate`

适合聊天模型消息模板：

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个 AI 助手"),
    ("human", "请解释 {topic}"),
])
```

### 8.3 学习建议

如果你主要写聊天应用，优先掌握：

- `ChatPromptTemplate.from_messages`
- 变量占位
- 和 `Runnable` / model 的管道组合

---

## 9. Runnable 相关 API

这是 LangChain 很底层、也很核心的一组抽象。

一句话理解：

`Runnable = 一个可 invoke / stream / batch / 组合 的执行单元`

常见类：

- `Runnable`
- `RunnableLambda`
- `RunnableSequence`
- `RunnableParallel`
- `RunnablePassthrough`

### 9.1 最经典的组合方式：`|`

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("讲一个关于 {topic} 的冷笑话")
chain = prompt | model | StrOutputParser()
result = chain.invoke({"topic": "数据库"})
```

### 9.2 `RunnableLambda`

把普通 Python 函数包装成 `Runnable`：

```python
from langchain_core.runnables import RunnableLambda

step = RunnableLambda(lambda x: x.upper())
print(step.invoke("hello"))
```

### 9.3 `RunnableParallel`

让多个步骤并行处理同一个输入。

### 9.4 `RunnablePassthrough`

输入透传，常用于：

- 保留原始输入
- 给流水线补字段
- 组装复杂上下文

### 9.5 Runnable 的通用方法

- `invoke`
- `ainvoke`
- `batch`
- `abatch`
- `stream`
- `with_retry`
- `with_fallbacks`
- `with_config`

这套 API 很值得单独练，因为它决定了你能不能把 LangChain 用成“工程化组合框架”，而不是只会写几个 demo。

---

## 10. Output Parser 相关 API

常见输出解析器：

- `StrOutputParser`
- `JsonOutputParser`
- `PydanticOutputParser`

### 10.1 `StrOutputParser`

最简单，直接把模型输出抽成字符串：

```python
from langchain_core.output_parsers import StrOutputParser

chain = prompt | model | StrOutputParser()
```

### 10.2 什么时候需要 parser

- 你不想手动从 `AIMessage` 里取文本
- 你想转 JSON
- 你想转成 Pydantic 对象
- 你想把输出接给下游代码继续处理

---

## 11. Structured Output 相关 API

这是新版非常值得重点学的一块。

核心入口：`create_agent(..., response_format=...)`

### 11.1 `response_format`

官方定义里常见几种写法：

- `response_format=SchemaType`
- `response_format=ProviderStrategy(SchemaType)`
- `response_format=ToolStrategy(SchemaType)`

### 11.2 `ProviderStrategy`

适用于 provider 原生支持结构化输出的场景。

```python
from pydantic import BaseModel, Field
from langchain.agents import create_agent

class ContactInfo(BaseModel):
    name: str = Field(description="姓名")
    email: str = Field(description="邮箱")
    phone: str = Field(description="电话")

agent = create_agent(
    model="gpt-5",
    response_format=ContactInfo,
)
```

如果模型支持原生 structured output，LangChain 会优先走 `ProviderStrategy`。

### 11.3 `ToolStrategy`

适用于不支持原生 structured output，但支持 tool calling 的模型。

```python
from langchain.agents.structured_output import ToolStrategy

agent = create_agent(
    model="gpt-5",
    response_format=ToolStrategy(ContactInfo),
)
```

它还支持：

- `tool_message_content`
- `handle_errors`

这对于“抽取类任务”特别实用，比如：

- 信息提取
- 分类
- 评分
- 审核结果结构化返回

---

## 12. Streaming 相关 API

新版 LangChain 非常重视流式交互。

常见方式：

- `model.stream(...)`
- `agent.stream(...)`

### 12.1 Agent 流式模式

官方文档里常见：

- `stream_mode="updates"`
- `stream_mode="custom"`
- 也可以同时传多个 mode

示意：

```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    stream_mode=["updates", "custom"],
    version="v2",
):
    print(chunk)
```

流式的价值在于：

- 实时展示模型 token 输出
- 观察工具调用过程
- 展示长任务执行进度

---

## 13. Retrieval / RAG 相关 API

官方新版对 Retrieval 的描述更偏“搭积木”。

核心构件包括：

- `Document loaders`
- `Text splitters`
- `Embedding models`
- `Vector stores`
- `Retrievers`

### 13.1 检索链路怎么记

可以记成：

`原始文档 -> loader -> splitter -> embeddings -> vector store -> retriever -> LLM`

### 13.2 相关常见包/对象

- `Document`
- 各类 loader
- `RecursiveCharacterTextSplitter`
- embedding 模型
- vector store
- retriever

### 13.3 官方对 RAG 的三种常见架构

- `2-step RAG`
- `Agentic RAG`
- `Hybrid RAG`

理解重点：

- `2-step RAG`：先检索，再生成，流程稳定
- `Agentic RAG`：Agent 自己决定何时检索
- `Hybrid RAG`：在两者之间折中，增加验证/检查步骤

---

## 14. 新旧 API 对照：现在最容易混淆的地方

下面这张表很重要，因为你在网上看到的大量教程仍然是旧写法。

| 旧教程常见写法 | 新版主线更推荐 | 说明 |
| --- | --- | --- |
| `OpenAI()` / `ChatOpenAI()` 直接硬写很多初始化 | `init_chat_model()` 或 provider 类 | 新版更强调统一初始化入口 |
| `LLMChain` | `prompt | model | parser` | Runnable 风格更主流 |
| `SequentialChain` | `RunnableSequence` / `|` | 管道式组合更自然 |
| `initialize_agent()` | `create_agent()` | 新版官方主推入口 |
| 旧 `Memory` 系列类 | `Short-term memory` + `state/store` + middleware | 新版更偏图状态与运行时状态 |
| 大量能力都在 `langchain` 主包 | provider 独立包 | 新版生态拆分更清晰 |

### 14.1 为什么很多旧教程学起来“哪里都不太对”

因为现在官方已经明确保留了：

- `langchain`
- `langchain-core`
- `langchain-classic`

这意味着：

- 老 API 没完全消失
- 但它们不再是官方最推荐主线
- 学新项目时，优先跟新版主线走

---

## 15. 最推荐的学习顺序

如果你要真正把 API 学扎实，我建议按这个顺序：

1. `init_chat_model`
2. `SystemMessage / HumanMessage / AIMessage`
3. `PromptTemplate / ChatPromptTemplate`
4. `invoke / stream / batch`
5. `StrOutputParser`
6. `Runnable` 管道：`prompt | model | parser`
7. `@tool`
8. `bind_tools`
9. `create_agent`
10. `response_format` / structured output
11. Retrieval / RAG 组件
12. middleware / state / store

这个顺序比一上来就啃 Agent 更稳，因为：

- 你先懂“模型调用”
- 再懂“流水线组合”
- 最后再懂“智能体调度”

---

## 16. 一份最小新版 LangChain 示例

```python
from langchain.chat_models import init_chat_model
from langchain.tools import tool
from langchain.agents import create_agent

model = init_chat_model("gpt-5.2")

@tool
def get_weather(city: str) -> str:
    """查询城市天气"""
    return f"{city} is sunny."

agent = create_agent(
    model=model,
    tools=[get_weather],
    system_prompt="你是一个简洁的 AI 助手",
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "上海天气怎么样？"}]}
)

print(result)
```

如果你能真正看懂这个最小示例背后的结构，其实就已经摸到新版 LangChain 主脉络了：

- `model` 负责推理
- `tool` 负责行动
- `agent` 负责调度
- `messages` 负责承载上下文

---

## 17. 你现在最该记住的 API

如果只保留一页速记，我建议你记下面这些：

- `from langchain.chat_models import init_chat_model`
- `from langchain.agents import create_agent`
- `from langchain.tools import tool, ToolRuntime`
- `from langchain.messages import SystemMessage, HumanMessage, AIMessage, ToolMessage`
- `from langchain_core.prompts import PromptTemplate, ChatPromptTemplate`
- `from langchain_core.output_parsers import StrOutputParser, JsonOutputParser`
- `from langchain_core.runnables import RunnableLambda, RunnableParallel, RunnablePassthrough`
- `from langchain.agents.structured_output import ToolStrategy, ProviderStrategy`

---

## 18. 参考来源

以下内容主要依据 2026-03-25 可访问的官方入口整理：

- LangChain GitHub: https://github.com/langchain-ai/langchain
- LangChain Overview: https://docs.langchain.com/oss/python/langchain/overview
- Install: https://docs.langchain.com/oss/python/langchain/install
- Models: https://docs.langchain.com/oss/python/langchain/models
- Messages: https://docs.langchain.com/oss/python/langchain/messages
- Tools: https://docs.langchain.com/oss/python/langchain/tools
- Agents: https://docs.langchain.com/oss/python/langchain/agents
- Structured Output: https://docs.langchain.com/oss/python/langchain/structured-output
- Streaming: https://docs.langchain.com/oss/python/langchain/streaming
- Retrieval: https://docs.langchain.com/oss/python/langchain/retrieval
- Python Reference: https://reference.langchain.com/python/langchain/

这份文档的目标不是覆盖所有细枝末节，而是把“新版 LangChain 到底该学哪些 API”整理成一个清晰骨架。
