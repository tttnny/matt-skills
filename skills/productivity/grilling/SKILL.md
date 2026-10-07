---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

> **核心原则：把沟通对象当产品负责人，只聊业务结果，不谈代码实现。**
>
> - **视角落点**：只确认**用户能感知的行为**（页面表现、交互体验、业务流程），屏蔽底层工程细节（接口、依赖库、实现模块）。
> - **语言脱敏**：问题中杜绝代码片段、文件路径及未经解释的专业黑话，统一用业务语言交流。
> - **技术翻译**：若底层选型影响重大，先讲**"对用户有什么影响"**，再确认**"业务上怎么取舍"**。

一轮就是一次 `grilling_form` 调用，整条 frontier 就是 `questions[]`，别再拿散文投一遍。每道题：

- `id`：本轮内唯一（重复即整轮拒收），表单按它存用户的作答；
- `number` / `header`：Q 编号 / 短标题，界面拼成 `Q2 · 标题`；
- `question`：一句话的提问（纯文本单行，markdown 与换行都不生效），背景进 `detail`；
- `detail`：只写各选项共用的背景（markdown）——选项在这里再列一遍会被整轮拒收；
- `options[]`：每个候选一条 `label` + 一句话的 `description`（各自的代价 / 影响，推荐项把推荐理由也写在这），选项**只**写在这里；
- 推荐项放最前、给 `recommended: true`：它默认勾上、也算那题已作答，只标你真心会选的；
- 开放题不给 `options`，用户在那个自填框里答；每题都能勾选加自填、勾选可多选（不互斥）。

用户在对话区的表单里勾选、自填，点「导入输入框」写进他自己的输入框，再用原生发送键发出；他的作答会作为**下一条用户消息**到达，不要等它、也不要把表单当成已经有答案。

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

> **会话结束的判定需要同时满足**：
>
> - **The frontier is empty**: every branch of the design tree visited, nothing left silently assumed.
> - 本会话派遣过的**每一个子代理都已结算**——只要还有一个没回来，`frontier` 空了也不作数，不得当成最终共识、不得向用户确认或据此行动；先等它结算（结果可能推翻已定下来的决定、需要重开一部分树）。
>
> Do not act on it until the user confirms you have reached a shared understanding.
