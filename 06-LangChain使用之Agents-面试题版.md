# 第06章 LangChain 使用之 Agents 面试题版

## 1. 什么是 Agent

### 标准回答

Agent 是一种基于大语言模型的任务执行机制。它会把 LLM 作为决策中枢，根据用户问题动态选择工具、决定调用顺序，并结合工具返回结果完成更复杂的任务。和固定流程的 Chain 相比，Agent 具备更强的动态规划与自主决策能力。

### 白话解释

Chain 像提前写好的流水线，Agent 像会临场判断下一步该做什么的“智能执行者”。

### 面试加分点

- Agent 本质不是一个单独模型，而是一种“模型 + 工具 + 执行流程”的应用架构。
- Agent 的核心价值在于把 LLM 的推理能力和外部工具能力结合起来。

---

## 2. Agent 和 Chain 有什么区别

### 标准回答

Chain 的执行流程通常是固定的，适合步骤明确、结构稳定的任务；Agent 的执行流程是动态的，LLM 会根据当前任务和中间结果自主决定下一步动作，因此更适合复杂、多步骤、需要工具协作的场景。

### 可直接背诵版

- Chain：固定流程，强可控。
- Agent：动态决策，更灵活。

### 深一点的回答

两者差别不只是 API 不同，而是控制权不同：

- Chain 的控制权主要在开发者手里。
- Agent 的部分控制权交给了模型。

因此，Chain 通常更稳定、成本更低；Agent 更智能，但也更复杂、更容易出现不确定性。

---

## 3. 为什么需要 Agent

### 标准回答

因为大模型虽然具备较强的语言理解和推理能力，但在实时信息获取、精确计算、外部系统访问、多步任务执行等方面存在局限。Agent 可以让模型借助工具获取实时数据、完成计算、访问数据库或 API，从而显著扩展模型能力边界。

### 关键词

- 实时信息
- 工具调用
- 多步任务
- 动态规划

### 高频追问

#### 追问：大模型不是已经很强了吗，为什么还需要工具？

可以回答：

大模型擅长“基于已有知识做生成和推理”，但不天然具备联网搜索、查数据库、执行代码、读取业务系统的能力。工具是 Agent 接触外部世界的接口。

---

## 4. Agent 的核心组成有哪些

### 标准回答

一个完整的 Agent 系统通常包括以下几个部分：

- `LLM`：负责推理、规划和决策
- `Tools`：负责执行外部能力
- `Memory`：负责保存多轮上下文
- `Planning`：负责任务拆解和步骤安排
- `Action`：负责实际执行
- `AgentExecutor`：负责调度 Agent 与工具执行整个流程

### 面试表达优化版

可以把它概括为：

- LLM 是大脑
- Tool 是手脚
- Memory 是上下文记忆
- AgentExecutor 是运行时调度器

---

## 5. Tool、Toolkit、Agent、AgentExecutor 分别是什么

### 标准回答

- `Tool`：单个可调用能力单元，比如搜索、计算、数据库查询。
- `Toolkit`：多个工具的集合。
- `Agent`：决策者，负责判断调用什么工具、什么时候结束。
- `AgentExecutor`：执行器，负责驱动 Agent 循环运行、调用工具并返回最终结果。

### 高频易错点

很多人会把 AgentExecutor 说成“只是个包装器”，这不够准确。

更准确的说法是：

**AgentExecutor 是 Agent 的运行时。**

它承担了执行循环、工具调度和结果回传的职责。

---

## 6. LangChain 中 Agent 有哪些主流模式

### 标准回答

课件主要介绍了两种主流模式：

- `Function Call` 模式
- `ReAct` 模式

---

## 7. 什么是 Function Call 模式

### 标准回答

Function Call 模式是一种基于结构化函数调用的 Agent 方式。模型不会只输出自然语言，而是按结构化格式生成需要调用的工具和参数。这样可以提高工具调用的稳定性和效率，适合工具边界清晰、调用目标明确的场景。

### 关键词

- 结构化输出
- JSON 参数
- 稳定调用
- 效率更高

### 常见 AgentType

- `AgentType.OPENAI_FUNCTIONS`
- `AgentType.OPENAI_MULTI_FUNCTIONS`

### 适用场景

- 工具调用路径明确
- 追求高效稳定
- 底层模型支持函数调用能力

---

## 8. 什么是 ReAct 模式

### 标准回答

ReAct 是 `Reasoning + Acting` 的组合。它通过“思考-行动-观察-再思考”的循环方式完成任务。模型会先分析当前问题，决定是否需要调用工具，然后根据工具返回结果继续推理，直到得到最终答案。

### 常见流程

- `Thought`
- `Action`
- `Action Input`
- `Observation`
- `Final Answer`

### 适用场景

- 多步任务
- 需要展示推理路径
- 需要更强解释性

### 常见 AgentType

- `AgentType.ZERO_SHOT_REACT_DESCRIPTION`
- `AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION`
- `AgentType.CONVERSATIONAL_REACT_DESCRIPTION`

---

## 9. Function Call 和 ReAct 的区别是什么

### 标准回答

Function Call 更强调结构化工具调用，输出更稳定、执行更高效；ReAct 更强调“推理 + 行动”的可解释过程，适合复杂多步任务，但对 Prompt 和解析器要求更高，也更容易出现格式解析问题。

### 对比式回答

- 底层机制：Function Call 是结构化调用，ReAct 是自然语言推理。
- 输出形式：Function Call 偏 JSON，ReAct 偏自由文本。
- 执行效率：Function Call 往往更高。
- 可解释性：ReAct 更强。
- 鲁棒性风险：ReAct 更容易发生解析失败。

### 面试加分总结

如果系统更强调稳定性和工程效率，优先考虑 Function Call；如果更强调多步推理过程和可解释性，优先考虑 ReAct。

---

## 10. `initialize_agent()` 和 `create_xxx_agent()` 的区别

### 标准回答

`initialize_agent()` 属于传统快速创建方式，配置简单，适合快速上手；`create_xxx_agent()` 配合 `AgentExecutor()` 属于更通用、更灵活的方式，支持自定义 Prompt 和更清晰的组件拆分，适合进阶和生产环境。

### 对比回答

- `initialize_agent()`：简单、上手快、定制性弱
- `create_xxx_agent()`：代码多、理解成本高、但扩展性强

### 面试建议说法

如果是 Demo、教学和快速验证，`initialize_agent()` 很方便；如果是正式项目，我更倾向于用 `create_react_agent()` 或 `create_tool_calling_agent()` 再显式创建 `AgentExecutor`，因为这样控制力更强。

---

## 11. `AgentExecutor` 的作用是什么

### 标准回答

`AgentExecutor` 是 Agent 的运行时执行器。它负责接收用户输入，驱动 Agent 推理，调用工具，接收工具结果，再将结果反馈给 Agent 继续推理，直到产出最终答案。

### 一句话版本

Agent 负责决策，AgentExecutor 负责把决策真正执行起来。

### 高频追问

#### 追问：为什么不能只创建 Agent？

因为 Agent 更像“策略对象”或“决策逻辑”，真正的循环执行、工具调度与流程控制通常由 AgentExecutor 负责。

---

## 12. Tool 的 `description` 为什么重要

### 标准回答

因为大模型在选择工具时，并不是像程序那样靠硬编码匹配，而是依赖工具名称和描述来理解工具用途。`description` 写得越准确，模型越容易正确选择工具并正确传参。

### 加分表达

在 Agent 系统中，工具描述本质上就是给 LLM 的“工具使用说明书”，不是普通注释。

### 反例

如果描述写得过于模糊，模型可能：

- 选错工具
- 传错参数
- 让工具失效

---

## 13. LangChain 中如何定义自定义工具

### 标准回答

可以先定义一个普通 Python 函数，再将其封装为 `Tool` 对象，并提供名称、函数入口以及清晰的功能描述。这样 Agent 就能把它当作一个可调用能力来使用。

### 课件案例

自定义了一个 `simple_calculator(expression: str)` 函数，用于执行数学表达式计算，然后包装成 `Math_Calculator` 工具，最终让 Agent 自动完成“计算 3 的平方”。

### 面试加分点

这说明 Agent 的扩展能力非常强，任何业务函数理论上都可以被封装为工具。

---

## 14. 多工具协同的典型场景是什么

### 标准回答

典型场景是一个任务需要多个能力串联完成，比如“查询股价并计算同比涨幅”。这个任务既需要搜索工具获取当前股价和历史股价，也需要计算工具计算涨跌百分比。Agent 的价值就在于能够根据任务动态组合这些工具。

### 关键词

- 搜索 + 计算
- 多步推理
- 中间结果驱动下一步动作

### 面试加分总结

这类题可以体现 Agent 相比普通问答系统的工程价值，因为它不是简单回答，而是在执行任务。

---

## 15. 为什么说 ReAct 更容易出现解析错误

### 标准回答

因为 ReAct 模式依赖模型按照特定文本格式输出，例如 `Thought`、`Action`、`Action Input`、`Observation` 等标签。如果模型直接输出自由文本，解析器就可能无法识别，导致执行失败。

### 高频考点

这也是为什么实际开发中经常要设置：

```python
handle_parsing_errors=True
```

---

## 16. `handle_parsing_errors=True` 有什么作用

### 标准回答

它用于提高 Agent 在解析工具调用或中间输出时的容错能力。当模型输出不符合预期格式时，系统不会立刻报错退出，而是可以尝试把错误反馈给模型，让模型修正并重试；如果依然失败，则可能进行降级处理，返回更友好的错误信息。

### 面试总结版

- 开发阶段：可以关闭，方便暴露问题
- 生产阶段：建议开启，提高鲁棒性

### 加分点

这个参数体现了 Agent 工程化中的“容错设计”思路。

---

## 17. `agent_scratchpad` 是什么

### 标准回答

`agent_scratchpad` 是 Agent 用于保存中间思考过程和中间执行步骤的占位区域。特别是在 ReAct 或通用 Agent 创建方式中，它用于承接前面工具调用产生的中间结果，帮助模型在多步任务中保持上下文连续性。

### 为什么重要

如果 Prompt 中缺少 `agent_scratchpad`，Agent 可能无法正常传递 `intermediate_steps`，甚至出现 `KeyError: 'intermediate_steps'` 等错误。

### 白话理解

它就像 Agent 的草稿纸。

---

## 18. `chat_history` 为什么重要

### 标准回答

`chat_history` 通常用于在多轮对话中存储历史消息，配合 Memory 组件注入到 Prompt 中，让 Agent 能理解上下文并继续之前的话题。

### 课件中的关键点

在使用 `ConversationBufferMemory` 时，常见写法是：

```python
memory_key="chat_history"
```

Prompt 中也要有：

```python
("placeholder", "{chat_history}")
```

两者要保持一致。

### 高频易错点

如果 Memory 的 key 和 Prompt 占位符不一致，多轮上下文就无法正确注入。

---

## 19. 为什么 Agent 需要 Memory

### 标准回答

因为很多对话不是单轮孤立的，后续问题往往依赖前文信息。Memory 的作用是保存历史对话，使 Agent 在多轮交互中保持上下文一致性，从而支持省略式表达、连续追问和上下文推理。

### 课件中的典型例子

第一轮问：

> 北京明天的天气怎么样？

第二轮问：

> 上海呢？

如果没有记忆，Agent 很可能无法正确理解“上海呢”是在继续问天气。

---

## 20. `ConversationBufferMemory` 是什么

### 标准回答

`ConversationBufferMemory` 是一种最基础的对话记忆方式，它会把历史对话直接缓存在上下文中，适合多轮对话场景。它实现简单、直观，但随着对话变长，可能带来上下文膨胀问题。

### 面试加分点

你可以顺带补一句：

它适合短中等长度对话，但如果历史过长，后续可能需要摘要记忆或窗口记忆来控制上下文成本。

---

## 21. `return_messages=True` 的作用是什么

### 标准回答

它表示 Memory 在返回历史记录时，以消息对象列表而不是普通字符串的形式返回。这种形式更适合聊天模型，也更方便直接注入 `ChatPromptTemplate`。

---

## 22. 通用方式下接入记忆要注意什么

### 标准回答

在通用方式下，Memory 不是自动生效的，通常要同时满足两个条件：

1. Prompt 中显式预留历史记录占位符，例如 `{chat_history}`
2. 在 `AgentExecutor` 中显式传入 `memory`

如果缺少其中任意一步，多轮记忆都可能无法正常工作。

---

## 23. 为什么在 Function Call 模式中常使用 `ChatPromptTemplate`

### 标准回答

因为 Function Call 模式通常面向聊天模型，`ChatPromptTemplate` 更适合组织系统消息、用户消息、历史消息以及 `agent_scratchpad` 这样的中间状态，占位更自然，也更符合聊天模型的输入结构。

---

## 24. 为什么在 ReAct 模式中经常使用 `PromptTemplate`

### 标准回答

因为 ReAct 模式通常要求模型输出严格的文本格式，例如 `Thought`、`Action`、`Observation`、`Final Answer` 等，这种格式很适合通过 `PromptTemplate` 明确定义整段模板结构。

### 进一步回答

当然，不是说 ReAct 只能配合 `PromptTemplate`，而是文本模板形式更直观，尤其适合把工具说明和输出格式要求完整写清楚。

---

## 25. LangChain Hub 中的官方 Prompt 模板有什么价值

### 标准回答

LangChain Hub 提供了像 `hwchase17/react` 这样的官方 Prompt 模板，能够帮助开发者减少从零编写 Prompt 时的格式错误和变量遗漏问题。对于 ReAct 这类格式要求较高的 Agent，使用官方模板可以提升稳定性和开发效率。

### 面试加分点

可以补一句：

自己写 ReAct Prompt 时，最容易出错的不是表达不通顺，而是缺少 `tools`、`tool_names`、`agent_scratchpad` 等关键变量。

---

## 26. Agent 适合哪些应用场景

### 标准回答

Agent 适合以下类型场景：

- 需要调用外部工具的任务
- 流程不固定的复杂问答
- 多步任务执行
- 需要基于中间结果继续决策的问题
- 多轮对话且上下文依赖较强的场景

### 具体例子

- 天气查询
- 新闻总结
- 查股价并做计算
- 调用数据库查询用户信息
- 调用企业内部 API 完成任务流

---

## 27. Agent 不适合哪些场景

### 标准回答

如果任务流程很固定、步骤明确、对稳定性和成本控制要求极高，那么优先使用 Chain 或普通工作流往往更合适。并不是所有问题都需要 Agent。

### 面试加分表达

我的理解是，Agent 是“灵活性换复杂度”的方案。能用固定流程解决的问题，通常没必要过早引入 Agent。

---

## 28. 如何回答“你会怎么设计一个 Agent 系统”

### 参考回答

我一般会按以下顺序设计：

1. 先判断这个任务是否真的需要动态决策。
2. 如果需要，再明确要接哪些工具，比如搜索、计算、数据库、代码执行等。
3. 根据场景选择 Function Call 还是 ReAct。
4. 设计清晰的工具描述和输入输出约束。
5. 选择合适的 Prompt 模板，并确保包含 `agent_scratchpad` 或 `chat_history` 等关键变量。
6. 用 `AgentExecutor` 统一执行，并在必要时增加 `handle_parsing_errors=True` 提升鲁棒性。
7. 如果有多轮上下文需求，再引入 Memory。

### 这个回答的价值

它能体现你不只是会写示例代码，而是具备系统设计意识。

---

## 29. 这章最容易被问到的“坑点题”

### 题目一：为什么 Agent 明明创建成功了，但工具就是不调用？

参考回答：

可能原因包括：

- 工具描述不清晰，模型不知道何时用它
- Prompt 没有明确要求可以或必须使用工具
- 工具没有正确传给 Agent/AgentExecutor
- 当前模型能力不足，判断失误

### 题目二：为什么 ReAct 模式下会解析失败？

参考回答：

因为模型没有按约定格式输出 `Thought`、`Action` 等字段，而是直接输出自由文本，导致解析器无法识别。

### 题目三：为什么多轮对话时第二轮理解错了？

参考回答：

通常是因为没有接入 Memory，或者 Prompt 没有注入 `chat_history`，导致上下文没有被传给模型。

### 题目四：为什么 Prompt 中必须包含 `agent_scratchpad`？

参考回答：

因为 Agent 的中间执行步骤需要有地方存放和回传，缺少这个占位区域会导致多步执行上下文断裂，甚至出现 `intermediate_steps` 相关错误。

---

## 30. 一段高质量总结陈述

如果面试官让你整体总结这一章，可以这样说：

LangChain 中的 Agent 主要用于解决固定 Chain 难以处理的动态任务。它通过让大语言模型充当决策中枢，结合搜索、计算、数据库等工具，实现多步任务执行。和 Chain 相比，Agent 的核心优势在于动态规划和工具调度能力。实现上，常见模式包括 Function Call 和 ReAct，其中前者更强调结构化工具调用，后者更强调思考与行动循环。工程实践中，除了会用 `initialize_agent()`，更重要的是理解 `create_react_agent()`、`create_tool_calling_agent()`、`AgentExecutor`、`agent_scratchpad`、`chat_history`、`Memory` 和 `handle_parsing_errors` 这些关键点，因为它们决定了 Agent 系统的稳定性、可扩展性和多轮对话能力。

---

## 31. 速背版答案

### 速背 1：什么是 Agent

Agent 是一种基于大模型的动态任务执行机制，能根据问题自主选择工具并完成多步任务。

### 速背 2：Agent 和 Chain 的区别

Chain 是固定流程，Agent 是动态决策。

### 速背 3：Function Call 和 ReAct 的区别

Function Call 更结构化、更高效；ReAct 更可解释、更适合复杂多步任务。

### 速背 4：AgentExecutor 的作用

负责驱动 Agent 运行、调用工具、回传结果，直到得到最终答案。

### 速背 5：为什么需要 Memory

为了支持多轮对话中的上下文延续。

### 速背 6：为什么需要 `agent_scratchpad`

为了保存 Agent 的中间执行步骤和思考过程，保证多步任务连续执行。

### 速背 7：为什么需要 `handle_parsing_errors=True`

为了在 ReAct 输出格式不规范时提高容错能力，避免系统直接崩溃。

