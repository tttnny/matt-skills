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

一轮就是一次 `grilling_form` 调用，整条 frontier 就是 `questions[]`，别再把这一轮写成散文；它是本轮**唯一且最后一次**调用。每道题：

- `id`：本轮内唯一（重复即整轮拒收），表单按它存用户的作答；
- `question`：一句话的提问（纯文本单行，markdown 与换行都不生效），背景进 `detail`；
- `number` / `header`：可选的 Q 编号 / 短标题，给一个显示一个，两个都给才由界面拼成 `Q2 · 标题`；题序按 `questions[]`，与编号无关；
- `detail`：只写各选项共用的背景（markdown），选项在这里再列一遍会被整轮拒收；
- `options[]`：每个候选一条 `label` + 一句话的 `description`（各自的代价 / 影响，推荐项把推荐理由也写在这），选项**只**写在这里；`label` 用同意即可接受的肯定句，同题两条不得渲染成同一串文字（重复即整轮拒收），`label` 里不用「、」（那是作答回填时的项分隔符）；
- 推荐项放最前、给 `recommended: true`：它默认勾上、也算那题已作答，只标你真心会选的；
- 开放题不给 `options`，用户在那个自填框里答；每题都能勾选加自填、勾选可多选（不互斥）。

Word each question so agreeing means picking your recommended option, not rejecting the question.

入参被拒时这一轮**不会**结束：结果里带着 `violations`，当轮照着改掉、把同一次调用重投一遍；被拒不是改写法的理由，别退回散文。字段名与工具名不进 `question` / `detail` 正文。

表单只负责让用户勾选和自填；作答由他自己在原生输入框发出，作为**下一条用户消息**到达，通常包在 `【盘问作答】…【/盘问作答】` 之间。不要等它、也不要把表单当成已经有了答案，本轮到此停住，不要替用户把答案填上。

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

> **The session is done when the frontier is empty**: every branch of the design tree visited, nothing left silently assumed. 本会话派遣过的**每一个子代理也都已结算**：只要还有一个没回来，`frontier` 空了也不作数，不得当成最终共识、不得向用户确认或据此行动；先等它结算（结果可能推翻已定下来的决定、需要重开一部分树）。
>
> Do not act on it until the user confirms you have reached a shared understanding.
