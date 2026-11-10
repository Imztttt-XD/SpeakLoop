# SpeakLoop 项目骨架生成 Prompt

用途：粘贴给 AI 编程助手（Claude Code、Cursor 等），生成 SpeakLoop 的初始项目骨架。需求依据见 [docs/PRD.md](../PRD.md)。

```text
请帮我创建一个 AI 英语口语陪练网页应用「SpeakLoop」的前后端分离项目骨架，具体要求如下：

【技术栈】
后端（backend/）：
- JDK 21, Spring Boot 3（当前稳定 3.x 版本）
- Spring Web + Spring WebSocket + Spring Validation
- Spring Security + JWT（jjwt）认证
- MyBatis-Plus 作为 ORM 框架
- MySQL 8.0 作为主数据库，Flyway 管理表结构迁移
- Caffeine 作为进程内缓存（本期不使用 Redis）
- Spring WebClient 用于调用外部 AI 服务（支持 SSE 流式）
- Lombok
- Maven 构建
前端（frontend/）：
- Vue 3（组合式 API + <script setup>）+ TypeScript + Vite
- Vue Router + Pinia
- Element Plus 组件库，ECharts 图表
- Axios 封装请求
- pnpm 构建
部署：
- Docker Compose 编排 app（Spring Boot）、nginx（托管前端静态资源并反向代理 /api 和 /ws）、mysql 三个服务

【后端目录结构】
采用经典分层架构，包名 com.speakloop：
- controller：接口层（auth、me、scenario、practice、review、learning、song、admin）
- websocket：实时练习 WebSocket 处理器、握手拦截器、消息定义
- service：业务逻辑层（含 interface 和 impl）
- mapper：数据访问层
- entity：数据库实体
- dto：数据传输对象（request / response 分包）
- provider：外部 AI 服务适配层（LlmProvider、AsrProvider、TtsProvider、PronunciationProvider 接口及实现）
- security：JWT 过滤器、认证入口、当前用户工具类
- config：配置类（Security、WebSocket、MyBatis-Plus、Jackson、WebClient、Caffeine）
- common：通用组件（统一响应体、错误码、异常处理等）
- utils：工具类（加解密、雪花 ID 转字符串等）
- resources/db/migration：Flyway 脚本

【前端目录结构】
- src/api：按模块封装的接口请求
- src/views：页面（login、register、setup、home、scenarios、practice、review、songs、learn、settings、admin）
- src/components：通用组件
- src/stores：Pinia 状态（user、practice、settings）
- src/router：路由配置与登录、管理员权限守卫
- src/utils：request.ts（Axios 实例、令牌注入、401 自动刷新、统一错误提示）
- src/layouts：用户端布局（左侧导航）、管理后台布局

【通用组件】
1. 统一响应体 Result<T>，包含 code、message、data 三个字段；分页响应 PageResult<T> 包含 records、total、page、size
2. 错误码枚举 ErrorCode：0 成功、40001 参数校验失败、40101 未登录或令牌过期、40102 账号或密码错误、40103 账号已锁定或禁用、40301 无权限、40401 资源不存在、40901 状态冲突、42901 平台额度已用完、42902 请求过于频繁、50001 系统异常、50201 AI 服务未配置、50202 AI 服务认证失败、50203 AI 服务超时或不可用；message 使用可直接展示的中文
3. 全局异常处理器，覆盖参数校验异常、业务异常、认证与权限异常、系统异常
4. 自定义业务异常类 BusinessException（携带 ErrorCode）
5. JWT 认证：访问令牌 2 小时，刷新令牌 7 天放 HttpOnly Cookie；登录连续失败 5 次锁定 15 分钟
6. 数据隔离：提供 CurrentUser 工具类获取当前用户 ID，所有业务 Service 查询必须带 user_id 条件
7. AI Key 加解密工具：AES-256-GCM，主密钥从环境变量 SPEAKLOOP_MASTER_KEY 读取
8. 雪花 ID 主键（MyBatis-Plus ASSIGN_ID），Jackson 全局把 Long 序列化为字符串
9. 跨域配置（仅 dev 配置文件启用，允许 http://localhost:5173）
10. MyBatis-Plus 自动填充 create_time、update_time，分页插件

【数据库】
请生成 Flyway 迁移脚本 V1__init.sql（MySQL 8.0，utf8mb4），包含以下表：
1. 用户表(t_user)：id, username, password_hash, role, status, failed_login_count, locked_until, last_login_time, create_time, update_time
2. 邀请码表(t_invite_code)：id, code, max_uses, used_count, expire_time, created_by, status, create_time
3. 用户档案表(t_user_profile)：user_id, estimated_level, goals, daily_goal_minutes, accent, interests, ielts_target, work_profile, preferences, prefer_own_key, onboarding_done, create_time, update_time
4. AI 凭证表(t_api_credential)：id, scope, user_id, provider_type, vendor, base_url, model, api_key_cipher, api_key_hint, extra_config, unit_price, enabled, last_check_time, last_check_ok, create_time, update_time
5. 额度策略表(t_quota_policy)：id, user_id, daily_voice_minutes, monthly_llm_tokens, monthly_tts_chars, monthly_pronunciation_calls, create_time, update_time
6. 用量记录表(t_usage_record)：id, user_id, session_id, provider_type, key_scope, usage_type, amount, estimated_cost, create_time
7. 场景表(t_scenario)：id, owner_id, category, subcategory, title, title_zh, description, level, estimated_minutes, ai_role, objectives, warmup_expressions, exam_structure, evaluator, hints_enabled, sort_order, status, create_time, update_time
8. 雅思题目表(t_ielts_question)：id, part, topic, prompt, cue_card_points, follow_ups, related_part2_id, season, status, create_time, update_time
9. 练习会话表(t_practice_session)：id, user_id, scenario_id, category, mode, level, correction_strength, focus, carried_weak_point_ids, exam_question_ids, key_scope, status, review_status, objective_progress, song_id, started_at, ended_at, last_active_at, duration_seconds, user_speaking_seconds, create_time, update_time
10. 对话回合表(t_turn)：id, session_id, user_id, turn_index, speaker, text, interrupted, exam_part, question_id, started_at, ended_at, create_time
11. 复盘表(t_review)：id, session_id, user_id, summary, next_practice_suggestion, estimated_cefr, general_scores, metrics, weak_points, recommended_vocabulary, scenario_assessment, model, error_message, create_time, update_time
12. 纠错表(t_review_issue)：id, review_id, user_id, turn_id, original, improved, reason, category
13. 薄弱点表(t_weak_point)：id, user_id, normalized_key, title, category, status, occurrence_count, consecutive_absent_sessions, first_seen_at, last_seen_at
14. 薄弱点出现记录表(t_weak_point_occurrence)：weak_point_id, session_id, evidence
15. 词汇表(t_vocabulary)：id, user_id, normalized_term, term, phonetic, meaning, contextual_meaning, example, source, familiarity, occurrence_count, first_saved_at, last_saved_at
16. 收藏句子表(t_sentence)：id, user_id, session_id, turn_id, lyric_line_id, speaker, text, analysis, saved_at
17. 复习计划表(t_srs_schedule)：id, user_id, item_type, item_id, interval_days, ease_factor, repetitions, next_review_at, last_reviewed_at, last_rating
18. 歌曲表(t_song)：id, user_id, title, artist, audio_path, duration_seconds, lyrics_source, last_practiced_at, create_time
19. 歌词行表(t_lyric_line)：id, song_id, line_index, text, start_ms, end_ms, explanation
20. 跟唱记录表(t_shadowing_attempt)：id, user_id, lyric_line_id, recording_path, score, word_scores, create_time
21. 每日统计表(t_daily_stat)：user_id, stat_date, practice_minutes, session_count, by_category, reviewed_items, goal_met
22. 词典表(t_dict_entry)：word, phonetic, translation, definition, exchange

请为每张表设置合理的字段类型（JSON 结构的字段用 MySQL JSON 类型，并在 COMMENT 中说明结构）、主键策略（雪花 ID 用 BIGINT；关联表和统计表用联合主键）和必要的索引，至少包括：
- t_user.username 唯一，t_invite_code.code 唯一
- t_api_credential(scope, user_id, provider_type) 唯一
- t_usage_record(user_id, create_time)
- t_practice_session(user_id, started_at)、(status, last_active_at)
- t_turn(session_id, turn_index) 唯一
- t_review.session_id 唯一
- t_weak_point(user_id, normalized_key) 唯一
- t_vocabulary(user_id, normalized_term) 唯一
- t_srs_schedule(item_type, item_id) 唯一、(user_id, next_review_at)
- t_lyric_line(song_id, line_index) 唯一
每张表和字段都要写 COMMENT。

【交付要求】
- 另写 V2__seed.sql：插入默认额度策略，以及 2 个日常生活场景、1 个雅思模拟考场景、1 个商务场景的示例数据
- 实现以下完整链路，其余模块只建空的 Controller / Service 骨架：
  1. 初始化管理员：GET /api/setup/status、POST /api/setup/admin
  2. 登录与刷新令牌：POST /api/auth/login、POST /api/auth/refresh、POST /api/auth/logout
  3. 场景列表：GET /api/scenarios，前端场景中心页面展示 seed 中的 4 个场景
- 前端完成登录页、初始化页、带左侧导航的主布局、场景中心页，以及管理员路由守卫
- Provider 只定义接口，并提供一个 OpenAI 兼容的 LlmProvider 实现（WebClient + SSE 流式），其余用抛出“未实现”的占位实现
- 为 Result、全局异常处理器、AES 加解密工具、AuthService 编写 JUnit 5 单元测试
- 提供 docker-compose.yml、backend/Dockerfile、nginx.conf（含 /ws 的 WebSocket 反向代理配置）和 .env.example（DB_PASSWORD、JWT_SECRET、SPEAKLOOP_MASTER_KEY）
- README 说明本地开发启动步骤（MySQL、后端、前端）和 Docker 部署步骤，并提醒：局域网或公网访问必须配置 HTTPS，否则浏览器无法使用麦克风
- mvn test 通过，pnpm build 通过，docker compose up 后能登录并看到场景中心
- 本次只搭骨架，不实现语音链路、复盘和额度逻辑
```
