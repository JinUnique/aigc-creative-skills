# 动态场景设计与抗趋同

本文件负责从任意 brief 推导场景、保护用户锚点与续写连续性，并在有真实历史时抑制结构性重复。它是类级长期决策协议，不是地点百科、题材池、天气套餐或成片场景模板。所有示意只使用占位符与抽象维度。

## 1. 核心原则

- **生成顺序固定为**：`叙事功能 → 空间逻辑 → 使用与权力 → 物质和光线 → 环境运动 → 视觉表面`。不得先抽一个醒目地点名，再为它补故事。
- 场景差异必须改变叙事意义或生成控制，例如行动路径、遮挡关系、进入权限、主体距离、光源方向、表面响应、环境受力或跨镜连续性；只换地点名、同义词或装饰名词不算差异。
- brief、参考素材与续写连续性高于新颖性。不得为去重随机换地点、时段或视觉母题，破坏用户明确设定。
- 去掉固定场景先验不等于降低细节。最终仍要保留可见的光影、构图、空间层次、材质光学、使用痕迹、物理因果、环境运动与制作可执行性。
- 内部候选、签名、评分和拒绝原因默认不进入最终 prompt；最终只渲染选中的场景设计。

## 2. ScenePlan 内部契约

每个场景先建立内部 `ScenePlan`。下列字段至少全部完成；不要求输出 JSON，也不把字段名原样交给用户。

```text
ScenePlan
  anchors
    <dimension>: {value: <value>, state: locked|free, source: <brief|reference|continuity|derived>}
  narrative_function
  spatial_topology
  use_logic
  power_geometry
  material_family
  light_source_logic
  atmosphere
  environmental_motion
  history_trace
  staging_motif
  continuity_effect
```

| 字段 | 决策问题 | 必须产生的生成控制 |
|---|---|---|
| `narrative_function` | 场景必须让什么关系、信息、行动或状态发生变化？ | 本场景不可替代的戏剧职责；先于视觉表面确定 |
| `spatial_topology` | 空间如何连通、分层、开合、遮蔽？入口、出口、边界和视线如何组织？ | 可复原的路径、层级、遮挡、轴线与构图深度 |
| `use_logic` | 空间原本由谁、为何、按何种流程使用？ | 布局、触点、物件层级与行动路径彼此有因果 |
| `power_geometry` | 谁能进入、占据、阻挡、观察或退出？权力如何通过位置变化？ | 高低、中心/边缘、距离、边界、通道与出入口关系 |
| `material_family` | 结构与表面为何采用这组材料属性？如何反光、吸光、磨损、受力？ | 少量同源材料家族及粗糙度、接缝、反射、形变与接触反馈 |
| `light_source_logic` | 主光从哪里来，方向、光质、光比、遮挡和变化由何触发？ | 可解释光源与一致阴影、反射、暗部和时序变化 |
| `atmosphere` | 空气、温湿、颗粒或天气是否真的改变行动、能见度或光线？ | 仅在叙事需要时加入介质，并写明物理来源和影响 |
| `environmental_motion` | 环境中什么在动，动力来源是什么，如何影响主体或镜头？ | 方向、速度层级、接触/遮挡、残留和持续性 |
| `history_trace` | 过去的使用、维护、冲突或时间留下什么可推断证据？ | 痕迹位置与使用路径、材质和事件因果一致，不做随机做旧 |
| `staging_motif` | 主体与环境以何种调度关系反复组织注意力？ | 可执行的距离、方向、层级、视线和重构图规则 |
| `continuity_effect` | 选定设计必须延续什么，又会把哪些状态传给下一镜/下一段？ | 地理、光线、材质、物件、环境运动和最终构图的连续性结果 |

`narrative_function` 决定“为什么需要这个空间”；其余字段决定“这个空间如何工作”；视觉表面只是在这些逻辑成立后对结构、光、材质和痕迹的具体渲染。

## 3. locked / free 锚点

### 3.1 锁定来源

以下内容只要用户或输入明确提供，就标为 `locked`：

- brief 中明确的时空、空间关系、地点属性、时间、天气、光源、材质、构图或行动限制；
- 参考图、首尾帧、视频、风格素材或其他参考资产明确控制的场景维度；
- 延长、续写、多段连续创作中已经建立的地理、屏幕方向、主光、物件状态、环境状态、损坏/磨损、人物站位和动作接点；
- 用户要求复用、保持、不改变或必须承接的任何场景信息。

其余由 Agent 为本次设计推导的维度标为 `free`。没有明确证据时不得把偏好猜成 locked，也不得把模型常见默认当作用户锚点。

### 3.2 锁定规则

1. `locked` 值进入所有候选，不参与新颖性惩罚，不得被重建覆盖。
2. 参考素材只锁定其明确控制职责；不能因为有参考图就假定所有场景维度都锁定。
3. 连续性锚点先于历史去重。若连续性要求重复结构，应如实复用，不为“更新鲜”而制造跳变。
4. 不同来源冲突时，先执行用户最新明确纠偏；无法判断且会改变交付时再询问，不偷偷选新颖项。
5. 最终渲染只写对生成必要的 locked 锚点，不输出 `locked/free` 标签和内部来源记录。

内部锚点占位符：

```text
<dimension>:
  value: <用户值 / 参考素材可见值 / 连续性值 / 推导值>
  state: <locked|free>
  source: <brief|reference:<asset>|continuity:<record>|derived>
```

## 4. 三候选生成与选择

当场景没有被完全锁定时，内部生成 **3 个** `ScenePlan` 候选。候选与分数默认不向用户展示。

### 4.1 生成候选

1. 复制所有 locked 锚点。
2. 为三个候选保持同一 `narrative_function` 和核心事件，不把“换故事”伪装成场景多样性。
3. 从 `spatial_topology` 开始推导 free 维度，再依次完成 `use_logic`、`power_geometry`、`material_family`、`light_source_logic`、`atmosphere`、`environmental_motion`、`history_trace`、`staging_motif` 与 `continuity_effect`。
4. 候选之间至少有 3 个 free 维度产生实质差异；若 brief 只留下少于 3 个 free 维度，则只改变可用维度，不触碰 locked，并把相关性与连续性置于多样性之前。
5. 每个差异都回答：“它改变了哪项叙事意义或生成控制？”答不出则判为词面改名，重建该候选。
6. 没有真实历史时仍执行**高频默认母题自检**：若三个候选都由 brief 未支持的高显著性默认先验主导，必须回到 `narrative_function → spatial_topology` 重新推导至少一个候选。该自检只能判断“先验是否压过输入证据”，严禁建立地点、天气、时段、配色黑名单或任何具体组合包。

### 4.2 硬门槛与排序

先过硬门槛，再按优先级做内部选择：

1. **叙事相关性**：是否直接服务 `narrative_function`、核心动作、信息揭示和观众体验；任何硬性叙事约束失败即淘汰。
2. **可生成性**：空间是否可读，动作是否有时间和路径，光、材质、环境运动与物理因果是否一致，信息是否未过载。
3. **连续性**：是否保持所有 locked 锚点，并正确传递 `continuity_effect`。
4. **新颖性**：只在前三项相当时作为同分项，优先结构上不重复近期自由维度的候选。

这是词典序优先级，不是让新颖性以加权总分抵消叙事失败。不得因为候选“更少见”就选择与 brief 无关或不可生成的设计。

内部选择模板：

```text
candidate_id: <internal only>
locked_gate: <pass|reject>
narrative_relevance: <internal assessment>
generatability: <internal assessment>
continuity: <internal assessment>
novelty_tiebreak: <internal assessment>
selection_or_rejection_reason: <internal only>
```

## 5. 八维 scene_signature

每个已选场景形成不进入最终 prompt 的八维分类签名。签名记录结构控制，不记录可直接复制的场景句子。

```text
scene_signature
  spatial_topology
  use_logic
  access_and_power
  material_system
  light_logic
  atmosphere
  active_environment
  staging_motif
```

### 5.1 与 ScenePlan 的映射

| scene_signature | ScenePlan 来源 | 归一化内容 |
|---|---|---|
| `spatial_topology` | `spatial_topology` | 连通、层级、边界、遮挡、入口/出口结构 |
| `use_logic` | `use_logic` | 使用者、目的、流程与行动路径关系 |
| `access_and_power` | `power_geometry` | 进入权、中心/边缘、高低、阻挡和退出关系 |
| `material_system` | `material_family` | 结构/表面属性、粗糙度、反射、磨损和受力规律 |
| `light_logic` | `light_source_logic` | 光源、方向、光质、光比、遮挡与变化触发 |
| `atmosphere` | `atmosphere` | 介质状态、来源及对能见度、光或行动的作用；无必要时记录为 neutral |
| `active_environment` | `environmental_motion` | 动力来源、方向、节奏、交互和残留 |
| `staging_motif` | `staging_motif` | 主体距离、视线、层级、方向与重构图关系 |

签名值用抽象分类或短语归一化。同一结构只换地点名、修辞或同义词时，归一化后仍必须得到相同值。`history_trace` 与 `continuity_effect` 不进入首期八维签名，但仍是 ScenePlan 和质量审计必填项。

### 5.2 近期窗口

- 只有批量任务、连续创作或调用方提供真实历史记录时，才建立 `dedup_window` 并比较最近已接受的签名。
- 窗口大小由调用方配置或真实工作流决定，不从 reference、样例行或当前日期猜默认值。
- 只读入已接受且结构完整的历史签名；拒绝稿可保留在审计记录中，但不作为“已发布分布”。
- 单条任务没有历史时，不声称完成“近期历史去重”；除本次 3 个内部候选的结构差异比较外，仍必须执行高频默认母题自检。若全部候选受 brief 未支持的高显著性默认先验主导，从 `narrative_function` 和 `spatial_topology` 重推至少一个；不得把自检实现为地点、天气、时段、配色黑名单或具体组合包。

### 5.3 首期重建规则

首期分类规则的**可校准初值**为：对八维签名中的 free 维度逐项比较，若与窗口内任一近期已接受结果有 **5 个或以上相同**，则拒绝当前候选并重建。该 `initial_rebuild_threshold = 5` 只是 v0.8 冷启动起点，必须通过真实样本、生成成功率和盲评校准，不是永久 universal 真理。

执行细则：

1. 比较前先识别连续性真正必要的维度，逐项标为 `locked` 并记录证据；这些维度保留在签名和连续性记录中，但排除在碰撞计数与新颖性惩罚之外。无法指出连续性证据的维度仍为 `free`，不得事后改标以规避碰撞。
2. 对每个近期结果只统计可比较的 free 维度。只有与所有近期结果的 free 相同数都低于当前阈值，候选才通过新颖性门槛；若锁定后可比较的 free 维度本身少于阈值，则不降低阈值、不改动 locked，碰撞条件自然不可达，候选仍须通过其余硬门槛。
3. 触发重建后至少改变 3 个未锁定维度，并重新从空间逻辑向视觉表面推导，不能只替换同义词。
4. 改变必须影响叙事意义或生成控制；随机换地点、天气或材料名但不改变路径、权力、光学、环境运动或调度，仍按重复处理。
5. 连续性只保护预先锁定且有证据的维度，不构成接受 free 碰撞的例外。free 维度仍达到阈值且已到调用方配置的最大重建次数时，只能标记 `needs_review` 或 `rejected`，不得写为 accepted。
6. v0.8 首期不依赖 embedding。后续系统可把语义相似度作为辅助，但阈值仍须以真实样本校准，不能覆盖分类签名与 locked 规则。

## 6. 重建与拒绝原因

重建不是在原句上换名词，而是回到最早发生重复的 free 逻辑层重新推导。内部拒绝原因使用抽象代码，便于批量审计：

| reason_code | 含义 | 重建入口 |
|---|---|---|
| `LOCKED_CONFLICT` | 候选改变了 brief、参考素材或连续性锚点 | 恢复 locked 后从第一个 free 维度重建 |
| `NARRATIVE_MISMATCH` | 场景不服务叙事功能或核心事件 | 回到 `narrative_function → spatial_topology` |
| `GENERATABILITY_OVERLOAD` | 路径、主体、光线、环境运动或信息量不可稳定生成 | 简化空间关系与同时变化的控制面 |
| `CONTINUITY_BREAK` | 地理、方向、光线、材质、物件或动作接点断裂 | 从 `continuity_effect` 回填各相关字段 |
| `SIGNATURE_COLLISION` | 八维 free 签名达到当前重建阈值 | 至少重建 3 个 free 维度 |
| `LEXICAL_ONLY_CHANGE` | 只换名词、同义词或表面装饰，结构控制未变 | 回到空间/使用/权力逻辑，不做词面修补 |
| `RANDOM_LOCATION_SWAP` | 为新颖性随机换地点并损害相关性或连续性 | 恢复叙事功能和 locked 锚点后重建 |
| `TERM_OR_EFFECT_CONFLICT` | 光、材质、镜头或艺术术语互斥 | 只保留与可观察目标一致的最高价值控制 |

拒绝原因不进入最终 prompt；批量记录方法见 `text-to-video-batch-workflow.md`。

## 7. 渲染成最终场景描述

选定候选后，按“逻辑 → 证据”写成紧凑自然语言，不输出 ScenePlan、候选或签名字段。仅保留能影响生成的内容：

```text
<空间如何连通、分层与限制行动>；
<主体如何使用、占据或穿越空间，位置如何体现关系>；
<少量同源材质的表面、反射、接缝、磨损与接触响应>；
<主光源、方向、光质、遮挡、光比和变化触发>；
<环境运动的物理来源、路径、交互与残留>；
<历史痕迹如何证明过去使用>；
<需要延续到下一镜/下一段的地理、物件、光线、环境与构图状态>。
```

地点、天气、时段与表面名词只能来自用户锚点或上述逻辑的最终具体化，不能从固定清单抽取。无叙事作用的空气、天气、破坏、异常反差或 VFX 不强行加入。

## 8. 验收

- ScenePlan 至少完整包含 `narrative_function`、`spatial_topology`、`use_logic`、`power_geometry`、`material_family`、`light_source_logic`、`environmental_motion`、`history_trace`、`continuity_effect`。
- brief 明确场景锚点、参考素材职责与续写连续性均标为 locked；新颖性没有覆盖 locked。
- 场景未完全锁定时内部生成 3 个候选，最终不泄露候选、评分、签名或拒绝原因。
- 选择顺序为相关性 > 可生成性 > 连续性 > 新颖性；新颖性只作同分项。
- 八维 `scene_signature` 完整；有真实历史时才使用近期窗口，无历史时做本次候选比较与高频默认母题自检。
- 5/8 相同重建被明确标为可校准初值；触发后至少改变 3 个 free 维度。
- 连续性必要维度在比较前锁定并排除碰撞计数；free 相同数只有低于阈值才可接受，达到重建上限仍碰撞时只能 `needs_review` 或 `rejected`。
- 词面不同但空间、使用、权力、材质、光线、环境和调度结构相同，仍判为重复。
- 差异改变叙事意义或生成控制，不靠随机地点交换。
- 最终描述保留空间层次、构图、光源逻辑、材质光学、物理因果、环境运动、历史痕迹和连续性。
- 文件中没有固定场景包、地点清单、天气/时段/材质套餐、固定异常反差、题材池、角色、剧情、对白或 IP 内容。
