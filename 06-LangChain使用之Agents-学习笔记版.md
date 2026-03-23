# 第06章 LangChain 使用之 Agents 学习笔记版

## 1. 这章到底在讲什么

这一章的核心主题只有一句话：

**让大模型不只是“回答问题”，而是“会自己决定该调用什么工具来完成任务”。**

如果把前面学过的 `Chain` 理解为“固定流程的流水线”，那么 `Agent` 就更像“带有判断能力的执行者”。

它们最本质的区别是：

- `Chain`：流程通常是提前写死的。先做 A，再做 B，再做 C。
- `Agent`：流程不是提前完全写死，而是由大模型根据当前问题动态决定下一步做什么。

比如用户问：

> “特斯拉当前股价是多少？比去年上涨了百分之几？”

如果用普通 `Chain`，你往往要提前写好：

1. 先搜当前股价。
2. 再搜去年股价。
3. 再调用计算工具。
4. 最后总结答案。

而 `Agent` 的思路是：

1. 我先判断这个问题需要哪些能力。
2. 发现需要“搜索”与“计算”。
3. 先调用搜索工具拿到股价。
4. 再调用计算工具算涨幅。
5. 最后组织自然语言答案。

这就是智能体最核心的价值：

**把“大模型的推理能力”与“外部工具的执行能力”结合起来。**

---

## 2. 为什么需要 Agent

很多初学者会有一个误区：

> “大模型已经很强了，为什么还要工具？”

原因是大模型虽然强，但有天然边界。

### 2.1 大模型的典型局限

- 不擅长获取实时信息，比如今天的天气、最新股价、刚发布的新闻。
- 纯靠参数记忆时，可能出现过时信息。
- 复杂计算、严谨逻辑、多步任务执行时，容易出错。
- 无法直接访问数据库、搜索引擎、代码解释器、业务系统。

所以，Agent 的目标并不是“替代大模型”，而是：

**让大模型学会调度外部能力。**

可以把它理解成：

- LLM 负责“想”
- Tool 负责“做”
- Agent 负责“决定什么时候让谁做”
- AgentExecutor 负责“把整个执行过程跑起来”

---

## 3. Agent 的核心组成

课件里把 Agent 的核心能力拆成了几个部分，这里我用更容易理解的话重新组织一下。

### 3.1 LLM：大脑

这是 Agent 的“决策中心”。

它负责：

- 理解用户意图
- 判断当前问题需不需要工具
- 选择合适的工具
- 决定工具调用顺序
- 基于工具结果生成最终答案

常见示例：

- `OpenAI()`
- `ChatOpenAI()`

### 3.2 Tools：手和脚

工具是 Agent 真正接触外部世界的方式。

比如：

- 搜索工具：查天气、新闻、股价
- 计算工具：算百分比、数学表达式
- 数据库工具：查订单、查用户信息
- Python 工具：执行脚本或数学计算

工具本质上就是一个“可被大模型调用的函数接口”。

### 3.3 Memory：记忆

记忆用于解决“多轮对话上下文延续”问题。

例如：

1. 你先问：“北京明天的天气怎么样？”
2. 再问：“上海呢？”

第二句并不完整，但人能理解“上海呢”是在继续追问天气。

如果没有记忆，模型可能根本不知道你在问什么。

所以 Memory 的作用就是：

- 保存历史对话
- 让 Agent 在下一轮还能接着前一轮理解

### 3.4 Planning：规划

规划指的是：

- 拆任务
- 排顺序
- 判断是否还需要继续调用工具

例如“查股价并计算涨幅”就是典型的多步规划问题。

### 3.5 Action：行动

Agent 真正执行外部操作，比如：

- 搜索
- 调用 API
- 计算
- 读取数据库

### 3.6 Collaboration：协作

更复杂的系统里，可能不是一个 Agent 单独完成所有事，而是多个 Agent 协同处理不同子任务。

这章课件提到这个概念，但重点仍然是单 Agent 的基本使用。

---

## 4. Agent、Tool、Toolkit、AgentExecutor 之间是什么关系

这是非常容易混淆的一组概念。

### 4.1 Tool

一个具体能力单元。

例如：

- `Search`
- `Calculator`

### 4.2 Toolkit

一组工具的集合。

例如：

- 搜索工具
- 计算工具
- 数据查询工具

一起给 Agent 使用。

### 4.3 Agent

Agent 是“决策者”。

它不直接做搜索，不直接做计算，而是决定：

- 当前该用哪个工具
- 先用哪个
- 后用哪个
- 什么时候结束

### 4.4 AgentExecutor

这个非常关键。

**Agent 负责想，AgentExecutor 负责跑。**

可以把 AgentExecutor 理解成 Agent 的运行时执行器。

它负责：

- 驱动 Agent 开始工作
- 调用工具
- 接收工具返回结果
- 把结果再交还给 Agent 继续思考
- 循环直到得到最终答案

所以，一般不是“直接运行 Agent”，而是通过 `AgentExecutor` 来运行。

---

## 5. Agent 和 Chain 的区别

这是面试和学习都很重要的基础题。

### 5.1 Chain 的特点

- 流程固定
- 节点顺序通常提前定义好
- 可控性强
- 适合流程清晰、步骤确定的任务

例如：

- 固定 Prompt 模板
- 固定检索后总结
- 固定格式转换

### 5.2 Agent 的特点

- 流程动态
- 由模型决定下一步动作
- 更灵活
- 更适合复杂、多步骤、需要调用外部工具的任务

### 5.3 什么时候用 Chain，什么时候用 Agent

优先用 `Chain` 的情况：

- 业务流程很固定
- 你明确知道先做什么后做什么
- 要求稳定、低成本、低延迟

优先用 `Agent` 的情况：

- 用户问题不固定
- 需要自主选择工具
- 需要多步推理
- 需要根据中间结果再决定下一步

一句话总结：

**能用 Chain 解决的问题，尽量先用 Chain；必须动态决策时，再上 Agent。**

---

## 6. LangChain 中 Agent 的两种主流模式

课件把 Agent 的核心类型归纳成两种模式：

- `Function Call` 模式
- `ReAct` 模式

这部分非常关键。

---

## 7. Function Call 模式

### 7.1 它是什么

`Function Call` 模式本质上是：

**让模型按结构化格式输出“我要调用哪个工具、传什么参数”。**

也就是模型不是输出一大段自由文本，而是更像输出一条“工具调用指令”。

比如：

```json
{
  "tool": "Search",
  "args": {
    "query": "北京天气"
  }
}
```

### 7.2 它的优点

- 输出结构化，稳定性更高
- 工具调用更直接
- 延迟通常更低
- 对于“工具明确”的任务很合适

### 7.3 它的缺点

- 通常要求底层模型支持函数调用能力
- 推理过程不如 ReAct 显性
- 有时解释性不如 ReAct

### 7.4 典型 AgentType

课件中提到的典型类型：

- `AgentType.OPENAI_FUNCTIONS`
- `AgentType.OPENAI_MULTI_FUNCTIONS`

### 7.5 适合什么场景

- 工具集合比较明确
- 更看重稳定调用
- 更在意执行效率
- 不强求模型把中间思考过程“写出来”

---

## 8. ReAct 模式

### 8.1 它是什么

`ReAct` = `Reasoning + Acting`

也就是：

- 先推理
- 再行动
- 再观察结果
- 再继续推理

这是很多 Agent 教程里最经典的模式。

### 8.2 它的典型过程

ReAct 常见流程是：

1. `Thought`：我该怎么做？
2. `Action`：我要调用哪个工具？
3. `Action Input`：给工具传什么参数？
4. `Observation`：工具返回了什么？
5. 回到 `Thought`
6. 最后输出 `Final Answer`

例如：

```text
问题：我想知道今天北京的天气
Thought：需要先搜索天气信息
Action：Search
Action Input：今天 北京 天气
Observation：查到了天气结果
Thought：我已经拿到足够信息
Final Answer：今天北京天气……
```

### 8.3 它的优点

- 推理链条更清晰
- 可解释性更强
- 多步任务时更直观
- 对“先思考再行动”的复杂任务更友好

### 8.4 它的缺点

- 输出是自然语言格式，更容易解析失败
- 延迟通常更高
- 对提示词设计要求更高

### 8.5 典型 AgentType

课件中提到：

- `AgentType.ZERO_SHOT_REACT_DESCRIPTION`
- `AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION`
- `AgentType.CONVERSATIONAL_REACT_DESCRIPTION`

可这样理解：

- `ZERO_SHOT_REACT_DESCRIPTION`：基础 ReAct
- `STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION`：面向结构化聊天场景
- `CONVERSATIONAL_REACT_DESCRIPTION`：带记忆的对话式 ReAct

---

## 9. Function Call 和 ReAct 的对比

### 9.1 底层机制

- Function Call：结构化函数调用
- ReAct：自然语言推理 + 行动

### 9.2 输出形式

- Function Call：JSON/结构化参数
- ReAct：自由文本格式

### 9.3 执行效率

- Function Call：通常更高效
- ReAct：通常更慢一些

### 9.4 解释能力

- Function Call：更像“直接调工具”
- ReAct：更像“先写出思考，再决定动作”

### 9.5 使用建议

如果你更看重：

- 稳定性
- 工具调用效率
- 结构化输出

优先考虑 `Function Call`。

如果你更看重：

- 推理过程可见
- 多步复杂任务
- 需要显式展示思考路径

优先考虑 `ReAct`。

---

## 10. Agent 的两种创建方式

课件给出了两类思路：

- 传统方式：`initialize_agent()`
- 通用方式：`create_xxx_agent() + AgentExecutor()`

---

## 11. 传统方式：initialize_agent()

### 11.1 它是什么

这是一种“快速上手”的方式。

你只要传：

- `llm`
- `tools`
- `agent` 类型

基本就能跑起来。

### 11.2 典型代码结构

```python
agent_executor = initialize_agent(
    llm=llm,
    tools=[search_tool],
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

result = agent_executor.invoke(query)
```

### 11.3 优点

- 上手快
- 代码少
- 学习成本低

### 11.4 缺点

- 定制能力弱
- 提示词往往不透明或不方便改
- 更底层的控制力有限

### 11.5 适合谁

- 初学者
- 做最小可运行 Demo
- 快速验证工具调用逻辑

---

## 12. 通用方式：create_xxx_agent + AgentExecutor

### 12.1 它是什么

先创建 Agent，再显式创建 AgentExecutor。

例如：

- `create_react_agent()`
- `create_tool_calling_agent()`

然后：

```python
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

### 12.2 优点

- 可自定义 Prompt
- Agent 与执行器职责更清晰
- 扩展性和可控性更强

### 12.3 缺点

- 代码更多
- 理解门槛更高
- 需要知道 Prompt 要包含哪些变量

### 12.4 适合谁

- 想做可控系统的人
- 想深入 Agent 原理的人
- 生产环境开发者

---

## 13. 传统方式和通用方式怎么选

### 13.1 学习阶段

先用 `initialize_agent()`。

原因：

- 更容易看到整体流程
- 不容易被 Prompt 细节绊住

### 13.2 进阶与生产阶段

逐步过渡到：

- `create_react_agent()`
- `create_tool_calling_agent()`
- `AgentExecutor(...)`

因为这套方式更灵活。

---

## 14. Agent 中工具的使用

这一章中，课件用了几个非常典型的案例来说明工具。

### 14.1 案例一：单工具查询天气

需求：

> “今天北京的天气怎么样？”

这里最典型的工具是 `TavilySearchResults`。

### 14.2 为什么用 Tavily

因为 Tavily 是专门面向 Agent/LLM 场景的搜索工具，适合做联网搜索。

它的价值在于：

- 能提供实时搜索结果
- 比单纯靠模型记忆更适合处理最新信息
- 和 LangChain 集成较方便

### 14.3 工具如何定义

课件里展示了两种写法。

#### 写法一：再包一层 Tool

```python
search = TavilySearchResults(max_results=3)
search_tool = Tool(
    name="Search",
    func=search.run,
    description="用于搜索互联网上的信息"
)
```

这种方式的好处是：

- 你可以自定义工具名
- 可以自定义描述
- 让模型更清楚工具用途

#### 写法二：直接把内置工具传进去

```python
search = TavilySearchResults(max_results=3)
tools = [search]
```

这种方式更简洁。

### 14.4 description 为什么重要

很多初学者会忽视工具描述。

但实际上，Agent 能否选对工具，很大程度依赖：

- 工具名
- 工具描述

也就是：

**工具描述不是注释，而是给大模型看的“使用说明书”。**

描述写得越清楚，模型越容易：

- 正确选择工具
- 正确传参
- 降低误调用概率

---

## 15. 多工具协同：搜索 + 计算

这是课件中非常有代表性的 Agent 例子。

问题：

> “特斯拉当前股价是多少？比去年上涨了百分之几？”

这不是一个单工具问题，而是一个典型的多步骤任务。

### 15.1 它需要哪些能力

- 搜索当前股价
- 搜索去年股价
- 数学计算涨幅百分比
- 组织最终答案

### 15.2 因此它至少需要两个工具

- `Search`
- `Calculator`

### 15.3 搜索工具

课件中用的是 `TavilySearchResults`。

### 15.4 计算工具

课件中使用的是：

```python
from langchain_experimental.utilities.python import PythonREPL
```

然后包装成计算工具：

```python
python_repl = PythonREPL()
calc_tool = Tool(
    name="Calculator",
    func=python_repl.run,
    description="用于执行数学计算，例如计算百分比变化"
)
```

### 15.5 这个案例说明了什么

它说明 Agent 最大的价值不是“单次调用一个工具”。

而是：

**能够根据任务需要，自己把多个工具串起来。**

也就是：

- 先拿数据
- 再算结果
- 最后总结

这已经明显比普通单轮问答更接近“任务执行”。

---

## 16. 自定义工具的本质

课件中还举了一个很适合初学者理解的例子：

> “计算 3 的平方”

这里作者自己定义了一个函数：

```python
def simple_calculator(expression: str) -> str:
    return str(eval(expression))
```

再包装成 Tool：

```python
math_calculator_tool = Tool(
    name="Math_Calculator",
    func=simple_calculator,
    description="用于数学计算，输入必须是纯数学表达式"
)
```

然后 Agent 会把“计算 3 的平方”转换成：

```text
3**2
```

再调用工具，得到结果 `9`。

### 16.1 这个案例要你真正理解什么

不是“平方怎么计算”，而是：

**任何普通 Python 函数，只要包装成 Tool，就能被 Agent 当作能力使用。**

这意味着你未来可以把很多业务能力都做成工具：

- 查订单
- 发邮件
- 算运费
- 查库存
- 调业务 API

---

## 17. Prompt 在 Agent 中为什么格外重要

Chain 里也有 Prompt，但 Agent 对 Prompt 的依赖更强。

因为 Agent 不是只负责生成语言，它还要：

- 选择工具
- 输出工具调用格式
- 管理思考过程

所以如果 Prompt 不对，Agent 可能根本跑不起来。

---

## 18. create_tool_calling_agent 中的 Prompt 要点

课件中在 Function Call 通用方式里，用了：

```python
ChatPromptTemplate.from_messages([
    ("system", "您是一位乐于助人的助手，请务必使用 tavily_search_results_json 工具来获取信息。"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])
```

### 18.1 为什么要有 `{agent_scratchpad}`

这是非常重要的知识点。

`agent_scratchpad` 可以理解成：

**Agent 的中间思考与中间步骤缓存区。**

它用来存：

- 前面调用过什么工具
- 工具返回了什么
- 下一步该怎么继续

如果没有它，Agent 的多步过程就可能断掉。

课件明确指出：

- 如果不传 `intermediate_steps`
- 或 Prompt 中缺少 `agent_scratchpad`
- 可能导致 `KeyError: 'intermediate_steps'`

这背后的本质是：

**Agent 是要“边做边想”的，scratchpad 就是这个“边做边想”的工作区。**

---

## 19. create_react_agent 中的 Prompt 要点

ReAct 模式比 Function Call 更依赖 Prompt 格式。

因为模型要输出类似：

- `Thought`
- `Action`
- `Action Input`
- `Observation`
- `Final Answer`

如果格式不稳定，解析器就可能失败。

### 19.1 Prompt 必须包含的核心信息

- 可用工具列表 `tools`
- 工具名称集合 `tool_names`
- 用户输入 `input`
- 中间思考 `agent_scratchpad`

课件里还提到了可以直接使用 LangChain Hub 的官方模板：

```python
prompt = hub.pull("hwchase17/react")
```

这个思路很实用，因为官方模板已经把常见必需变量都处理好了。

### 19.2 为什么官方模板有价值

因为你自己从零写 ReAct Prompt 时，最容易出错的不是“语言表达”，而是：

- 漏变量
- 漏格式
- 输出结构不符合解析器要求

官方模板能减少很多低级错误。

---

## 20. handle_parsing_errors 是什么

这是这章很实战的一个点。

### 20.1 为什么会有解析错误

在 ReAct 模式中，Agent 期待模型输出某种严格格式，比如：

```text
Thought: ...
Action: ...
Action Input: ...
```

但模型有时会直接返回一段自由文本。

这就会导致：

- 解析器看不懂
- Agent 执行失败

### 20.2 `handle_parsing_errors=True` 的作用

课件里解释得很清楚，本质上有两层作用：

- 自动捕获解析错误
- 把错误信息返回给模型，让模型重试修正

如果还不行，则会：

- 进行降级处理
- 返回更友好的失败结果

### 20.3 什么时候该开

建议：

- 生产环境：建议开
- 开发调试：可以先关，方便尽快发现问题

### 20.4 初学者怎么记

一句话记忆：

**ReAct 容易“格式不规范”，`handle_parsing_errors=True` 就是给 Agent 加一个“容错垫”。**

---

## 21. Agent 如何接入记忆

课件后半部分重点讲了这一点。

### 21.1 为什么需要记忆

没有记忆时：

1. 用户问：“北京明天的天气怎么样？”
2. 再问：“上海呢？”

第二句会缺主语，系统可能不知道是在问“上海明天的天气”。

有记忆时，Agent 可以根据历史对话补全语义。

### 21.2 课件中使用的记忆类型

```python
ConversationBufferMemory
```

这是最基础、最直观的一种记忆方式。

它的本质就是：

**把历史对话原样缓存在上下文里。**

### 21.3 关键参数：`memory_key="chat_history"`

这个点非常重要。

课件强调：

```python
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)
```

为什么要这样写？

因为 Prompt 中往往要用：

```python
("placeholder", "{chat_history}")
```

也就是说：

- Memory 存的数据键叫 `chat_history`
- Prompt 取历史对话时也必须用 `chat_history`

两边要对上。

如果对不上，就会出现上下文无法注入的问题。

### 21.4 `return_messages=True` 是什么意思

表示记忆以“消息对象”的形式返回，而不是简单字符串。

这对聊天模型更自然，也更适合多轮对话。

---

## 22. 传统方式下接入记忆

在传统方式中，课件使用：

```python
agent_executor = initialize_agent(
    tools=[search_tool],
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)
```

这说明：

- 使用对话型 ReAct AgentType
- 再把 memory 注入进去

这样第二轮问“上海呢”时，Agent 才能接住上下文。

---

## 23. 通用方式下接入记忆

在通用方式中，记忆通常既和 Prompt 有关，也和 AgentExecutor 有关。

例如课件中的思路：

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有用的助手，可以回答问题并使用工具。"),
    ("placeholder", "{chat_history}"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])
```

然后：

```python
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)
```

最后：

```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True
)
```

### 23.1 这里的关键理解

通用方式下，记忆不是“神奇自动生效”的。

而是要满足两件事：

1. Prompt 里留出 `chat_history` 占位符
2. Executor 里把 memory 真正传进去

少一个都可能不工作。

---

## 24. 这章代码里最值得你真正掌握的知识点

如果你是 AI 初学者，我建议不要死记所有 API，而要抓住以下几个“骨架级”知识点。

### 24.1 第一层：Agent 不是模型，Agent 是“模型 + 工具调度机制”

很多新手会把 Agent 理解成一种新的模型。

这其实不对。

更准确地说：

**Agent 是一种应用架构。**

它把：

- 模型推理能力
- 工具调用能力
- 任务执行流程

组合在一起。

### 24.2 第二层：Tool 决定“能做什么”

如果不给 Agent 工具，它再聪明也只能“想”，不能真正“做”。

### 24.3 第三层：Prompt 决定“怎么做”

尤其在 ReAct 中，Prompt 直接决定：

- 工具调用格式
- 输出规范
- 解析是否成功

### 24.4 第四层：Memory 决定“能不能连续对话”

没有记忆，多轮对话体验会差很多。

### 24.5 第五层：AgentExecutor 决定“能不能跑起来”

它是运行时，不只是一个普通包装器。

---

## 25. 这章容易踩的坑

### 25.1 以为 Agent 会自动联网

不会。

模型本身默认并不会联网。

如果你想查实时天气，必须给它搜索工具。

### 25.2 以为工具 description 无所谓

错。

description 很重要，它直接影响模型选工具的准确率。

### 25.3 ReAct Prompt 漏掉 `agent_scratchpad`

很容易报错，或者中间步骤无法正常传递。

### 25.4 记忆里 `memory_key` 和 Prompt 占位符不一致

比如一个叫 `chat_history`，另一个写成 `history`，就可能出问题。

### 25.5 以为多轮对话“自然就有上下文”

不会。

如果你没有接入 Memory，多轮上下文不会可靠保留。

### 25.6 自定义工具输入约束写得太模糊

模型可能乱传参数，导致工具调用失败。

所以工具说明最好明确：

- 输入类型
- 输入格式
- 不支持什么

---

## 26. 从“代码会写”到“真正理解”之间的关键跨越

如果你只看代码，容易停留在：

- 会导包
- 会复制示例
- 会跑天气查询

但真正的理解应该更进一步：

### 26.1 你要知道 Agent 解决的是哪类问题

它解决的是：

**流程不固定、需要动态决策、需要工具协作的问题。**

### 26.2 你要知道为什么要有 ReAct

因为很多任务不是一次工具调用能解决，而是：

- 思考一次
- 做一步
- 看结果
- 再继续

### 26.3 你要知道为什么要有 Function Call

因为很多实际业务不需要把“思考过程”写出来，更需要：

- 稳定
- 结构化
- 高效

### 26.4 你要知道为什么 Prompt 和 Memory 在 Agent 中更敏感

因为 Agent 本身是一个“循环执行系统”，不是单次文本生成。

---

## 27. 一套适合初学者的学习顺序

如果你现在刚接触 Agent，我建议你按下面顺序学。

### 第一步：先理解概念，不急着背 API

先彻底搞懂：

- Agent 和 Chain 的区别
- Tool、Agent、AgentExecutor 的关系
- Function Call 和 ReAct 的区别

### 第二步：只做单工具案例

比如：

- 搜索天气
- 搜索新闻

先确认你能理解：

- 为什么需要工具
- Agent 是怎么决定调用工具的

### 第三步：做多工具案例

比如：

- 查股价 + 算涨幅

这一步能让你真正体会 Agent 的价值。

### 第四步：做自定义工具

比如：

- 自己写一个 `simple_calculator`
- 再包装成 Tool

这一步会让你从“会用别人的工具”升级成“会给 Agent 增加新能力”。

### 第五步：做带记忆对话

比如：

- 第一轮问北京天气
- 第二轮问上海呢

这一步会帮你理解 Memory。

### 第六步：再深入通用方式

开始自己写：

- `PromptTemplate`
- `ChatPromptTemplate`
- `create_react_agent`
- `create_tool_calling_agent`

---

## 28. 这一章的整体知识地图

你可以把这一章整理成下面这张脑图。

### 28.1 基础认知层

- 什么是 Agent
- 为什么需要 Agent
- Agent 和 Chain 的区别

### 28.2 组件层

- LLM
- Tools
- Memory
- Planning
- Action
- AgentExecutor

### 28.3 类型层

- Function Call 模式
- ReAct 模式

### 28.4 使用层

- `initialize_agent()`
- `create_xxx_agent() + AgentExecutor()`

### 28.5 实战层

- 单工具天气查询
- 多工具查股价并计算涨幅
- 自定义数学工具
- 接入记忆实现多轮对话

### 28.6 容错层

- `handle_parsing_errors=True`
- `agent_scratchpad`
- `chat_history`

---

## 29. 一段适合你背下来的总结

`Agent` 是 LangChain 中用于构建智能体的一种机制。它和固定流程的 `Chain` 不同，能够借助大模型的推理能力，动态决定调用哪些工具、按什么顺序调用，并根据中间结果继续推进任务。一个完整的 Agent 系统通常包括 LLM、Tools、Memory、Agent 和 AgentExecutor。其中，LLM 负责推理决策，Tools 负责执行外部能力，Memory 负责维护多轮上下文，AgentExecutor 负责驱动整个运行流程。

在实现方式上，LangChain 中常见的 Agent 主要有两种模式：`Function Call` 和 `ReAct`。前者更偏结构化工具调用，效率更高；后者更偏“思考-行动-观察”的循环过程，可解释性更强。工程实践中，`initialize_agent()` 适合快速入门，而 `create_react_agent()`、`create_tool_calling_agent()` 配合 `AgentExecutor()` 更适合做可控和可定制的系统。

如果想真正学会 Agent，不要只停留在“能跑代码”，而要真正理解它为什么需要工具、为什么需要 Prompt、为什么需要 Memory，以及为什么 ReAct 会依赖 `agent_scratchpad`。只有把这些骨架想明白，后面你做检索、工作流、RAG、甚至多智能体系统时，才不会只是机械套 API。

---

## 30. 你的最小行动建议

如果你准备继续往下学，我建议你立刻亲手做这 4 个小实验：

1. 只用一个搜索工具，做“天气查询 Agent”。
2. 加一个计算工具，做“股价涨幅 Agent”。
3. 自己写一个 Python 函数，包装成 Tool。
4. 给 Agent 加上 Memory，做两轮连续问答。

做完这 4 个实验，你对这一章的理解会从“看懂”变成“真正会用”。

