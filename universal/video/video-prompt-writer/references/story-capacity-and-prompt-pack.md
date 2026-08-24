# 故事容量编译与连续 Prompt Pack

本文件定义如何把一个可能超过单次生成容量的完整故事，编译成一个可直接执行的单条提示词或一组连续视频提示词。它只提供跨题材适用的意图路由、容量判断、分段、连续性与验收规则，不提供固定人物、场景、对白、题材或成片样本。

## 一、两个层级必须分开

- **外层 prompt 分段**：把完整故事拆成多次独立生成所需的多个 Prompt Payload；每个 payload 是一个生成单元。
- **片内时间片 / 分镜**：在单个 Prompt Payload 内按动作、信息、视点、声音或状态变化组织时间轴。

外层多段不等于片内多切镜。不得因为故事被拆成多个 prompt，就把每个 prompt 再机械切成大量极短镜头；能在一个逻辑镜头中通过 blocking、焦点、遮挡、构图、表演、声音或光线完成的变化，优先保留为镜内多节拍。

## 二、先判断用户意图

内部记录：

```text
delivery_intent: single_clip | story_complete | auto
segment_duration_cap: 15 | 30
locked_story_facts: <不可删减或改写的用户事实、动作、台词、顺序、结尾>
```

### `single_clip`

用户明确要求“一条提示词”“一个片段”“必须在单个 15 秒 / 30 秒片段内完成”时使用。

- 不静默拆成多个 prompt；
- 不擅自删除 locked story facts；
- 容量不足时标记 `mode_feasibility: blocked`，说明最主要的超载来源，并请用户授权以下任一路径：删减 / 改写次要内容、提高单片时长、改为 `story_complete`；
- 用户仍要求强行单片时，只能把未验证风险放在正文外，不得声称内容可自然完成。

### `story_complete`

用户强调完整故事、完整动作链、不得删剧情、按指定时长连续分段，或明确要求多段 prompt pack 时使用。

- locked story facts 全部进入 master story plan；
- 单段容量不足时自动输出连续 StoryPromptPack；
- `segment_duration_cap` 根据用户要求选择 15 或 30；未指定时按主 skill 的能力档规则选择，能力未知且澄清后仍未定时暂按 30 规划、标 `conditional`；15 / 30 之外不静默取整，也不进入 v0.8.4 pack 合同，必须要求改用 15 / 30；
- 用户明确要求“按 15 / 30 秒分段”时，即使故事能放进一个更长单片，也尊重该格式偏好；
- 每段独立可复制、可生成、可验收，且与前后段有明确状态交接。

### `auto`

仅在用户未明确指定时使用：

- brief 是一个可在容量内完成的画面瞬间、单一动作或单一关系变化：按 `single_clip`；
- brief 明确列出必须按顺序保留的完整故事节点，且容量检查确认单片放不下：按 `story_complete`；
- 是否允许删减、是否必须单片仍无法从语义判断，且不同选择会改变交付物：只问一个聚焦问题，不自作主张；
- 用户答复后仍不明确时，默认优先保全 locked story facts，按 `story_complete` 编译，并在外部标明 auto 推断依据与“可改为 strict single”的选项。

用户改口 intent 时必须从路由与容量审计重新开始，重新渲染全组；不得在同一交付中混入两种 intent 的旧 payload。

不要只根据文字长短判断意图。短 brief 也可能隐含长时动作，长 brief 也可能只是对单一画面的细致描述。

## 三、容量编译

### 3.1 建立 master story plan

先把用户输入整理为有顺序的 locked beats。每个 beat 至少记录：

```text
beat_id
story_function
required_visible_action
required_dialogue_or_audio
required_reaction_or_readability
required_transition
required_end_state
can_overlap_with
cannot_split_inside
```

不得把导演补全误写成用户事实。生产性补全只解决机位、光线、材质、blocking、声音层次和衔接，不新增人物、动机、对白、前史、关键动作或结局。

### 3.2 估算真实完成时间

容量估算的结果按置信度记录：`high / medium / low`，并写明估算方法。置信度低或估算与目标偏差较大时，不凭估算自动分段；把两案（单片 master 与分段方案）一起交付请用户确认。

估算落在目标容量的灰区（保守估算 ≈ 0.85×cap ~ cap）时，按 `conditional_split` 处理：同时给出单段 master 与连续分段方案，标注 `mode_feasibility: conditional`，由用户选择；不得静默选择其中任一。

容量判断至少覆盖：

1. 动作准备、路径、接触、反应、恢复；
2. 观众辨识关键信息、尺度变化或空间变化所需时间；
3. 对白 / 画外音的完整交付时长；
4. 停顿、response gap、表演反应与声音桥；
5. 转场和最终稳定画面；
6. 同时发生的动作与声音是否在物理、听觉和信息理解上真的可并行。

对白优先使用真人朗读或目标 TTS；没有实测时标记 `unverified_estimate`，不能用字符速率声称已验证。重叠对白不会让理解容量自动翻倍；多条信息都必须听清时，按可懂度而不是物理发声总时长判断。

### 3.3 决策

```text
if user explicitly requested segmentation at a named cap:
    compile StoryPromptPack even when a longer single clip could fit
elif locked beats fit one requested clip:
    render one Prompt Payload
elif delivery_intent == story_complete:
    compile StoryPromptPack
elif delivery_intent == single_clip:  # 容量放不下时
    mode_feasibility = blocked
else:
    resolve auto intent; ask only if materially ambiguous
```

不得用加速所有动作、取消反应、把台词顶到末尾、堆叠极短镜头或删掉结尾余量来伪造“放得下”。

## 四、StoryPromptPack 规划

内部结构：

```text
StoryPromptPack
  delivery_intent: story_complete
  segment_duration_cap: 15 | 30
  story_duration_estimate:
    duration_or_range
    verification: verified_human_read | verified_target_tts | unverified_estimate
  global_story_anchors
  continuity_ledger
  segments:
    - segment_id
      global_story_range
      local_duration
      narrative_unit
      locked_beats
      continuity_in
      continuity_out
      transition_carrier
      prompt_payload
```

### 4.1 段数与时长

- 每个片段满足 `0 < local_duration <= segment_duration_cap`；
- 每段本地时间轴从 0 开始，最后一个时间码等于该段 `local_duration`；
- 全局故事时间范围只放在代码块外的片段清单或交接说明中，不把全局时间码混入本地 Prompt Payload；外部清单每段按 `片段 <N>：全局 <开始–结束> / 本地 <X>s / <叙事摘要> / continuity_in：<非空可观察状态> / continuity_out：<非空可观察状态>` 提供可解析交接；
- 初始容器可按“若干个完整 cap 段 + 最后一个 `0 < Y <= cap` 的余段”规划；
- 余段短于最小可用时长（可校准初值约 3 秒）时，把最后两个相邻段重切为更均衡的段（如 32 秒 cap=30 → 16+16，而不是 30+2），每段仍满足 `0 < duration <= cap`；平台 capability 还必须给出 `min_duration` 与可用离散时长档，重切结果低于平台最短时长或不在可选档时不得声称可执行；
- 短尾至少承载一个完整可辨识 beat 并形成可截取尾帧；不足时重平衡。安全叙事接点优先于满段偏好，满段只是初始容器；
- 每次重平衡后重新计算受影响段的本地时间轴、对白余量和 continuity ledger，并重新执行质量检查；重切后的边界无叙事接点或无法提交时标记需人工复核，不得静默通过；
- `can_overlap_with` 存在并行动作轨时，只有所有轨都到达可观察、可恢复状态才允许切段；
- 总时长恰好是 cap 的整数倍时，不生成空的尾段；
- 不为凑满 cap 添加剧情、无效停顿或装饰镜头；
- 某个不可中断 beat 本身超过 cap 时，不从中间硬切。若用户和能力允许，改用更长 cap、已确认的 extend；否则说明阻塞并请求改写授权。

段数由故事容量和安全交接点决定，不由 prompt 字数决定。

### 4.2 分段边界

优先把边界放在以下位置：

- 一个动作完成、接触结果可见，但下一行动尚未启动；
- 人物位置、视线、重心、手部或道具状态形成可复现定格；
- 遮挡、出画、入画、门/物件经过镜头、烟雾/光线覆盖等可作为视觉接点；
- 声音尾音、环境底层或画外声可自然跨段；
- VFX 已留下明确残留状态，下一段可以从该残留继续；
- 情绪或关系完成一个阶段，但仍保留下一段的行动压力。

避免在以下位置切断：

- 动作准备与接触之间；
- 必须连续理解的一句对白中间；
- 表演变化尚未出现可见结果时；
- VFX 只有起点、没有可继承状态时；
- 观众尚未辨认地点、主体或因果时。

`transition_carrier` 说明边界靠什么可重建：优先动作匹配 / 入出画 / 遮挡等视觉接点，再辅以声音桥；无法使用视觉接点时才用情绪延续或稳定状态切。段首镜负责重新确立场景、主体和动作方向，段尾镜停在可复现、可截取状态，不停在半动作。

## 五、连续性台账

每个片段必须同时拥有 `continuity_in / continuity_out`。至少核对：

```text
identity_and_appearance
pose_gaze_breath_weight_and_motion_vector
spatial_geography_screen_direction_and_axis
prop_position_ownership_damage_and_consumption
body_costume_environment_damage_or_contamination
light_time_weather_and_exposure
vfx_source_path_contact_peak_residual
ambient_audio_event_tail_voice_and_sound_bridge
emotion_relationship_attention_and_information_state
camera_distance_angle_focus_occlusion_and_handoff
```

- pack 级 continuity ledger 是唯一状态源；每段 `continuity_in / continuity_out` 必须由对应时间点的台账快照派生，不得各段独立编写。首段 `continuity_in` 来自用户素材、前作尾帧或本次故事初始状态；
- `continuity_out` 必须是可观察、可复现的状态，不写“保持连续”“接下一段”等空指令；
- 下一段 `continuity_in` 必须与上一段 `continuity_out` 一致；外部清单为便于机器核验应原样复制同一摘要。只有明确设计了有动机的跳切、时间跳跃或状态突变时，下一段才可改写 `continuity_in`，并必须额外声明非空 `continuity_transition` / `transition_carrier` 说明动机与状态变化；
- 对日落、燃烧、伤势、污染、崩坏等持续变化，不只记录快照，还记录 `drift_direction` / 变化速率，防止多段间局部一致但整体反向；
- 多角色场景同时核对角色间相对距离、视线链和屏幕方向；
- 外部台账用于人和执行系统核对；会影响生成的起始姿态、空间、道具、损坏、光线、声音、情绪、VFX 残留和镜头接点，必须同时渲染进对应 Prompt Payload 的基础设定、首个时间片、最后一个时间片或画面 / 声音约束；
- 独立生成的每段都重复必要的身份、场景和风格锚点。这种跨 Prompt 重复是独立可生成所必需，不算正文内无效重复；同一 Prompt 内仍应消除无控制增益的复述。

## 六、每段 Prompt Payload

每个片段都是一份独立、完整、可直接粘贴的提示词：

- 复制正文从 `【基础设定】` 开始；
- 保留 `【基础设定】`、`【氛围与画质】`、`【画面内容】`、`【声音约束】`、`【画面约束】`；
- 每个时间片保留 `时间段`、`景别/镜头`、`运镜`、`画面内容`、`视效/VFX`、`声音/对白`、`情绪转折` 七字段及固定顺序；
- 每段复制正文分别按 Unicode 字符计数并包含空格与换行，分别不得超过 5000 字；5000 是单段上限，不是整个 pack 共用预算；
- 片段编号、全局故事时间、容量诊断、continuity ledger 摘要放在代码块外；
- 会影响生成的 continuity in/out 不能只停留在外部诊断，必须嵌入正文；
- 不在正文出现内部 schema、评分、风险说明或“请与上一段保持一致”等对执行者有意义但对模型不可生成的空话。

## 七、择优保留视觉、表演与声音质量

智能分段的目标是为质量腾出真实时间，而不是只把文本切开。

### 表演

- 为视线、呼吸、手指、重心、距离、声线、停顿和对方反应留出可见 / 可听时间；
- 边界前形成可复现的表演状态，下一段从该状态或有动机的新状态开始；
- 不因分段把自然表演改成连续口令式动作，不用抽象情绪词替代行为证据。

### 镜头与场面调度

- 维持地理锚点、视线轴、动作轴、屏幕方向和进入 / 离开方向；
- 优先用动作、视线、构图、遮挡、焦点、声音或光线做有动机的跨段接点；
- 外层 prompt 数量不决定片内镜头数。15 秒片段仍优先 2–4 个节拍、1–4 个逻辑镜头；30 秒片段仍优先 4–7 个节拍、1–7 个逻辑相连镜头；这些是软预算，不为凑数量切镜；
- 能由同一镜头中的 blocking、焦点或遮挡完成的信息，不拆成机械 animatic。

### VFX 与物理连续性

- 每个效果仍按来源 → 路径 → 接触 → 峰值 → 衰减 / 残留组织；
- 跨段时，上一段至少留下下一段可继承的材质、形状、位置、运动方向、光照或环境损伤状态；
- 不能在段尾让效果凭空消失，也不能在下一段无来源重启；
- 冲击、破碎、烟尘、液体、火光、能量或其他变化继续服从遮挡、反射、交互光、受力方向、介质扰动和主体反应。

### 声音

- 环境底层、事件音尾响、静默、呼吸、人声或声音桥的跨段职责写清；
- 独立生成时若无法真实保留前段尾音，用可重新建立的声源状态替代“沿用上一段声音”空话；
- 每段台词都在本段可用说话时间内完成，不能把未说完的半句无意切到下一段。

## 七之补、能力路由优先级

- **extend 优先于 segment**：已知后端支持延长时，同一叙事单元先走 `extend`；只有故事总容量明确超出可延长范围，或执行流程明确要求分段时，才选 `segment`；
- 用户工作流偏好（如“我要 3 条 30s 好拼接”）是合法的覆盖条件，须记录在路由说明中；
- 能力 unknown 时不得声称原生支持、可延长或可分段，只允许 `conditional` + 生成前核验；
- 一个 pack 的段数建议不超过 8；超过时优先评估是否应改用更长单段、`video-scripts-writer` 上游脚本设计或项目工作流，而不是无限细分。

## 八、与 `video-scripts-writer` 的边界

`video-prompt-writer` 必须独立完成已有故事的容量编译、分段、连续性台账和 Prompt Payload 渲染，不把 `video-scripts-writer` 设为硬依赖。

仅在以下情况可选先使用 `video-scripts-writer`：

- 用户要求从概念扩写完整多场景脚本；
- 需要项目 bible、长篇情绪结构、多人协作脚本表、参考图映射或正式创作 handoff；
- 输入还没有形成可直接编译的故事节拍、角色关系或结局。

可复用的上游字段只有：项目 bible、连续性台账、台词结束时间、给提示词 writer 的画面要点、参考图映射和连贯性注意点。`video-prompt-writer` 消费但重新估算台词时长与单段容量，最终以生成级预算和 pack 级 continuity ledger 为准；若与上游台账冲突，在外部列出差异，不静默篡改上游文档。不得让上游 skill 代替 Prompt Payload 输出，也不得为了使用上游 skill 擅自扩写用户已完成的故事。

`story_shot_planner` 只在用户需要分镜规划表、连续性/风险台账或 handoff prompts、而非直接交付完整视频生成提示词时使用；本技能可消费其已确认的镜头与状态依据，但仍自行完成容量审计、分段和 Prompt Payload 渲染。

## 九、交付前检查

- 用户意图是否被正确路由，且 strict single 没有被静默拆段？
- locked story facts 是否全部且仅一次进入 master plan，没有删除、改写顺序或新增关键情节？
- 每段是否 `0 < duration <= cap`，本地时间轴从 0 开始且连续覆盖本段？
- 全局时间是否只在外部清单，未污染本地时间码？
- 是否没有空尾段、无意义过短尾段、为凑满时长添加内容或切断不可中断 beat？
- continuity out 是否与下一段 continuity in 一致，并已把生成相关锚点嵌入正文？
- 每段是否独立保留完整标题和七字段，正文分别不超过 5000 字？
- 片内镜头是否按叙事功能组织，没有因外层分段机械过切？
- 表演、空间轴、声音桥和 VFX / 物理残留是否跨段可恢复？
- platform capability、generation mode 与 mode feasibility 是否独立判断，未知能力没有被写成已确认？
