---
name: video-prompt-writer
description: "写、改、润色单条或连续多段视频生成提示词时加载：视频提示词 / 即梦 / Seedance / 文生视频 / 图生视频 / 多参考图 / 首尾帧 / 编辑 / 延长 / 融合 / 分镜 / 时间轴 / 智能分段 / 连续性 / 特效 / 画外音。把一句话需求、完整故事或已有草稿，编译成一条或一组可直接粘贴的中文提示词。"
version: 0.8.4
metadata:
  hermes:
    tags: [seedance, 即梦, 视频, 提示词, 图生视频, 多参考, 分镜, 镜头语言, 时间轴, 智能分段, 连续性, 特效, 画外音]
    related_skills: [video-scripts-writer, story_shot_planner, creative_quality_reviewer]
---

# Video Prompt Writer｜单片与连续分段视频提示词编写器

把用户的需求、完整故事或已有草稿，写成一条或一组可直接复制到目标视频生成入口的中文提示词。每个 Prompt Payload 支持 **1–30 秒**；完整故事允许超过单个 payload 的容量，并可按用户意图编译为以 15 秒或 30 秒为单段上限的连续 StoryPromptPack。单个 payload 可以由多个逻辑相连镜头组成，也可以是一条包含多个内部节拍的连续长镜头，但不能把无关事件拼成蒙太奇。

本技能采用两层结构：内部先完成 **Creative Plan / StoryPromptPack**（规划与校验），最终按主模板渲染为一条或多条可直接复制的提示词。模板字段完整、子维度写实是每条 payload 的交付基线；能力提升来自专业信息、真实时序、跨段连续性与约束，而不是压缩输出或把超载剧情加速硬塞。

## 触发条件

用户出现以下需求时加载本技能：

- 写、改、润色、补全或压缩单条视频生成提示词；
- 把完整故事、长剧情或已有脚本按 15 秒 / 30 秒单段上限智能拆成连续多段提示词；
- 文生视频、图生视频、多参考、首尾帧、编辑、延长或融合；
- 需要 1–30 秒的单个叙事单元，或一组跨片段连续的 Prompt Payload；
- 需要时间轴、镜头语言、声音、对白、VFX/物理因果或跨段连续性设计；
- 给出参考素材并要求锁定身份、外观、动作、场景、声音、首尾帧或编辑范围。

已有完整故事、只需容量编译与提示词渲染时，本技能独立完成，不强制加载其他脚本 skill。只有用户还需要从概念扩写完整多场景脚本、项目 bible、正式脚本表、参考图映射或协作 handoff 时，才可选先加载 `video-scripts-writer`，再消费其连续性台账、台词结束时间、画面要点与连贯性注意点；不得让上游 skill 代替最终 Prompt Payload。批量、定时生产或结构化记录写入时，额外加载 `references/text-to-video-batch-workflow.md`。

## 输入

开工前收集；非关键缺失按合理默认推进，关键缺失才询问：

- brief：主体、可见事件、状态变化、不可删改的动作/台词/顺序与结尾落点；
- `delivery_intent: single_clip | story_complete | auto`：严格单片、完整故事优先或按语义推断；
- 单片规格：`target_duration` 为 1–30 秒；多段规格：`segment_duration_cap: 15 | 30`，表示每个 Prompt Payload 的时长上限而非整个故事总时长。用户未指定 cap 时，先取平台已确认支持的 30 秒档（减少接缝），否则取已确认的 15 秒档；平台能力未知时只问一次，仍未明确则以 30 秒为暂定规划值并标 `conditional`，执行前核验。用户提出 15 / 30 之外的 cap 时不静默取整，必须要求改用 15 / 30；v0.8.4 不接受 custom cap，平台特定离散时长档需要后续显式扩展合同、校验器与测试后才能启用；
- 任务模式：文生视频 / 图生视频 / 多参考 / 首尾帧 / 编辑 / 延长 / 融合；
- 画幅、语言、参考素材及精确 `@素材名`；
- `platform_capability`：目标后端、模型或入口已知的原生时长上限、最短可生成时长、可用离散时长档、延长能力、分段能力和输入承载；
- 硬锚点：身份、外观、道具、空间、动作方向、首尾帧、文字、logo、字幕、声音或其他不可变内容；

没有可靠 capability profile 时写明 `platform_capability: unknown`，不得凭平台名称猜测、虚报原生 30 秒支持或静默截短。能力未知不妨碍先编译 prompt pack，但执行可行性只能标为 `conditional`，实际提交前仍需核验。

## 工作流程

1. **定任务与意图路由**：确认任务模式、目标时长 / 单段上限和平台能力。先按 `delivery_intent` 分派：
   - `single_clip`：只渲染一条 payload；容量不足时标 `mode_feasibility: blocked` 并请求授权（删减 / 提高时长 / 改 story_complete），绝不静默拆段。
   - `story_complete`：把完整故事编译为 master story plan；单个 `segment_duration_cap` 放不下时输出连续 StoryPromptPack（N 个满段 + 最后一个 `0 < Y <= cap` 的余段，或按安全接点重平衡）。用户明确要求“按 15 / 30 秒分段”时，将格式偏好视为硬输入，即使更长单条也放得下仍按指定 cap 编译；
   - `auto`：按语义推断，推断依据与单问澄清见 `references/story-capacity-and-prompt-pack.md` 第二节；单问后仍不明确时默认保全完整故事，按 `story_complete` 编译，在外部说明推断依据与可改选项，不擅自删剧情。
   用户交付后改口 intent 时，重新执行意图路由、容量审计和全组渲染；同一交付不得混入旧 intent 的 payload。
   skill 能写 30 秒 prompt，不代表后端能原生生成 30 秒。内部/诊断使用 `generation_mode: native | extend | segment`：
   - `native`：能力资料明确覆盖目标时长；
   - `extend`：后端原生上限较短，但支持在连续性锚点下延长同一叙事单元；
   - `segment`：已知执行流程要求分段，或任务容量明确需要拆分。
   `platform_capability`、计划的 `generation_mode` 与 `mode_feasibility: confirmed | conditional | blocked` 是三个独立判断。能力未知本身不自动推出 `segment`；只能依据用户偏好、已知工具链流程、连续性需要或容量判断选择计划模式，并标为 `conditional`。若没有任何执行路径得到确认，保留目标时长的 master prompt / pack，在正文外列出条件式 `extend/segment` 方案并要求执行前确认，不得虚报 `native`、可延长或可分段。
2. **建立内部 Creative Plan / StoryPromptPack**：规划与校验至少覆盖：
   ```text
   Creative Plan
     task_mode
     target_duration | segment_duration_cap
     delivery_intent: single_clip | story_complete | auto
     platform_capability
     generation_mode: native | extend | segment
     mode_feasibility: confirmed | conditional | blocked
     global_anchors
     scene_plan
     beats
     dialogue_plan
     directorial_plan
     shot_and_timeline_plan
     sound
     constraints
     quality_audit
   ```
   多段任务在 beats、dialogue plan、directorial plan、shot plan 之上追加 master story plan 与 continuity ledger，规则见 `references/story-capacity-and-prompt-pack.md`；每段渲染为一个独立 Prompt Payload。Creative Plan / StoryPromptPack 默认不作为渲染字段展示，也不要求输出 JSON。
3. **锁定叙事单元或完整故事链**：`single_clip` 锁定一个起始状态、核心行动或关系问题、关键变化、结果和最后一帧功能；16–30 秒允许建立、发展、转折与收束，但所有节拍必须服务同一因果链、关系变化或空间推进。`story_complete` 先把用户输入的 locked beats 按真实完成时间编译为 master story plan，再决定单个 payload 还是连续 pack。
4. **建立场景锚点与 ScenePlan**：先在 `global_anchors` 中锁定主体、素材职责、场景连续性、风格、光线、材质、声音基线和硬约束，再明确各场景维度的 `locked/free` 状态并建立 `scene_plan`。场景推导、三候选、无历史默认先验自检、八维签名与连续性碰撞处理必须按 `references/scene-design-and-diversity.md` 执行；只把会变化的信息下放到时间片，避免逐片重复全局设定。多段任务中，独立生成的每段仍需重复必要锚点：身份、主体外观、场景 ID / 空间基线、风格与光线基线每段必留；未变化且不影响生成的静态装饰和材质细枝可压缩。跨段重复必要锚点不算段内冗余；同一 payload 内无控制增益的重复仍应删除。
5. **按事件切时间轴**：按行动阶段、信息揭示、空间关系、视点、声音职责或状态变化切分，不机械均分。采用 3–8、9–15、16–30 秒不同软预算；连续长镜头允许少镜头、多节拍。时间码必须连续、不重叠、无无意空洞，并完整覆盖目标时长。详见 `references/timeline-segment-structure.md`。
6. **完成导演与摄影决策**：先确定观众此刻应知道、感到和等待什么，再选择 blocking、视点、景别、机位、焦点与运镜。每次切镜或摄影机重构必须带来新信息、关系、视点或节奏变化；不用术语堆砌代替动机。
7. **安排画面变化、声音和对白**：视觉或物理变化按来源、路径、交互和余波写进对应时间片字段；极简现实片段不强行添加人工 VFX。先判定 `audio_policy`：正常生成音频时写声源、材质、空间、远近和时间关系；外部成曲且生成端静音时，不编造音效或未知鼓点，但逐片写清与已提供音轨/歌词段的同步职责；纯静音任务逐片写清静默如何由可见动作、停顿或构图承担节奏。对白按可用说话时间估算，并在片段结束前完成，尾部留给反应、声音余波或稳定画面。
8. **编译分段与连续性（story_complete 时）**：按 master story plan 的 locked beats、真实完成时间与安全交接点切段，所有段满足 `0 < duration <= segment_duration_cap`；安全叙事接点优先于满段偏好，满段 + 余段只作初始容器。短尾必须能承载至少一个完整可辨识 beat 并形成可截取尾帧；不足时重平衡最后两段或相邻段。每次重平衡后重新计算受影响段的本地时间轴、对白余量和 continuity ledger；并行动作轨必须全部到达可恢复状态才允许切段。为每段写由 pack 级 continuity ledger 状态快照派生的 `continuity_in / continuity_out`，覆盖身份外观、姿态视线、空间轴线、道具/损坏、光线时间与变化方向、VFX 残留、声音、情绪、信息和镜头接点，不得各段独立编造；`transition_carrier` 明确写为动作匹配 / 入出画 / 遮挡 / 声音桥 / 情绪延续之一或组合。影响生成的状态必须同时嵌入对应 Prompt Payload 正文。段首镜承担场景与主体重建 / 定向功能，段尾停在可复现、可截取状态，不停在半动作。外层分段边界是生成单元交接，不是片内切镜；片内仍按叙事功能组织逻辑镜头与镜内多节拍，不因 pack 机械过切。完整规则见 `references/story-capacity-and-prompt-pack.md`。
9. **渲染最终提示词**：`single_clip` 渲染一条；`story_complete` 渲染一组。每条 payload 都按主模板输出 `【基础设定】`、`【氛围与画质】`、`【画面内容】`、`【声音约束】`、`【画面约束】`。`【画面内容】` 中每个时间片固定按 `时间段`、`景别/镜头`、`运镜`、`画面内容`、`视效/VFX`、`声音/对白`、`情绪转折` 顺序写实；保留字段结构与顺序，不省略标签、不把字段内容合并成一段概述。模板骨架见 `references/prompt-structure-framework.md`。
10. **检查、修复、复检**：按 `references/video-prompt-quality-checks.md` 验证四大块完整、模板字段齐全、时间码连续覆盖、时长可实现性、平台路由、约束冲突、对白收尾余量和专业信息密度；多段 pack 另按 `references/story-capacity-and-prompt-pack.md` 第九节检查意图路由、locked facts、段时长、连续性、单段 5000 字上限与片内镜头质量。压缩只能删重复与空结构，不能以丢失镜头动机、连续性、声音或物理因果换短。

## 时长与容量骨架

这些是软预算，不是镜头数硬模板：

- `1–2 秒`：一个可辨识动作点、反应点或状态变化，通常一个节拍；
- `3–8 秒`（归入 `1–8 秒` micro 档）：1–3 个节拍，通常 1–2 个镜头；
- `9–15 秒` short 档：2–4 个节拍，通常 1–4 个镜头；
- `16–30 秒` extended 档：4–7 个节拍，通常 1–7 个逻辑相连镜头，形成完整叙事单元。

连续长镜头不受镜头数建议约束，仍用行动节点、遮挡/显露、焦点、空间关系、声音或表演变化组织内部节拍。时长增加不自动增加场景、角色、对白或特效。若动作、停顿、反应、声音和结尾余量放不下：

1. 先删次要镜头、重复信息和装饰性变化；
2. 仍超载且 `delivery_intent: single_clip`：标 blocked 并请求授权，不用加速一切硬塞；
3. 仍超载且 `delivery_intent: story_complete`：按 `references/story-capacity-and-prompt-pack.md` 编译连续 StoryPromptPack，为质量腾出真实时间而不是压缩关键内容。

分段是容量不足时的正路之一，不是质量降级；但它只解决“一个生成单元放不下”的问题，不能替代片内真实的表演时间、动作完成时间或对白时长。

## 输出契约

诊断信息（生成模式、入口模式、意图路由、时长 / 单段上限、能力路由、素材职责）放在复制用提示词**外部**。`single_clip` 只交付一个复制用代码块；`story_complete` 多段时交付一个外部片段清单 + 每段一个独立复制用代码块。块内不写诊断、评审、备选或解释，从 `【基础设定】` 直接开始。

外部诊断（按需，能力相关必写）：

- `生成模式：<文生视频 / 图生视频 / 多参考图 / 首尾帧 / 编辑 / 延长 / 融合>`
- `入口模式：<全能参考 / 首尾帧 / 编辑 / 延长 / 融合 / 无参考资产>`
- `delivery_intent: single_clip | story_complete | auto`　`segment_duration_cap: 15 | 30`（多段时）
- `时长：<X秒>`　`画幅：<16:9 / 9:16 / ...>`
- `platform_capability: <已知能力与来源 / unknown>`　`generation_mode: native | extend | segment`　`mode_feasibility: confirmed | conditional | blocked`
- `能力说明：<能力未知、低于目标时长或需要连续性传递时必写；不得虚报或静默截断>`
- `素材职责：<@素材名>：<控制层：身份/外观/服饰/材质/道具/场景/首帧/尾帧/动作/运镜/表情/声音/节奏/编辑对象/风格>`

多段 pack 的外部片段清单（每段一条）：

- `片段 <N>：全局 <开始–结束> / 本地 <X>s / <叙事单元摘要> / continuity_in：<可观察起始状态摘要> / continuity_out：<可观察结束状态摘要> / <生成模式与素材职责>`
- 相邻段的外部台账摘要必须把上一段 `continuity_out` 原样复制为下一段 `continuity_in`，便于机器核验；若边界本身设计了有动机的跳切、时间跳跃或状态突变，则在下一段额外写 `/ continuity_transition：<非空动机与状态变化>`，不得用改写措辞伪装普通连续交接。

全局故事时间只出现在外部清单，每段复制用正文内部的时间码一律从该段本地 0 开始。

复制用提示词模板（每个 `<...>` 槽位都要填实，不能整段省略，也不能把多个子维度塞进一句泛化描述；不需要的子维度按合理默认补写，不留空）：

```text
【基础设定】
角色设定：
（凡是有台词、被招呼、对视、做出反应或与主体互动的主体，都按完整角色逐项写下面五组子维度，不因"次要"而压成一句；只有完全不参与互动的纯背景人群才归入"群体角色"压缩处理。）
<角色名/类型>：<身份/职业/物种、数量/年龄/性别、体态轮廓、时代/美术风格>。<性格状态/行为属性>。<面部/五官/表情/神态>。身穿/携带<服装、配饰、道具、装备>。<颜色、材质、质感、破损、限制>。需连续保持的写 <@素材名>。
群体角色（如有，仅限不参与互动的纯背景人群）：<数量/规模、群体类型、危险程度/视觉气质>。<身体状态、毛发、皮肤、伤口、污痕等细节>。身着<服装类型/时代风格>。<破损、血渍、灰尘、材质细节>。
场景设定：
<年代/地域/空间类型，整体美术风格/世界观>。<事件背景/剧情阶段/冲突状态>。<季节/天气/时间/光照>。场景中有<建筑元素、街道/室内/自然环境、交通工具/大型道具、小型环境道具>。<战斗/灾难痕迹/秩序破坏程度>。<地面、墙面、空气、远景、近景细节>。<日常元素>与<异常元素>形成<反差类型>。

【氛围与画质】
风格核心：<主美术风格、题材类型、真实感等级、影像质感、媒介模拟>。杜绝<不想要的质感/生成问题/风格偏差>。
反差关系：<正常/美好元素>与<危险/异常元素>形成<强烈/微妙/黑色幽默式>反差。
动态与动作：保持<速度感/冲击力/运动感/流畅度>。避免<僵硬/拖沓/假动作/游戏CG感>。
情绪基调：整体氛围<幽默/紧张/压抑/荒诞/史诗/浪漫/死寂>。
摄影语言：<画幅比例/质感>。使用<摄影机类型/媒介>。搭配<镜头型号/风格>。添加<动态模糊/景深/拖影/手持感/长焦压缩/广角透视>。
色彩与影调：<年代/类型美学>。<主色A>+<主色B>的<高对比/低对比/互补/冷暖>色调。<高饱和/低饱和/复古/脏旧/清透>影调。<胶片颗粒/数字锐度/柔光/高锐度>质感。
光照设计：采用<自然光/人工光/霓虹/火光/月光/顶光/侧逆光>。整体光线<通透/阴郁/硬朗/柔和/戏剧化>。高光<不过曝/溢出/边缘泛光/强反射>。暗部<保留细节/压暗/高反差/柔和过渡>。明暗过渡<自然/锐利/柔和/强烈>。营造<通透明亮/恐怖压抑/史诗肃穆/浪漫梦幻>的光感。

【画面内容】
时间段：<开始时间>-<结束时间>
景别/镜头：<景别、拍摄方向、机位高度、角度、构图方式、主体位置、环境/背景位置、空间层次、焦点优先级与本镜头叙事功能>
运镜：<运动方式、节奏、是否跟随主体、焦点转移、固定/摇/推拉/升降>
画面内容：主体、动作、表情、服装/道具、空间关系、环境细节、光线、色彩、材质、前中后景与可见因果。
视效/VFX：可见视效或物理变化的现象、位置、路径、运动、强度变化、与主体/镜头/环境的交互；无明显视效时写"无人工视效，仅保留真实自然变化：..."。
声音/对白：环境声、动作声、音乐、对白、语气、停顿；不写视觉特效。对白/画外音安排在本镜结束前一点说完，结尾留出收尾余量，不要顶到时间末尾被截断
情绪转折：<开始时情绪> -> <结束时情绪>。<是否制造与上一镜头的反差>
时间片2: <继续按同一结构书写>

【声音约束】
<环境声/持续音效跨镜连续；拟音/事件音绑定动作节点和材质；声音桥、静默或余波；人声/画外音归属、远近、音量和可懂度；台词在片段结束前留出收尾余量说完，避免声音戛然而止；默认禁止BGM/配乐>

【画面约束】
禁止任何字幕。
```

简单片段可压缩外部诊断或合并少量设定，但复制用正文仍须保持可生成、可拍摄、无内部评审痕迹。文生批量场景按 `references/text-to-video-batch-workflow.md` 的差异化规则。

## 数字与单位规范

复制用提示词正文内所有**精确可量化参数**统一使用阿拉伯数字 + 标准单位，中文数词仅保留在不可改写内容中：

- 焦段：24mm、50mm、85mm；仅当精确焦段具有生成控制价值时才写具体毫米数，否则按 `references/shot-language-library.md` 原则写「自然广角 / 标准视角 / 中长焦压缩」等效果描述。
- 距离 / 尺度：30米、80米、2米。
- 时长（含小数）：1秒、2秒、4秒、0.1秒、0.2秒、0.4秒。
- 画幅：16:9、9:16（不写「十六比九」等中文比例）。
- 时间片时间段：维持现有时间码格式（如 `00:00-00:02`、`00:00-00:02.0`），时间码本身已是阿拉伯数字，不强制改写为「0–2秒」表述。
- 中文数词豁免范围：成语 / 诗句原文等不可改写内容；修辞性夸张数字（如「千万粒墨点」「万卷书」）不属于精确可量化参数，不强求阿拉伯化。
- 除豁免外，叙述性语句中的精确数字一律阿拉伯化（如「十二座书架」→「12座书架」，「每秒五字」→「每秒5字」）。

## 质量门槛

交付前必须同时满足：

- 目标时长在 1–30 秒，且 prompt 时长能力与后端原生能力被明确区分；
- `delivery_intent` 被正确判定：严格单片没有被静默拆段，容量不足时以 blocked + 授权请求处理；story_complete 没有被悄悄删减关键剧情；
- 16–30 秒的单个 payload 仍是一个完整、逻辑相连的叙事单元，不是无关蒙太奇；多段 pack 的每段同样自洽，且段间通过 continuity ledger 连续；
- 多段 pack 每段时长满足 `0 < duration <= segment_duration_cap`，每段本地时间轴从 0 开始，全局故事时间只在外部清单；
- Creative Plan / StoryPromptPack 的 duration、delivery intent、platform capability、generation_mode、mode feasibility、global anchors、scene plan、beats、dialogue plan、directorial plan、sound、constraints、master story plan（多段）、continuity ledger（多段）已内部覆盖；
- 复制正文覆盖四大块：`【基础设定】`（角色+场景）、`【氛围与画质】`、`【画面内容】`（分镜）、`【声音约束】+【画面约束】`；独立硬约束标题按需出现，不输出空标题；
- 时间码连续且完整覆盖该 payload 的本地时长（单条时即目标时长），片段按事件或状态变化切分；
- 复制用正文不设固定字数下限；篇幅由目标时长、事件密度、角色/场景复杂度、VFX/声音职责与连续性需要动态决定。每句都应增加可见、可听、因果、连续性或硬约束控制，不为凑字数补写重复锚点、泛化形容词或无生成价值扩写；
- 每条复制用正文不得超过 5000 字。计数范围为该条从 `【基础设定】` 到最后一个约束块的完整复制正文，按 Unicode 字符计数并包含空格与换行；超过时优先删除重复的全局设定、诊断解释、泛化质量词和无控制增益的装饰细节，不得删除用户事实、时间片字段、关键动作、声音职责或硬约束；
- 每个时间片固定按 `时间段`、`景别/镜头`、`运镜`、`画面内容`、`视效/VFX`、`声音/对白`、`情绪转折` 顺序书写，字段齐全且能从对应字段恢复具体景别/视点、摄影机行为及动机、行动、表演、适用的视觉变化、音频时序职责与转折语义；
- 多段 pack 的 continuity_in / continuity_out 至少覆盖身份外观、姿态视线、空间轴线、道具/损坏、光线时间、VFX 残留、声音、情绪、信息与镜头接点，且影响生成的交接状态已嵌入正文；
- 外层分段边界是生成单元交接，不是片内切镜；片内按叙事功能组织逻辑镜头与镜内多节拍，没有因 pack 机械过切；
- 连续长镜头允许少镜头多节拍，不因镜头数软预算被强拆；
- 极简现实片段不强行添加人工 VFX，也不输出无生成价值的 VFX 占位；
- 对白可在分配时段内自然说完，结尾有反应、声音余波或稳定构图余量；
- 约束之间、约束与素材、动作与镜头、声音与画面没有冲突；
- 压缩删除重复与空结构，不删除身份锚点、关键动作、逐片景别、摄影机行为与动机、时序、连续性、音频时序职责或硬约束；
- universal runtime 只提供抽象决策规则和占位符，不引入垂类、IP、固定人物、固定剧情或可复制的确定性创意样本。

## 引用文档

- `references/prompt-structure-framework.md`：Creative Plan 与最终提示词分层、主模板骨架、字段写法和压缩顺序。
- `references/story-capacity-and-prompt-pack.md`：完整故事容量编译、`delivery_intent` 意图路由、15 / 30 秒单段上限的 StoryPromptPack、分段边界、continuity ledger、跨段表演 / 镜头 / VFX / 声音质量与 `video-scripts-writer` 边界。
- `references/scene-design-and-diversity.md`：场景锚点、ScenePlan、三候选、默认先验自检、签名去重与连续性碰撞边界。
- `references/timeline-segment-structure.md`：1–30 秒时长档、事件驱动分段、连续覆盖、长镜头与对白余量。
- `references/video-prompt-quality-checks.md`：v0.8.4 二元检查、五维评估与修复闭环。
- `references/video-element-knowledge-base.md`：影视/摄影控制词汇。
- `references/shot-language-library.md`：景别、视角、构图和运镜选择。
- `references/cinematography-storyboard-production-design.md`：DirectorialPlan、mise-en-scène、blocking、staging、轴线、声音桥、production design 与 DirectorialPlan→ShotDecision 接口。
- `references/narrative-script-performance-direction.md`：叙事变化、DialoguePlan、角色声线、潜台词、权力弧、信息台账、表演方向与逐轮完整交付时长。
- `references/vfx-design-dimensions.md`：有必要时的视觉变化与物理因果。
- `references/action-contact-compositing-continuity.md`：动作接触、遮挡、交互光影与连续性。
- `references/dialogue-pace-and-voiceover.md`：不重复扣时的对白预算、完整 delivery 实测、真人朗读/目标 TTS、重叠打断、画外音与人声可懂度。
- `references/sound-design-foley-ambient-mix.md`：环境声、拟音、声音桥、静默/余波和混音层级。
- `references/text-to-video-batch-workflow.md`：批量生产差异化规则。
