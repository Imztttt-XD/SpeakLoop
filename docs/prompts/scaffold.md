# SpeakLoop 项目骨架生成 Prompt

用途：粘贴给 AI 编程助手（Claude Code、Cursor 等），生成 SpeakLoop 的初始项目骨架。需求依据见 [docs/PRD.md](../PRD.md)。

```text
请帮我创建一个 macOS 桌面端 AI 英语口语陪练应用「SpeakLoop」的项目骨架，具体要求如下：

【技术栈】
- Node.js 22 LTS，pnpm 构建
- Electron（当前稳定版）+ electron-vite
- 界面：Vue 3（组合式 API + <script setup>）+ Vue Router + Pinia + TypeScript 5（strict 模式）
- 本地数据库：SQLite（better-sqlite3），手写 SQL + 版本化迁移脚本
- 参数校验：zod
- 测试：Vitest + @vue/test-utils
- 打包：electron-builder，产出 macOS 的 dmg 和 zip（arm64 + x64），预留签名与公证配置
- API Key 存储：Electron safeStorage（macOS 下由钥匙串保护），不写入数据库和配置文件

【目录结构】
主进程采用类似 Spring Boot 的经典分层架构，界面采用 Vue 标准结构：
- src/main/ipc：接口层，按领域拆分（profile、scenario、practice、review、learning、song、settings），相当于 controller
- src/main/services：业务逻辑层（练习编排、复盘、间隔复习、歌曲等），相当于 service
- src/main/repositories：数据访问层，学习数据的唯一写入入口，相当于 mapper
- src/main/db：数据库连接、迁移脚本（migrations/001_init.sql 起）
- src/main/providers：外部 AI 服务适配层（LLM、ASR、TTS、发音评测、实时语音），每类一个接口 + 实现
- src/main/common：通用组件（统一返回体、错误码、异常、IPC 包装器、日志）
- src/preload：预加载脚本，只暴露白名单 API（window.speakloop）
- src/renderer/views：页面（首页、场景中心、场景详情、练习中、复盘、歌曲、学习中心、设置）
- src/renderer/components：通用组件
- src/renderer/stores：Pinia 状态（profile、practice、learning、settings）
- src/renderer/router：路由配置
- src/renderer/api：对 window.speakloop 的类型化封装，页面只通过这里调用主进程
- src/subtitle：悬浮字幕窗（独立 HTML 入口，Vue 应用）
- src/shared：跨进程共享的类型、IPC 通道常量、zod schema，不得依赖 Electron 或 Node API
- resources：内置场景与雅思题库 JSON、ECDICT 词典、应用图标

【通用组件】
1. 统一返回体 Result<T>：成功为 { ok: true, data }，失败为 { ok: false, error: { code, message, retryable, detail? } }，message 使用可直接展示的中文
2. 错误码枚举 ErrorCode：VALIDATION_FAILED、NOT_FOUND、CONFLICT、PROVIDER_UNAUTHORIZED、PROVIDER_TIMEOUT、PROVIDER_RATE_LIMITED、PROVIDER_UNAVAILABLE、MIC_PERMISSION_DENIED、AUDIO_DEVICE_UNAVAILABLE、SESSION_NOT_ACTIVE、SESSION_ALREADY_ACTIVE、INTERNAL
3. 自定义业务异常类 AppError（携带 ErrorCode），以及统一的 IPC 处理包装器（相当于全局异常处理器）：用 zod 校验入参，捕获 AppError、校验异常和未知异常并转换为 Result
4. 事件推送封装：主进程 emit(event, payload)，预加载层 on(event, handler) 返回取消订阅函数；界面侧提供 useIpcEvent 组合式函数，组件卸载时自动取消订阅
5. Provider 接口定义：LlmProvider（支持 SSE 流式和 AbortSignal）、AsrProvider、TtsProvider、PronunciationProvider、RealtimeVoiceProvider，先提供一个 OpenAI 兼容的 LlmProvider 实现，其余用占位实现
6. 本地诊断日志：按天滚动、保留 7 天，写日志前自动脱敏 API Key
7. 安全配置：contextIsolation 开启、nodeIntegration 关闭、外链只允许白名单域名打开
8. Info.plist 中声明麦克风权限用途说明（NSMicrophoneUsageDescription）
9. 界面基础布局：左侧边栏导航 + 右侧路由视图，支持系统深色模式

【数据库】
请生成 SQLite 建表迁移脚本（001_init.sql），包含以下表：
1. 用户档案表(user_profile)：id, estimated_level, goals, daily_goal_minutes, accent, interests, ielts_target, work_profile, created_at, updated_at
2. 场景表(scenarios)：id, category, subcategory, title, title_zh, level, estimated_minutes, evaluator_id, built_in, version, payload
3. 雅思题目表(ielts_questions)：id, part, topic, prompt, cue_card_points, follow_ups, season
4. 练习会话表(sessions)：id, scenario_id, category, mode, level, correction_strength, focus, status, started_at, ended_at, duration_seconds, user_speaking_seconds, archive_path
5. 对话回合表(turns)：id, session_id, turn_index, speaker, text, status, interrupted, exam_part, question_id, started_at, ended_at
6. 复盘表(reviews)：id, session_id, status, summary, next_practice_suggestion, estimated_cefr, metrics, scenario_assessment, model, error_message, created_at
7. 纠错表(review_issues)：id, review_id, turn_id, original, improved, reason, category
8. 薄弱点表(weak_points)：id, normalized_key, title, category, status, occurrence_count, first_seen_at, last_seen_at, consecutive_absent_sessions
9. 薄弱点出现记录表(weak_point_occurrences)：weak_point_id, session_id, evidence
10. 词汇表(vocabulary)：id, normalized_term, term, phonetic, meaning, contextual_meaning, example, source, familiarity, occurrence_count, first_saved_at, last_saved_at
11. 收藏句子表(sentences)：id, session_id, turn_id, song_line_id, speaker, text, analysis, saved_at
12. 复习计划表(review_schedules)：item_type, item_id, interval_days, ease_factor, repetitions, next_review_at, last_reviewed_at, last_rating
13. 歌曲表(songs)：id, title, artist, audio_path, duration_seconds, lyrics_source, imported_at, last_practiced_at
14. 歌词行表(lyric_lines)：id, song_id, line_index, text, start_ms, end_ms, explanation
15. 跟唱记录表(shadowing_attempts)：id, lyric_line_id, recording_path, score, word_scores, created_at
16. 每日统计表(daily_stats)：date, practice_minutes, session_count, by_category, reviewed_items, goal_met

请为每张表设置合理的字段类型（JSON 字段用 TEXT 存储并在注释中说明结构）、主键策略（使用 UUID v7 字符串，关联表用联合主键）、外键与级联删除规则，以及必要的索引（如 sessions.started_at、turns(session_id, turn_index)、vocabulary.normalized_term 唯一、weak_points.normalized_key 唯一、review_schedules.next_review_at）。

【交付要求】
- 同时生成 src/shared/types.ts 中与上述表对应的 TypeScript 类型
- 内置 2 个日常生活场景、1 个雅思模拟考场景、1 个商务场景的示例 JSON，以及首次启动时导入它们的 seed 逻辑
- 场景中心页面能从数据库读取并展示这 4 个场景（打通 Vue 页面 → api → preload → ipc → service → repository 一条完整链路）
- 为 Result 包装器、迁移脚本、一个 Repository 和场景中心页面编写 Vitest 测试
- pnpm lint、pnpm test、pnpm build 均能通过，pnpm dev 能启动出带侧边栏导航的主窗口
- 在 README 中用一张表说明各目录与 Spring Boot / Vue 项目中对应概念的关系
- 本次只搭骨架，不实现语音链路和复盘逻辑
```
