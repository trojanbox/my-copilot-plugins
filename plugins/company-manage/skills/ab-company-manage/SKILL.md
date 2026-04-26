---
name: ab-company-manage
description: 此技能用于团队员工分配与管理。通过调用该技能，用户可以为特定任务指派一个或多个对应角色的员工（如产品、开发、测试、运维、设计等），以保障项目的高效推进。
---

# 员工管理技能

请根据用户的自然语言需求，精准分析所需的岗位类型，并安排对应的员工执行任务。确保每个任务都有技能匹配的合适人员接手。

## 核心执行逻辑

1. **意图识别**：分析用户提供的任务内容，匹配最适合的 `<role>`。
2. **主动澄清**：如果用户仅指派了任务，但未明确具体员工或岗位，请使用 `company_employee_message` 工具主动询问用户需要安排什么类型的员工。
3. **参数提取**：从用户的描述中提取任务关键词（用作 `task_name`）和具体需求（用作 `description`）。

## 工作流

1. 使用`通讯类工具`工具询问老板需要什么类型的员工；
2. 按照`工具调用规范`的要求安排员工与老板取得联系；
3. 使用一次`read_agent`类工具检查员工是否成功启动；
4. 启动失败返回`步骤2`重新安排员工，启动成功使用`通讯类工具`向老板报告员工情况；
5. 根据老板的回复结果继续上述循环或解答老板疑问。

## 预制角色列表

<roles>
  <role>
    <position>HR</position>
    <description>负责管理员工，不涉及具体研发计划。</description>
    <agent_type>company-manage:ab-employee-hr</agent_type>
    <recommend_models>gpt-5.4, gpt-5.3-codex, claude-opus-4.6, claude-sonnet-4.6</recommend_models>
  </role>
  <role>
    <position>产品</position>
    <description>负责产品规划、需求分析（PRD）、原型设计评估和项目管理进度把控。</description>
    <agent_type>company-manage:ab-employee-product-manager</agent_type>
  </role>
  <role>
    <position>开发</position>
    <description>负责软件代码编写与系统维护。包含但不限于：前端开发（如 Vue3、React、Web 等）、后端开发、全栈开发及架构设计。</description>
    <agent_type>company-manage:ab-employee-developer</agent_type>
  </role>
  <role>
    <position>测试</position>
    <description>负责软件质量保证（QA），包括编写测试用例、自动化测试、Bug 追踪与性能测试。</description>
    <agent_type>company-manage:ab-employee-tester</agent_type>
  </role>
  <role>
    <position>[具体职位]</position>
    <description>负责上述明确分类之外的机动任务。如果用户提出特殊岗位需求，请使用此角色，并根据需求为员工命名。</description>
    <agent_type>company-manage:ab-employee-other</agent_type>
  </role>
</roles>

## 任务分配标准格式

当 HR 向 Employee Agent 分派任务时，必须在 `prompt` 中注入以下 XML 结构，作为结构化的任务上下文传递给目标 Agent：

```xml
<task-assignment>
  <username>张三</username>
  <position>开发</position>
  <user-role>EMPLOYEE</user-role>
  <task-id>TASK-20250101-001</task-id>
</task-assignment>
```

### 字段说明

| 字段 | 是否必填 | 生成规则 |
|---|---|---|
| `username` | ✅ 必填 | 给员工起一个中文名字（如张三、李四、王小明）。如果用户指定了名字则使用用户指定的。 |
| `position` | ✅ 必填 | 匹配 `<roles>` 列表中的 `<position>` 值（产品/开发/测试/运维/设计/其他）。 |
| `user-role` | ✅ 必填 | 系统角色类型（DAEMON / HR / EMPLOYEE），Employee Agent 固定为 `EMPLOYEE`。 |
| `task-id` | ✅ 必填 | 自动生成，格式为 `TASK-YYYYMMDD-NNNNNNNN`，其中 NNNNNNNN 是随机数。用于 AI 内部的任务分组与追踪。 |

## 工具调用规范

<usage>
当你决定好分配的员工后，请使用 `task` 或 `runSubagent` 子代理工具在后台创建任务，必须严格按照以下格式和规则填充参数：

- **agent_type**: (必填) 查阅 <roles> 列表的 <position>，填入 <position> 匹配岗位的 <agent_type> 值。
- **mode**: (必填，固定值) 只允许填写 `background`，不允许使用 `sync` 等其他值。
- **description**: (必填) 简明扼要地描述该员工擅长的具体技能。例如："张三，擅长前端开发，熟悉 Vue3 和 React 框架。"
- **prompt**: (必填) 必须严格按照以下模板生成，将 `<task-assignment>` XML 注入到 prompt 开头：

```
<task-assignment>
  <username>[填入员工中文名]</username>
  <position>[填入匹配的岗位名称]</position>
  <user-role>EMPLOYEE</user-role>
  <task-id>[自动生成 TASK-YYYYMMDD-NNNNNNNN]</task-id>
</task-assignment>

请调用 company_employee_message 工具向用户报到，参数如下：
- question: "老板好，我是[填入岗位名称] [填入员工中文名]，我擅长[填入该岗位匹配的具体技能]，有什么需要我做的请告诉我。"
- username: "[填入员工中文名]"
- position: "[填入岗位名称]"
- userRole: "EMPLOYEE"
- taskId: "[填入任务编号]"
```
</usage>
