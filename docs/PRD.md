# SpeakLoop（说练环）产品需求文档

| 项目 | 内容 |
| --- | --- |
| 产品名称 | SpeakLoop（说练环） |
| 平台 | macOS 13 Ventura 及以上，Apple Silicon 优先，兼容 Intel |
| 文档版本 | v1.0 草稿 |
| 日期 | 2026-11-10 |
| 文档状态 | 待评审 |
| 参考产品 | SpeakHub v0.2.5（Windows，本仓库） |

## 修订记录

| 版本 | 日期 | 修订人 | 说明 |
| --- | --- | --- | --- |
| v1.0 | 2026-11-10 | 待填 | 初稿 |

## 目录

1. 背景与产品定位
2. 产品目标
3. 目标用户与用户画像
4. 用户故事
5. 功能模块拆解
6. 核心业务流程
7. 页面结构设计
8. 数据结构设计（初步）
9. API 初稿设计
10. 非功能需求
11. 版本规划
12. 风险与待决问题
13. 附录：与 SpeakHub 的复用关系

---

## 1. 背景与产品定位

### 1.1 背景

SpeakHub 已在 Windows 上验证了一条完整的口语学习链路：开口对话、实时字幕、查词收藏、LLM 复盘、词汇和句子复习，再把薄弱点带入下一次练习。它也暴露了几个问题：

- 练习内容按“场景提示词”组织，但不同目标（考试、工作、娱乐）的练习方式和评估标准差别很大，一套通用流程难以兼顾。
- 网页自动化（ChatGPT、Gemini 网页）依赖第三方页面结构，维护成本高。
- 仅支持 Windows，Mac 用户无法使用。

### 1.2 产品定位

SpeakLoop 是一款 macOS 原生体验的 AI 英语口语陪练软件。产品以**场景**为一级入口，为日常生活、雅思口语、商务工作、英文歌曲四类目标分别设计练习形式、AI 角色和评估维度，并共用一套“练习 → 复盘 → 复习 → 再练习”的学习闭环。

### 1.3 一句话价值

打开就能用英语聊起来，每次练习后知道自己错在哪、下次该练什么。

### 1.4 范围

| 本期包含 | 本期不包含 |
| --- | --- |
| macOS 桌面客户端 | Windows、iOS、Android、Web 版本 |
| 四大场景练习与通用自由对话 | 真人外教、社区与排行榜 |
| 用户自带 API Key 的 AI 服务接入 | 自建账号体系与云端同步 |
| 本地存储全部学习数据 | 付费订阅与内购 |
| 歌曲场景使用用户自己导入的音频和歌词 | 内置受版权保护的歌曲或歌词库 |
| | 网页自动化（ChatGPT / Gemini 网页） |

---

## 2. 产品目标

### 2.1 业务目标

| 编号 | 目标 | 衡量方式 |
| --- | --- | --- |
| G1 | 降低开口门槛：新用户 10 分钟内完成第一次语音对话 | 首次启动到首次对话完成的时长中位数 |
| G2 | 形成学习习惯：用户每周至少练习 3 天 | 周活跃用户中练习 ≥ 3 天的占比 |
| G3 | 让进步可见：用户能看到各场景的评分趋势 | 查看学习报告的用户占比 |
| G4 | 稳定可用：练习过程少中断，复盘可靠生成 | 练习异常中断率、复盘生成成功率 |

### 2.2 用户目标

- 有一个随时可用、不怕犯错的英语对话对象。
- 针对自己的目标（日常交流、雅思、工作、唱歌）练习，而不是泛泛闲聊。
- 每次练完拿到具体纠错和下一步建议。
- 把学到的单词、句子沉淀下来并按时复习。

### 2.3 成功指标（上线 3 个月）

| 指标 | 目标值 | 说明 |
| --- | --- | --- |
| 首次对话完成率 | ≥ 70% | 完成安装并启动的用户中，完成至少 1 次 ≥ 3 分钟对话的比例 |
| 首次对话耗时中位数 | ≤ 10 分钟 | 含引导、配置 Key、授权麦克风 |
| 次周留存 | ≥ 35% | 首周练习过的用户第二周仍练习 |
| 周练习 ≥ 3 天占比 | ≥ 30% | 周活跃用户口径 |
| 复盘生成成功率 | ≥ 98% | 失败也不得丢失对话 |
| 语音回合延迟 P50 | ≤ 1.5 秒 | 用户说完到 AI 开始播放 |

> 目标值为初稿建议，需在内测后根据基线调整。所有指标只通过匿名统计采集，不包含练习内容。

---

## 3. 目标用户与用户画像

目标用户：有英语口语提升需求、使用 Mac 的中文母语者。

| 画像 | 典型身份 | 英语水平 | 核心诉求 | 主场景 |
| --- | --- | --- | --- | --- |
| 小林·日常交流者 | 准备出国旅行或刚移居海外的上班族 | A2–B1 | 能应付点餐、问路、看病、闲聊，不再紧张 | 日常生活 |
| 小周·雅思考生 | 大三学生，3 个月后考雅思，口语目标 6.5 | B1–B2 | 熟悉题型、练答题结构、知道自己大概几分 | 英语考试 |
| 陈经理·职场人 | 外企产品经理，要参加英文周会与客户电话 | B2 | 开会能说清楚、表达更专业得体 | 商务工作 |
| 阿宁·兴趣学习者 | 喜欢欧美流行音乐的大学生 | A2–B2 | 唱准歌词、学地道表达和连读 | 英文歌曲 |

---

## 4. 用户故事

格式：作为【角色】，我想要【功能】，以便【价值】。优先级 P0 为 MVP 必须，P1 为首个迭代，P2 为后续候选。

### 4.1 通用与上手

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-01 | 作为新用户，我想要一个引导流程帮我完成水平自测、服务配置和麦克风授权，以便尽快开始第一次对话 | P0 | 引导可跳过；每步失败给出中文原因和解决办法；完成后直接进入推荐场景 |
| US-02 | 作为用户，我想要选择语音或文字方式练习，以便在不方便说话时也能练 | P0 | 两种模式共用场景、复盘和存档 |
| US-03 | 作为用户，我想要在 AI 说话时直接打断它，以便对话更自然 | P0 | 检测到用户开口后 300 ms 内停止 AI 播放 |
| US-04 | 作为用户，我想要一个悬浮字幕窗显示双方的话，以便边听边看、不漏信息 | P0 | 字幕窗置顶、可拖动、可锁定穿透；跨桌面空间可见 |
| US-05 | 作为用户，我想要点击字幕中的单词查看释义并收藏，以便积累生词 | P0 | 显示音标、中文释义、语境含义；收藏后进入词汇本 |
| US-06 | 作为用户，我想要收藏整句话，以便之后复习好的表达 | P1 | 只有已完成的字幕可收藏；重复收藏不重复入库 |
| US-07 | 作为用户，我想要用快捷键开关麦克风，以便不用切换窗口 | P1 | 默认 ⌥Space，可自定义；菜单栏图标显示当前状态 |

### 4.2 日常生活场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-10 | 作为日常交流者，我想要从“点餐、问路、酒店入住、看病”等情景卡片中选择练习，以便有针对性地准备 | P0 | 首发 ≥ 20 个情景，每个情景有目标、AI 角色和 3 个任务点 |
| US-11 | 作为日常交流者，我想要 AI 扮演店员、医生等角色并主动推进对话，以便练习真实交流 | P0 | AI 保持角色，不主动切换到教学口吻，除非用户请求帮助 |
| US-12 | 作为日常交流者，我想要卡住时点“提示”获得可以说的句子，以便对话不中断 | P0 | 提供 2–3 个难度递进的参考说法，可点读 |
| US-13 | 作为日常交流者，我想要自由闲聊模式，以便练习无固定主题的对话 | P1 | AI 根据用户兴趣标签选话题 |

### 4.3 英语考试（雅思）场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-20 | 作为雅思考生，我想要完整模拟 Part 1/2/3 口语考试，以便熟悉流程和节奏 | P0 | 考官角色按题型推进；Part 2 有 1 分钟准备、2 分钟陈述计时，可记笔记 |
| US-21 | 作为雅思考生，我想要单独练习某一个 Part 或某个话题，以便集中突破 | P0 | 可按 Part、话题筛选题目；题库支持按季度更新 |
| US-22 | 作为雅思考生，我想要考后获得按官方四项标准的预估分数，以便了解差距 | P0 | 输出流利度与连贯性、词汇丰富度、语法多样性与准确性、发音四项分和总分，并附“非官方评分”声明 |
| US-23 | 作为雅思考生，我想要看到参考高分答案和我回答的对比，以便学习结构和表达 | P1 | 按题目展示我的回答、改进版回答和改动说明 |
| US-24 | 作为雅思考生，我想要看到我的分数随时间变化，以便判断是否可以报名考试 | P1 | 学习中心展示四项分趋势 |

### 4.4 商务工作场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-30 | 作为职场人，我想要练习会议发言、项目汇报、客户电话、谈判、英文面试等场景，以便应对真实工作 | P0 | 首发 ≥ 12 个商务情景 |
| US-31 | 作为职场人，我想要填写自己的行业、职位和真实议题，以便 AI 模拟我实际会遇到的对话 | P0 | 背景信息可保存为“我的工作档案”，开始练习时自动带入 |
| US-32 | 作为职场人，我想要得到语气正式度和礼貌程度的反馈，以便说话更得体 | P0 | 复盘中单独列出“语气与得体性”问题，并给出更专业的说法 |
| US-33 | 作为职场人，我想要上传会议议程或演讲稿，让 AI 基于它提问，以便会前演练 | P1 | 支持 .txt / .md / .pdf / .docx，≤ 20 页；内容只发送给用户配置的模型 |

### 4.5 英文歌曲场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-40 | 作为兴趣学习者，我想要导入自己的歌曲音频和歌词，以便用喜欢的歌学英语 | P0 | 支持 mp3/m4a/wav 和 .lrc/.txt 歌词；无时间轴时可手动或自动对齐 |
| US-41 | 作为兴趣学习者，我想要逐句跟唱或跟读，以便练习发音和连读 | P0 | 单句循环、0.5–1.0 倍速、录音回放对比 |
| US-42 | 作为兴趣学习者，我想要每句的发音评分和问题单词标注，以便知道哪里唱得不准 | P1 | 显示句子得分和低分单词，点击可听标准读音 |
| US-43 | 作为兴趣学习者，我想要 AI 讲解歌词中的俚语、连读和文化背景，以便理解歌曲含义 | P0 | 按句展示讲解，可收藏到词汇和句子本 |
| US-44 | 作为兴趣学习者，我想要和 AI 聊这首歌的主题，以便把歌词表达用到对话里 | P1 | 从歌曲页一键进入“歌曲话题对话” |

### 4.6 复盘与学习

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-50 | 作为用户，我想要每次练习后自动生成复盘，以便知道自己哪里说错了 | P0 | 包含总结、逐句纠错、按场景的评分、薄弱点、推荐下一次练习 |
| US-51 | 作为用户，复盘失败时我不想丢失对话，并能重新生成 | P0 | 对话先落盘；历史详情提供“重新生成复盘” |
| US-52 | 作为用户，我想要按间隔重复复习单词和句子，以便记得牢 | P0 | 四档评分（重来/困难/一般/简单）调整下次复习时间；复习过程不调用 LLM |
| US-53 | 作为用户，我想要看到反复出现的薄弱点以及它们是否在改善，以便有针对性地练 | P1 | 同类问题合并；显示出现次数、最近出现日期、状态 |
| US-54 | 作为用户，我想要一键带着上次的薄弱点开始下一次练习，以便持续改进 | P0 | 新练习的系统提示词包含薄弱点，复盘中单独评价这些点的表现 |
| US-55 | 作为用户，我想要设置每日练习目标并收到提醒，以便坚持下去 | P1 | 目标分钟数可设；macOS 通知提醒；显示连续天数 |
| US-56 | 作为用户，我想要导出我的学习数据，以便备份或导入 Anki | P2 | 全量备份为 zip；词汇和句子可导出 CSV |

---

## 5. 功能模块拆解

### 5.1 模块总览

```text
SpeakLoop
├── M1 账户与引导（本地）
│   ├── 首次引导向导
│   ├── 水平自测（CEFR 估计）
│   └── 学习目标与兴趣设置
├── M2 场景中心
│   ├── M2.1 日常生活
│   ├── M2.2 英语考试（雅思）
│   ├── M2.3 商务工作
│   ├── M2.4 英文歌曲
│   └── M2.5 自由对话 / 自定义场景
├── M3 练习引擎（场景共用）
│   ├── 会话编排（提示词组装、角色、回合管理）
│   ├── 语音链路（录音、VAD、ASR、TTS、打断）
│   ├── 文字链路
│   ├── 实时提示（卡壳求助）
│   └── 断点保存与恢复
├── M4 字幕与查词
│   ├── 悬浮字幕窗
│   ├── 点词查询（本地词典 + LLM 语境释义）
│   └── 单词 / 句子收藏
├── M5 复盘与评估
│   ├── 通用复盘（纠错、总结、建议）
│   ├── 场景评估器（日常 / 雅思 / 商务 / 歌曲）
│   ├── 长对话分段复盘
│   └── 复盘重试
├── M6 学习中心
│   ├── 学习概览与趋势
│   ├── 历史记录与检索
│   ├── 词汇本与复习
│   ├── 句子本与复习
│   └── 薄弱点追踪
├── M7 设置
│   ├── AI 服务配置（LLM / ASR / TTS / 发音评测）
│   ├── 音频设备与快捷键
│   ├── 字幕外观
│   └── 隐私与数据管理
└── M8 系统能力
    ├── 菜单栏常驻与全局快捷键
    ├── 通知提醒
    ├── 自动更新
    └── 诊断日志
```

### 5.2 M1 账户与引导

不设云端账号，所有数据保存在本机，用“本地档案”保存用户偏好。

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 引导向导 | 欢迎 → 选择学习目标（可多选四大场景）→ 水平自测 → 配置 AI 服务 → 授权麦克风并测试 → 推荐首个练习 | P0 |
| 水平自测 | AI 进行 3 分钟问答，给出 CEFR 估计（A1–C1），用户可手动修改 | P1 |
| 学习目标 | 每日目标分钟数、目标考试分数（雅思）、行业与职位（商务） | P0 |
| 服务预设 | 提供“推荐配置”（一个 LLM Key 即可开始，ASR/TTS 使用系统能力兜底），以及“高级配置” | P0 |

### 5.3 M2 场景中心

所有场景共用一个**场景定义模型**（见 8.3 `Scenario`），差异体现在 AI 角色、练习结构、计时规则和评估器上。

| 维度 | 日常生活 | 英语考试（雅思） | 商务工作 | 英文歌曲 |
| --- | --- | --- | --- | --- |
| 练习形式 | 角色扮演、任务驱动对话 | 结构化模拟考试 | 角色扮演、材料驱动演练 | 跟读跟唱 + 歌词讲解 + 话题对话 |
| AI 角色 | 店员、医生、房东、新朋友等 | 考官 | 同事、经理、客户、面试官 | 歌词讲解老师、发音教练 |
| 结构 | 开场 → 3 个任务点 → 自然结束 | Part 1（4–5 分钟）→ Part 2（准备 1 分钟 + 陈述 2 分钟 + 追问）→ Part 3（4–5 分钟） | 设定背景 → 主题讨论 → 追问与施压 → 总结 | 逐句练习，按句推进 |
| 计时 | 无 | 严格计时并显示倒计时 | 可选（如“3 分钟汇报”） | 跟随音频时间轴 |
| 评估重点 | 任务完成度、可理解度、常用表达 | 四项官方维度及预估分数 | 清晰度、专业度、得体性、逻辑结构 | 发音准确度、连读、节奏 |
| 卡壳提示 | 支持 | 模拟考试模式关闭，单项练习开启 | 支持 | 不适用 |

#### M2.1 日常生活

- **情景库**：按“出行、餐饮、购物、住宿、医疗、社交、办事”7 个分类组织，首发 ≥ 20 个情景。
- **情景卡片**：标题、难度、预计时长、AI 角色、任务点、关键表达预习（3–5 个）。
- **任务点**：例如“点一份不含坚果的主菜”“询问能否分开付款”。完成情况在对话中实时打勾，复盘中统计完成度。
- **自由闲聊**：根据用户兴趣标签生成话题，AI 以朋友口吻对话。

#### M2.2 英语考试（雅思）

- **模式**：完整模拟（Part 1–3 连续）、单 Part 练习、话题专项。
- **题库**：Part 1 话题、Part 2 话题卡（Cue Card）及 Part 3 追问。题库以 JSON 资源随版本发布，支持按季度更新。题目由团队原创或改编，不收录真题原文（见风险 R3）。
- **Part 2 流程**：展示话题卡 → 1 分钟准备（可打字记笔记）→ 2 分钟陈述（计时，到点由考官打断）→ 1–2 个追问。
- **评分**：按流利度与连贯性（FC）、词汇丰富度（LR）、语法多样性与准确性（GRA）、发音（P）四项给出 0–9 分（0.5 分一档）及总分，并给出每项依据。发音分依赖发音评测服务，未配置时标注“未评估”。
- **答案对比**：每道题展示“我的回答（转写）/ 改进版回答 / 修改要点”。
- **考试倒计时**：可设置考试日期，首页显示剩余天数和推荐练习强度。

#### M2.3 商务工作

- **情景库**：会议发言、项目进度汇报、客户电话、价格谈判、英文面试、闲聊破冰（Small Talk）、处理投诉、产品演示等，首发 ≥ 12 个。
- **工作档案**：行业、职位、公司类型、常用议题，开始练习时自动注入提示词。
- **材料驱动演练**：上传议程、演讲稿、简历或职位描述，AI 据此扮演对应角色提问。
- **得体性反馈**：复盘中单独列出过于口语、过于直接或不礼貌的表达，给出商务场合说法。
- **表达模板**：按情景沉淀常用句型（如表达异议、争取时间、总结行动项），可收藏复习。

#### M2.4 英文歌曲

- **版权原则**：应用不内置、不下载、不上传受版权保护的歌曲和歌词。用户只能导入自己合法拥有的音频和歌词；歌词与音频只在本地使用，送去 LLM 讲解时按句发送。
- **歌曲导入**：拖入音频文件和歌词文件；没有 .lrc 时间轴的，提供“自动对齐”（基于 ASR 时间戳）和“手动打点”两种方式。
- **歌词学习**：逐句展示原文、中文释义、俚语、连读弱读标注、文化背景。
- **跟唱跟读**：单句循环、A-B 复读、0.5–1.0 倍速；录下用户的跟读，回放对比；配置发音评测服务后给出句子得分和低分单词。
- **歌曲话题对话**：从歌曲页进入，AI 围绕歌曲主题和其中的表达与用户聊天。

#### M2.5 自由对话与自定义场景

- 用户可新建场景：名称、AI 角色设定、目标、难度、纠错强度，复用通用评估器。
- 场景可导出为 JSON 文件分享，导入时校验格式。

### 5.4 M3 练习引擎

| 子模块 | 功能说明 | 参考 SpeakHub |
| --- | --- | --- |
| 会话编排 | 组装系统提示词：基础规则（英语优先、处理残缺输入）+ 场景角色 + 难度 + 纠错强度 + 本次重点 + 薄弱点；维护 `system → user/assistant` 历史顺序 | `direct-chat-prompt.ts`、`learning-service.ts` |
| 语音输入 | 麦克风采集 16 kHz → 本地 VAD 断句 → 流式 ASR；支持选择输入设备 | `local-speech-audio.ts`、`speech-vad-worker.ts`、`aliyun-asr.ts` |
| 语音输出 | LLM 流式文本 → 分句 → 流式 TTS → 连续播放；最多预取 2 句 | `speech-segments.ts`、`speech-synthesis-queue.ts` |
| 打断 | 用户开口或提交新文字时，中止当前 LLM 请求和 TTS 播放，未说完的回复标记为“被打断” | `barge-in-policy.ts`、`interruptible-task-handoff.ts` |
| 实时语音模型（可选） | 支持端到端实时语音模型（如 OpenAI Realtime、Gemini Live API），由用户自带 Key；转写仍走统一字幕与存档链路 | 新增 |
| 卡壳提示 | 用户点“提示”或沉默超过 8 秒（可配置）时，生成 2–3 条参考说法 | 新增 |
| 断点保存 | 每条完整字幕即时写入本地；异常退出后下次启动提示恢复 | `session-checkpoint.ts` |
| 关闭安全 | 结束练习时先停止接收新语音，等待原生推理调用结束后再释放 worker | SpeakHub “API Voice Shutdown Safety” |

### 5.5 M4 字幕与查词

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 悬浮字幕窗 | 置顶、可拖动缩放、可锁定（锁定后鼠标穿透）、在所有桌面空间和全屏应用上方可见；显示双方或仅一方 | P0 |
| 字幕外观 | 字号、颜色、背景（透明 / 毛玻璃 / 纯色）、最大行数 | P0 |
| 点词查询 | 本地 ECDICT 词典即时返回音标和释义；LLM 补充语境含义和更自然的说法 | P0 |
| 收藏 | 单词收藏进入词汇本；整句收藏进入句子本，练习结束时随复盘一起生成句子讲解 | P0 / P1 |
| 主窗口内字幕 | 不开悬浮窗时，练习页内显示对话流 | P0 |

### 5.6 M5 复盘与评估

- **通用复盘**：总结、逐句纠错（原句 / 改写 / 原因 / 错误类型）、推荐词汇、薄弱点（最多 6 条，合并相近问题）、下一次练习建议。
- **场景评估器**：在通用复盘之上输出场景专属结果。

| 评估器 | 输出 |
| --- | --- |
| 日常 | 任务点完成情况、可理解度 1–5 分、常用表达推荐 |
| 雅思 | FC / LR / GRA / P 四项分数与总分、每项依据、逐题改进答案 |
| 商务 | 清晰度、专业度、得体性、结构四项 1–5 分，语气问题清单、表达模板 |
| 歌曲 | 逐句发音得分、问题单词、连读建议（依赖发音评测服务） |

- **客观指标**（本地计算，不依赖 LLM）：语速（词/分钟，仅语音）、平均回合词数、填充词次数（um、uh、like 等）、中文混用比例。
- **长对话处理**：先整份发送并流式接收结构化结果；只有超时或网关报 504 时才按完整回合切分重试，本地合并纠错，不设条数上限。
- **失败处理**：复盘失败时对话照常归档并标记“暂无复盘”，可在历史详情中重新生成。

### 5.7 M6 学习中心

| 子模块 | 功能 | 优先级 |
| --- | --- | --- |
| 概览 | 本周/本月练习时长、练习天数、连续天数、各场景评分趋势、CEFR 趋势、常见错误类型 | P0 |
| 历史记录 | 按场景、日期、模式、关键词筛选；详情页显示完整对话、复盘、收藏 | P0 |
| 词汇本 | 列表、熟悉度分组、到期复习；复习卡片四档评分 | P0 |
| 句子本 | 句子讲解（翻译、结构、可复用句型、表达、例句）、间隔重复复习 | P1 |
| 薄弱点追踪 | 同类薄弱点合并，显示出现次数、最近出现、状态（待改善 / 改善中 / 已克服），可一键专项练习 | P1 |
| 学习报告 | 每周生成一份学习报告，可导出为 PDF 或图片 | P2 |

**间隔重复规则（初版）**：采用简化 SM-2。初始间隔 1 天，评分“重来”重置为 10 分钟后再次出现，“困难”×1.2，“一般”×2.5，“简单”×3.5；连续 3 次“一般”或以上且间隔 ≥ 21 天标记为“已掌握”。

### 5.8 M7 设置

| 分组 | 设置项 |
| --- | --- |
| AI 服务 | LLM（Base URL、模型、Key，兼容 OpenAI 接口）、ASR（Apple 语音识别 / 云端服务）、TTS（Apple 系统语音 / 云端服务 / 本地 Kokoro）、发音评测服务、实时语音模型；每项提供“检测连通性” |
| 音频 | 输入设备、输出设备、麦克风检测、VAD 灵敏度、静音提示时长 |
| 练习偏好 | 默认难度、纠错强度、AI 语速、口音偏好（美式 / 英式） |
| 字幕 | 外观、默认显示方式 |
| 快捷键 | 麦克风开关、显示/隐藏字幕、结束练习 |
| 提醒 | 每日目标、提醒时间 |
| 隐私与数据 | 数据目录、导出/导入、清除全部数据、匿名统计开关 |
| 关于 | 版本、检查更新、导出诊断包 |

### 5.9 M8 系统能力

- **菜单栏常驻**：显示今日进度、麦克风状态，可快速开始上次的场景。
- **全局快捷键**：使用系统全局快捷键注册，冲突时提示。
- **通知**：每日提醒、复盘生成完成、复习到期。
- **自动更新**：DMG 首次安装，后续通过签名并公证的更新包自动升级。
- **诊断**：本地诊断日志保留 7 天，导出时自动脱敏 API Key 和对话正文。

---

## 6. 核心业务流程

### 6.1 一次练习的主流程

```text
选择场景 → 场景详情（目标、角色、预习表达）→ 确认设置（模式、难度、纠错强度、本次重点）
→ 开始练习
    → 会话编排生成系统提示词
    → AI 开场
    → 循环：用户说话 → VAD 断句 → ASR 转写 → 字幕 + 本地保存 → LLM 流式回复 → 分句 TTS → 播放
             （任意时刻用户开口 → 打断当前回复）
→ 结束练习
    → 停止音频链路（等待推理调用结束）
    → 生成复盘（通用 + 场景评估器 + 收藏句子讲解）
    → 归档入库、更新词汇本 / 句子本 / 薄弱点 / 统计
→ 复盘页 → “带薄弱点开始下一次练习”
```

### 6.2 雅思模拟考流程

```text
选择“完整模拟” → 系统抽取 Part 1 话题 2 个、Part 2 话题卡 1 张、对应 Part 3 话题
→ Part 1：考官提问 4–6 题（约 4–5 分钟）
→ Part 2：展示话题卡 → 准备 60 秒（可记笔记）→ 陈述 ≤ 120 秒（到时考官打断）→ 追问 1–2 题
→ Part 3：围绕 Part 2 主题深入讨论（约 4–5 分钟）
→ 结束 → 雅思评估器输出四项分数、逐题改进答案 → 记入雅思分数趋势
```

### 6.3 歌曲学习流程

```text
导入音频 + 歌词 → 时间轴检查（有 LRC 直接使用 / 自动对齐 / 手动打点）
→ 歌曲详情：逐句歌词 + 讲解（首次打开时按需生成并缓存）
→ 选择句子 → 播放原声 → 跟读/跟唱录音 → 回放对比 → 发音评分（已配置时）
→ 收藏表达 → 进入词汇本 / 句子本
→ 可选：进入“歌曲话题对话”
```

### 6.4 异常恢复流程

```text
应用启动 → 检测到未完成练习（current session 存在且未结束）
→ 弹窗：“发现一次未完成的练习（时间、场景、回合数）”
   → 归档并生成复盘 / 仅归档 / 丢弃
```

---

## 7. 页面结构设计

### 7.1 信息架构

```text
主窗口（侧边栏导航）
├── 首页 Today
├── 场景 Practice
│   ├── 场景中心（四大场景 + 自由对话 + 我的场景）
│   ├── 场景列表（某一大类下的情景卡片）
│   ├── 场景详情 / 练习设置
│   └── 练习中（Live Session）
├── 歌曲 Songs
│   ├── 歌曲库
│   └── 歌曲详情 / 跟唱
├── 复盘 Review（练习结束后进入，也可从历史打开）
├── 学习 Learn
│   ├── 概览
│   ├── 历史记录 → 历史详情
│   ├── 词汇本 → 词汇复习
│   ├── 句子本 → 句子复习
│   └── 薄弱点
└── 设置 Settings

独立窗口与系统入口
├── 首次引导向导（独立窗口）
├── 悬浮字幕窗
├── 菜单栏下拉面板
└── 系统通知
```

### 7.2 页面清单

| 编号 | 页面 | 入口 | 主要内容 | 关键操作 | 优先级 |
| --- | --- | --- | --- | --- | --- |
| P01 | 引导向导 | 首次启动 | 6 步：目标 → 自测 → AI 服务 → 麦克风 → 偏好 → 推荐练习 | 下一步、跳过、检测连通性 | P0 |
| P02 | 首页 | 侧边栏 | 今日目标进度环、连续天数、今日待复习数、继续上次练习、推荐练习（基于薄弱点）、雅思考试倒计时 | 一键开始、去复习 | P0 |
| P03 | 场景中心 | 侧边栏 | 四大场景大卡片（含进度）、自由对话、我的场景 | 进入场景、新建场景 | P0 |
| P04 | 场景列表 | P03 | 分类标签、情景卡片（难度、时长、完成状态） | 筛选、搜索、收藏情景 | P0 |
| P05 | 场景详情 / 练习设置 | P04 | 情景目标、AI 角色、任务点、预习表达；模式、难度、纠错强度、本次重点、是否带薄弱点 | 开始练习 | P0 |
| P06 | 练习中 | P05 | 对话流、AI 状态（聆听/思考/说话）、任务点进度或雅思计时器、文字输入框、提示按钮、麦克风开关 | 打断、求助、开关字幕窗、结束并复盘 | P0 |
| P07 | 复盘页 | P06 结束 / 历史详情 | 总结、场景评分卡、客观指标、逐句纠错、薄弱点、推荐词汇、收藏句子讲解、下一次建议 | 收藏、重新生成、带薄弱点再练 | P0 |
| P08 | 歌曲库 | 侧边栏 | 已导入歌曲列表、学习进度 | 导入歌曲、删除 | P0 |
| P09 | 歌曲详情 / 跟唱 | P08 | 歌词逐句列表、当前句讲解、播放控制、录音对比、发音得分 | 单句循环、变速、录音、收藏、进入话题对话 | P0 |
| P10 | 学习概览 | 侧边栏 | 时长与天数、各场景评分趋势、CEFR 趋势、雅思四项趋势、常见错误 | 切换周/月 | P0 |
| P11 | 历史记录 / 详情 | P10 | 列表 + 筛选；详情同复盘页并附完整对话 | 搜索、删除、重新生成复盘 | P0 |
| P12 | 词汇本 / 复习 | P10 | 词汇列表与熟悉度；复习卡片 | 评分、编辑释义、导出 | P0 |
| P13 | 句子本 / 复习 | P10 | 句子与讲解；复习卡片 | 评分、标记已掌握 | P1 |
| P14 | 薄弱点 | P10 | 薄弱点列表、出现次数、状态、关联练习 | 专项练习 | P1 |
| P15 | 设置 | 侧边栏 / ⌘, | 见 5.8 | 保存、检测、导出 | P0 |
| P16 | 悬浮字幕窗 | 练习中开启 | 双方字幕、点词释义浮层、收藏按钮、文字输入（可选） | 拖动、缩放、锁定、点词、收藏 | P0 |
| P17 | 菜单栏面板 | 菜单栏图标 | 今日进度、麦克风状态、继续练习 | 开始/结束、打开主窗口 | P1 |

### 7.3 关键页面线框说明

**P06 练习中**

```text
┌───────────────────────────────────────────────────────────┐
│ ← 日常生活 · 餐厅点餐   B1 · 标准纠错        ⏱ 06:32   [结束并复盘] │
├───────────────────────────────────────────────────────────┤
│ 任务点：☑ 告诉服务员人数  ☐ 点一份不含坚果的主菜  ☐ 分开付款     │
├───────────────────────────────────────────────────────────┤
│  🤖 Good evening! Table for how many?                     │
│                         I'd like a table for two, please. 🧑 │
│  🤖 Sure, right this way. ...（说话中）                      │
│                                                           │
├───────────────────────────────────────────────────────────┤
│ ● 聆听中   [🎤 开/关 ⌥Space]  [💡 提示]  [🗔 字幕窗]  [输入文字…]  │
└───────────────────────────────────────────────────────────┘
```

雅思模式下，任务点栏替换为“Part 进度 + 倒计时 + 笔记区（仅 Part 2 准备阶段）”，提示按钮隐藏。

**P07 复盘页**

```text
┌ 总结 ──────────────────────────────────────────┐
│ 一段话总结 + 场景评分卡（如雅思：FC 6.0 LR 6.5 GRA 5.5 P 6.0 → 6.0）│
├ 客观指标 ──────────────────────────────────────┤
│ 语速 118 词/分 · 平均回合 14 词 · 填充词 9 次 · 中文混用 3%       │
├ 逐句纠错（默认显示 5 条，可展开全部）────────────────┤
├ 薄弱点（≤ 6）· 推荐词汇 · 收藏句子讲解 ───────────────┤
└ [带薄弱点开始下一次]  [重新生成复盘]                     ┘
```

---

## 8. 数据结构设计（初步）

### 8.1 存储方案

| 数据 | 存储方式 | 说明 |
| --- | --- | --- |
| 结构化学习数据 | SQLite（`~/Library/Application Support/SpeakLoop/speakloop.db`） | 会话、回合、复盘、词汇、句子、薄弱点、统计 |
| 对话归档 | Markdown 文件（同目录 `archives/`） | 便于用户阅读和备份；数据库是检索和统计的来源 |
| API Key | macOS 钥匙串 | 不写入数据库和配置文件 |
| 场景与题库 | 应用内置 JSON 资源 + 用户自定义（数据库） | 内置资源随版本更新 |
| 歌曲音频与录音 | 本地文件（`songs/`、`recordings/`） | 数据库只存路径和元数据 |
| 设置 | JSON 配置文件 | 不含密钥 |

写入规则：所有学习数据只通过数据层（Repository）写入，与 SpeakHub 中 `store.ts` 是唯一写入入口的规则一致；数据库变更通过迁移脚本执行。

### 8.2 实体关系

```text
UserProfile 1 ── * PracticeSession * ── 1 Scenario
PracticeSession 1 ── * Turn
PracticeSession 1 ── 0..1 Review 1 ── * ReviewIssue
Review 1 ── 0..1 ScenarioAssessment
PracticeSession * ── * WeakPoint        （WeakPointOccurrence 关联）
VocabularyItem * ── * PracticeSession   （VocabularyOccurrence 关联）
SavedSentence * ── 1 Turn
Song 1 ── * LyricLine 1 ── * ShadowingAttempt
VocabularyItem / SavedSentence 1 ── 1 ReviewSchedule
```

### 8.3 核心实体定义（TypeScript）

```ts
// ---------- 通用枚举 ----------
type ScenarioCategory = 'daily' | 'exam-ielts' | 'business' | 'song' | 'free' | 'custom'
type PracticeMode = 'voice' | 'text'
type CefrLevel = 'A1' | 'A2' | 'B1' | 'B2' | 'C1' | 'C2'
type CorrectionStrength = 'light' | 'normal' | 'strict'
type SessionStatus = 'active' | 'completed' | 'interrupted' | 'recovered'
type Speaker = 'user' | 'assistant' | 'system-event'
type ErrorCategory =
  | 'grammar' | 'tense' | 'articles' | 'prepositions' | 'word-choice'
  | 'collocation' | 'pronunciation' | 'fluency' | 'coherence'
  | 'register' /* 语气与得体性 */ | 'interaction' | 'other'
type ReviewRating = 'again' | 'hard' | 'good' | 'easy'

// ---------- 用户档案 ----------
interface UserProfile {
  id: string
  displayName?: string
  nativeLanguage: 'zh-CN'
  estimatedLevel: CefrLevel
  goals: ScenarioCategory[]
  dailyGoalMinutes: number            // 默认 15
  accent: 'us' | 'uk'
  interests: string[]                 // 自由对话选题用
  ieltsTarget?: { overall: number; examDate?: string }
  workProfile?: {
    industry: string
    role: string
    companyType?: string
    commonTopics: string[]
  }
  createdAt: string
  updatedAt: string
}

// ---------- 场景 ----------
interface Scenario {
  id: string
  category: ScenarioCategory
  subcategory?: string                // 如 'dining'、'meeting'、'part2'
  title: string
  titleZh: string
  description: string
  level: CefrLevel                    // 推荐难度
  estimatedMinutes: number
  aiRole: { name: string; persona: string; voice?: string }
  objectives: ScenarioObjective[]     // 日常/商务任务点
  warmupExpressions: Array<{ phrase: string; meaning: string }>
  structure?: ExamStructure           // 仅雅思
  evaluatorId: 'daily' | 'ielts' | 'business' | 'song' | 'general'
  hintsEnabled: boolean
  builtIn: boolean
  version: number
}

interface ScenarioObjective {
  id: string
  description: string                 // 例：“点一份不含坚果的主菜”
}

interface ExamStructure {
  parts: Array<{
    part: 1 | 2 | 3
    durationSeconds: number
    prepSeconds?: number              // Part 2 = 60
    speakSeconds?: number             // Part 2 = 120
    questionIds: string[]
  }>
}

interface IeltsQuestion {
  id: string
  part: 1 | 2 | 3
  topic: string
  prompt: string
  cueCardPoints?: string[]            // Part 2 提示要点
  followUps?: string[]
  season?: string                     // 题库季度，如 '2026-Q4'
}

// ---------- 练习会话 ----------
interface PracticeSession {
  id: string
  scenarioId: string
  scenarioCategory: ScenarioCategory
  mode: PracticeMode
  level: CefrLevel
  correctionStrength: CorrectionStrength
  focus?: string                      // 本次重点
  carriedWeakPointIds: string[]       // 带入的薄弱点
  aiProvider: { llm: string; asr?: string; tts?: string; realtime?: string }
  status: SessionStatus
  startedAt: string
  endedAt?: string
  durationSeconds: number
  userSpeakingSeconds?: number        // 计算语速用
  objectiveProgress: Array<{ objectiveId: string; done: boolean; turnId?: string }>
  songId?: string                     // 歌曲话题对话
  archivePath: string                 // Markdown 归档
}

interface Turn {
  id: string
  sessionId: string
  index: number
  speaker: Speaker
  text: string
  status: 'streaming' | 'complete'
  interrupted: boolean
  examPart?: 1 | 2 | 3
  questionId?: string
  startedAt: string
  endedAt?: string                    // 语音回合起止，计算语速
  audioPath?: string                  // 可选保存用户录音（默认关闭）
}

// ---------- 复盘与评估 ----------
interface Review {
  id: string
  sessionId: string
  status: 'pending' | 'completed' | 'failed'
  summary: string
  nextPracticeSuggestion: string
  estimatedCefr?: CefrLevel
  generalScores?: { accuracy: number; vocabulary: number; fluency: number; interaction: number } // 1–5
  metrics: SpeakingMetrics
  weakPoints: string[]                // 本次摘要，≤ 6 条为目标，超出不报错
  recommendedVocabulary: Array<{ term: string; meaning: string; example?: string }>
  scenarioAssessment?: ScenarioAssessment
  model: string
  errorMessage?: string
  createdAt: string
}

interface ReviewIssue {
  id: string
  reviewId: string
  turnId?: string
  original: string
  improved: string
  reason: string
  category: ErrorCategory
}

interface SpeakingMetrics {
  wordsPerMinute?: number             // 仅语音模式
  averageTurnWords: number
  fillerWordCount: number
  chineseMixRatio: number             // 0–1
  userTurnCount: number
}

type ScenarioAssessment =
  | { type: 'daily'; objectivesCompleted: number; objectivesTotal: number; intelligibility: number }
  | { type: 'ielts'; fc: number; lr: number; gra: number; p?: number; overall: number;
      rationale: Record<'fc' | 'lr' | 'gra' | 'p', string>;
      answers: Array<{ questionId: string; userAnswer: string; improvedAnswer: string; notes: string[] }> }
  | { type: 'business'; clarity: number; professionalism: number; register: number; structure: number;
      registerIssues: Array<{ original: string; better: string; reason: string }> }
  | { type: 'song'; averageLineScore?: number; weakWords: string[] }

// ---------- 薄弱点 ----------
interface WeakPoint {
  id: string
  normalizedKey: string               // 合并相近问题用
  title: string                       // 例：“过去时态使用不一致”
  category: ErrorCategory
  status: 'open' | 'improving' | 'resolved'
  occurrenceCount: number
  firstSeenAt: string
  lastSeenAt: string
  consecutiveAbsentSessions: number   // 连续 N 次未出现 → improving / resolved
}

interface WeakPointOccurrence {
  weakPointId: string
  sessionId: string
  evidence?: string                   // 原句片段
}

// ---------- 词汇与句子 ----------
interface VocabularyItem {
  id: string
  normalizedTerm: string
  term: string
  phonetic?: string
  meaning?: string
  contextualMeaning?: string
  example?: string
  source: 'subtitle' | 'review' | 'song' | 'manual'
  familiarity: 'new' | 'learning' | 'mastered'
  occurrenceCount: number
  firstSavedAt: string
  lastSavedAt: string
}

interface SavedSentence {
  id: string
  turnId?: string
  sessionId?: string
  songLineId?: string
  speaker: Speaker | 'song'
  text: string
  analysis?: {
    translation: string
    structure: string
    reusablePattern: string
    expressions: Array<{ phrase: string; meaning: string }>
    examples: string[]
    tip?: string
  }
  savedAt: string
}

interface ReviewSchedule {
  itemType: 'vocabulary' | 'sentence'
  itemId: string
  intervalDays: number
  easeFactor: number                  // 初始 2.5
  repetitions: number
  nextReviewAt: string
  lastReviewedAt?: string
  lastRating?: ReviewRating
}

// ---------- 歌曲 ----------
interface Song {
  id: string
  title: string
  artist?: string
  audioPath: string                   // 用户导入的本地文件
  durationSeconds: number
  lyricsSource: 'lrc' | 'txt-auto-aligned' | 'txt-manual'
  importedAt: string
  lastPracticedAt?: string
}

interface LyricLine {
  id: string
  songId: string
  index: number
  text: string
  startMs?: number
  endMs?: number
  explanation?: {                     // 首次查看时生成并缓存
    translation: string
    slangAndIdioms: Array<{ phrase: string; meaning: string }>
    connectedSpeech: string[]         // 连读、弱读说明
    culturalNote?: string
  }
}

interface ShadowingAttempt {
  id: string
  lyricLineId: string
  recordingPath: string
  score?: number                      // 0–100，需发音评测服务
  wordScores?: Array<{ word: string; score: number }>
  createdAt: string
}

// ---------- 统计（每日聚合） ----------
interface DailyStat {
  date: string                        // YYYY-MM-DD，本地时区
  practiceMinutes: number
  sessionCount: number
  byCategory: Partial<Record<ScenarioCategory, number>>  // 分钟
  reviewedItems: number
  goalMet: boolean
}
```

### 8.4 SQLite 表清单（初稿）

| 表名 | 主键 | 关键字段 | 索引 |
| --- | --- | --- | --- |
| `user_profile` | id | level, goals(json), daily_goal_minutes, work_profile(json) | – |
| `scenarios` | id | category, subcategory, built_in, version, payload(json) | category |
| `ielts_questions` | id | part, topic, season | part, topic |
| `sessions` | id | scenario_id, category, mode, status, started_at, duration_seconds | started_at, category, status |
| `turns` | id | session_id, index, speaker, text, status, interrupted, started_at, ended_at | (session_id, index) |
| `reviews` | id | session_id, status, summary, metrics(json), assessment(json), model | session_id |
| `review_issues` | id | review_id, turn_id, category | review_id, category |
| `weak_points` | id | normalized_key, status, occurrence_count, last_seen_at | normalized_key 唯一 |
| `weak_point_occurrences` | (weak_point_id, session_id) | evidence | session_id |
| `vocabulary` | id | normalized_term, familiarity | normalized_term 唯一 |
| `vocabulary_occurrences` | (vocabulary_id, session_id) | – | session_id |
| `sentences` | id | turn_id, song_line_id, text, analysis(json) | session_id |
| `review_schedules` | (item_type, item_id) | next_review_at, interval_days, ease_factor | next_review_at |
| `songs` | id | title, artist, audio_path | – |
| `lyric_lines` | id | song_id, index, start_ms, explanation(json) | (song_id, index) |
| `shadowing_attempts` | id | lyric_line_id, score | lyric_line_id |
| `daily_stats` | date | practice_minutes, by_category(json) | – |
| `schema_migrations` | version | applied_at | – |

---

## 9. API 初稿设计

本产品是本地桌面应用，“API”分为三层：

1. **应用内部接口**：Vue 界面层通过 Electron IPC 调用主进程服务层。下文按“`域.动作`”命名，统一请求/响应格式。
2. **事件推送**：服务层向界面推送实时字幕、语音状态等。
3. **外部 AI 服务适配层**：对接用户配置的 LLM、ASR、TTS、发音评测和实时语音服务。

### 9.1 通用约定

- 调用：`invoke(channel, payload) → Promise<Result<T>>`
- 返回格式：

```ts
type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: { code: ErrorCode; message: string /* 中文可展示 */; retryable: boolean; detail?: unknown } }

type ErrorCode =
  | 'VALIDATION_FAILED' | 'NOT_FOUND' | 'CONFLICT'
  | 'PROVIDER_UNAUTHORIZED' | 'PROVIDER_TIMEOUT' | 'PROVIDER_RATE_LIMITED' | 'PROVIDER_UNAVAILABLE'
  | 'MIC_PERMISSION_DENIED' | 'AUDIO_DEVICE_UNAVAILABLE'
  | 'SESSION_NOT_ACTIVE' | 'SESSION_ALREADY_ACTIVE'
  | 'INTERNAL'
```

- 所有请求参数用 schema（如 zod）在服务层校验；时间统一 ISO 8601；ID 使用 UUID v7。
- 事件订阅：`on(event, handler) → unsubscribe()`。

### 9.2 接口清单

#### 档案与引导

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `profile.get` | 获取用户档案 | – | `UserProfile` |
| `profile.update` | 更新档案（部分字段） | `Partial<UserProfile>` | `UserProfile` |
| `onboarding.getState` | 引导完成情况 | – | `{ completedSteps: string[]; done: boolean }` |
| `onboarding.completeStep` | 标记步骤完成 | `{ step: string }` | `{ done: boolean }` |
| `assessment.startPlacement` | 开始水平自测（一次特殊练习） | `{ mode: PracticeMode }` | `{ sessionId: string }` |

#### 场景与题库

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `scenario.list` | 场景列表 | `{ category?: ScenarioCategory; subcategory?: string; keyword?: string }` | `ScenarioSummary[]`（含完成次数、最近分数） |
| `scenario.get` | 场景详情 | `{ id }` | `Scenario` |
| `scenario.create` / `scenario.update` / `scenario.delete` | 自定义场景管理 | `Scenario` 字段 | `Scenario` / `{}` |
| `scenario.import` / `scenario.export` | 导入导出 JSON | `{ filePath }` / `{ id, filePath }` | `Scenario` / `{}` |
| `ielts.listQuestions` | 题库查询 | `{ part?: 1\|2\|3; topic?: string }` | `IeltsQuestion[]` |
| `ielts.drawMockExam` | 抽取一套模拟题 | `{ excludeRecentDays?: number }` | `ExamStructure` |

#### 练习会话

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `practice.start` | 开始练习 | 见下 | `{ session: PracticeSession; warnings: string[] }` |
| `practice.sendText` | 发送文字（会打断进行中的回复） | `{ sessionId; text; speakReply?: boolean }` | `{ turnId }` |
| `practice.interrupt` | 主动打断 AI | `{ sessionId }` | `{}` |
| `practice.requestHint` | 卡壳求助 | `{ sessionId }` | `{ hints: Array<{ level: 'simple'\|'natural'\|'advanced'; text: string }> }` |
| `practice.setMicrophone` | 开关麦克风 | `{ enabled: boolean }` | `{ enabled: boolean }` |
| `practice.examAdvance` | 雅思：进入下一阶段（准备结束、提前结束陈述） | `{ sessionId; to: 'part2-speak'\|'part2-followup'\|'part3' }` | `{ phase }` |
| `practice.end` | 结束并生成复盘 | `{ sessionId; generateReview: boolean }` | `{ session; reviewId?: string; reviewStatus: 'pending'\|'skipped' }` |
| `practice.getRecoverable` | 启动时检查未完成练习 | – | `RecoverableSession \| null` |
| `practice.recover` | 处理未完成练习 | `{ sessionId; action: 'archive-and-review'\|'archive'\|'discard' }` | `{ session? }` |

`practice.start` 请求示例：

```json
{
  "scenarioId": "ielts-mock-full",
  "mode": "voice",
  "level": "B2",
  "correctionStrength": "normal",
  "focus": "Use more complex sentences in Part 3",
  "carryWeakPointIds": ["wp_01H..."],
  "examQuestionSet": { "part1": ["q_101", "q_115"], "part2": ["q_207"], "part3": ["q_301", "q_302"] }
}
```

#### 复盘

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `review.get` | 获取复盘 | `{ sessionId }` | `Review & { issues: ReviewIssue[] }` |
| `review.regenerate` | 重新生成（失败或旧数据） | `{ sessionId }` | `{ reviewId; status: 'pending' }` |
| `review.estimateCost` | 复盘前估算字符数和 token | `{ sessionId }` | `{ characters: number; estimatedTokens: number }` |

#### 字幕、查词与收藏

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `dictionary.lookup` | 查词（本地优先，可选语境释义） | `{ word; contextSentence?: string; withContext: boolean }` | `{ query; phonetic?; definitions: string[]; contextualMeaning?; naturalAlternative? }` |
| `vocabulary.save` | 收藏单词 | `{ term; sessionId?; turnId?; source }` | `VocabularyItem` |
| `sentence.save` | 收藏句子（仅已完成的字幕） | `{ turnId } \| { songLineId }` | `SavedSentence` |
| `subtitle.updatePreferences` | 字幕外观 | `Partial<SubtitlePreferences>` | `SubtitlePreferences` |
| `subtitle.setVisible` / `subtitle.setLocked` | 显示、锁定 | `{ value: boolean }` | `{}` |

#### 学习中心

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `learning.dashboard` | 概览 | `{ period: 'week'\|'month'; category?: ScenarioCategory }` | `Dashboard`（时长、天数、连续天数、评分趋势、CEFR 趋势、雅思四项趋势、常见错误） |
| `history.search` | 历史检索 | `{ text?; category?; mode?; status?; dateFrom?; dateTo?; cursor?; limit? }` | `{ items: SessionSummary[]; nextCursor? }` |
| `history.get` | 历史详情 | `{ sessionId }` | `{ session; turns; review?; savedSentences }` |
| `history.delete` | 删除记录（级联删除关联数据） | `{ sessionId }` | `{}` |
| `vocabulary.list` | 词汇列表 | `{ familiarity?; due?: boolean; keyword?; cursor? }` | `{ items; nextCursor? }` |
| `sentence.list` | 句子列表 | `{ due?: boolean; keyword?; cursor? }` | `{ items; nextCursor? }` |
| `srs.getQueue` | 今日复习队列（不调用 LLM） | `{ itemType?: 'vocabulary'\|'sentence'; limit?: number }` | `Array<{ itemType; item; schedule }>` |
| `srs.rate` | 提交复习评分 | `{ itemType; itemId; rating: ReviewRating }` | `ReviewSchedule` |
| `weakPoint.list` | 薄弱点列表 | `{ status? }` | `WeakPoint[]` |
| `weakPoint.update` | 手动修改状态 | `{ id; status }` | `WeakPoint` |
| `weakPoint.startDrill` | 针对薄弱点生成专项练习 | `{ ids: string[]; mode }` | `{ session }` |
| `data.export` / `data.import` | 备份与恢复 | `{ filePath }` | `{ counts }` |
| `data.exportCsv` | 导出词汇/句子 CSV（Anki 可导入） | `{ itemType; filePath }` | `{ rows: number }` |

#### 歌曲

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `song.import` | 导入音频和歌词 | `{ audioPath; lyricsPath?; lyricsText? }` | `{ song; alignment: 'lrc'\|'needs-alignment' }` |
| `song.align` | 自动对齐歌词时间轴 | `{ songId }` | `{ lines: LyricLine[]; confidence: number }` |
| `song.updateTimings` | 手动打点 | `{ songId; timings: Array<{ lineId; startMs; endMs }> }` | `{}` |
| `song.list` / `song.get` / `song.delete` | 歌曲管理 | – | – |
| `song.explainLine` | 获取或生成单句讲解（结果缓存） | `{ lineId }` | `LyricLine['explanation']` |
| `song.submitAttempt` | 提交跟唱录音并评分 | `{ lineId; recordingPath }` | `ShadowingAttempt` |
| `song.startTopicChat` | 进入歌曲话题对话 | `{ songId; mode }` | `{ session }` |

#### 设置与服务

| 接口 | 说明 | 请求 | 响应 |
| --- | --- | --- | --- |
| `settings.get` / `settings.update` | 普通设置 | – / `Partial<Settings>` | `Settings` |
| `provider.saveCredentials` | 保存密钥到钥匙串 | `{ provider: ProviderKind; apiKey }` | `{ hasKey: true }` |
| `provider.check` | 检测连通性（不写入练习数据） | `{ provider: ProviderKind; config }` | `{ ok: boolean; message: string; latencyMs?: number }` |
| `provider.listModels` | 探测可用模型 | `{ baseUrl; apiKey? }` | `{ models: string[] }` |
| `audio.listDevices` | 音频设备 | – | `{ inputs; outputs }` |
| `audio.testMicrophone` | 4 秒麦克风检测 | `{ deviceId? }` | `{ result: 'ok'\|'silent'\|'permission-denied'\|'device-error'; peakLevel }` |
| `tts.preview` | 试听音色 | `{ provider; voice }` | `{ audio: Float32Array; sampleRate }` |
| `diagnostics.export` | 导出脱敏诊断包 | `{ filePath }` | `{ filePath }` |
| `app.checkUpdate` | 检查更新 | – | `{ available: boolean; version?; notes? }` |

### 9.3 事件推送

| 事件 | 负载 | 说明 |
| --- | --- | --- |
| `practice.lifecycle` | `{ sessionId; state: 'starting'\|'active'\|'ending'\|'ended'\|'error'; message? }` | 会话生命周期 |
| `practice.turn` | `Turn` | 字幕增量与完成；`status: 'streaming'` 时同一 `id` 持续更新 |
| `practice.voicePhase` | `{ phase: 'idle'\|'listening'\|'thinking'\|'synthesizing'\|'speaking' }` | 语音状态 |
| `practice.objective` | `{ objectiveId; done: true; turnId }` | 任务点完成 |
| `practice.examTimer` | `{ part; phase; remainingSeconds }` | 雅思计时 |
| `practice.audioChunk` | `{ packetId; generation; pcm: Float32Array; sampleRate; final: boolean }` | TTS 音频流 |
| `review.progress` | `{ sessionId; stage: 'requesting'\|'chunking'\|'merging'\|'saving'; percent? }` | 复盘进度 |
| `review.completed` / `review.failed` | `{ sessionId; reviewId }` / `{ sessionId; error }` | 复盘结果 |
| `srs.dueChanged` | `{ vocabulary: number; sentences: number }` | 待复习数变化 |
| `microphone.state` | `{ enabled; deviceId; level? }` | 麦克风状态 |

### 9.4 外部 AI 服务适配层

所有外部服务通过统一接口接入，可替换供应商。

```ts
interface LlmProvider {
  chat(req: {
    messages: Array<{ role: 'system' | 'user' | 'assistant'; content: string }>
    stream: boolean
    responseFormat?: 'text' | 'json'
    signal: AbortSignal
    timeoutMs: number                 // 对话 30 s，复盘 300 s
  }): AsyncIterable<{ delta: string }> | Promise<{ content: string }>
}

interface AsrProvider {
  startStream(opts: { language: 'en' | 'auto'; sampleRate: 16000; signal: AbortSignal }): {
    write(pcm: Float32Array): void
    end(): Promise<void>
    onResult(cb: (r: { text: string; final: boolean; startMs?: number; endMs?: number }) => void): void
  }
}

interface TtsProvider {
  synthesize(req: { text: string; voice: string; speed?: number; signal: AbortSignal }):
    AsyncIterable<{ pcm: Float32Array; sampleRate: number }>
}

interface PronunciationProvider {
  assess(req: { referenceText: string; audioPath: string; signal: AbortSignal }): Promise<{
    overall: number                   // 0–100
    words: Array<{ word: string; score: number; errorType?: 'mispronunciation' | 'omission' | 'insertion' }>
  }>
}

interface RealtimeVoiceProvider {
  connect(opts: { systemPrompt: string; voice: string; signal: AbortSignal }): {
    sendAudio(pcm: Float32Array): void
    sendText(text: string): void
    interrupt(): void
    onTranscript(cb: (t: { speaker: Speaker; text: string; final: boolean }) => void): void
    onAudio(cb: (pcm: Float32Array, sampleRate: number) => void): void
    close(): Promise<void>
  }
}
```

| 能力 | 默认方案 | 可选方案 |
| --- | --- | --- |
| LLM | 任意 OpenAI 兼容 `/v1/chat/completions`（SSE 流式） | DeepSeek、通义、Claude、GPT 等 |
| ASR | macOS 系统语音识别（Speech 框架，免费，离线可用） | 阿里 Paraformer、Whisper 类云端服务 |
| TTS | macOS 系统语音（免费兜底） | 云端流式 TTS、本地 Kokoro |
| 发音评测 | 未配置时不评估 | 支持发音评测的云服务（待选型） |
| 实时语音 | 不启用 | OpenAI Realtime、Gemini Live API |

复盘请求约定（与 SpeakHub 一致）：先整份发送转写并要求返回 JSON；超过 8,000 字符时使用流式接收；只有 504 或超时才按完整回合切分重试；解析时去除空项和重复项，薄弱点超出 6 条不报错。

---

## 10. 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 语音回合延迟 P50 ≤ 1.5 s、P90 ≤ 3 s（说完到开始播放，网络正常）；打断响应 ≤ 300 ms；学习中心在 1,000 次练习内打开 ≤ 1 s；冷启动 ≤ 3 s |
| 稳定性 | 每条完整字幕即时落盘；复盘失败不丢对话；结束练习时等待原生推理调用结束再释放资源；外部服务断线自动重连并提示 |
| 隐私 | 数据默认只存本机；API Key 存钥匙串；匿名统计不含练习内容，可关闭；录音默认不保存；诊断包自动脱敏 |
| 安全 | 渲染层与系统能力隔离，只暴露白名单接口；外链只允许白名单域名；更新包校验签名 |
| 兼容性 | macOS 13+；Apple Silicon 与 Intel；外接麦克风、AirPods 等蓝牙耳机；深色模式 |
| macOS 规范 | 申请麦克风权限并提供用途说明；Developer ID 签名与公证；支持 ⌘, 打开设置、⌘W 关闭窗口；字幕窗在全屏应用上方显示 |
| 可访问性 | 支持系统字号；主要操作有键盘快捷键；VoiceOver 可读出主要控件 |
| 可测试性 | 解析、分句、入库、复习调度、评估结果校验均有单元测试；练习主链路有集成测试 |
| 本地化 | 界面中文；练习内容英文；后续可扩展英文界面 |

---

## 11. 版本规划

| 版本 | 主题 | 范围 | 发布门槛 |
| --- | --- | --- | --- |
| v0.1 内测 | 能练起来 | 引导、日常生活场景（10 个情景）、通用练习引擎（语音 + 文字）、字幕窗与查词、通用复盘、历史记录、设置 | 三种 AI 配置组合下完成 30 分钟连续对话无中断 |
| v0.5 公测 | 四大场景 | 雅思模拟考与评分、商务场景与工作档案、歌曲导入与讲解、词汇本复习、学习概览 | 雅思评分结果结构校验通过；歌曲导入覆盖 LRC 与纯文本 |
| v1.0 正式版 | 学习闭环 | 句子本复习、薄弱点追踪、带薄弱点再练、跟唱发音评分、每日目标与提醒、菜单栏、自动更新 | 成功指标埋点上线；签名公证通过 |
| v1.x | 增强 | 实时语音模型、材料驱动演练、学习报告、数据导出、自定义场景分享 | 按用户反馈排序 |

---

## 12. 风险与待决问题

### 12.1 风险

| 编号 | 风险 | 影响 | 应对 |
| --- | --- | --- | --- |
| R1 | 雅思评分不准，用户误把预估当真实成绩 | 信任受损 | 明确标注“AI 预估，非官方评分”；展示每项评分依据；内测用有真实成绩的用户校准 |
| R2 | 歌曲与歌词版权 | 法律风险 | 不内置、不下载、不分享歌曲和歌词；只处理用户本地导入的文件；讲解按句发送；分享功能不包含歌词 |
| R3 | 雅思题库版权与时效 | 法律风险、内容过时 | 题目原创或改编，不收录真题原文；按季度更新话题 |
| R4 | 外部服务延迟和费用不可控 | 体验差、用户成本 | 系统 ASR/TTS 兜底；显示延迟和复盘消耗估算；推荐性价比配置 |
| R5 | 发音评测服务选型未定 | 雅思发音分、歌曲评分无法上线 | v0.5 前完成选型；未配置时显示“未评估”，不估造分数 |
| R6 | 用户配置门槛高 | 首次对话完成率低 | 引导向导 + 推荐配置（一个 LLM Key 起步）+ 每步连通性检测 |
| R7 | 开发者需新学 TypeScript 与 Electron | 前期进度变慢 | 主进程按 Spring 式分层（ipc / service / repository）组织；先搭骨架再逐个功能迭代；优先迁移 SpeakHub 已验证的模块 |

### 12.2 待决问题

- [x] 技术栈：已定为 Electron + Vue 3 + TypeScript（2026-11-10），理由见附录 13.2。
- [ ] 是否需要提供官方托管的 AI 服务（免配置，需要账号和付费），还是坚持用户自带 Key？
- [ ] 发音评测服务选型和费用。
- [ ] 雅思题库由谁来编写和维护？
- [ ] 是否保存用户录音（用于回放和发音分析），默认开还是关？
- [ ] 产品是否收费，以及收费模式。

---

## 13. 附录：技术选型与 SpeakHub 复用

### 13.1 技术栈

| 层 | 选型 |
| --- | --- |
| 桌面框架 | Electron + electron-vite |
| 界面 | Vue 3（组合式 API）+ Vue Router + Pinia + TypeScript |
| 主进程 | Node.js 22 LTS + TypeScript，按 ipc / service / repository / provider 分层 |
| 数据库 | SQLite（better-sqlite3）+ 版本化迁移脚本 |
| 校验 | zod |
| 测试 | Vitest + @vue/test-utils |
| 打包 | electron-builder，dmg + zip，arm64 与 x64，签名与公证 |

### 13.2 选型理由

- **复用 SpeakHub 主进程**：流式对话、分句朗读、打断、复盘重试、本地语音检测等难点都在 SpeakHub 的 TypeScript 主进程中，Electron 方案可直接迁移。
- **界面用 Vue**：开发者熟悉 Java + Vue；界面需要按四大场景重新设计，改用 Vue 只损失少量 SpeakHub 的 React 界面代码。
- **未选方案**：JavaFX（悬浮字幕窗、流式音频和 Mac 公证支持弱）；Spring Boot 本地后端 + Vue（需内置 JRE、两个运行时，单机应用过重）；Swift 原生（体验最好，但需从零学习新语言且无法复用 SpeakHub 代码）。

### 13.3 与 SpeakHub 的复用关系

| SpeakHub 能力 | 在 SpeakLoop 中的处理 | 说明 |
| --- | --- | --- |
| API 直连练习链路（LLM 流式、分句、TTS 预取、打断） | 复用并抽象为 Provider 接口 | 核心资产，Electron 方案可直接迁移 |
| 本地 VAD（Silero，sherpa-onnx） | 复用 | sherpa-onnx 有 macOS 构建，需验证 arm64 打包 |
| 阿里 Paraformer ASR | 作为可选 ASR | 默认改用 macOS 系统语音识别 |
| MiMo / Kokoro TTS | 作为可选 TTS | 默认改用 macOS 系统语音兜底 |
| ChatGPT / Gemini 网页自动化 | 不迁移 | 依赖第三方页面结构，维护成本高；由实时语音模型 API 替代 |
| 悬浮字幕、点词查询、ECDICT 词典 | 复用并适配 macOS 窗口层级 | 需支持全屏应用和多桌面空间 |
| 复盘（长对话分段、504 重试、薄弱点 ≤ 6 条） | 复用并扩展场景评估器 | 保留“复盘失败不丢对话”的规则 |
| Markdown 归档 + `learning-index.json` | 改为 SQLite + Markdown 归档 | 支持更复杂的统计和薄弱点追踪 |
| 词汇四档复习 | 复用，扩展到句子 | 统一 `ReviewSchedule` |
| GitHub Release 自动更新、NSIS 安装 | 改为 DMG + 签名公证 + 自动更新 | macOS 发布流程需重做 |
| Windows 安全存储 | 改为 macOS 钥匙串 | – |
