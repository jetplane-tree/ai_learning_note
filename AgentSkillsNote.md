# AgentSkillsWithAnthropic

## Skill的结构
- `SKILL.md`：主说明文档，描述 Skill 用途、输入输出、核心流程。必须包含YAML元数据
- `references/`：存放参考规则、模板、辅助文档等
- `scripts/`：存放数据处理、公式重算等 Python 脚本
### 多文件结构：在SKILL.md里告知其可索引的路径
```
my-skill/
├── SKILL.md (required - overview and navigation)
├── reference.md (detailed API docs - loaded when needed)
├── examples.md (usage examples - loaded when needed)
└── scripts/
    └── helper.py (utility script - executed, not loaded)
```

```
## Overview

[Essential instructions here]

## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)

## Utility scripts

To validate input files, run the helper script. It checks for required fields and returns any validation errors:
```bash
python scripts/helper.py input.txt
```


## Skills和Agent的关系
Skills 是 Agent 能力的扩展载体。通过将专业知识、工作流程和可重复操作封装为 Skills，Agent 能够：
1. **突破原生限制**：获得代码执行之外的专业能力
2. **保证输出一致性**：通过标准化流程确保结果稳定
3. **提升效率**：避免每次重复描述相同的需求和上下文

本质上是思考：Agent完成任务需要什么样的能力

### 官方安装Skill在哪
- 在~/.claude里，可以在Finder里通过command+shift+. 查看隐藏文件夹
- 通过插件市场下载的话就在claude的plugin文件夹里


## Skills vs Tools, MCP, and Subagents
- **MCP 服务器**：提供所需的上下文，带来所有底层工具和资源的连接器
- **Subagents**：用于多线程和并行处理。
	- 为执行单一、明确定义的任务而专门构建的特化AI Agent。
	- 主智能体可以生成或创建**Subagents**，子代理可以向父智能体报告
	- 提供独立的上下文环境、限制工具使用权限、访问特定的技能
	- 被委派任务的独立智能体。可复用技能，隔离上下文。
- **Skills**：用于可重复的主线程工作流，渐进式披露
	- 通过代码和资源打包提示词和对话
	- 可预测、可重复、可移植
- **Tools**：**工具定义**（名称、描述、参数）始终存在于上下文窗口中

### **Filesystem 与 Skills 层构成了系统的能力基础设施**
![[Pasted image 20260323013527.png]]
Filesystem 作为技能容器，封装了多个可复用的 Skill 模块，这些 Skill 是经过抽象的业务能力单元。左侧的指导文档（"A guide for how to categorize feedback and how to summarize findings"）作为元指令（Meta-prompt），定义了系统处理数据的标准方法论——包括分类维度、总结框架和质量标准。实现了"知识即配置"的理念：通过修改指导文档即可调整系统行为，无需改动底层代码，Skills 层向下为分析器提供标准化工具支持，向上为 Agent 提供可编排的能力单元，确保分析过程的一致性和可维护性。


## 自定义skills

### YAML
- description不仅要说明Skills **做什么**，还要明确说明何时使用以及**如何使用**
- description还应该包含Skills的输入要求、输出格式以及任何特殊的使用条件

|必填字段|约束条件|
|---|---|
|**name (name)**|• 最多 64 个字符  <br>• 只能包含小写字母、数字和连字符  <br>• 不能以连字符开头或结尾  <br>• 必须与父目录name匹配  <br>• 建议使用动名词形式（动词+-ing）|
|**description (description)**|• 最多 1024 个字符  <br>• 不能为空  <br>• 应descriptionSkills的功能以及何时使用它  <br>• 应包含帮助智能体识别相关任务的具体关键词|

| 可选字段                     | 约束条件                |
| ------------------------ | ------------------- |
| **license（许可证）**         | 许可证名称或对许可证文件的引用     |
| **compatibility（兼容性）**   | 最多 500 个字符，指示环境要求   |
| **metadata（元数据）**        | 任意键值对               |
| **allowed-tools（允许的工具）** | 预批准工具的空格分隔列表（实验性功能） |

### 正文
**核心要点** 简洁优先：正文控制在 500 行以内，避免冗长 分层组织：基础内容放正文，详细内容放引用文件 扁平引用：只使用一级文件引用，不嵌套 灵活度选择：根据任务复杂度选择高/中/低自由度 步骤化：复杂任务必须拆分为清晰的顺序步骤，每个步骤都有明确的目标和操作

### 其他目录
| 目录          | 内容                                                                                             | 备注                                       |
| ----------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------- |
| /scripts    | - 清晰记录依赖项  <br>- 脚本有清晰的文档  <br>- 错误处理明确且有帮助                                                    | 注意：在您的说明中明确Claude是应执行该脚本，还是将其作为参考阅读。     |
| /references | - 包含代理在需要时可阅读的附加文档。  <br>- 保持单个参考文件的专注性。                                                       | 注意：对于超过100行的参考文件，在顶部包含内容目录。这确保代理能看到完整范围。 |
| /assets     | - 模板：  <br>- 文档模板  <br>- 配置模板  <br>- 图像：  <br>- 图表  <br>- 标志  <br>- 数据文件：  <br>- 查找表  <br>- 模式 |                                          |
