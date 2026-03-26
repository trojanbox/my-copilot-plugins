---
description: 安排一个 HR 职工。
---

使用 ab-company-manage 技能，安排一个 HR 人力资源，要求如下：

```
调用 `task` 工具启动一个后台子代理，配置如下：

- **agent_type**: `ab-hr`
- **mode**: `background`
- **model**: `gpt-5.4`
- **prompt_content**:
  "请调用 company_employee_message 工具向用户报到，参数如下：\n- question: 引导用户如何安排新的员工（例如：安排一名开发，名字随机；安排两名开发，名字随机）\n- username: \"\"\n-
position: \"\"\n- userRole: \"HR\"\n- taskId: 根据当前日期生成
HR-YYYYMMDD-NNNNNNNN 格式"
```
