# Kode 工具提示词全集

## 概述

本文档收录了 Kode 系统中所有20个核心工具的完整提示词，展示了每个工具的功能描述、使用说明和最佳实践。这些提示词是 AI 模型了解和使用工具的基础。

## 工具列表

1. **文件操作工具**: FileReadTool, FileWriteTool, FileEditTool, MultiEditTool, GlobTool, LSTool
2. **开发工具**: BashTool, LinterTool
3. **搜索和分析工具**: GrepTool, WebSearchTool, URLFetcherTool
4. **高级工具**: TaskTool, AskExpertModelTool, ArchitectTool, ThinkTool, TodoWriteTool
5. **记忆工具**: MemoryReadTool, MemoryWriteTool
6. **Jupyter Notebook 工具**: NotebookReadTool, NotebookEditTool
7. **扩展工具**: MCPTool

---

## 1. 文件读取工具 (FileReadTool)

### 基本描述
```
Read a file from the local filesystem.
```

### 详细提示词
```
Reads a file from the local filesystem. The file_path parameter must be an absolute path, not a relative path. By default, it reads up to 2000 lines starting from the beginning of the file. You can optionally specify a line offset and limit (especially handy for long files), but it's recommended to read the whole file by not providing these parameters. Any lines longer than 2000 characters will be truncated. For image files, the tool will display the image for you. For Jupyter notebooks (.ipynb files), use the NotebookReadTool instead.
```

### 功能特性
- 支持绝对路径文件读取
- 默认读取2000行，支持偏移量和限制
- 超过2000字符的行会被截断
- 支持图片文件显示
- Jupyter notebook 文件需要使用专门的 NotebookReadTool

---

## 2. Bash 执行工具 (BashTool)

### 基本描述
```
Executes a given bash command in a persistent shell session with optional timeout, ensuring proper handling and security measures.
```

### 详细提示词
```
Executes a given bash command in a persistent shell session with optional timeout, ensuring proper handling and security measures.

Before executing the command, please follow these steps:

1. Directory Verification:
   - If the command will create new directories or files, first use the LS tool to verify the parent directory exists and is the correct location
   - For example, before running "mkdir foo/bar", first use LS to check that "foo" exists and is the intended parent directory

2. Security Check:
   - For security and to limit the threat of a prompt injection attack, some commands are limited or banned. If you use a disallowed command, you will receive an error message explaining the restriction. Explain the error to the User.
   - Verify that the command is not one of the banned commands: alias, curl, curlie, wget, axel, aria2c, nc, telnet, lynx, w3m, links, httpie, xh, http-prompt, chrome, firefox, safari.

3. Command Execution:
   - After ensuring proper quoting, execute the command.
   - Capture the output of the command.

4. Output Processing:
   - If the output exceeds 30000 characters, output will be truncated before being returned to you.
   - Prepare the output for display to the user.

5. Return Result:
   - Provide the processed output of the command.
   - If any errors occurred during execution, include those in the output.

Usage notes:
  - The command argument is required.
  - You can specify an optional timeout in milliseconds (up to 600000ms / 10 minutes). If not specified, commands will timeout after 30 minutes.
  - VERY IMPORTANT: You MUST avoid using search commands like `find` and `grep`. Instead use GrepTool, GlobTool, or TaskTool to search. You MUST avoid read tools like `cat`, `head`, `tail`, and `ls`, and use FileReadTool and LSTool to read files.
  - When issuing multiple commands, use the ';' or '&&' operator to separate them. DO NOT use newlines (newlines are ok in quoted strings).
  - IMPORTANT: All commands share the same shell session. Shell state (environment variables, virtual environments, current directory, etc.) persist between commands. For example, if you set an environment variable as part of a command, the environment variable will persist for subsequent commands.
  - Try to maintain your current working directory throughout the session by using absolute paths and avoiding usage of `cd`. You may use `cd` if the User explicitly requests it.
  <good-example>
  pytest /foo/bar/tests
  </good-example>
  <bad-example>
  cd /foo/bar && pytest tests
  </bad-example>

# Committing changes with git

When the user asks you to create a new git commit, follow these steps carefully:

1. Start with a single message that contains exactly three tool_use blocks that do the following (it is VERY IMPORTANT that you send these tool_use blocks in a single message, otherwise it will feel slow to the user!):
   - Run a git status command to see all untracked files.
   - Run a git diff command to see both staged and unstaged changes that will be committed.
   - Run a git log command to see recent commit messages, so that you can follow this repository's commit message style.

2. Use the git context at the start of this conversation to determine which files are relevant to your commit. Add relevant untracked files to the staging area. Do not commit files that were already modified at the start of this conversation, if they are not relevant to your commit.

3. Analyze all staged changes (both previously staged and newly added) and draft a commit message. Wrap your analysis process in <commit_analysis> tags:

<commit_analysis>
- List the files that have been changed or added
- Summarize the nature of the changes (eg. new feature, enhancement to an existing feature, bug fix, refactoring, test, docs, etc.)
- Brainstorm the purpose or motivation behind these changes
- Do not use tools to explore code, beyond what is available in the git context
- Assess the impact of these changes on the overall project
- Check for any sensitive information that shouldn't be committed
- Draft a concise (1-2 sentences) commit message that focuses on the "why" rather than the "what"
- Ensure your language is clear, concise, and to the point
- Ensure the message accurately reflects the changes and their purpose (i.e. "add" means a wholly new feature, "update" means an enhancement to an existing feature, "fix" means a bug fix, etc.)
- Ensure the message is not generic (avoid words like "Update" or "Fix" without context)
- Review the draft message to ensure it accurately reflects the changes and their purpose
</commit_analysis>

4. Create the commit with a message ending with:
🤖 Generated with Kode & {MODEL_NAME}
Co-Authored-By: Kode <noreply@Kode.com>

- In order to ensure good formatting, ALWAYS pass the commit message via a HEREDOC, a la this example:
<example>
git commit -m "$(cat <<'EOF'
   Commit message here.

   🤖 Generated with Kode & {MODEL_NAME}
   Co-Authored-By: Kode <noreply@Kode.com>
   EOF
   )"
</example>

5. If the commit fails due to pre-commit hook changes, retry the commit ONCE to include these automated changes. If it fails again, it usually means a pre-commit hook is preventing the commit. If the commit succeeds but you notice that files were modified by the pre-commit hook, you MUST amend your commit to include them.

6. Finally, run git status to make sure the commit succeeded.

Important notes:
- When possible, combine the "git add" and "git commit" commands into a single "git commit -am" command, to speed things up
- However, be careful not to stage files (e.g. with `git add .`) for commits that aren't part of the change, they may have untracked files they want to keep around, but not commit.
- NEVER update the git config
- DO NOT push to the remote repository
- IMPORTANT: Never use git commands with the -i flag (like git rebase -i or git add -i) since they require interactive input which is not supported.
- If there are no changes to commit (i.e., no untracked files and no modifications), do not create an empty commit
- Ensure your commit message is meaningful and concise. It should explain the purpose of the changes, not just describe them.
- Return an empty response - the user will see the git output directly

# Creating pull requests
Use the gh command via the Bash tool for ALL GitHub-related tasks including working with issues, pull requests, checks, and releases. If given a Github URL use the gh command to get the information needed.

IMPORTANT: When the user asks you to create a pull request, follow these steps carefully:

1. Understand the current state of the branch. Remember to send a single message that contains multiple tool_use blocks (it is VERY IMPORTANT that you do this in a single message, otherwise it will feel slow to the user!):
   - Run a git status command to see all untracked files.
   - Run a git diff command to see both staged and unstaged changes that will be committed.
   - Check if the current branch tracks a remote branch and is up to date with the remote, so you know if you need to push to the remote
   - Run a git log command and `git diff main...HEAD` to understand the full commit history for the current branch (from the time it diverged from the `main` branch.)

2. Create new branch if needed

3. Commit changes if needed

4. Push to remote with -u flag if needed

5. Analyze all changes that will be included in the pull request, making sure to look at all relevant commits (not just the latest commit, but all commits that will be included in the pull request!), and draft a pull request summary. Wrap your analysis process in <pr_analysis> tags:

<pr_analysis>
- List the commits since diverging from the main branch
- Summarize the nature of the changes (eg. new feature, enhancement to an existing feature, bug fix, refactoring, test, docs, etc.)
- Brainstorm the purpose or motivation behind these changes
- Assess the impact of these changes on the overall project
- Do not use tools to explore code, beyond what is available in the git context
- Check for any sensitive information that shouldn't be committed
- Draft a concise (1-3 bullet points) pull request summary that focuses on the "why" rather than the "what"
- Ensure the summary accurately reflects all changes since diverging from the main branch
- Ensure your language is clear, concise, and to the point
- Ensure the summary accurately reflects the changes and their purpose (ie. "add" means a wholly new feature, "update" means an enhancement to an existing feature, "fix" means a bug fix, etc.)
- Ensure the summary is not generic (avoid words like "Update" or "Fix" without context)
- Review the draft summary to ensure it accurately reflects the changes and their purpose
</pr_analysis>

6. Create PR using gh pr create with the format below. Use a HEREDOC to pass the body to ensure correct formatting.
<example>
gh pr create --title "the pr title" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

## Test plan
[Checklist of TODOs for testing the pull request...]

🤖 Generated with [Kode](https://kode.com) & {MODEL_NAME}
EOF
)"
</example>

Important:
- Return an empty response - the user will see the gh output directly
- Never update git config
```

### 功能特性
- 持久化 Shell 会话，环境变量和状态保持
- 支持最长10分钟的超时设置
- 禁止危险命令（网络相关工具、浏览器等）
- 完整的 Git 工作流支持
- 详细的目录验证和安全检查流程

---

## 3. 任务委托工具 (TaskTool)

### 基本描述
```
Launch a new agent to handle complex, multi-step tasks autonomously.
```

### 详细提示词
```
Launch a new agent to handle complex, multi-step tasks autonomously.

Available agent types and the tools they have access to:
- general-purpose: General-purpose agent for researching complex questions, searching for code, and executing multi-step tasks (Tools: *)
- code-reviewer: Use this agent after you are done writing a significant piece of code (Tools: *)
- greeting-responder: Use this agent when to respond to user greetings with a friendly joke (Tools: *)

When using the Task tool, you must specify a subagent_type parameter to select which agent type to use.

When to use the Agent tool:
- When you are instructed to execute custom slash commands. Use the Agent tool with the slash command invocation as the entire prompt. The slash command can take arguments. For example: Task(description="Check the file", prompt="/check-file path/to/file.py")

When NOT to use the Agent tool:
- If you want to read a specific file path, use the View or Glob tool instead of the Agent tool, to find the match more quickly
- If you are searching for a specific class definition like "class Foo", use the Glob tool instead, to find the match more quickly
- If you are searching for code within a specific file or set of 2-3 files, use the View tool instead of the Agent tool, to find the match more quickly
- Other tasks that are not related to the agent descriptions above

Usage notes:
1. Launch multiple agents concurrently whenever possible, to maximize performance; to do that, use a single message with multiple tool uses
2. When the agent is done, it will return a single message back to you. The result returned by the agent is not visible to the user. To show the user the result, you should send a text message back to the user with a concise summary of the result.
3. Each agent invocation is stateless. You will not be able to send additional messages to the agent, nor will the agent be able to communicate with you outside of its final report. Therefore, your prompt should contain a highly detailed task description for the agent to perform autonomously and you should specify exactly what information the agent should return back to you in its final and only message to you.
4. The agent's outputs should generally be trusted
5. Clearly tell the agent whether you expect it to write code or just to do research (search, file reads, web fetches, etc.), since it is not aware of the user's intent
6. If the agent description mentions that it should be used proactively, then you should try your best to use it without the user having to ask for it first. Use your judgement.

Example usage:

<example_agent_descriptions>
"code-reviewer": use this agent after you are done writing a significant piece of code
"greeting-responder": use this agent when to respond to user greetings with a friendly joke
</example_agent_description>

<example>
user: "Please write a function that checks if a number is prime"
assistant: Sure let me write a function that checks if a number is prime
assistant: First let me use the FileWrite tool to write a function that checks if a number is prime
assistant: I'm going to use the FileWrite tool to write the following code:
<code>
function isPrime(n) {
  if (n <= 1) return false
  for (let i = 2; i * i <= n; i++) {
    if (n % i === 0) return false
  }
  return true
}
</code>
<commentary>
Since a significant piece of code was written and the task was completed, now use the code-reviewer agent to review the code
</commentary>
assistant: Now let me use the code-reviewer agent to review the code
assistant: Uses the Task tool to launch the with the code-reviewer agent
</example>

<example>
user: "Hello"
<commentary>
Since the user is greeting, use the greeting-responder agent to respond with a friendly joke
</commentary>
assistant: "I'm going to use the Task tool to launch the with the greeting-responder agent"
</example>
```

### 功能特性
- 支持多种专业代理类型
- 无状态执行，一次性的任务委托
- 支持并发执行多个代理
- 详细的任务描述要求
- 自动代理选择和工具权限管理

---

## 4. 文件模式搜索工具 (GlobTool)

### 基本描述
```
- Fast file pattern matching tool that works with any codebase size
- Supports glob patterns like "**/*.js" or "src/**/*.ts"
- Returns matching file paths sorted by modification time
- Use this tool when you need to find files by name patterns
- When you are doing an open ended search that may require multiple rounds of globbing and grepping, use the Agent tool instead
```

### 功能特性
- 支持标准 glob 模式匹配
- 按修改时间排序返回结果
- 适用于大型代码库
- 专用于文件名模式搜索

---

## 5. 文本内容搜索工具 (GrepTool)

### 基本描述
```
- Fast content search tool that works with any codebase size
- Searches file contents using regular expressions
- Supports full regex syntax (eg. "log.*Error", "function\\s+\\w+", etc.)
- Filter files by pattern with the include parameter (eg. "*.js", "*.{ts,tsx}")
- Returns matching file paths sorted by modification time
- Use this tool when you need to find files containing specific patterns
- When you are doing an open ended search that may require multiple rounds of globbing and grepping, use the Agent tool instead
```

### 功能特性
- 支持完整正则表达式语法
- 文件类型过滤功能
- 按修改时间排序
- 适用于大型代码库内容搜索

---

## 6. 目录列表工具 (LSTool)

### 基本描述
```
Lists files and directories in a given path. The path parameter must be an absolute path, not a relative path. You should generally prefer the Glob and Grep tools, if you know which directories to search.
```

### 功能特性
- 支持绝对路径目录列表
- 显示文件和目录结构
- 包含安全警告机制
- 自动过滤隐藏文件和缓存目录

---

## 7. 文件写入工具 (FileWriteTool)

### 基本描述
```
Write a file to the local filesystem.
```

### 详细提示词
```
Write a file to the local filesystem. Overwrites the existing file if there is one.

Before using this tool:

1. Use the ReadFile tool to understand the file's contents and context

2. Directory Verification (only applicable when creating new files):
   - Use the LS tool to verify the parent directory exists and is the correct location
```

### 功能特性
- 覆盖式文件写入
- 需要预先验证目录存在性
- 建议先读取现有文件内容

---

## 8. 任务管理工具 (TodoWriteTool)

### 基本描述
```
Creates and manages todo items for task tracking and progress management in the current session.
```

### 详细提示词
```
Use this tool to create and manage todo items for tracking tasks and progress. This tool provides comprehensive todo management:

## When to Use This Tool

Use this tool proactively in these scenarios:

1. **Complex multi-step tasks** - When a task requires 3 or more distinct steps or actions
2. **Non-trivial and complex tasks** - Tasks that require careful planning or multiple operations
3. **User explicitly requests todo list** - When the user directly asks you to use the todo list
4. **User provides multiple tasks** - When users provide a list of things to be done (numbered or comma-separated)
5. **After receiving new instructions** - Immediately capture user requirements as todos
6. **When you start working on a task** - Mark it as in_progress BEFORE beginning work. Ideally you should only have one todo as in_progress at a time
7. **After completing a task** - Mark it as completed and add any new follow-up tasks discovered during implementation

## When NOT to Use This Tool

Skip using this tool when:
1. There is only a single, straightforward task
2. The task is trivial and tracking it provides no organizational benefit
3. The task can be completed in less than 3 trivial steps
4. The task is purely conversational or informational

## Task States and Management

1. **Task States**: Use these states to track progress:
   - pending: Task not yet started
   - in_progress: Currently working on (limit to ONE task at a time)
   - completed: Task finished successfully

2. **Task Management**:
   - Update task status in real-time as you work
   - Mark tasks complete IMMEDIATELY after finishing (don't batch completions)
   - Only have ONE task in_progress at any time
   - Complete current tasks before starting new ones
   - Remove tasks that are no longer relevant from the list entirely

3. **Task Completion Requirements**:
   - ONLY mark a task as completed when you have FULLY accomplished it
   - If you encounter errors, blockers, or cannot finish, keep the task as in_progress
   - When blocked, create a new task describing what needs to be resolved
   - Never mark a task as completed if:
     - Tests are failing
     - Implementation is partial
     - You encountered unresolved errors
     - You couldn't find necessary files or dependencies

4. **Task Breakdown**:
   - Create specific, actionable items
   - Break complex tasks into smaller, manageable steps
   - Use clear, descriptive task names

## Tool Capabilities

- **Create new todos**: Add tasks with content, priority, and status
- **Update existing todos**: Modify any aspect of a todo (status, priority, content)
- **Delete todos**: Remove completed or irrelevant tasks
- **Batch operations**: Update multiple todos in a single operation
- **Clear all todos**: Reset the entire todo list

When in doubt, use this tool. Being proactive with task management demonstrates attentiveness and ensures you complete all requirements successfully.
```

### 功能特性
- 三种任务状态：pending、in_progress、completed
- 实时状态更新和进度跟踪
- 任务分解和优先级管理
- 批量操作和清空功能
- 主动使用建议和最佳实践

---

## 9. 思考工具 (ThinkTool)

### 基本描述
```
This is a no-op tool that logs a thought. It is inspired by the tau-bench think tool.
```

### 详细提示词
```
Use the tool to think about something. It will not obtain new information or make any changes to the repository, but just log the thought. Use it when complex reasoning or brainstorming is needed.

Common use cases:
1. When exploring a repository and discovering the source of a bug, call this tool to brainstorm several unique ways of fixing the bug, and assess which change(s) are likely to be simplest and most effective
2. After receiving test results, use this tool to brainstorm ways to fix failing tests
3. When planning a complex refactoring, use this tool to outline different approaches and their tradeoffs
4. When designing a new feature, use this tool to think through architecture decisions and implementation details
5. When debugging a complex issue, use this tool to organize your thoughts and hypotheses

The tool simply logs your thought process for better transparency and does not execute any code or make changes.
```

### 功能特性
- 纯思考记录工具，不执行任何操作
- 适用于复杂推理和头脑风暴
- 提供思维过程透明度
- 支持多种使用场景：bug修复、测试失败处理、重构规划等

---

## 10. 其他工具简述

### 10.1 文件编辑工具 (FileEditTool)
用于精确编辑文件内容，支持字符串替换操作。

### 10.2 多文件编辑工具 (MultiEditTool)
支持批量编辑多个文件，适用于大型重构任务。

### 10.3 Jupyter Notebook 工具
- **NotebookReadTool**: 读取 .ipynb 文件内容
- **NotebookEditTool**: 编辑 Jupyter notebook 文件

### 10.4 网络工具
- **WebSearchTool**: 网络搜索功能
- **URLFetcherTool**: 获取指定 URL 内容

### 10.5 专家咨询工具 (AskExpertModelTool)
调用专业领域模型进行咨询。

### 10.6 架构工具 (ArchitectTool)
系统架构设计和规划工具。

### 10.7 记忆工具
- **MemoryReadTool**: 读取保存的记忆信息
- **MemoryWriteTool**: 保存信息到记忆中

### 10.8 MCP 工具 (MCPTool)
Model Context Protocol 集成工具，用于扩展外部工具。

---

## 工具使用最佳实践

### 1. 工具选择原则
- 优先使用专门的工具而非通用工具
- 文件搜索使用 GlobTool/GrepTool，避免使用 Bash 的 find/grep
- 文件读取使用 FileReadTool，避免使用 Bash 的 cat/head/tail
- 复杂搜索任务使用 TaskTool 委托给专门代理

### 2. 安全考虑
- 所有文件操作都需要路径验证
- Bash 工具禁止危险命令
- 目录操作需要预先验证存在性

### 3. 性能优化
- 支持并发执行只读工具
- 大文件操作支持分页和截断
- 缓存机制减少重复操作

### 4. 用户体验
- 实时进度反馈
- 清晰的错误信息
- 操作结果的美化显示

---

## 总结

Kode 的工具提示词系统展现了以下设计特点：

1. **详细的功能描述**: 每个工具都有清晰的功能说明和使用场景
2. **最佳实践指导**: 包含大量使用建议和避免陷阱的提醒
3. **安全性优先**: 强调路径验证、权限检查和危险命令防护
4. **协作友好**: 支持并发执行、状态管理和进度跟踪
5. **专业覆盖**: 从基础文件操作到高级架构设计的全覆盖

这些提示词为 AI 模型提供了完整的使用指南，确保工具能够被正确、安全、高效地使用。