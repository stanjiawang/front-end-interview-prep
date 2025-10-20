# 🧩 Front-End System Design — Tech Stack & Trade-off Matrix

| **Step** | **主题 / Category** | **可选技术栈 / Tools** | **适用场景 / When to Use** | **优点 / Pros** | **缺点 / Cons** | **取舍与面试讨论点 / Trade-offs & Discussion Topics** |
|:--:|:--|:--|:--|:--|:--|:--|
| **1** | 🎯 **需求澄清与定界 (Requirements Clarification)** | N/A（分析阶段） | 明确目标用户、核心功能、非功能指标 | 能聚焦范围、控制复杂度、对齐目标指标 (LCP, INP, SEO 等) | 若范围定义不清，后续架构可能偏离 | - 优先定义性能目标（LCP/INP/CLS）<br>- 是否需要 SEO？<br>- 用户基数与国际化是否影响选型？ |
|  | 用户基数 & 平台 | Web / Mobile Web / PWA / Desktop(Electron/Tauri) | 不同终端支持 | 统一技术栈 | 性能与体验差异化难 | 跨端需权衡“代码复用 vs 端体验优化” |
|  | 国际化 (i18n/l10n) | react-intl / next-intl / i18next | 多语言站点 | 自动语言切换、动态翻译 | 构建复杂、包体大 | SSR/SSG 的 i18n 路由分流方式？ |
|  | 性能指标 | LCP / INP / CLS / TBT / TTI | 通用性能目标 | 量化体验 | 调优复杂 | 哪个指标是业务关键？如何监控？ |
|  | 浏览器支持 | modern-only / polyfill / legacy bundle | 用户群不同 | 简化构建或扩展兼容 | 成本 vs 覆盖率 | 现代化构建策略（Vite/Next 自动分包） |

| **2** | 🏗️ **高层架构 (High-Level Architecture)** | CSR (SPA) | 内部工具、仪表盘、B2B SaaS | 前端简单、静态可缓存 | 首屏慢、SEO 弱 | SEO 不重要可选；LCP 优化靠骨架屏 |
|  |  | SSR (Next.js/Remix/Nuxt) | 电商、内容+应用混合 | 首屏快、SEO 好 | 服务器成本高、缓存复杂 | SSR vs CSR 的权衡：性能 vs 成本 |
|  |  | SSG/ISR | 内容型网站、博客 | 极快首屏、可 CDN | 动态性差 | 内容更新频率决定是否可用 |
|  |  | 混合渲染 (Next.js App Router) | 同时有动态与静态页面 | SEO+性能兼顾 | 约束多、学习曲线高 | 面试：解释 “何时 SSR / 何时 CSR” |
|  | 微前端 | Module Federation / single-spa / Web Components | 多团队协作、大型系统 | 独立部署、渐进迁移 | 架构重、性能损耗 | 团队自治 vs 性能开销 |
|  | 前端框架 | React / Vue / Svelte / Solid | 通用前端开发 | React 生态最广 | 复杂度高 | React 主流、Svelte 小而快、Vue 学习曲线低 |
|  | 构建工具 | Vite / Webpack / Turbopack | 通用 | 快速热更、现代特性 | 部分插件兼容性 | Vite 开发快；Webpack 兼容性好 |
|  | 样式方案 | CSS Modules / Tailwind / Styled-Components / Emotion | 组件样式管理 | Tailwind 一致性高 | CSS-in-JS 性能差 | Tailwind vs CSS Modules 的语义性取舍 |
|  | Monorepo | Turborepo / Nx / pnpm workspace | 多包项目 | 共享类型、依赖隔离 | 初期复杂 | 大型前端项目推荐 |

| **3** | 📡 **数据模型与 API 设计 (Data Model & API Interface)** | REST API | 通用系统 | 简单、缓存友好 | Over-fetch / Under-fetch | REST 标准化最佳实践 |
|  |  | GraphQL / Apollo | 多端多视图 | 灵活、强类型 | 缓存复杂、学习曲线 | 前端灵活取数 vs 后端复杂性 |
|  |  | tRPC | 全栈 TS 团队 | 类型共享、开发快 | 耦合强、生态小 | 若后端也 TS 则效率高 |
|  |  | WebSocket / SSE | 聊天、协作、行情 | 实时推送、低延迟 | 连接管理复杂 | 实时性 vs 伸缩性 |
|  | 数据结构设计 | TypeScript 类型定义 / OpenAPI | 通用 | 自文档化、校验一致 | 初始建模复杂 | Entity 字段与关系需前期清晰定义 |
|  | 分页策略 | Offset / Cursor / Keyset | 长列表分页 | Cursor 稳定性好 | Offset 跳页风险 | 面试常问 Cursor Pagination 机制 |
|  | 缓存策略 | ETag / Cache-Control / SWR | API 性能优化 | 响应更快、降负载 | 缓存一致性复杂 | React Query + SWR 的组合使用策略 |
|  | 通信协议 | HTTP/3 / gRPC-Web / GraphQL over HTTP | 高并发系统 | 更高效传输 | 部署门槛 | gRPC-Web 对类型安全友好 |

| **4** | 🔁 **状态管理与数据流 (State Management & Data Flow)** | React Query / SWR | 服务端数据管理 | 缓存/重试/并发/乐观更新 | 概念多 | Server State 的首选 |
|  |  | Zustand | UI / 本地状态管理 | 轻量、简洁 | 缺乏中间件生态 | 替代 Redux 小场景理想 |
|  |  | Redux Toolkit | 大型复杂状态 | 可调试、规范 | 样板代码多 | 面试点：Redux vs Zustand |
|  |  | Context + Hook | 局部状态 | 原生简洁 | 滥用易重渲染 | 小范围状态传递最佳 |
|  |  | Apollo Client | GraphQL + 缓存 | 强类型、内置缓存 | 重 | GraphQL 项目首选 |
|  | 状态划分 | Local vs Global vs Server | 所有应用 | 可维护 | 划分难 | 关键在于边界：谁是权威数据源？ |
|  | 数据同步 | 乐观更新 / 后台刷新 / 去重 | 高交互应用 | 更流畅体验 | 需防止竞态 | “Optimistic Update 回滚机制” 常被问 |
|  | 离线支持 | PWA + IndexedDB / localStorage | 移动端 | 离线体验好 | 同步难 | React Query + PWA 离线缓存策略 |

| **5** | ⚡ **性能优化与细节 (Performance, Security, A11y)** | Code Splitting / Dynamic Import | 大型应用 | 降低首屏 JS 体积 | 配置复杂 | 按路由分包、动态加载 |
|  | Lazy Loading / Suspense | 异步模块加载 | 提升加载感知 | 边界多 | 骨架屏、Spinner 配合 |
|  | 图片优化 | Next/Image / responsive srcset | 所有网站 | 显著降低 LCP | SSR 集成复杂 | 用 WebP + Lazy Loading |
|  | Core Web Vitals | LCP / INP / CLS / TBT | 性能评估 | 可量化 | 调优难 | 监控与 CI 门控 |
|  | 虚拟列表 | react-window / react-virtual | 长列表 | 优化渲染性能 | 实现复杂 | 面试常问：Virtualization 原理 |
|  | Design System | MUI / Radix UI / shadcn/ui | 组件一致性 | 可复用、可访问性好 | 初期建设成本高 | 设计系统 vs 快速开发 |
|  | 安全 | CSP / Trusted Types / CSRF Token / HttpOnly Cookie | 所有应用 | 防御常见攻击 | 实施复杂 | JWT vs Session 权衡 |
|  | a11y | ARIA / axe / eslint-plugin-jsx-a11y | 法规或企业要求 | 合规 | 成本高 | 可访问性是质量的一部分 |
|  | 测试与监控 | Jest / RTL / Playwright / Lighthouse CI / Sentry | 通用 | 保证质量 | 维护开销 | 性能监控 + 错误追踪 |

| **6** | 🧮 **总结与权衡 (Summary & Trade-offs)** | - | 汇总与汇报阶段 | 展示全局视角 | 需逻辑清晰 | 面试收尾关键：解释取舍 |
|  | 渲染策略取舍 | CSR vs SSR vs SSG | 各类站点 | SEO vs 成本 | 部署复杂度 | 用业务场景说明选择原因 |
|  | 状态管理取舍 | React Query vs Redux vs Zustand | 各规模项目 | 清晰职责 | 学习曲线 | React Query 管远端数据；Redux 管流程状态 |
|  | 样式取舍 | Tailwind vs CSS Modules | 所有应用 | 开发效率 vs 语义性 | 一致性 vs 可读性 | Tailwind: 组件一致性；CSS Modules: 强语义 |
|  | API 取舍 | REST vs GraphQL | API 层 | 简单 vs 灵活 | 端到端一致性 | REST 更通用，GraphQL 更灵活 |
|  | Auth 模式取舍 | Session vs JWT | 登录系统 | 安全 vs 无状态 | 吊销难 | 面试时强调 HttpOnly Cookie 的好处 |
|  | 架构取舍 | 单体 vs 微前端 | 大型项目 | 自治 vs 开销 | 集成难 | 说明组织结构驱动架构 |
|  | 最佳实践输出 | 文档 + CI/CD + 性能监控 | 交付阶段 | 可复用、透明 | 持续维护成本 | 有助展示工程视角 |
