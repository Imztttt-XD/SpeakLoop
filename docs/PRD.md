# SpeakLoop（说练环）产品需求文档

| 项目 | 内容 |
| --- | --- |
| 产品名称 | SpeakLoop（说练环） |
| 产品形态 | 网页应用（B/S 架构），自部署 |
| 使用范围 | 个人或小范围（≤ 50 个账号）使用，不开放公众注册 |
| 支持浏览器 | Chrome / Edge 最新两个大版本（主要），Safari 17+（基本可用） |
| 技术栈 | 后端 Java 21 + Spring Boot 3，前端 Vue 3，数据库 MySQL 8 |
| 文档版本 | v2.0 草稿 |
| 日期 | 2026-11-10 |
| 文档状态 | 待评审 |
| 参考产品 | [SpeakHub](https://github.com/yin-yizhen/SpeakHub) v0.2.5（Windows 桌面端） |

## 修订记录

| 版本 | 日期 | 修订人 | 说明 |
| --- | --- | --- | --- |
| v2.0 | 2026-11-10 | 待填 | 产品形态由 macOS 桌面端改为自部署网页应用；技术栈改为 Spring Boot + Vue 3；新增账号、额度与管理后台；AI Key 支持平台提供与用户自带两种方式 |
| v1.0 | 2026-11-10 | 待填 | 初稿（macOS 桌面端，Electron 方案） |

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
13. 附录：技术选型与 SpeakHub 借鉴

---

## 1. 背景与产品定位

### 1.1 背景

SpeakHub 已在 Windows 桌面端验证了一条完整的口语学习链路：开口对话、实时字幕、查词收藏、LLM 复盘、词汇和句子复习，再把薄弱点带入下一次练习。它也暴露了几个问题：

- 练习内容按“场景提示词”组织，但不同目标（考试、工作、娱乐）的练习方式和评估标准差别很大，一套通用流程难以兼顾。
- 每台电脑都要单独安装、配置 Key、下载语音模型，换设备后学习记录不能延续。
- 仅支持 Windows。

### 1.2 产品定位

SpeakLoop 是一款可自部署的 AI 英语口语陪练网页应用。部署者（管理员）在自己的服务器或电脑上运行一套服务，自己和家人、同学、同事用浏览器登录即可练习，Mac、Windows 和平板都能用，学习记录保存在服务器上。

产品以**场景**为一级入口，为日常生活、雅思口语、商务工作、英文歌曲四类目标分别设计练习形式、AI 角色和评估维度，并共用一套“练习 → 复盘 → 复习 → 再练习”的学习闭环。

### 1.3 一句话价值

打开浏览器就能用英语聊起来，每次练习后知道自己错在哪、下次该练什么。

### 1.4 范围

| 本期包含 | 本期不包含 |
| --- | --- |
| 网页应用（桌面浏览器优先，平板可用） | 原生桌面客户端、手机 App、小程序 |
| 管理员创建账号或发放邀请码，账号密码登录 | 开放注册、第三方登录、找回密码邮件/短信 |
| 四大场景练习与通用自由对话 | 真人外教、社区与排行榜 |
| 平台统一提供 AI Key（带额度）+ 用户自带 Key | 在线支付与订阅 |
| Docker Compose 一键部署 | 多机集群、高可用部署 |
| 歌曲场景使用用户自己上传的音频和歌词，仅上传者本人可见 | 内置或共享受版权保护的歌曲和歌词 |

---

## 2. 产品目标

### 2.1 业务目标

| 编号 | 目标 | 衡量方式 |
| --- | --- | --- |
| G1 | 降低开口门槛：新用户登录后 3 分钟内开始第一次语音对话 | 首次登录到首次对话开始的时长中位数 |
| G2 | 形成学习习惯：用户每周至少练习 3 天 | 活跃用户中每周练习 ≥ 3 天的占比 |
| G3 | 让进步可见：用户能看到各场景的评分趋势 | 查看学习概览的用户占比 |
| G4 | 稳定可用：练习过程少中断，复盘可靠生成 | 练习异常中断率、复盘生成成功率 |
| G5 | 成本可控：平台 AI 费用不超出管理员设定的预算 | 每月平台 Key 实际用量与额度之比 |

### 2.2 用户目标

- 有一个随时可用、不怕犯错的英语对话对象，任何设备打开浏览器都能练。
- 针对自己的目标（日常交流、雅思、工作、唱歌）练习，而不是泛泛闲聊。
- 每次练完拿到具体纠错和下一步建议。
- 把学到的单词、句子沉淀下来并按时复习。

### 2.3 成功指标（上线 3 个月）

| 指标 | 目标值 | 说明 |
| --- | --- | --- |
| 首次对话开始耗时中位数 | ≤ 3 分钟 | 使用平台 Key，从首次登录算起 |
| 次周留存 | ≥ 50% | 首周练习过的用户第二周仍练习（小范围使用，基数小，目标偏高） |
| 周练习 ≥ 3 天占比 | ≥ 30% | 活跃用户口径 |
| 复盘生成成功率 | ≥ 98% | 失败也不得丢失对话 |
| 语音回合延迟 P50 | ≤ 2 秒 | 用户说完到 AI 开始播放（含网络往返） |
| 平台额度超支次数 | 0 | 超出额度时应拦截，而不是继续计费 |

> 目标值为初稿建议。所有指标从服务器自身数据统计，不接入第三方统计服务。

---

## 3. 目标用户与用户画像

目标用户：有英语口语提升需求的中文母语者，以及负责部署和管理这套服务的人。

| 画像 | 典型身份 | 英语水平 | 核心诉求 | 主场景 |
| --- | --- | --- | --- | --- |
| 老张·管理员 | 会一点 Java 的开发者，在家里 NAS 或云服务器上部署，给自己和朋友用 | 不限 | 部署简单、能控制 AI 费用、账号好管理 | 后台管理 |
| 小林·日常交流者 | 准备出国旅行或刚移居海外的上班族 | A2–B1 | 能应付点餐、问路、看病、闲聊，不再紧张 | 日常生活 |
| 小周·雅思考生 | 大三学生，3 个月后考雅思，口语目标 6.5 | B1–B2 | 熟悉题型、练答题结构、知道自己大概几分 | 英语考试 |
| 陈经理·职场人 | 外企产品经理，要参加英文周会与客户电话 | B2 | 开会能说清楚、表达更专业得体 | 商务工作 |
| 阿宁·兴趣学习者 | 喜欢欧美流行音乐的大学生 | A2–B2 | 唱准歌词、学地道表达和连读 | 英文歌曲 |

---

## 4. 用户故事

格式：作为【角色】，我想要【功能】，以便【价值】。优先级 P0 为 MVP 必须，P1 为首个迭代，P2 为后续候选。

### 4.1 账号、部署与额度

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-01 | 作为管理员，我想要用一条命令部署整套服务，以便不用逐个安装组件 | P0 | `docker compose up -d` 启动后端、前端（Nginx）和 MySQL；首次启动引导创建管理员账号 |
| US-02 | 作为管理员，我想要创建账号或生成邀请码，以便只让我认识的人使用 | P0 | 支持直接创建账号、生成一次性邀请码（可设有效期）；可禁用账号、重置密码 |
| US-03 | 作为管理员，我想要配置平台统一的 AI Key，以便用户登录后免配置直接练习 | P0 | 后台可分别配置 LLM、ASR、TTS、发音评测的 Key 并检测连通性；Key 加密存储，页面不回显明文 |
| US-04 | 作为管理员，我想要给每个用户设置平台 Key 的使用额度，以便控制费用 | P0 | 按“每日练习分钟数 + 每月 LLM token 数”设置默认额度和单人额度；超出后提示用户，不再使用平台 Key |
| US-05 | 作为管理员，我想要查看各用户的用量，以便了解费用去向 | P1 | 按用户、日期、服务类型统计用量和估算费用 |
| US-06 | 作为用户，我想要填写自己的 AI Key，以便在平台额度用完后继续练习，或使用我偏好的模型 | P0 | 个人设置可分别配置各类服务的 Key；可选择优先使用自己的 Key 或平台 Key；使用自己的 Key 不占平台额度 |
| US-07 | 作为用户，我想要看到今天还剩多少平台额度，以便合理安排练习 | P0 | 首页和练习页显示剩余分钟数；剩余 ≤ 10% 时提醒 |

### 4.2 通用与上手

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-10 | 作为新用户，我想要一个引导流程帮我完成学习目标设置、水平自测和麦克风授权，以便尽快开始第一次对话 | P0 | 引导可跳过；麦克风授权失败时说明浏览器设置方法；完成后直接进入推荐场景 |
| US-11 | 作为用户，我想要选择语音或文字方式练习，以便在不方便说话时也能练 | P0 | 两种模式共用场景、复盘和存档 |
| US-12 | 作为用户，我想要在 AI 说话时直接打断它，以便对话更自然 | P0 | 浏览器检测到用户开口后 500 ms 内停止 AI 播放 |
| US-13 | 作为用户，我想要在练习页看到双方的实时字幕，以便边听边看、不漏信息 | P0 | 字幕随识别与回复逐字更新；可调字号、只看一方 |
| US-14 | 作为用户，我想要把字幕弹出成一个置顶小窗，以便边看其他材料边练 | P2 | 支持 Document Picture-in-Picture 的浏览器（Chrome/Edge）可弹出字幕小窗；其他浏览器隐藏该按钮 |
| US-15 | 作为用户，我想要点击字幕中的单词查看释义并收藏，以便积累生词 | P0 | 显示音标、中文释义、语境含义；收藏后进入词汇本 |
| US-16 | 作为用户，我想要收藏整句话，以便之后复习好的表达 | P1 | 只有已完成的字幕可收藏；重复收藏不重复入库 |
| US-17 | 作为用户，我想要用键盘快捷键开关麦克风，以便不用找按钮 | P1 | 练习页内按空格键（输入框未聚焦时）切换麦克风，可在设置中修改 |

### 4.3 日常生活场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-20 | 作为日常交流者，我想要从“点餐、问路、酒店入住、看病”等情景卡片中选择练习，以便有针对性地准备 | P0 | 首发 ≥ 20 个情景，每个情景有目标、AI 角色和 3 个任务点 |
| US-21 | 作为日常交流者，我想要 AI 扮演店员、医生等角色并主动推进对话，以便练习真实交流 | P0 | AI 保持角色，不主动切换到教学口吻，除非用户请求帮助 |
| US-22 | 作为日常交流者，我想要卡住时点“提示”获得可以说的句子，以便对话不中断 | P0 | 提供 2–3 个难度递进的参考说法，可点读 |
| US-23 | 作为日常交流者，我想要自由闲聊模式，以便练习无固定主题的对话 | P1 | AI 根据用户兴趣标签选话题 |

### 4.4 英语考试（雅思）场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-30 | 作为雅思考生，我想要完整模拟 Part 1/2/3 口语考试，以便熟悉流程和节奏 | P0 | 考官角色按题型推进；Part 2 有 1 分钟准备、2 分钟陈述计时，可记笔记 |
| US-31 | 作为雅思考生，我想要单独练习某一个 Part 或某个话题，以便集中突破 | P0 | 可按 Part、话题筛选题目；管理员可在后台维护题库 |
| US-32 | 作为雅思考生，我想要考后获得按官方四项标准的预估分数，以便了解差距 | P0 | 输出流利度与连贯性、词汇丰富度、语法多样性与准确性、发音四项分和总分，并附“非官方评分”声明 |
| US-33 | 作为雅思考生，我想要看到参考高分答案和我回答的对比，以便学习结构和表达 | P1 | 按题目展示我的回答、改进版回答和改动说明 |
| US-34 | 作为雅思考生，我想要看到我的分数随时间变化，以便判断是否可以报名考试 | P1 | 学习中心展示四项分趋势 |

### 4.5 商务工作场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-40 | 作为职场人，我想要练习会议发言、项目汇报、客户电话、谈判、英文面试等场景，以便应对真实工作 | P0 | 首发 ≥ 12 个商务情景 |
| US-41 | 作为职场人，我想要填写自己的行业、职位和真实议题，以便 AI 模拟我实际会遇到的对话 | P0 | 背景信息保存为“我的工作档案”，开始练习时自动带入 |
| US-42 | 作为职场人，我想要得到语气正式度和礼貌程度的反馈，以便说话更得体 | P0 | 复盘中单独列出“语气与得体性”问题，并给出更专业的说法 |
| US-43 | 作为职场人，我想要上传会议议程或演讲稿，让 AI 基于它提问，以便会前演练 | P1 | 支持 .txt / .md / .pdf / .docx，≤ 20 页、≤ 10 MB；文件仅本人可见 |

### 4.6 英文歌曲场景

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-50 | 作为兴趣学习者，我想要上传自己的歌曲音频和歌词，以便用喜欢的歌学英语 | P0 | 支持 mp3/m4a/wav（≤ 20 MB）和 .lrc/.txt 歌词；无时间轴时可自动或手动对齐；歌曲仅上传者本人可见 |
| US-51 | 作为兴趣学习者，我想要逐句跟唱或跟读，以便练习发音和连读 | P0 | 单句循环、0.5–1.0 倍速、录音回放对比 |
| US-52 | 作为兴趣学习者，我想要每句的发音评分和问题单词标注，以便知道哪里唱得不准 | P1 | 显示句子得分和低分单词，点击可听标准读音 |
| US-53 | 作为兴趣学习者，我想要 AI 讲解歌词中的俚语、连读和文化背景，以便理解歌曲含义 | P0 | 按句展示讲解，可收藏到词汇和句子本 |
| US-54 | 作为兴趣学习者，我想要和 AI 聊这首歌的主题，以便把歌词表达用到对话里 | P1 | 从歌曲页一键进入“歌曲话题对话” |

### 4.7 复盘与学习

| 编号 | 用户故事 | 优先级 | 验收标准 |
| --- | --- | --- | --- |
| US-60 | 作为用户，我想要每次练习后自动生成复盘，以便知道自己哪里说错了 | P0 | 包含总结、逐句纠错、按场景的评分、薄弱点、推荐下一次练习 |
| US-61 | 作为用户，复盘失败时我不想丢失对话，并能重新生成 | P0 | 对话先入库；历史详情提供“重新生成复盘” |
| US-62 | 作为用户，我想要关掉浏览器或断网后，回来还能找到这次练习 | P0 | 每条完整字幕即时入库；30 分钟无活动的练习自动结束并归档，用户可补生成复盘 |
| US-63 | 作为用户，我想要按间隔重复复习单词和句子，以便记得牢 | P0 | 四档评分（重来/困难/一般/简单）调整下次复习时间；复习过程不调用 LLM |
| US-64 | 作为用户，我想要看到反复出现的薄弱点以及它们是否在改善，以便有针对性地练 | P1 | 同类问题合并；显示出现次数、最近出现日期、状态 |
| US-65 | 作为用户，我想要一键带着上次的薄弱点开始下一次练习，以便持续改进 | P0 | 新练习的系统提示词包含薄弱点，复盘中单独评价这些点的表现 |
| US-66 | 作为用户，我想要设置每日练习目标，以便坚持下去 | P1 | 目标分钟数可设；首页显示进度和连续天数；可选浏览器通知提醒 |
| US-67 | 作为用户，我想要导出我的学习数据，以便备份或导入 Anki | P2 | 词汇和句子可导出 CSV；全部练习记录可导出 zip |

---

## 5. 功能模块拆解

### 5.1 模块总览

```text
SpeakLoop
├── M1 账号与权限
│   ├── 登录 / 退出 / 修改密码
│   ├── 邀请码注册
│   └── 角色：管理员 ADMIN、普通用户 USER
├── M2 引导与个人档案
│   ├── 首次引导
│   ├── 水平自测（CEFR 估计）
│   └── 学习目标、兴趣、工作档案、雅思目标
├── M3 场景中心
│   ├── M3.1 日常生活
│   ├── M3.2 英语考试（雅思）
│   ├── M3.3 商务工作
│   ├── M3.4 英文歌曲
│   └── M3.5 自由对话 / 自定义场景
├── M4 练习引擎（场景共用）
│   ├── 会话编排（提示词组装、角色、回合管理）
│   ├── 实时语音通道（浏览器 ↔ 服务器 WebSocket）
│   ├── 语音链路（浏览器 VAD、服务器 ASR / LLM / TTS、打断）
│   ├── 文字链路
│   ├── 卡壳提示
│   └── 断线恢复与超时归档
├── M5 字幕与查词
├── M6 复盘与评估
├── M7 学习中心
├── M8 AI 服务与额度
│   ├── Provider 适配层（LLM / ASR / TTS / 发音评测）
│   ├── 密钥管理（平台 Key、用户 Key，加密存储）
│   ├── Key 路由（决定本次调用用哪把 Key）
│   └── 用量记录与额度校验
├── M9 个人设置
└── M10 管理后台
    ├── 用户与邀请码管理
    ├── 平台 AI 服务配置
    ├── 额度策略
    ├── 用量统计
    ├── 内容管理（内置场景、雅思题库）
    └── 系统信息与诊断日志
```

### 5.2 M1 账号与权限

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 初始化管理员 | 首次访问时数据库无用户，跳转到“创建管理员”页面；创建后该入口关闭 | P0 |
| 登录 | 用户名 + 密码；连续失败 5 次锁定 15 分钟 | P0 |
| 邀请码注册 | 管理员生成邀请码（可设有效期、可用次数）；用户凭邀请码设置用户名和密码 | P0 |
| 账号管理 | 管理员创建、禁用、重置密码、修改角色 | P0 |
| 会话 | 登录后签发 JWT（访问令牌 2 小时 + 刷新令牌 7 天，刷新令牌放 HttpOnly Cookie） | P0 |
| 数据隔离 | 所有业务数据带 `user_id`，接口只能访问本人数据；管理员只能看用量统计，不能查看他人练习内容 | P0 |

### 5.3 M2 引导与个人档案

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 引导 | 欢迎 → 选择学习目标（可多选四大场景）→ 麦克风授权与测试 → 水平自测（可跳过）→ 推荐首个练习 | P0 |
| 水平自测 | AI 进行 3 分钟问答，给出 CEFR 估计（A1–C1），用户可手动修改 | P1 |
| 个人档案 | 每日目标分钟数、口音偏好（美式/英式）、兴趣标签、工作档案（商务）、雅思目标分与考试日期 | P0 |

### 5.4 M3 场景中心

所有场景共用一个**场景定义模型**（见 8.3 `t_scenario`），差异体现在 AI 角色、练习结构、计时规则和评估器上。内置场景由管理员在后台维护，用户可创建仅自己可见的自定义场景。

| 维度 | 日常生活 | 英语考试（雅思） | 商务工作 | 英文歌曲 |
| --- | --- | --- | --- | --- |
| 练习形式 | 角色扮演、任务驱动对话 | 结构化模拟考试 | 角色扮演、材料驱动演练 | 跟读跟唱 + 歌词讲解 + 话题对话 |
| AI 角色 | 店员、医生、房东、新朋友等 | 考官 | 同事、经理、客户、面试官 | 歌词讲解老师、发音教练 |
| 结构 | 开场 → 3 个任务点 → 自然结束 | Part 1（4–5 分钟）→ Part 2（准备 1 分钟 + 陈述 2 分钟 + 追问）→ Part 3（4–5 分钟） | 设定背景 → 主题讨论 → 追问与施压 → 总结 | 逐句练习，按句推进 |
| 计时 | 无 | 严格计时并显示倒计时 | 可选（如“3 分钟汇报”） | 跟随音频时间轴 |
| 评估重点 | 任务完成度、可理解度、常用表达 | 四项官方维度及预估分数 | 清晰度、专业度、得体性、逻辑结构 | 发音准确度、连读、节奏 |
| 卡壳提示 | 支持 | 模拟考试关闭，单项练习开启 | 支持 | 不适用 |

#### M3.1 日常生活

- **情景库**：按“出行、餐饮、购物、住宿、医疗、社交、办事”7 个分类组织，首发 ≥ 20 个情景。
- **情景卡片**：标题、难度、预计时长、AI 角色、任务点、关键表达预习（3–5 个）。
- **任务点**：例如“点一份不含坚果的主菜”“询问能否分开付款”。完成情况在对话中实时打勾，复盘中统计完成度。
- **自由闲聊**：根据用户兴趣标签生成话题，AI 以朋友口吻对话。

#### M3.2 英语考试（雅思）

- **模式**：完整模拟（Part 1–3 连续）、单 Part 练习、话题专项。
- **题库**：Part 1 话题、Part 2 话题卡（Cue Card）及 Part 3 追问，存在数据库中，管理员可在后台增删改、批量导入 JSON。内置题目为原创或改编，不收录真题原文（见风险 R3）。
- **Part 2 流程**：展示话题卡 → 1 分钟准备（可打字记笔记）→ 2 分钟陈述（计时，到点由考官打断）→ 1–2 个追问。
- **评分**：按流利度与连贯性（FC）、词汇丰富度（LR）、语法多样性与准确性（GRA）、发音（P）四项给出 0–9 分（0.5 分一档）及总分，并给出每项依据。发音分依赖发音评测服务，未配置时标注“未评估”。
- **答案对比**：每道题展示“我的回答（转写）/ 改进版回答 / 修改要点”。
- **考试倒计时**：可设置考试日期，首页显示剩余天数和推荐练习强度。

#### M3.3 商务工作

- **情景库**：会议发言、项目进度汇报、客户电话、价格谈判、英文面试、闲聊破冰（Small Talk）、处理投诉、产品演示等，首发 ≥ 12 个。
- **工作档案**：行业、职位、公司类型、常用议题，开始练习时自动注入提示词。
- **材料驱动演练**：上传议程、演讲稿、简历或职位描述，后端提取文本后交给 AI，由 AI 扮演对应角色提问。
- **得体性反馈**：复盘中单独列出过于口语、过于直接或不礼貌的表达，给出商务场合说法。
- **表达模板**：按情景沉淀常用句型（如表达异议、争取时间、总结行动项），可收藏复习。

#### M3.4 英文歌曲

- **版权原则**：系统不内置、不抓取歌曲和歌词。用户上传的音频和歌词只对上传者本人可见，不提供分享或公开链接；送去 LLM 讲解时按句发送。
- **歌曲上传**：上传音频文件和歌词文件；没有 .lrc 时间轴的，提供“自动对齐”（ASR 时间戳）和“手动打点”（边播放边按键标记）两种方式。
- **歌词学习**：逐句展示原文、中文释义、俚语、连读弱读标注、文化背景。
- **跟唱跟读**：浏览器内单句循环、A-B 复读、0.5–1.0 倍速播放；录下用户的跟读并回放对比；配置了发音评测服务时给出句子得分和低分单词。
- **歌曲话题对话**：从歌曲页进入，AI 围绕歌曲主题和其中的表达与用户聊天。

#### M3.5 自由对话与自定义场景

- 用户可新建场景：名称、AI 角色设定、目标、难度、纠错强度，复用通用评估器。
- 自定义场景默认仅本人可见；管理员可将其“提升为内置场景”供所有人使用。

### 5.5 M4 练习引擎

#### 5.5.1 语音链路架构

```text
浏览器                                         服务器（Spring Boot）
┌──────────────────────────┐   WebSocket     ┌──────────────────────────────────────┐
│ 麦克风 getUserMedia       │  /ws/practice   │ PracticeSessionHandler               │
│ AudioWorklet 转 16kHz PCM │ ──音频二进制帧──▶ │  → AsrProvider（流式识别）            │
│ VAD（检测开口/说完）       │ ──控制消息──────▶ │  → 字幕入库 → 推送 transcript 消息     │
│                          │                 │  → LlmProvider（流式回复）            │
│ 字幕渲染                  │ ◀──transcript── │  → 分句器 → TtsProvider（流式合成）    │
│ 音频播放队列 Web Audio     │ ◀──音频二进制帧── │  → 用量记录（UsageRecorder）          │
│ 检测到开口 → 停止播放      │ ──interrupt───▶ │  → 中止 LLM 与 TTS 请求               │
└──────────────────────────┘                 └──────────────────────────────────────┘
```

- **为什么经过服务器**：AI Key 只在服务器上，浏览器拿不到明文；服务器统一记录用量、校验额度、保存字幕。
- **VAD 放在浏览器**：开口检测和打断要求低延迟，在浏览器本地完成（如基于 onnxruntime-web 的 Silero VAD），只把有语音的片段送到服务器，节省 ASR 费用。

#### 5.5.2 子模块

| 子模块 | 功能说明 | 借鉴 SpeakHub |
| --- | --- | --- |
| 会话编排 | 组装系统提示词：基础规则（英语优先、处理残缺输入）+ 场景角色 + 难度 + 纠错强度 + 本次重点 + 薄弱点；维护 `system → user/assistant` 历史顺序 | `direct-chat-prompt.ts`、`learning-service.ts` 的规则 |
| 语音输入 | 浏览器采集 → VAD 断句 → WebSocket 发送 → 服务器流式 ASR | `local-speech-audio.ts` 的采集与重采样思路 |
| 语音输出 | LLM 流式文本 → 分句（首句短、后续合并）→ 流式 TTS → 推送音频 → 浏览器连续排队播放，最多预取 2 句 | `speech-segments.ts`、`speech-synthesis-queue.ts` 的分句和预取规则 |
| 打断 | 用户开口或提交新文字时，服务器中止当前 LLM 与 TTS 请求，浏览器清空播放队列，未说完的回复标记为“被打断” | `barge-in-policy.ts`、`interruptible-task-handoff.ts` 的顺序规则 |
| 文字链路 | 文字输入走同一 WebSocket；可选“朗读 AI 回复” | SpeakHub 文字模式 |
| 卡壳提示 | 用户点“提示”或沉默超过 8 秒（可配置）时，生成 2–3 条参考说法 | 新增 |
| 断线恢复 | WebSocket 断开后浏览器自动重连（指数退避，最多 5 次），重连后继续同一会话；会话状态以数据库为准 | 新增 |
| 超时归档 | 定时任务每 5 分钟扫描，30 分钟无活动的练习自动结束并归档，复盘状态为“未生成” | `session-checkpoint.ts` 的落盘思路 |

### 5.6 M5 字幕与查词

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 练习页字幕 | 双方字幕逐字更新；可只看一方；字号与配色可调 | P0 |
| 字幕小窗 | 支持 Document Picture-in-Picture 的浏览器可弹出置顶字幕小窗 | P2 |
| 点词查询 | 服务器内置 ECDICT 词典即时返回音标和释义；可选调用 LLM 补充语境含义和更自然的说法 | P0 |
| 收藏 | 单词进入词汇本；整句进入句子本，练习结束时随复盘一起生成句子讲解 | P0 / P1 |

### 5.7 M6 复盘与评估

- **通用复盘**：总结、逐句纠错（原句 / 改写 / 原因 / 错误类型）、推荐词汇、薄弱点（目标最多 6 条，合并相近问题）、下一次练习建议。
- **场景评估器**：在通用复盘之上输出场景专属结果。

| 评估器 | 输出 |
| --- | --- |
| 日常 | 任务点完成情况、可理解度 1–5 分、常用表达推荐 |
| 雅思 | FC / LR / GRA / P 四项分数与总分、每项依据、逐题改进答案 |
| 商务 | 清晰度、专业度、得体性、结构四项 1–5 分，语气问题清单、表达模板 |
| 歌曲 | 逐句发音得分、问题单词、连读建议（依赖发音评测服务） |

- **客观指标**（服务器计算，不调用 LLM）：语速（词/分钟，仅语音）、平均回合词数、填充词次数（um、uh、like 等）、中文混用比例。
- **异步执行**：结束练习后复盘放入异步任务执行，页面通过轮询或 WebSocket 获取进度，用户可以先离开页面。
- **长对话处理**：先整份发送并流式接收结构化结果；只有超时或网关报 504 时才按完整回合切分重试，本地合并纠错，不设条数上限。
- **结果校验**：LLM 返回的 JSON 按 schema 校验；薄弱点超过 6 条时保留全部有效条目，不因超量判定失败。
- **失败处理**：复盘失败时对话照常归档并标记“复盘失败”，可在历史详情中重新生成。

### 5.8 M7 学习中心

| 子模块 | 功能 | 优先级 |
| --- | --- | --- |
| 概览 | 本周/本月练习时长、练习天数、连续天数、各场景评分趋势、CEFR 趋势、常见错误类型 | P0 |
| 历史记录 | 按场景、日期、模式、关键词筛选；详情页显示完整对话、复盘、收藏 | P0 |
| 词汇本 | 列表、熟悉度分组、到期复习；复习卡片四档评分 | P0 |
| 句子本 | 句子讲解（翻译、结构、可复用句型、表达、例句）、间隔重复复习 | P1 |
| 薄弱点追踪 | 同类薄弱点合并，显示出现次数、最近出现、状态（待改善 / 改善中 / 已克服），可一键专项练习 | P1 |
| 雅思成绩单 | 历次模拟考四项分与总分趋势 | P1 |

**间隔重复规则（初版）**：采用简化 SM-2。初始间隔 1 天；“重来”在本轮复习末尾再次出现，次日重新开始；“困难”间隔 ×1.2；“一般”×易记因子（初始 2.5）；“简单”×易记因子×1.3。连续 3 次“一般”及以上且间隔 ≥ 21 天，标记为“已掌握”。

### 5.9 M8 AI 服务与额度

#### 5.9.1 Key 路由规则

每次调用 AI 服务前，由 `KeyRouter` 决定使用哪把 Key：

```text
用户偏好 = 优先使用自己的 Key？
├── 是：用户配置了该类服务的 Key 且可用 → 用用户 Key（不计平台额度）
│       否则 → 走“平台 Key”分支
└── 否：平台 Key 分支
        ├── 平台已配置该类服务 且 用户额度未用完 → 用平台 Key，记录用量
        ├── 额度已用完 且 用户有自己的 Key → 自动改用用户 Key，并提示一次
        └── 都没有 → 返回 QUOTA_EXCEEDED 或 PROVIDER_NOT_CONFIGURED，前端引导填写自己的 Key
```

- 一次练习开始时锁定所用 Key 来源，练习中途额度用完时允许本次练习继续到结束（最多超出 10 分钟），避免对话被突然切断；超出部分照常记账。

#### 5.9.2 额度

| 维度 | 默认值（管理员可改） | 计量方式 |
| --- | --- | --- |
| 每日语音练习分钟数 | 30 分钟 | 按练习会话中用户和 AI 的语音时长累计 |
| 每月 LLM token | 1,000,000 | 按 LLM 接口返回的 usage 累计；无 usage 时按字符数估算 |
| 每月 TTS 字符数 | 200,000 | 送去合成的字符数 |
| 每月发音评测次数 | 500 | 调用次数 |

- 额度按用户单独设置，未设置时使用默认策略。
- 用户自带 Key 的调用同样记录用量（便于用户查看），但不计入平台额度。

#### 5.9.3 Provider 适配

| 能力 | 接入方式 | 备注 |
| --- | --- | --- |
| LLM | OpenAI 兼容 `/v1/chat/completions`（SSE 流式） | DeepSeek、通义千问、Claude、GPT 等均可 |
| ASR | 阿里云 Paraformer 实时识别（WebSocket） | 首选，国内网络稳定；预留其他实现 |
| TTS | 云端流式 TTS（如阿里云 CosyVoice、小米 MiMo） | 预留其他实现 |
| 发音评测 | 待选型 | 未配置时不评估 |

### 5.10 M9 个人设置

| 分组 | 设置项 |
| --- | --- |
| 账号 | 修改密码、退出所有设备 |
| 我的 AI Key | 各类服务的 Base URL、模型、Key；“检测连通性”；“优先使用我的 Key”开关 |
| 额度 | 今日/本月平台额度剩余、我的用量明细 |
| 练习偏好 | 默认难度、纠错强度、AI 语速、口音偏好、AI 音色 |
| 音频 | 输入设备、麦克风检测、VAD 灵敏度、静音提示时长 |
| 字幕 | 字号、配色、默认显示方式 |
| 学习目标 | 每日目标、浏览器通知提醒 |
| 数据 | 导出词汇/句子 CSV、导出全部记录、删除全部学习数据 |

### 5.11 M10 管理后台

| 功能 | 描述 | 优先级 |
| --- | --- | --- |
| 用户管理 | 列表、创建、禁用、重置密码、设置角色和单人额度 | P0 |
| 邀请码 | 生成、设有效期和次数、作废 | P0 |
| 平台 AI 服务 | 配置各类服务的 Key、模型和价格（用于估算费用），检测连通性 | P0 |
| 额度策略 | 默认额度设置 | P0 |
| 用量统计 | 按用户、服务、日期统计用量和估算费用，可导出 CSV | P1 |
| 内容管理 | 内置场景、雅思题库的增删改与 JSON 导入导出 | P1 |
| 系统信息 | 版本、数据库连接状态、磁盘占用、最近错误日志 | P1 |

---

## 6. 核心业务流程

### 6.1 一次练习的主流程

```text
选择场景 → 场景详情（目标、角色、预习表达）→ 确认设置（模式、难度、纠错强度、本次重点、是否带薄弱点）
→ POST /api/practice/sessions 创建会话（服务器确定 Key 来源并校验额度）
→ 浏览器建立 WebSocket /ws/practice?sessionId=…
→ AI 开场
→ 循环：
     浏览器 VAD 检测到开口 → 发送音频帧 → 服务器 ASR 流式识别 → 推送用户字幕
     VAD 检测到说完 → 服务器定稿用户字幕并入库 → LLM 流式回复 → 推送 AI 字幕
     → 分句 TTS → 推送音频 → 浏览器排队播放
     （任意时刻用户开口 → 发送 interrupt → 中止当前回复）
→ 用户点击“结束并复盘” → POST /api/practice/sessions/{id}/end
     → 关闭 WebSocket、停止 ASR / LLM / TTS
     → 归档会话，异步生成复盘（通用 + 场景评估器 + 收藏句子讲解）
     → 更新词汇本 / 句子本 / 薄弱点 / 每日统计
→ 复盘页（生成中显示进度）→ “带薄弱点开始下一次练习”
```

### 6.2 雅思模拟考流程

```text
选择“完整模拟” → 服务器抽题（Part 1 话题 2 个、Part 2 话题卡 1 张、对应 Part 3 话题；排除近 30 天做过的题）
→ Part 1：考官提问 4–6 题（约 4–5 分钟）
→ Part 2：展示话题卡 → 准备 60 秒（可记笔记）→ 陈述 ≤ 120 秒（到时考官打断）→ 追问 1–2 题
→ Part 3：围绕 Part 2 主题深入讨论（约 4–5 分钟）
→ 结束 → 雅思评估器输出四项分数、逐题改进答案 → 记入雅思成绩单
```

计时由服务器控制并推送 `exam_timer` 消息，避免浏览器标签页切到后台时计时不准。

### 6.3 歌曲学习流程

```text
上传音频 + 歌词 → 时间轴检查（有 LRC 直接使用 / 自动对齐 / 手动打点）
→ 歌曲详情：逐句歌词 + 讲解（首次查看时生成并缓存）
→ 选择句子 → 播放原声 → 浏览器录音 → 回放对比 → 上传录音做发音评分（已配置时）
→ 收藏表达 → 进入词汇本 / 句子本
→ 可选：进入“歌曲话题对话”
```

### 6.4 部署与初始化流程

```text
管理员下载项目 → 复制 .env.example 为 .env，填写数据库密码、JWT 密钥、加密主密钥、访问域名
→ docker compose up -d（app + nginx + mysql）
→ 浏览器访问 → 检测到无用户 → 创建管理员账号
→ 后台配置平台 AI Key 并检测连通性 → 设置默认额度
→ 生成邀请码发给朋友
```

> 浏览器只允许 HTTPS 页面或 `localhost` 使用麦克风。局域网或公网访问必须配置 HTTPS 证书，部署文档需给出 Nginx + Let's Encrypt 示例。

---

## 7. 页面结构设计

### 7.1 信息架构

```text
未登录
├── 登录
├── 邀请码注册
└── 初始化管理员（仅系统无用户时）

用户端（左侧导航 + 右侧内容）
├── 首页
├── 场景
│   ├── 场景中心
│   ├── 场景列表
│   ├── 场景详情 / 练习设置
│   └── 练习中
├── 歌曲
│   ├── 歌曲库
│   └── 歌曲详情 / 跟唱
├── 复盘（练习结束后进入，也可从历史打开）
├── 学习
│   ├── 概览
│   ├── 历史记录 → 历史详情
│   ├── 词汇本 → 词汇复习
│   ├── 句子本 → 句子复习
│   ├── 薄弱点
│   └── 雅思成绩单
└── 设置

管理后台（仅 ADMIN，路由前缀 /admin）
├── 用户管理
├── 邀请码
├── AI 服务配置
├── 额度策略
├── 用量统计
├── 内容管理（场景、雅思题库）
└── 系统信息
```

### 7.2 页面清单

| 编号 | 页面 | 路由 | 主要内容 | 关键操作 | 优先级 |
| --- | --- | --- | --- | --- | --- |
| P01 | 登录 | `/login` | 用户名、密码 | 登录、去注册 | P0 |
| P02 | 邀请码注册 | `/register` | 邀请码、用户名、密码 | 注册 | P0 |
| P03 | 初始化管理员 | `/setup` | 管理员用户名、密码 | 创建 | P0 |
| P04 | 引导 | `/onboarding` | 4 步：目标 → 麦克风 → 自测 → 推荐练习 | 下一步、跳过 | P0 |
| P05 | 首页 | `/` | 今日目标进度、连续天数、今日待复习数、平台额度剩余、继续上次练习、推荐练习、雅思考试倒计时 | 一键开始、去复习 | P0 |
| P06 | 场景中心 | `/scenarios` | 四大场景卡片（含进度）、自由对话、我的场景 | 进入场景、新建场景 | P0 |
| P07 | 场景列表 | `/scenarios/:category` | 分类标签、情景卡片 | 筛选、搜索 | P0 |
| P08 | 场景详情 | `/scenarios/detail/:id` | 目标、AI 角色、任务点、预习表达；练习设置 | 开始练习 | P0 |
| P09 | 练习中 | `/practice/:sessionId` | 对话字幕流、AI 状态、任务点或雅思计时器、文字输入框、提示按钮、麦克风开关、额度剩余 | 打断、求助、弹出字幕小窗、结束并复盘 | P0 |
| P10 | 复盘 | `/review/:sessionId` | 总结、场景评分卡、客观指标、逐句纠错、薄弱点、推荐词汇、收藏句子讲解、下一次建议 | 收藏、重新生成、带薄弱点再练 | P0 |
| P11 | 歌曲库 | `/songs` | 已上传歌曲列表 | 上传、删除 | P0 |
| P12 | 歌曲详情 | `/songs/:id` | 逐句歌词、讲解、播放控制、录音对比、发音得分 | 单句循环、变速、录音、收藏、话题对话 | P0 |
| P13 | 学习概览 | `/learn` | 时长与天数、各场景评分趋势、CEFR 趋势、常见错误 | 切换周/月 | P0 |
| P14 | 历史记录 / 详情 | `/learn/history`、`/learn/history/:id` | 列表 + 筛选；详情同复盘页并附完整对话 | 搜索、删除、重新生成复盘 | P0 |
| P15 | 词汇本 / 复习 | `/learn/vocabulary`、`/learn/vocabulary/review` | 词汇列表；复习卡片 | 评分、编辑、导出 | P0 |
| P16 | 句子本 / 复习 | `/learn/sentences`、`/learn/sentences/review` | 句子与讲解；复习卡片 | 评分 | P1 |
| P17 | 薄弱点 | `/learn/weak-points` | 列表、次数、状态 | 专项练习 | P1 |
| P18 | 雅思成绩单 | `/learn/ielts` | 历次四项分与总分 | 查看对应复盘 | P1 |
| P19 | 设置 | `/settings` | 见 5.10 | 保存、检测、导出 | P0 |
| A01 | 用户管理 | `/admin/users` | 用户列表、状态、额度、最近练习时间 | 创建、禁用、重置密码、设额度 | P0 |
| A02 | 邀请码 | `/admin/invites` | 邀请码列表 | 生成、作废、复制 | P0 |
| A03 | AI 服务配置 | `/admin/providers` | 各类服务的 Key、模型、单价 | 保存、检测 | P0 |
| A04 | 额度策略 | `/admin/quota` | 默认额度 | 保存 | P0 |
| A05 | 用量统计 | `/admin/usage` | 按用户/服务/日期的用量和费用 | 筛选、导出 | P1 |
| A06 | 内容管理 | `/admin/content` | 场景、雅思题库 | 增删改、导入导出 | P1 |
| A07 | 系统信息 | `/admin/system` | 版本、数据库状态、磁盘、错误日志 | 下载日志 | P1 |

### 7.3 关键页面线框说明

**P09 练习中**

```text
┌──────────────────────────────────────────────────────────────────┐
│ ← 日常生活 · 餐厅点餐   B1 · 标准纠错   ⏱ 06:32   今日剩余 18 分钟  [结束并复盘] │
├──────────────────────────────────────────────────────────────────┤
│ 任务点：☑ 告诉服务员人数  ☐ 点一份不含坚果的主菜  ☐ 分开付款            │
├──────────────────────────────────────────────────────────────────┤
│  AI  Good evening! Table for how many?                            │
│                                 I'd like a table for two, please.  我 │
│  AI  Sure, right this way. ...（说话中）                            │
├──────────────────────────────────────────────────────────────────┤
│ ● 聆听中  [麦克风 开/关 · 空格]  [提示]  [弹出字幕]  [输入文字…]  [发送] │
└──────────────────────────────────────────────────────────────────┘
```

雅思模式下，任务点栏替换为“Part 进度 + 倒计时 + 笔记区（仅 Part 2 准备阶段）”，提示按钮隐藏。WebSocket 断线时顶部显示“连接中断，正在重连（2/5）”。

**P10 复盘页**

```text
┌ 总结 ──────────────────────────────────────────────┐
│ 一段话总结 + 场景评分卡（如雅思：FC 6.0 LR 6.5 GRA 5.5 P 6.0 → 6.0） │
├ 客观指标 ──────────────────────────────────────────┤
│ 语速 118 词/分 · 平均回合 14 词 · 填充词 9 次 · 中文混用 3%          │
├ 逐句纠错（默认显示 5 条，可展开全部）─────────────────────┤
├ 薄弱点 · 推荐词汇 · 收藏句子讲解 ────────────────────────┤
└ [带薄弱点开始下一次]  [重新生成复盘]                         ┘
```

---

## 8. 数据结构设计（初步）

### 8.1 存储方案

| 数据 | 存储方式 | 说明 |
| --- | --- | --- |
| 业务数据 | MySQL 8.0（utf8mb4） | 账号、会话、回合、复盘、词汇、句子、薄弱点、统计、用量 |
| 上传文件 | 服务器本地目录（Docker 卷 `/data/files`），按 `userId/类型/日期` 分目录 | 歌曲音频、跟读录音、商务材料；数据库存相对路径 |
| AI Key | MySQL，AES-256-GCM 加密 | 加密主密钥来自环境变量 `SPEAKLOOP_MASTER_KEY`，不入库 |
| 词典 | ECDICT 导入 MySQL 表 `t_dict_entry` | 首次启动时自动导入 |
| 缓存 | 本期不引入 Redis | 小范围使用，额度统计和限流用 MySQL + 进程内缓存（Caffeine）足够；后续用户增多再引入 |

通用约定：
- 主键使用雪花算法 `BIGINT`（MyBatis-Plus `ASSIGN_ID`），前端以字符串接收，避免 JS 精度丢失。
- 每张业务表包含 `create_time`、`update_time`；需要软删除的表包含 `deleted TINYINT`。
- 所有用户数据表带 `user_id` 并建立索引，数据访问层统一追加 `user_id` 条件。
- JSON 字段使用 MySQL `JSON` 类型，Java 侧用 `JacksonTypeHandler` 映射。
- 表结构变更使用 Flyway 迁移脚本管理。

### 8.2 实体关系

```text
t_user 1 ── 1 t_user_profile
t_user 1 ── * t_api_credential（scope=USER）     平台 Key：t_api_credential（scope=PLATFORM）
t_user 1 ── 0..1 t_quota_policy（单人额度）       默认额度：t_quota_policy（user_id 为空）
t_user 1 ── * t_usage_record
t_user 1 ── * t_practice_session * ── 1 t_scenario
t_practice_session 1 ── * t_turn
t_practice_session 1 ── 0..1 t_review 1 ── * t_review_issue
t_user 1 ── * t_weak_point 1 ── * t_weak_point_occurrence * ── 1 t_practice_session
t_user 1 ── * t_vocabulary / t_sentence 1 ── 1 t_srs_schedule
t_user 1 ── * t_song 1 ── * t_lyric_line 1 ── * t_shadowing_attempt
t_user 1 ── * t_daily_stat
```

### 8.3 建表清单

#### 账号与权限

**t_user 用户表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | 雪花 ID |
| username | VARCHAR(32) | 唯一 |
| password_hash | VARCHAR(100) | BCrypt |
| role | VARCHAR(16) | ADMIN / USER |
| status | TINYINT | 1 正常，0 禁用 |
| failed_login_count | INT | 连续失败次数 |
| locked_until | DATETIME | 锁定截止时间 |
| last_login_time | DATETIME | |
| create_time / update_time | DATETIME | |

索引：`uk_username(username)`

**t_invite_code 邀请码表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| code | VARCHAR(16) | 唯一 |
| max_uses / used_count | INT | |
| expire_time | DATETIME | |
| created_by | BIGINT | 管理员 ID |
| status | TINYINT | 1 有效，0 作废 |
| create_time | DATETIME | |

索引：`uk_code(code)`

**t_user_profile 用户档案表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| user_id | BIGINT PK | |
| estimated_level | VARCHAR(4) | A1–C2 |
| goals | JSON | 场景类别数组 |
| daily_goal_minutes | INT | 默认 15 |
| accent | VARCHAR(4) | us / uk |
| interests | JSON | 字符串数组 |
| ielts_target | JSON | `{ overall, examDate }` |
| work_profile | JSON | `{ industry, role, companyType, commonTopics[] }` |
| preferences | JSON | 练习、字幕、音频偏好 |
| prefer_own_key | TINYINT | 是否优先使用自己的 Key |
| onboarding_done | TINYINT | |
| create_time / update_time | DATETIME | |

#### AI 服务与额度

**t_api_credential AI 凭证表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| scope | VARCHAR(16) | PLATFORM / USER |
| user_id | BIGINT | scope=USER 时必填 |
| provider_type | VARCHAR(16) | LLM / ASR / TTS / PRONUNCIATION |
| vendor | VARCHAR(32) | openai-compatible / aliyun / mimo 等 |
| base_url | VARCHAR(255) | |
| model | VARCHAR(64) | |
| api_key_cipher | VARBINARY(512) | AES-GCM 密文（含 IV） |
| api_key_hint | VARCHAR(16) | 末 4 位，用于页面显示 |
| extra_config | JSON | 音色、语言等 |
| unit_price | JSON | 单价，用于估算费用 |
| enabled | TINYINT | |
| last_check_time / last_check_ok | DATETIME / TINYINT | |
| create_time / update_time | DATETIME | |

索引：`uk_scope_user_type(scope, user_id, provider_type)`

**t_quota_policy 额度策略表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | 为空表示默认策略 |
| daily_voice_minutes | INT | |
| monthly_llm_tokens | BIGINT | |
| monthly_tts_chars | BIGINT | |
| monthly_pronunciation_calls | INT | |
| create_time / update_time | DATETIME | |

索引：`uk_user(user_id)`

**t_usage_record 用量记录表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| session_id | BIGINT | 可空（如查词） |
| provider_type | VARCHAR(16) | |
| key_scope | VARCHAR(16) | PLATFORM / USER |
| usage_type | VARCHAR(16) | TOKENS_IN / TOKENS_OUT / VOICE_SECONDS / TTS_CHARS / CALLS |
| amount | BIGINT | |
| estimated_cost | DECIMAL(10,4) | 按单价估算，单位元 |
| create_time | DATETIME | |

索引：`idx_user_time(user_id, create_time)`、`idx_scope_time(key_scope, create_time)`

#### 场景与题库

**t_scenario 场景表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| owner_id | BIGINT | 为空表示内置场景 |
| category | VARCHAR(16) | DAILY / IELTS / BUSINESS / SONG / FREE / CUSTOM |
| subcategory | VARCHAR(32) | 如 dining、meeting、part2 |
| title / title_zh | VARCHAR(100) | |
| description | VARCHAR(500) | |
| level | VARCHAR(4) | 推荐难度 |
| estimated_minutes | INT | |
| ai_role | JSON | `{ name, persona, voice }` |
| objectives | JSON | 任务点数组 `[{ id, description }]` |
| warmup_expressions | JSON | `[{ phrase, meaning }]` |
| exam_structure | JSON | 仅雅思 |
| evaluator | VARCHAR(16) | DAILY / IELTS / BUSINESS / SONG / GENERAL |
| hints_enabled | TINYINT | |
| sort_order | INT | |
| status | TINYINT | 1 上架，0 下架 |
| create_time / update_time | DATETIME | |

索引：`idx_category(category, status)`、`idx_owner(owner_id)`

**t_ielts_question 雅思题目表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| part | TINYINT | 1 / 2 / 3 |
| topic | VARCHAR(64) | |
| prompt | VARCHAR(500) | |
| cue_card_points | JSON | Part 2 提示要点 |
| follow_ups | JSON | 追问 |
| related_part2_id | BIGINT | Part 3 关联的 Part 2 题 |
| season | VARCHAR(16) | 如 2026-Q4 |
| status | TINYINT | |
| create_time / update_time | DATETIME | |

索引：`idx_part_topic(part, topic)`

#### 练习与复盘

**t_practice_session 练习会话表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| scenario_id | BIGINT | |
| category | VARCHAR(16) | 冗余，便于统计 |
| mode | VARCHAR(8) | VOICE / TEXT |
| level | VARCHAR(4) | |
| correction_strength | VARCHAR(8) | LIGHT / NORMAL / STRICT |
| focus | VARCHAR(500) | 本次重点 |
| carried_weak_point_ids | JSON | 带入的薄弱点 |
| exam_question_ids | JSON | 雅思抽到的题 |
| key_scope | VARCHAR(16) | 本次使用的 Key 来源 |
| status | VARCHAR(16) | ACTIVE / COMPLETED / TIMEOUT |
| review_status | VARCHAR(16) | NONE / PENDING / COMPLETED / FAILED |
| objective_progress | JSON | `[{ objectiveId, done, turnId }]` |
| song_id | BIGINT | 歌曲话题对话 |
| started_at / ended_at / last_active_at | DATETIME | |
| duration_seconds / user_speaking_seconds | INT | |
| create_time / update_time | DATETIME | |

索引：`idx_user_started(user_id, started_at)`、`idx_status_active(status, last_active_at)`（超时扫描用）

**t_turn 对话回合表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| session_id | BIGINT | |
| user_id | BIGINT | |
| turn_index | INT | |
| speaker | VARCHAR(16) | USER / ASSISTANT |
| text | TEXT | |
| interrupted | TINYINT | |
| exam_part | TINYINT | |
| question_id | BIGINT | |
| started_at / ended_at | DATETIME(3) | 语音起止，计算语速 |
| create_time | DATETIME | |

索引：`uk_session_index(session_id, turn_index)`

> 只保存已定稿的回合；流式中的字幕只在内存和 WebSocket 中传递，不入库。

**t_review 复盘表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| session_id | BIGINT | 唯一 |
| user_id | BIGINT | |
| summary | TEXT | |
| next_practice_suggestion | TEXT | |
| estimated_cefr | VARCHAR(4) | |
| general_scores | JSON | `{ accuracy, vocabulary, fluency, interaction }`，1–5 |
| metrics | JSON | `{ wordsPerMinute, averageTurnWords, fillerWordCount, chineseMixRatio, userTurnCount }` |
| weak_points | JSON | 本次薄弱点摘要 |
| recommended_vocabulary | JSON | `[{ term, meaning, example }]` |
| scenario_assessment | JSON | 按评估器类型的结构，见 8.4 |
| model | VARCHAR(64) | |
| error_message | VARCHAR(500) | |
| create_time / update_time | DATETIME | |

索引：`uk_session(session_id)`、`idx_user_time(user_id, create_time)`

**t_review_issue 纠错表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| review_id | BIGINT | |
| user_id | BIGINT | |
| turn_id | BIGINT | 可空 |
| original / improved | TEXT | |
| reason | VARCHAR(500) | |
| category | VARCHAR(16) | GRAMMAR / TENSE / ARTICLES / PREPOSITIONS / WORD_CHOICE / COLLOCATION / PRONUNCIATION / FLUENCY / COHERENCE / REGISTER / INTERACTION / OTHER |

索引：`idx_review(review_id)`、`idx_user_category(user_id, category)`

#### 学习数据

**t_weak_point 薄弱点表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| normalized_key | VARCHAR(100) | 合并相近问题用 |
| title | VARCHAR(200) | |
| category | VARCHAR(16) | |
| status | VARCHAR(16) | OPEN / IMPROVING / RESOLVED |
| occurrence_count | INT | |
| consecutive_absent_sessions | INT | 连续 N 次未出现 → IMPROVING / RESOLVED |
| first_seen_at / last_seen_at | DATETIME | |

索引：`uk_user_key(user_id, normalized_key)`

**t_weak_point_occurrence 薄弱点出现记录表**：`weak_point_id`、`session_id`、`evidence VARCHAR(500)`；主键 `(weak_point_id, session_id)`。

**t_vocabulary 词汇表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| normalized_term | VARCHAR(100) | |
| term | VARCHAR(100) | |
| phonetic | VARCHAR(100) | |
| meaning | VARCHAR(500) | |
| contextual_meaning | VARCHAR(500) | |
| example | VARCHAR(500) | |
| source | VARCHAR(16) | SUBTITLE / REVIEW / SONG / MANUAL |
| familiarity | VARCHAR(16) | NEW / LEARNING / MASTERED |
| occurrence_count | INT | |
| first_saved_at / last_saved_at | DATETIME | |

索引：`uk_user_term(user_id, normalized_term)`

**t_sentence 收藏句子表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| session_id / turn_id / lyric_line_id | BIGINT | 来源，三选一 |
| speaker | VARCHAR(16) | USER / ASSISTANT / SONG |
| text | TEXT | |
| analysis | JSON | `{ translation, structure, reusablePattern, expressions[], examples[], tip }` |
| saved_at | DATETIME | |

索引：`idx_user(user_id)`、`uk_user_turn(user_id, turn_id)`

**t_srs_schedule 复习计划表**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | BIGINT PK | |
| user_id | BIGINT | |
| item_type | VARCHAR(16) | VOCABULARY / SENTENCE |
| item_id | BIGINT | |
| interval_days | DECIMAL(6,2) | |
| ease_factor | DECIMAL(4,2) | 初始 2.50 |
| repetitions | INT | |
| next_review_at | DATETIME | |
| last_reviewed_at | DATETIME | |
| last_rating | VARCHAR(8) | AGAIN / HARD / GOOD / EASY |

索引：`uk_item(item_type, item_id)`、`idx_user_due(user_id, next_review_at)`

#### 歌曲

**t_song 歌曲表**：`id`、`user_id`、`title`、`artist`、`audio_path`、`duration_seconds`、`lyrics_source`（LRC / AUTO_ALIGNED / MANUAL）、`last_practiced_at`、`create_time`；索引 `idx_user(user_id)`。

**t_lyric_line 歌词行表**：`id`、`song_id`、`line_index`、`text`、`start_ms`、`end_ms`、`explanation JSON`（`{ translation, slangAndIdioms[], connectedSpeech[], culturalNote }`）；索引 `uk_song_index(song_id, line_index)`。

**t_shadowing_attempt 跟唱记录表**：`id`、`user_id`、`lyric_line_id`、`recording_path`、`score`、`word_scores JSON`、`create_time`；索引 `idx_line(lyric_line_id)`。

#### 统计与词典

**t_daily_stat 每日统计表**：`user_id`、`stat_date DATE`、`practice_minutes`、`session_count`、`by_category JSON`、`reviewed_items`、`goal_met`；主键 `(user_id, stat_date)`。

**t_dict_entry 词典表**：`word`（主键）、`phonetic`、`translation`、`definition`、`exchange`（词形变化）。

### 8.4 场景评估结果结构（`t_review.scenario_assessment`）

```json
// 日常
{ "type": "DAILY", "objectivesCompleted": 2, "objectivesTotal": 3, "intelligibility": 4 }

// 雅思
{
  "type": "IELTS",
  "fc": 6.0, "lr": 6.5, "gra": 5.5, "p": 6.0, "overall": 6.0,
  "rationale": { "fc": "…", "lr": "…", "gra": "…", "p": "…" },
  "answers": [
    { "questionId": "1795…", "userAnswer": "…", "improvedAnswer": "…", "notes": ["…"] }
  ]
}

// 商务
{
  "type": "BUSINESS",
  "clarity": 4, "professionalism": 3, "register": 3, "structure": 4,
  "registerIssues": [ { "original": "…", "better": "…", "reason": "…" } ]
}

// 歌曲
{ "type": "SONG", "averageLineScore": 78, "weakWords": ["through", "world"] }
```

`p` 在未配置发音评测时为 `null`，前端显示“未评估”。

---

## 9. API 初稿设计

### 9.1 通用约定

- **风格**：RESTful，前缀 `/api`；管理接口前缀 `/api/admin`；实时语音使用 WebSocket `/ws/practice`。
- **认证**：请求头 `Authorization: Bearer <accessToken>`；刷新令牌放 HttpOnly Cookie，通过 `/api/auth/refresh` 换新。WebSocket 在连接参数中携带短期票据（`/api/practice/sessions/{id}/ws-ticket` 获取，60 秒有效）。
- **统一响应体**：

```json
{ "code": 0, "message": "success", "data": { } }
```

- **分页**：请求 `?page=1&size=20`，响应 `data` 为 `{ "records": [], "total": 0, "page": 1, "size": 20 }`。
- **ID**：雪花 ID 一律以字符串返回。
- **时间**：ISO 8601，带时区。
- **错误码**：

| code | 含义 | HTTP 状态 |
| --- | --- | --- |
| 0 | 成功 | 200 |
| 40001 | 参数校验失败 | 400 |
| 40101 | 未登录或令牌过期 | 401 |
| 40102 | 账号或密码错误 | 401 |
| 40103 | 账号已锁定或禁用 | 403 |
| 40301 | 无权限 | 403 |
| 40401 | 资源不存在 | 404 |
| 40901 | 状态冲突（如已有进行中的练习） | 409 |
| 42901 | 平台额度已用完 | 429 |
| 42902 | 请求过于频繁 | 429 |
| 50001 | 系统异常 | 500 |
| 50201 | AI 服务未配置 | 502 |
| 50202 | AI 服务认证失败（Key 无效） | 502 |
| 50203 | AI 服务超时或不可用 | 502 |

`message` 使用可直接展示给用户的中文。

### 9.2 认证与账号

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/setup/status` | 是否需要初始化 | – | `{ needSetup: boolean }` |
| POST | `/api/setup/admin` | 创建首个管理员（仅无用户时） | `{ username, password }` | `{ userId }` |
| POST | `/api/auth/login` | 登录 | `{ username, password }` | `{ accessToken, expiresIn, user }` |
| POST | `/api/auth/register` | 邀请码注册 | `{ inviteCode, username, password }` | `{ accessToken, expiresIn, user }` |
| POST | `/api/auth/refresh` | 刷新令牌 | Cookie | `{ accessToken, expiresIn }` |
| POST | `/api/auth/logout` | 退出 | – | – |
| PUT | `/api/auth/password` | 修改密码 | `{ oldPassword, newPassword }` | – |

### 9.3 个人档案与设置

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/me` | 当前用户与档案 | – | `{ user, profile }` |
| PUT | `/api/me/profile` | 更新档案 | 档案字段（部分更新） | `profile` |
| GET | `/api/me/credentials` | 我的 AI Key 列表（不含明文） | – | `[{ providerType, vendor, baseUrl, model, apiKeyHint, enabled }]` |
| PUT | `/api/me/credentials/{providerType}` | 保存我的 Key | `{ vendor, baseUrl, model, apiKey?, extraConfig? }` | 同上单项 |
| DELETE | `/api/me/credentials/{providerType}` | 删除我的 Key | – | – |
| POST | `/api/me/credentials/{providerType}/check` | 检测连通性 | `{ vendor, baseUrl, model, apiKey? }`（apiKey 为空时用已保存的） | `{ ok, message, latencyMs }` |
| GET | `/api/me/quota` | 我的额度与剩余 | – | `{ daily: { voiceMinutes: { limit, used } }, monthly: { llmTokens, ttsChars, pronunciationCalls } }` |
| GET | `/api/me/usage` | 我的用量明细 | `from, to, page, size` | 分页 |

### 9.4 场景与题库

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/scenarios` | 场景列表（内置 + 我的） | `category, subcategory, keyword` | `[ScenarioSummary]`（含我的完成次数、最近分数） |
| GET | `/api/scenarios/{id}` | 场景详情 | – | `Scenario` |
| POST | `/api/scenarios` | 新建自定义场景 | 场景字段 | `Scenario` |
| PUT | `/api/scenarios/{id}` | 修改自定义场景 | 场景字段 | `Scenario` |
| DELETE | `/api/scenarios/{id}` | 删除自定义场景 | – | – |
| GET | `/api/ielts/questions` | 题库查询 | `part, topic, page, size` | 分页 |
| POST | `/api/ielts/mock-exams/draw` | 抽一套模拟题 | `{ excludeRecentDays: 30 }` | `{ part1: [], part2: {}, part3: [] }` |

### 9.5 练习会话

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| POST | `/api/practice/sessions` | 创建练习（校验额度、锁定 Key 来源） | 见下 | `{ session, keyScope, quotaRemaining, warnings[] }` |
| GET | `/api/practice/sessions/active` | 查询我进行中的练习（用于恢复） | – | `session` 或 `null` |
| GET | `/api/practice/sessions/{id}` | 会话详情（含已定稿回合） | – | `{ session, turns }` |
| POST | `/api/practice/sessions/{id}/ws-ticket` | 获取 WebSocket 连接票据 | – | `{ ticket, expiresIn: 60 }` |
| POST | `/api/practice/sessions/{id}/hint` | 卡壳求助 | – | `{ hints: [{ level: "SIMPLE" \| "NATURAL" \| "ADVANCED", text }] }` |
| POST | `/api/practice/sessions/{id}/end` | 结束并触发复盘 | `{ generateReview: true }` | `{ session, reviewStatus: "PENDING" \| "NONE" }` |

`POST /api/practice/sessions` 请求示例：

```json
{
  "scenarioId": "1795230000000000001",
  "mode": "VOICE",
  "level": "B2",
  "correctionStrength": "NORMAL",
  "focus": "Use more complex sentences in Part 3",
  "carryWeakPointIds": ["1795230000000000123"],
  "examQuestionIds": { "part1": ["…", "…"], "part2": "…", "part3": ["…", "…"] }
}
```

### 9.6 WebSocket 实时练习协议

连接：`wss://<host>/ws/practice?sessionId=<id>&ticket=<ticket>`

- **文本帧**：JSON 控制消息，格式 `{ "type": "...", "data": { ... } }`
- **二进制帧**：音频。上行为 16 kHz、16 位、单声道 PCM；下行为 TTS 音频，帧头 8 字节（4 字节 `packetSeq` + 4 字节 `generation`），之后是 24 kHz 16 位单声道 PCM。

**上行消息（浏览器 → 服务器）**

| type | data | 说明 |
| --- | --- | --- |
| `speech_start` | `{}` | 浏览器 VAD 检测到开口；服务器开启 ASR 流，并中止当前 AI 回复 |
| `speech_end` | `{}` | 浏览器 VAD 检测到说完；服务器结束 ASR 流并定稿 |
| `text_message` | `{ text, speakReply: boolean }` | 文字输入，会打断当前回复 |
| `interrupt` | `{}` | 主动打断 |
| `exam_advance` | `{ to: "PART2_SPEAK" \| "PART2_FOLLOWUP" \| "PART3" }` | 雅思阶段推进（提前结束准备或陈述） |
| `playback_ended` | `{ generation }` | 某次回复的音频已播放完毕（用于计算 AI 说话时长） |
| `ping` | `{}` | 心跳，每 20 秒一次 |

**下行消息（服务器 → 浏览器）**

| type | data | 说明 |
| --- | --- | --- |
| `ready` | `{ sessionId, resumedTurns }` | 连接就绪；重连时返回已定稿的回合数 |
| `transcript` | `{ turnId, speaker, text, final, interrupted }` | 字幕增量；`final=true` 时已入库 |
| `voice_phase` | `{ phase: "LISTENING" \| "THINKING" \| "SPEAKING" \| "IDLE" }` | AI 状态 |
| `reply_start` / `reply_end` | `{ generation }` | 一次 AI 回复的开始与结束 |
| `objective_done` | `{ objectiveId, turnId }` | 任务点完成 |
| `exam_timer` | `{ part, phase, remainingSeconds }` | 雅思计时，每秒一次 |
| `quota_warning` | `{ remainingMinutes }` | 额度即将用完 |
| `error` | `{ code, message, retryable }` | 错误（如 ASR 断线） |
| `pong` | `{}` | 心跳响应 |

### 9.7 复盘

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/reviews/{sessionId}` | 获取复盘 | – | `{ status, review, issues[] }` |
| GET | `/api/reviews/{sessionId}/progress` | 复盘进度（轮询） | – | `{ status, stage: "REQUESTING" \| "CHUNKING" \| "MERGING" \| "SAVING", percent }` |
| POST | `/api/reviews/{sessionId}/regenerate` | 重新生成 | – | `{ status: "PENDING" }` |
| GET | `/api/reviews/{sessionId}/cost-estimate` | 复盘前估算消耗 | – | `{ characters, estimatedTokens }` |

### 9.8 字幕、查词与收藏

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/dictionary/lookup` | 查词 | `word, context?, withContext=false` | `{ word, phonetic, definitions[], contextualMeaning?, naturalAlternative? }` |
| POST | `/api/vocabulary` | 收藏单词 | `{ term, sessionId?, turnId?, source }` | `Vocabulary` |
| POST | `/api/sentences` | 收藏句子（仅已定稿回合或歌词行） | `{ turnId }` 或 `{ lyricLineId }` | `Sentence` |

### 9.9 学习中心

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| GET | `/api/learning/dashboard` | 概览 | `period=WEEK\|MONTH, category?` | 时长、天数、连续天数、评分趋势、CEFR 趋势、常见错误 |
| GET | `/api/learning/sessions` | 历史检索 | `keyword, category, mode, status, from, to, page, size` | 分页 |
| GET | `/api/learning/sessions/{id}` | 历史详情 | – | `{ session, turns, review, issues, sentences }` |
| DELETE | `/api/learning/sessions/{id}` | 删除记录（级联删除回合、复盘、关联记录） | – | – |
| GET | `/api/vocabulary` | 词汇列表 | `familiarity, due, keyword, page, size` | 分页 |
| PUT | `/api/vocabulary/{id}` | 编辑释义 | `{ meaning, example }` | `Vocabulary` |
| DELETE | `/api/vocabulary/{id}` | 删除 | – | – |
| GET | `/api/sentences` | 句子列表 | `due, keyword, page, size` | 分页 |
| GET | `/api/srs/queue` | 今日复习队列（不调用 LLM） | `itemType?, limit=50` | `[{ itemType, item, schedule }]` |
| POST | `/api/srs/rate` | 提交复习评分 | `{ itemType, itemId, rating }` | `schedule` |
| GET | `/api/weak-points` | 薄弱点列表 | `status?` | `[WeakPoint]` |
| PUT | `/api/weak-points/{id}` | 修改状态 | `{ status }` | `WeakPoint` |
| POST | `/api/weak-points/drill` | 针对薄弱点创建专项练习 | `{ ids[], mode }` | `{ session }` |
| GET | `/api/learning/ielts-history` | 雅思成绩单 | – | `[{ sessionId, date, fc, lr, gra, p, overall }]` |
| GET | `/api/export/vocabulary.csv` | 导出词汇 CSV | – | 文件 |
| GET | `/api/export/sentences.csv` | 导出句子 CSV | – | 文件 |

### 9.10 歌曲

| 方法 | 路径 | 说明 | 请求体 / 参数 | 响应 data |
| --- | --- | --- | --- | --- |
| POST | `/api/songs` | 上传歌曲（multipart） | `audio`、`lyricsFile?`、`lyricsText?`、`title`、`artist?` | `{ song, alignment: "LRC" \| "NEEDS_ALIGNMENT" }` |
| GET | `/api/songs` | 我的歌曲 | `page, size` | 分页 |
| GET | `/api/songs/{id}` | 详情（含歌词行） | – | `{ song, lines[] }` |
| GET | `/api/songs/{id}/audio` | 获取音频（校验归属，支持 Range） | – | 音频流 |
| DELETE | `/api/songs/{id}` | 删除（含文件） | – | – |
| POST | `/api/songs/{id}/align` | 自动对齐 | – | `{ lines[], confidence }` |
| PUT | `/api/songs/{id}/timings` | 手动打点 | `[{ lineId, startMs, endMs }]` | – |
| GET | `/api/songs/lines/{lineId}/explanation` | 获取或生成单句讲解（缓存） | – | `explanation` |
| POST | `/api/songs/lines/{lineId}/attempts` | 上传跟读录音并评分（multipart） | `recording` | `{ attempt }` |
| POST | `/api/songs/{id}/topic-chat` | 创建歌曲话题对话 | `{ mode }` | `{ session }` |

### 9.11 管理后台（需 ADMIN）

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET / POST | `/api/admin/users` | 用户列表 / 创建用户 |
| PUT | `/api/admin/users/{id}` | 修改角色、状态 |
| POST | `/api/admin/users/{id}/reset-password` | 重置密码，返回临时密码 |
| GET / POST | `/api/admin/invites` | 邀请码列表 / 生成 |
| DELETE | `/api/admin/invites/{id}` | 作废 |
| GET | `/api/admin/credentials` | 平台 Key 列表（不含明文） |
| PUT | `/api/admin/credentials/{providerType}` | 保存平台 Key |
| POST | `/api/admin/credentials/{providerType}/check` | 检测连通性 |
| GET / PUT | `/api/admin/quota/default` | 默认额度 |
| PUT | `/api/admin/quota/users/{userId}` | 单人额度 |
| GET | `/api/admin/usage/summary` | 用量汇总（`groupBy=USER\|PROVIDER\|DAY`, `from`, `to`） |
| GET | `/api/admin/usage/export.csv` | 导出用量 |
| GET / POST / PUT / DELETE | `/api/admin/scenarios` | 内置场景管理 |
| POST | `/api/admin/scenarios/import` | 导入场景 JSON |
| GET / POST / PUT / DELETE | `/api/admin/ielts/questions` | 雅思题库管理 |
| POST | `/api/admin/ielts/questions/import` | 导入题库 JSON |
| GET | `/api/admin/system/info` | 版本、数据库、磁盘 |
| GET | `/api/admin/system/logs` | 最近错误日志（已脱敏） |

### 9.12 AI 服务适配层（后端内部接口）

```java
public interface LlmProvider {
    /** 流式对话，onDelta 逐段回调；返回最终用量。可通过 CancellationToken 中止。 */
    LlmUsage streamChat(LlmRequest request, Consumer<String> onDelta, CancellationToken cancel);
    /** 非流式，复盘等需要完整 JSON 的场景使用，超时 300 秒。 */
    LlmResult chat(LlmRequest request, Duration timeout);
}

public interface AsrProvider {
    AsrStream open(AsrOptions options, Consumer<AsrResult> onResult);   // AsrResult: text, isFinal, startMs, endMs
}
public interface AsrStream extends AutoCloseable {
    void write(byte[] pcm16k);
    void finish();
}

public interface TtsProvider {
    /** 流式合成，onAudio 逐包回调 24kHz PCM。 */
    TtsUsage synthesize(TtsRequest request, Consumer<byte[]> onAudio, CancellationToken cancel);
}

public interface PronunciationProvider {
    PronunciationResult assess(String referenceText, Path audioFile);
}

public interface KeyRouter {
    /** 按用户偏好、额度和已配置的 Key 决定本次使用的凭证。 */
    ResolvedCredential resolve(Long userId, ProviderType type);
}

public interface QuotaService {
    void checkBeforeStart(Long userId);                     // 额度不足抛 QuotaExceededException
    void record(Long userId, Long sessionId, UsageRecord usage);
}
```

复盘请求约定（借鉴 SpeakHub）：先整份发送转写并要求返回 JSON；超过 8,000 字符时用流式接收；只有 504 或超时才按完整回合切分重试；解析时去除空项和重复项，薄弱点超出 6 条不报错。

---

## 10. 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 语音回合延迟 P50 ≤ 2 s、P90 ≤ 4 s（说完到开始播放）；打断响应 ≤ 500 ms；普通接口 P95 ≤ 300 ms；支持 20 人同时语音练习（4 核 8 GB 服务器） |
| 稳定性 | 已定稿回合即时入库；复盘失败不丢对话；WebSocket 断线自动重连并续接会话；外部 AI 服务断线自动重试并提示；30 分钟无活动的练习自动归档 |
| 安全 | 密码 BCrypt；JWT 签名密钥与 Key 加密主密钥来自环境变量；AI Key 只在服务器使用，接口不返回明文；所有业务查询带 `user_id` 条件；上传文件校验类型与大小，下载校验归属；登录失败锁定；生产环境禁止跨域 |
| 隐私 | 不接入第三方统计；管理员不能查看他人练习内容；用户可一键删除自己的全部学习数据；日志脱敏 Key 和对话正文 |
| 部署 | Docker Compose 一键启动；提供 `.env.example`；数据库与文件目录挂载为卷；提供备份脚本（mysqldump + 文件目录打包） |
| HTTPS | 局域网或公网访问必须使用 HTTPS（浏览器麦克风权限要求），部署文档提供 Nginx 证书配置示例 |
| 兼容性 | Chrome / Edge 最新两个大版本完整支持；Safari 17+ 基本可用（字幕小窗不可用）；屏幕宽度 ≥ 1024 px 优先，平板横屏可用 |
| 可维护性 | 后端分层清晰（controller / service / mapper），Provider 可替换；Flyway 管理表结构；核心逻辑单元测试覆盖率 ≥ 60% |
| 可测试性 | 分句、额度计算、Key 路由、复习调度、复盘 JSON 校验、长对话切分均有单元测试；练习主链路有集成测试（使用模拟 Provider） |

---

## 11. 版本规划

| 版本 | 主题 | 范围 | 发布门槛 |
| --- | --- | --- | --- |
| v0.1 | 能部署、能练 | 部署与初始化、登录与邀请码、平台 Key 配置、日常生活场景（10 个）、文字练习、通用复盘、历史记录 | Docker Compose 在全新服务器上一次部署成功；文字练习与复盘全链路可用 |
| v0.2 | 能说 | WebSocket 语音链路（浏览器 VAD + ASR + LLM + TTS）、打断、实时字幕、查词收藏、用户自带 Key、额度与用量记录 | 连续 30 分钟语音对话无中断；额度用完时正确拦截 |
| v0.5 | 四大场景 | 雅思模拟考与评分、商务场景与工作档案、歌曲上传与讲解、词汇本复习、学习概览、管理后台用量统计 | 雅思评分结果结构校验通过；歌曲上传覆盖 LRC 与纯文本 |
| v1.0 | 学习闭环 | 句子本复习、薄弱点追踪、带薄弱点再练、跟唱发音评分、每日目标、内容管理 | 成功指标可从后台统计；备份恢复演练通过 |
| v1.x | 增强 | 字幕小窗、材料驱动演练、数据导出、实时语音模型接入 | 按使用反馈排序 |

---

## 12. 风险与待决问题

### 12.1 风险

| 编号 | 风险 | 影响 | 应对 |
| --- | --- | --- | --- |
| R1 | 雅思评分不准，用户误把预估当真实成绩 | 信任受损 | 明确标注“AI 预估，非官方评分”；展示每项评分依据；用有真实成绩的用户校准 |
| R2 | 歌曲与歌词版权 | 法律风险 | 不内置、不抓取歌曲和歌词；上传内容仅本人可见、不提供分享；讲解按句发送 |
| R3 | 雅思题库版权与时效 | 法律风险、内容过时 | 题目原创或改编，不收录真题原文；管理员按季度更新 |
| R4 | 平台 AI 费用失控 | 管理员承担意外费用 | 额度校验 + 用量记录 + 后台汇总；单次练习超额最多 10 分钟 |
| R5 | 语音链路经服务器中转，延迟高于桌面端 | 对话不够流畅 | VAD 放浏览器；TTS 分句预取；服务器尽量部署在离用户近的地域；v0.2 实测后再调分句参数 |
| R6 | 未配置 HTTPS 时浏览器拒绝麦克风 | 语音模式不可用 | 部署文档重点说明；页面检测到非安全上下文时给出明确提示，并提供文字模式 |
| R7 | Java 端需要重写 SpeakHub 的流式语音逻辑 | 开发量大于直接复用 | 先做文字链路（v0.1）再做语音（v0.2）；按 SpeakHub 已验证的分句、打断、复盘规则实现，并照搬其测试用例的场景 |
| R8 | Safari 对 AudioWorklet、Document PiP 支持不完整 | 部分用户体验下降 | 主推 Chrome/Edge；Safari 降级为基本功能并提示 |
| R9 | 发音评测服务选型未定 | 雅思发音分、歌曲评分无法上线 | v0.5 前完成选型；未配置时显示“未评估” |

### 12.2 待决问题

- [x] 技术栈：已定为 Java 21 + Spring Boot 3 后端、Vue 3 前端、MySQL 8（2026-11-10），理由见附录 13.2。
- [x] 使用范围：自己或小范围自部署（2026-11-10）。
- [x] AI Key：平台提供（带额度）与用户自带两种都支持（2026-11-10）。
- [ ] ASR / TTS 首选供应商与音色确认（初稿按阿里云 Paraformer + 云端流式 TTS）。
- [ ] 发音评测服务选型和费用。
- [ ] 雅思题库由谁来编写和维护？
- [ ] 是否保存用户语音录音（目前只保存歌曲跟读录音）。
- [ ] 部署目标：家用 NAS、国内云服务器还是海外服务器？影响 AI 服务的网络可达性和延迟。

---

## 13. 附录：技术选型与 SpeakHub 借鉴

### 13.1 技术栈

| 层 | 选型 |
| --- | --- |
| 后端 | JDK 21、Spring Boot 3、Spring Web、Spring WebSocket、Spring Security + JWT、Spring Validation |
| 数据访问 | MyBatis-Plus、MySQL 8.0、Flyway |
| 外部 HTTP 调用 | Spring WebClient（流式 SSE）、Java-WebSocket / OkHttp（连接 ASR 服务） |
| 缓存 | Caffeine（进程内），本期不引入 Redis |
| 前端 | Vue 3（组合式 API）、Vite、TypeScript、Vue Router、Pinia、Element Plus、ECharts |
| 浏览器音频 | getUserMedia、AudioWorklet、Web Audio API、onnxruntime-web（Silero VAD） |
| 构建 | Maven（后端）、pnpm（前端） |
| 部署 | Docker Compose（app + nginx + mysql） |

### 13.2 选型理由

- **贴合开发者技能**：开发者熟悉 Java + Vue，Spring Boot + Vue 3 无需学习新语言，上手最快。
- **网页形态**：免安装，任何设备浏览器即可使用，学习记录集中在服务器，换设备不丢失。
- **自部署**：小范围使用，Docker Compose 单机部署即可，不需要集群和 Redis。
- **AI Key 在服务器**：浏览器不接触明文 Key，平台额度和用量统计都能在服务器统一控制。

### 13.3 从 SpeakHub 借鉴的内容

技术栈改为 Java 后，SpeakHub 的 TypeScript 代码不能直接复用，借鉴的是已验证的规则和设计。

| SpeakHub 能力 | 在 SpeakLoop 中的处理 |
| --- | --- |
| API 直连对话（SSE 流式、`system → user/assistant` 顺序） | 按相同规则用 Java 实现 |
| 分句规则（首句最少 24 字符、后续合并到 100 字符、短回复直接发送） | 按相同参数实现，v0.2 实测后再调 |
| TTS 预取（最多 2 句）与连续播放 | 服务器预取 + 浏览器 Web Audio 排队播放 |
| 打断顺序（先中止、再定稿被打断的回复、最后接收新输入） | 按相同顺序实现 |
| 复盘（整份优先、504 才切分、纠错不设上限、薄弱点超量不报错） | 按相同规则实现，测试用例场景照搬 |
| 词汇四档复习 | 扩展到句子，统一 `t_srs_schedule` |
| 提示词（英语优先、处理残缺输入、场景/难度/纠错模板） | 迁移为场景定义的一部分 |
| ChatGPT / Gemini 网页自动化 | 不迁移 |
| 桌面悬浮字幕窗 | 改为练习页字幕 + 可选 Document PiP 小窗 |
| 本地 Kokoro TTS、本地 VAD（sherpa-onnx） | VAD 改为浏览器端；TTS 使用云端服务 |
| GitHub Release 自动更新 | 改为 Docker 镜像版本升级 |
