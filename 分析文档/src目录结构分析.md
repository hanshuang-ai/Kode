# src目录结构分析

本文档详细分析了Kode项目中`src/`目录下各个子目录的功能和职责。

## 概述

Kode实现了一个**三层并行架构**，通过模块化的目录结构支持AI驱动的CLI工具功能，包括多模型协作和动态代理系统。

## 核心架构目录

### **[`src/entrypoints/`](../src/entrypoints/)**
- **功能**: 应用程序入口点
- **内容**:
  - `cli.tsx` - 主要的CLI应用程序入口
  - `mcp.ts` - MCP (Model Context Protocol) 相关入口

### **[`src/screens/`](../src/screens/)**
- **功能**: 用户界面屏幕组件
- **内容**:
  - `REPL.tsx` - 交互式终端界面 (核心UI)
  - `Doctor.tsx` - 诊断工具界面
  - `LogList.tsx` - 日志列表界面
  - `ResumeConversation.tsx` - 恢复对话界面

### **[`src/components/`](../src/components/)**
- **功能**: 可复用的React组件
- **主要组件**:
  - **UI组件**: `Logo.tsx`, `Help.tsx`, `Spinner.tsx`
  - **对话组件**: `Message.tsx`, `PromptInput.tsx`, `TextInput.tsx`
  - **配置组件**: `ModelSelector.tsx`, `Config.tsx`, `ModelConfig.tsx`
  - **权限组件**: `TrustDialog.tsx`, `MCPServerApprovalDialog.tsx`
  - **子目录**: `binary-feedback/`, `CustomSelect/`, `messages/`, `permissions/`

### **[`src/commands/`](../src/commands/)**
- **功能**: CLI命令实现
- **主要命令**:
  - `agents.tsx` - 代理管理
  - `config.tsx` - 配置管理
  - `model.tsx` - 模型管理
  - `help.tsx` - 帮助系统
  - `login.tsx`, `logout.tsx` - 认证
  - `review.ts` - 代码审查
  - `terminalSetup.ts` - 终端设置

## 工具系统目录

### **[`src/tools/`](../src/tools/)**
- **功能**: 核心工具实现 (每个工具都有独立目录)
- **工具类别**:
  - **文件操作**: `FileReadTool/`, `FileEditTool/`, `FileWriteTool/`, `GlobTool/`, `GrepTool/`
  - **系统交互**: `BashTool/`, `lsTool/`
  - **AI功能**: `TaskTool/`, `ThinkTool/`, `AskExpertModelTool/`
  - **数据管理**: `MemoryReadTool/`, `MemoryWriteTool/`, `TodoWriteTool/`
  - **网络功能**: `WebSearchTool/`, `URLFetcherTool/`
  - **高级功能**: `ArchitectTool/`, `MultiEditTool/`, `NotebookReadTool/`, `NotebookEditTool/`
  - **集成**: `MCPTool/`

## 服务与集成目录

### **[`src/services/`](../src/services/)**
- **功能**: 外部服务集成和适配器
- **主要内容**:
  - `claude.ts` - Claude API集成
  - `openai.ts` - OpenAI兼容模型集成
  - `mcpClient.ts` - MCP客户端
  - `adapters/` - 模型适配器
  - `oauth.ts` - OAuth认证
  - `customCommands.ts` - 自定义命令服务

### **[`src/utils/`](../src/utils/)**
- **功能**: 通用工具函数和实用程序
- **核心功能**:
  - `config.ts` - 配置管理
  - `model.ts` - 模型管理
  - `agentLoader.ts` - 代理加载器
  - `messageContextManager.ts` - 消息上下文管理
  - `debugLogger.ts` - 调试日志
  - `file.ts` - 文件操作工具
  - `commands.ts` - 命令处理
  - `PersistentShell.ts` - 持久化Shell

## 支持系统目录

### **[`src/hooks/`](../src/hooks/)**
- **功能**: React Hooks (自定义钩子)
- **主要钩子**:
  - `useTextInput.ts` - 文本输入处理
  - `useUnifiedCompletion.ts` - 统一自动完成
  - `useCanUseTool.ts` - 工具权限检查
  - `useApiKeyVerification.ts` - API密钥验证
  - `useArrowKeyHistory.ts` - 历史记录导航

### **[`src/types/`](../src/types/)**
- **功能**: TypeScript类型定义
- **内容**: 对话、日志、模型能力、权限模式等类型定义

### **[`src/constants/`](../src/constants/)**
- **功能**: 常量定义
- **内容**:
  - `models.ts` - 模型配置
  - `modelCapabilities.ts` - 模型能力
  - `prompts.ts` - 系统提示词
  - `product.ts` - 产品信息

### **[`src/context/`](../src/context/)**
- **功能**: React Context 提供者
- **内容**: `PermissionContext.tsx` - 权限上下文

### **[`src/test/`](../src/test/)**
- **功能**: 测试文件
- **子目录**: `diagnostic/`, `integration/`, `production/`, `regression/`, `unit/`

## 根目录文件

除了目录结构，`src/`根目录还包含一些核心文件:

| 文件名 | 功能描述 |
|--------|----------|
| `Tool.ts` | 工具接口定义 |
| `tools.ts` | 工具注册表 |
| `query.ts` | 查询处理 |
| `permissions.ts` | 权限系统 |
| `messages.ts` | 消息处理 |
| `context.ts` | 项目上下文 |
| `history.ts` | 历史记录 |
| `cost-tracker.ts` | 成本追踪 |

## 架构设计原则

### 三层并行架构
1. **用户交互层** (`src/screens/REPL.tsx`) - 交互式终端界面
2. **编排层** (`src/tools/TaskTool/`) - 动态代理系统和任务委托
3. **工具执行层** (`src/tools/`) - 专业化工具和权限系统

### 多模型支持
- **ModelManager** (`src/utils/model.ts`) - 统一模型配置和切换
- **模型适配器** (`src/services/adapters/`) - 不同AI服务的适配层
- **动态切换** - 运行时模型更改，无需重启会话

### 代理系统
- **动态加载** (`src/utils/agentLoader.ts`) - 5层优先级的代理配置加载
- **配置层次** - 从内置到项目级别的代理配置
- **权限控制** - 基于代理的工具能力过滤

这个架构体现了Kode作为AI驱动的CLI工具的设计理念，通过清晰的模块化分工支持复杂的AI协作工作流。

---

*创建时间: 2025-11-28*
*分析对象: Kode项目src目录结构*