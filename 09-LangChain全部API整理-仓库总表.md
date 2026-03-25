# 《09-LangChain 全部 API 整理》仓库总表版

## 1. 先说明范围

你给的是官方仓库：

- GitHub: `https://github.com/langchain-ai/langchain`

按 `2026-03-25` 这一天我查到的仓库结构，`libs/` 下面主要包含：

- `core`
- `langchain`
- `langchain_v1`
- `text-splitters`
- `partners`
- `standard-tests`
- `model-profiles`

其中，真正和“我们平时写代码直接相关”的 API，主要集中在：

1. `langchain`
2. `langchain-core`
3. `langchain-text-splitters`
4. `langchain-classic`
5. `partners/*` 里的 provider 包

所以这份整理里的“全部 API”，不是把几百个内部辅助函数机械抄一遍，而是按仓库真实分层，把公开 API 体系完整梳理出来。

---

## 2. 整个仓库的 API 分层图

建议你把 LangChain 仓库理解成 5 层：

### 第 1 层：应用层 API

主要包：

- `langchain`

定位：

- 新版主线开发入口
- 适合直接写 Agent、模型调用、工具调用、结构化输出

代表 API：

- `init_chat_model()`
- `init_embeddings()`
- `create_agent()`
- `ToolStrategy`
- `ProviderStrategy`
- Agent middleware 系列

### 第 2 层：核心抽象层 API

主要包：

- `langchain-core`

定位：

- 所有上层能力的底层协议和基础组件
- 几乎所有 provider 和应用层能力都依赖它

代表 API：

- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`
- `PromptTemplate`
- `ChatPromptTemplate`
- `Runnable`
- `RunnableSequence`
- `RunnableParallel`
- `RunnablePassthrough`
- `Document`
- `Embeddings`
- `VectorStore`
- `BaseRetriever`
- `StrOutputParser`
- `JsonOutputParser`
- `PydanticOutputParser`

### 第 3 层：RAG 辅助层 API

主要包：

- `langchain-text-splitters`

定位：

- 专门负责文档切分
- 通常在 RAG 预处理阶段使用

### 第 4 层：经典旧版 API

主要包：

- `langchain-classic`

定位：

- 兼容旧教程、旧项目、旧写法
- 很多老文章和视频仍然以它为主

代表 API：

- `LLMChain`
- `SequentialChain`
- `ConversationChain`
- `RetrievalQA`
- `AgentExecutor`
- `initialize_agent()`
- `ConversationBufferMemory`

### 第 5 层：Provider / Integration API

主要包：

- `langchain-openai`
- `langchain-anthropic`
- `langchain-deepseek`
- `langchain-groq`
- `langchain-ollama`
- `langchain-openrouter`
- `langchain-mistralai`
- `langchain-huggingface`
- `langchain-chroma`
- `langchain-qdrant`
- `langchain-exa`
- `langchain-fireworks`
- `langchain-nomic`
- `langchain-perplexity`
- `langchain-xai`

定位：

- 负责接具体模型、向量库、搜索、检索、外部服务

---

## 3. `langchain` 主包 API 总表

这是新版主线最重要的包。

### 3.1 模型初始化 API

- `langchain.chat_models.init_chat_model()`
- `langchain.embeddings.init_embeddings()`

作用：

- 统一初始化聊天模型和 embedding 模型
- 屏蔽不同 provider 的初始化差异

### 3.2 Agent API

主入口：

- `langchain.agents.create_agent()`

相关结构化输出 API：

- `langchain.agents.structured_output.ToolStrategy`
- `langchain.agents.structured_output.ProviderStrategy`
- `langchain.agents.structured_output.AutoStrategy`
- `langchain.agents.structured_output.ResponseFormat`

相关异常：

- `StructuredOutputError`
- `MultipleStructuredOutputsError`
- `StructuredOutputValidationError`

### 3.3 Agent Middleware API

这是新版 LangChain 非常重要的一块。

核心中间件类型：

- `AgentMiddleware`
- `ModelRequest`
- `ModelResponse`
- `ExtendedModelResponse`
- `AgentState`
- `ModelCallResult`

常见 hook / 装饰器：

- `before_model()`
- `after_model()`
- `before_agent()`
- `after_agent()`
- `dynamic_prompt()`
- `wrap_model_call()`
- `wrap_tool_call()`
- `hook_config()`

常见内置 middleware：

- `ModelFallbackMiddleware`
- `ModelRetryMiddleware`
- `ModelCallLimitMiddleware`
- `ToolCallLimitMiddleware`
- `ToolRetryMiddleware`
- `ContextEditingMiddleware`
- `HumanInTheLoopMiddleware`
- `SummarizationMiddleware`
- `PIIMiddleware`
- `LLMToolSelectorMiddleware`
- `TodoListMiddleware`
- `FilesystemFileSearchMiddleware`
- `ShellToolMiddleware`

相关辅助对象：

- `Todo`
- `PlanningState`
- `Action`
- `ActionRequest`
- `ApproveDecision`
- `EditDecision`
- `RejectDecision`
- `HITLRequest`
- `HITLResponse`
- `ReviewConfig`
- `InterruptOnConfig`
- `ContextEdit`
- `ClearToolUsesEdit`

### 3.4 Agent 层辅助函数

- `shell_tool()`
- `glob_search()`
- `grep_search()`
- `write_todos()`

PII / redaction 相关：

- `detect_email()`
- `detect_credit_card()`
- `detect_ip()`
- `detect_mac_address()`
- `detect_url()`
- `apply_strategy()`
- `resolve_detector()`

### 3.5 `langchain` 主包学习结论

如果你现在写新版项目，最该优先掌握的是：

1. `init_chat_model`
2. `init_embeddings`
3. `create_agent`
4. `ToolStrategy` / `ProviderStrategy`
5. `wrap_model_call`
6. `wrap_tool_call`
7. `HumanInTheLoopMiddleware`
8. `SummarizationMiddleware`

---

## 4. `langchain-core` API 总表

这是整个 LangChain 生态最底层、最稳定、最值得系统学习的部分。

---

## 5. `langchain-core.messages` 消息 API

### 5.1 核心消息类

- `BaseMessage`
- `BaseMessageChunk`
- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`
- `FunctionMessage`
- `ChatMessage`
- `RemoveMessage`

### 5.2 tool call / content block 相关

- `ToolCall`
- `ToolCallChunk`
- `InvalidToolCall`
- `ServerToolCall`
- `ServerToolCallChunk`
- `ServerToolResult`

内容块类型：

- `TextContentBlock`
- `ReasoningContentBlock`
- `ImageContentBlock`
- `VideoContentBlock`
- `AudioContentBlock`
- `FileContentBlock`
- `PlainTextContentBlock`
- `ToolContentBlock`
- `DataContentBlock`
- `NonStandardContentBlock`

### 5.3 消息工具函数

- `tool_call()`
- `tool_call_chunk()`
- `invalid_tool_call()`
- `default_tool_parser()`
- `default_tool_chunk_parser()`
- `merge_content()`
- `message_to_dict()`
- `messages_to_dict()`
- `messages_from_dict()`
- `convert_to_messages()`
- `filter_messages()`
- `merge_message_runs()`
- `trim_messages()`
- `convert_to_openai_messages()`
- `count_tokens_approximately()`
- `message_chunk_to_message()`

### 5.4 什么时候学这一层

以下场景必须懂 `messages`：

- 多轮对话
- tool calling
- agent 状态追踪
- 结构化消息转换
- 多模态输入输出

---

## 6. `langchain-core.prompts` 提示词 API

### 6.1 核心模板类

- `BasePromptTemplate`
- `StringPromptTemplate`
- `PromptTemplate`
- `BaseChatPromptTemplate`
- `ChatPromptTemplate`
- `MessagesPlaceholder`
- `StructuredPrompt`
- `DictPromptTemplate`

### 6.2 消息模板类

- `BaseMessagePromptTemplate`
- `BaseStringMessagePromptTemplate`
- `ChatMessagePromptTemplate`
- `HumanMessagePromptTemplate`
- `AIMessagePromptTemplate`
- `SystemMessagePromptTemplate`
- `ImagePromptTemplate`

### 6.3 few-shot 相关

- `FewShotPromptTemplate`
- `FewShotChatMessagePromptTemplate`
- `FewShotPromptWithTemplates`

### 6.4 prompt 工具函数

- `load_prompt()`
- `load_prompt_from_config()`
- `format_document()`
- `aformat_document()`
- `jinja2_formatter()`
- `validate_jinja2()`
- `mustache_formatter()`
- `mustache_template_vars()`
- `mustache_schema()`
- `check_valid_template()`
- `get_template_variables()`

---

## 7. `langchain-core.documents` 文档 API

核心对象：

- `BaseMedia`
- `Blob`
- `Document`
- `BaseDocumentTransformer`
- `BaseDocumentCompressor`

这是 RAG 最底层的数据载体层。

你平时最常直接接触的是：

- `Document`

---

## 8. `langchain-core.embeddings` 向量化 API

核心接口：

- `Embeddings`

测试 / 假实现：

- `FakeEmbeddings`
- `DeterministicFakeEmbedding`

典型用途：

- 把文本转向量
- 对接向量库
- 支持语义检索

---

## 9. `langchain-core.retrievers` 检索器 API

核心接口：

- `BaseRetriever`
- `LangSmithRetrieverParams`

说明：

- 所有具体 retriever 的抽象基类在这里
- 上层的向量检索、混合检索、压缩检索、路由检索，本质都建立在这里

---

## 10. `langchain-core.vectorstores` 向量库 API

核心对象：

- `VectorStore`
- `VectorStoreRetriever`
- `InMemoryVectorStore`

相关工具函数：

- `maximal_marginal_relevance()`

理解方式：

- `VectorStore` 负责存和查
- `VectorStoreRetriever` 负责把向量库包装成 retriever

---

## 11. `langchain-core.output_parsers` 输出解析 API

### 11.1 基础抽象

- `BaseLLMOutputParser`
- `BaseGenerationOutputParser`
- `BaseOutputParser`
- `BaseTransformOutputParser`
- `BaseCumulativeTransformOutputParser`

### 11.2 常用解析器

- `StrOutputParser`
- `JsonOutputParser`
- `PydanticOutputParser`
- `XMLOutputParser`

### 11.3 列表解析器

- `ListOutputParser`
- `CommaSeparatedListOutputParser`
- `NumberedListOutputParser`
- `MarkdownListOutputParser`

### 11.4 OpenAI function / tool 相关解析器

- `OutputFunctionsParser`
- `JsonOutputFunctionsParser`
- `JsonKeyOutputFunctionsParser`
- `PydanticOutputFunctionsParser`
- `PydanticAttrOutputFunctionsParser`
- `JsonOutputToolsParser`
- `JsonOutputKeyToolsParser`
- `PydanticToolsParser`

### 11.5 相关工具函数

- `parse_tool_call()`
- `make_invalid_tool_call()`
- `parse_tool_calls()`
- `droplastn()`
- `nested_element()`

---

## 12. `langchain-core.runnables` 执行流 API

这是 LangChain 工程化组合能力的核心。

### 12.1 核心 Runnable 类

- `Runnable`
- `RunnableSerializable`
- `RunnableSequence`
- `RunnableParallel`
- `RunnableGenerator`
- `RunnableLambda`
- `RunnableEach`
- `RunnableBinding`
- `RunnableWithFallbacks`
- `RunnableWithMessageHistory`
- `RunnableBranch`
- `RunnablePassthrough`
- `RunnableAssign`
- `RunnablePick`
- `RouterRunnable`
- `DynamicRunnable`
- `RunnableRetry`
- `RunnableConfigurableFields`
- `RunnableConfigurableAlternatives`

### 12.2 配置 / schema 相关

- `RunnableConfig`
- `ConfigurableField`
- `ConfigurableFieldSingleOption`
- `ConfigurableFieldMultiOption`
- `ConfigurableFieldSpec`
- `AnyConfigurableField`

### 12.3 图结构与可视化

- `Graph`
- `Node`
- `Edge`
- `Branch`
- `NodeStyles`
- `CurveStyle`
- `MermaidDrawMethod`
- `PngDrawer`
- `VertexViewer`
- `AsciiCanvas`

### 12.4 事件 / 流式 schema

- `EventData`
- `BaseStreamEvent`
- `StandardStreamEvent`
- `CustomStreamEvent`
- `StreamEvent`

### 12.5 Runnable 相关函数

- `coerce_to_runnable()`
- `chain()`
- `draw_ascii()`
- `draw_mermaid()`
- `draw_mermaid_png()`
- `set_config_context()`
- `ensure_config()`
- `get_config_list()`
- `patch_config()`
- `merge_configs()`
- `run_in_executor()`
- `gather_with_concurrency()`

### 12.6 这一层最值得背下来的 API

- `Runnable`
- `RunnableSequence`
- `RunnableParallel`
- `RunnableLambda`
- `RunnablePassthrough`
- `with_retry()`
- `with_fallbacks()`
- `with_config()`

---

## 13. `langchain-text-splitters` API 总表

这是 RAG 文档预处理阶段最常用的一组 API。

核心类：

- `TextSplitter`
- `TokenTextSplitter`
- `CharacterTextSplitter`
- `RecursiveCharacterTextSplitter`
- `MarkdownTextSplitter`
- `MarkdownHeaderTextSplitter`
- `ExperimentalMarkdownSyntaxTextSplitter`
- `HTMLHeaderTextSplitter`
- `JSFrameworkTextSplitter`
- `PythonCodeTextSplitter`
- `KonlpyTextSplitter`
- `SentenceTransformersTokenTextSplitter`
- `LatexTextSplitter`
- `NLTKTextSplitter`
- `SpacyTextSplitter`

学习建议：

- 初学者先掌握 `RecursiveCharacterTextSplitter`
- 文档结构明显时用 `MarkdownHeaderTextSplitter` / `HTMLHeaderTextSplitter`
- 代码库切分时看 `PythonCodeTextSplitter`

---

## 14. `langchain-classic` API 总表

这部分是旧版教程最常见的来源。

如果你在网上看到大量下面这些 API，不要意外，它们多数都来自 `langchain-classic`：

### 14.1 经典 Chain API

- `LLMChain`
- `ConversationChain`
- `SequentialChain`
- `SimpleSequentialChain`
- `TransformChain`
- `APIChain`
- `LLMMathChain`
- `FlareChain`
- `RetrievalQA`
- `BaseRetrievalQA`
- `VectorDBQA`
- `MRKLChain`
- `ReActChain`
- `SelfAskWithSearchChain`
- `MapReduceDocumentsChain`
- `ReduceDocumentsChain`
- `StuffDocumentsChain`
- `MapRerankDocumentsChain`

常见工厂函数：

- `create_retrieval_chain()`
- `create_history_aware_retriever()`
- `create_stuff_documents_chain()`
- `load_qa_chain()`
- `load_qa_with_sources_chain()`
- `load_summarize_chain()`
- `create_sql_query_chain()`

### 14.2 经典 Agent API

- `Agent`
- `AgentExecutor`
- `BaseSingleActionAgent`
- `BaseMultiActionAgent`
- `RunnableAgent`
- `RunnableMultiActionAgent`
- `LLMSingleActionAgent`
- `ZeroShotAgent`
- `ConversationalAgent`
- `ConversationalChatAgent`
- `ChatAgent`
- `StructuredChatAgent`
- `OpenAIFunctionsAgent`
- `OpenAIMultiFunctionsAgent`

常见工厂函数：

- `initialize_agent()`
- `create_react_agent()`
- `create_structured_chat_agent()`
- `create_tool_calling_agent()`
- `create_openai_functions_agent()`
- `create_openai_tools_agent()`
- `create_json_chat_agent()`
- `create_self_ask_with_search_agent()`
- `create_xml_agent()`
- `create_vectorstore_agent()`
- `create_vectorstore_router_agent()`
- `create_conversational_retrieval_agent()`

### 14.3 经典 Memory API

- `BaseMemory`
- `BaseChatMemory`
- `ConversationBufferMemory`
- `ConversationBufferWindowMemory`
- `ConversationTokenBufferMemory`
- `ConversationSummaryMemory`
- `ConversationSummaryBufferMemory`
- `ConversationEntityMemory`
- `VectorStoreRetrieverMemory`
- `ConversationVectorStoreTokenBufferMemory`
- `SimpleMemory`
- `CombinedMemory`
- `ReadOnlySharedMemory`

### 14.4 经典 Retriever API

- `ContextualCompressionRetriever`
- `MultiVectorRetriever`
- `ParentDocumentRetriever`
- `RePhraseQueryRetriever`
- `MergerRetriever`
- `TimeWeightedVectorStoreRetriever`
- `EnsembleRetriever`
- `MultiQueryRetriever`
- `SelfQueryRetriever`

文档压缩 / rerank：

- `DocumentCompressorPipeline`
- `EmbeddingsFilter`
- `LLMChainFilter`
- `LLMChainExtractor`
- `LLMListwiseRerank`
- `CrossEncoderReranker`
- `CohereRerank`

### 14.5 经典 Structured Output / Functions API

- `create_structured_output_chain()`
- `create_structured_output_runnable()`
- `create_openai_fn_chain()`
- `create_openai_fn_runnable()`
- `create_extraction_chain()`
- `create_extraction_chain_pydantic()`
- `create_tagging_chain()`
- `create_tagging_chain_pydantic()`
- `create_qa_with_structure_chain()`
- `create_qa_with_sources_chain()`

### 14.6 经典 Index / Storage API

- `CacheBackedEmbeddings`
- `LocalFileStore`
- `EncoderBackedStore`
- `SQLRecordManager`
- `VectorStoreIndexWrapper`
- `VectorstoreIndexCreator`

### 14.7 如何看待 `langchain-classic`

建议是：

- 老项目维护时要会看
- 新项目不建议把它当主线
- 面试里如果别人说 `LLMChain`、`ConversationBufferMemory`，你要知道它们属于经典体系

---

## 15. `partners` 目录 API 总表

官方仓库 `libs/partners/` 当前能看到的主要集成包有：

- `anthropic`
- `chroma`
- `deepseek`
- `exa`
- `fireworks`
- `groq`
- `huggingface`
- `mistralai`
- `nomic`
- `ollama`
- `openai`
- `openrouter`
- `perplexity`
- `qdrant`
- `xai`

这些包的共同规律是：

- provider-specific 类名放在各自包里
- 命名通常很直观
- 典型模式是 `ChatXxx`、`XxxEmbeddings`、`XxxVectorStore`

---

## 16. `langchain-openai` API 总表

这是最常用的 integration 包之一。

当前参考文档能确认的核心 API：

- `BaseChatOpenAI`
- `ChatOpenAI`
- `AzureChatOpenAI`
- `OpenAI`
- `AzureOpenAI`
- `OpenAIEmbeddings`
- `AzureOpenAIEmbeddings`
- `Middleware`

理解方式：

- `ChatOpenAI`：聊天模型主力入口
- `OpenAIEmbeddings`：向量化入口
- `AzureChatOpenAI` / `AzureOpenAIEmbeddings`：Azure 版本
- `OpenAI` / `AzureOpenAI`：偏 legacy completion 风格

---

## 17. 如何真正掌握“全部 API”

真正高效的方法不是死记所有类，而是按下面这个顺序建立地图：

1. 先学 `langchain`
2. 再学 `langchain-core`
3. 再学 `langchain-text-splitters`
4. 然后补 `langchain-openai` 这类 provider 包
5. 最后再回头看 `langchain-classic`

因为真实开发时的主线路径基本是：

`messages/prompt -> model -> runnable -> tool -> agent -> retrieval`

---

## 18. 最核心的 API 速记清单

如果你只想先抓住主干，建议先记住这些：

### 新版主线

- `init_chat_model()`
- `init_embeddings()`
- `create_agent()`
- `ToolStrategy`
- `ProviderStrategy`

### Core 主干

- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`
- `PromptTemplate`
- `ChatPromptTemplate`
- `Runnable`
- `RunnableSequence`
- `RunnableParallel`
- `RunnablePassthrough`
- `Document`
- `Embeddings`
- `VectorStore`
- `BaseRetriever`
- `StrOutputParser`
- `JsonOutputParser`
- `PydanticOutputParser`

### RAG 主干

- `RecursiveCharacterTextSplitter`
- `VectorStoreRetriever`

### 旧版识别点

- `LLMChain`
- `ConversationChain`
- `AgentExecutor`
- `initialize_agent()`
- `ConversationBufferMemory`
- `RetrievalQA`

---

## 19. 新旧 API 的一句话判断法

你以后看到某个 API，可以先这样判断：

- 名字像 `create_agent`、`middleware`、`ToolStrategy`，大概率是新版主线
- 名字像 `Runnable`、`PromptTemplate`、`AIMessage`，大概率是 `langchain-core`
- 名字像 `RecursiveCharacterTextSplitter`，大概率是 `text-splitters`
- 名字像 `LLMChain`、`initialize_agent`、`ConversationBufferMemory`，大概率是 `langchain-classic`
- 名字像 `ChatOpenAI`、`OpenAIEmbeddings`，大概率是 provider 包

---

## 20. 官方参考入口

按学习和查 API 的优先级，推荐直接看这些：

- GitHub 仓库: `https://github.com/langchain-ai/langchain`
- LangChain 主包参考: `https://reference.langchain.com/python/langchain/`
- LangChain Core 参考: `https://reference.langchain.com/python/langchain-core/`
- Text Splitters 参考: `https://reference.langchain.com/python/langchain-text-splitters/`
- Classic 参考: `https://reference.langchain.com/python/langchain-classic/`
- OpenAI 集成参考: `https://reference.langchain.com/python/integrations/langchain_openai/`

---

## 21. 这份总表怎么用

推荐用法：

1. 想知道某个 API 属于哪个包，先看这份总表
2. 确认它是“新版主线 / core / classic / integration”哪一类
3. 再去官方 reference 查具体签名和参数

这样比直接在 GitHub 代码里盲搜要快很多，也更不容易被旧版教程带偏。
