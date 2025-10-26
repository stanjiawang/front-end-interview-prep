# Behavioral Interview Playbook

## Table of Contents
1. [Ownership of a problem not your responsibility](#1-ownership-of-a-problem-not-your-responsibility)
2. [Project with significant impact](#2-project-with-significant-impact)
3. [Disagreed with teammate or manager](#3-disagreed-with-teammate-or-manager)
4. [Influencing others without authority](#4-influencing-others-without-authority)
5. [Balancing speed and quality](#5-balancing-speed-and-quality)
6. [Difficult decision under pressure](#6-difficult-decision-under-pressure)
7. [Cross-team collaboration](#7-cross-team-collaboration)
8. [Mentoring and supporting teammates](#8-mentoring-and-supporting-teammates)
9. [Mistake and what you learned](#9-mistake-and-what-you-learned)
10. [Handling conflict within a team](#10-handling-conflict-within-a-team)
11. [Managing multiple priorities](#11-managing-multiple-priorities)
12. [Working with unclear requirements](#12-working-with-unclear-requirements)
13. [Learning something new quickly](#13-learning-something-new-quickly)
14. [Why TikTok (Shop)?](#14-why-tiktok-shop)
15. [Tech Lead Responsibilities](#15-tech-lead-responsibilities)
16. [How to Systematically Approach a Complex Technical Problem](#16-systematically-approach-complex-technical-problem)

---

## 1. Ownership of a problem not your responsibility

Yeah, sure. So basically, during our on-call rotation at Cisco, we often receive internal tickets through BEMS — it’s our internal monitoring and support system. Normally, the issues are split between frontend and backend, but even when it’s not on the frontend side, I still try to take ownership from the user-experience point of view.

One time, after collecting logs and checking the traces, I realized it was actually a backend corner case that hadn’t been handled properly. Since the backend team was running out of bandwidth and couldn’t respond quickly, I went through their code myself to locate the issue and confirm the root cause.

We had a quick sync-up afterward, verified the logic gap, and aligned on the fix plan. Meanwhile, I made sure our frontend handled the error gracefully so users weren’t affected.

And honestly, for me, ownership isn’t about blaming which side caused the problem — it’s about stepping up, collaborating across teams, and doing whatever it takes to make the product and user experience better.

---

## 2. Project with significant impact

Yeah, sure — one project that really stands out to me was a big performance refactor we did for the Webex web client.

I’d already been on the team for a few years, and over time the app had gotten heavier — it was taking almost 10 seconds to load, which was painful for users, especially those in large enterprise accounts.

I decided to take a step back and rethink how we loaded data. Instead of fetching every conversation and message upfront, I proposed a staged, lazy-loading approach — just load the most recent 50 conversations first, and then pull more on demand when users open them.

On the frontend, I added virtualization using React Virtualized so the browser only rendered what’s actually visible, and also cleaned up our Redux logic to avoid redundant calls.

We worked closely with the backend team to add pagination and delta APIs, and tested the improvements locally and with the team using Lighthouse to measure the gains.

After rollout, the load time dropped from roughly 10 seconds to under 3. But more importantly, the whole app just felt faster and smoother. That project really reminded me how much performance impacts user perception — it completely changed how people felt about the product.

---

## 3. Disagreed with teammate or manager

Yeah, actually, one big example was when we had to fix accessibility issues for the Webex web client.

We suddenly got pressure from government compliance — they gave us around six months to make everything accessible, or we’d have to pay a big fine. At that time, there were more than two thousand tickets open.

Our senior manager wanted us to start fixing issues right away in the UI layer to make the ticket count go down. I totally understood the urgency, but I didn’t think that was the best long-term solution.

So I suggested we focus first on our shared component library, Momentum UI, because most of the problems came from those base components. If we fix them once, the same fix applies everywhere.

I showed my manager a quick example — like fixing one button component could close maybe fifty tickets at once. That helped him see the benefit, and he agreed to try that approach.

After we cleaned up the shared library, the tickets started dropping much faster than we expected, and we finished everything on time. It really showed me that sometimes the fastest way to move forward is to fix the foundation first — and when you communicate clearly, people usually get on board pretty quickly.

---

## 4. Influencing others without authority

Yeah, sure — one good example was when I noticed our CI pipeline was wasting a lot of time and resources.

Back then, the pipeline ran ESLint and Stylelint on the build server after every commit. Since everyone was committing multiple times a day, the queue kept growing and it really slowed down merges for the whole team.

I wasn’t a tech lead or anything, but I thought we could move those checks earlier. So I wrote a small script using Husky and lint-staged to run ESLint and Stylelint locally before each commit — basically shifting the lint step left to the developer’s machine instead of the CI server.

I showed it in our team meeting, did a quick demo, and people immediately saw the difference — their commits were faster and no one had to wait for lint errors from the server anymore. The change was super simple, but it made everyone’s workflow smoother and saved a ton of build time overall.

---

## 5. Balancing speed and quality

Yeah, sure — one good example was when we added the lobby experience feature in Webex meetings.

The idea was that participants could wait in a lobby before joining, and hosts could move them in or out during the meeting. It worked perfectly for regular meetings, but when it came to breakout sessions, things got tricky because that part of the system used a completely different set of APIs built by another team in China.

We were under a tight deadline since this feature had to roll out across all platforms — web, desktop, and mobile — at the same time. After looking into the dependency, I realized it would take much longer to fully support breakout sessions. So, to keep the overall release on track, I decided to cut the scope temporarily and hide the lobby feature inside breakout sessions for the first launch.

That decision helped us deliver on time without breaking anything, and later we worked with the China team to extend full support in the following release. It turned out to be the right call — we hit the release date and kept the user experience stable, which was the most important part for me.

---

## 6. Difficult decision under pressure

Yeah, sure — one of the most stressful situations I’ve had was during an on-call shift for the Webex web client.

We had just rolled out a new feature to production, and within minutes we started getting and error alerts — people couldn’t join meetings. The logs showed a spike in failed API calls right after the deployment.

At that moment, we had two options: try a quick hot-fix or roll back the release. The new feature was fairly isolated, but it touched part of the join flow, which made a live fix risky.

After a quick check and analysis, I decided to recommend a full rollback to restore stability first and investigate the root cause afterward.

We rolled back within about 30 minutes, the service went back to normal, and meetings resumed smoothly. Later we found it was a browser compatibility issue — the new feature wasn’t fully supported on certain versions, which caused the failure for some users.

It was a tough call under pressure, but it reinforced for me that stability always comes first — it’s better to roll back quickly, fix it properly, and protect the user experience rather than risk a bigger outage with a live patch.

---

## 7. Cross-team collaboration

Yeah, sure — one project that really stands out was when I worked as a senior front-end engineer on rebuilding our e-commerce platform purchase.webex.com around 2020.

The goal was to redesign the UI and improve performance for SMB users buying Webex products online. We decided to move to Next.js, which gave us faster rendering, better SEO, and more flexibility to support different payment vendors based on user geolocation.

The project involved multiple groups — UX/UI design, backend, and several third-party payment vendors — so collaboration was key.

Besides regular weekly syncs, I set up a shared document wiki to document all API specs and UI designs, and used design reviews and recorded demos to keep both the design and backend teams aligned.

In the end, we launched the new platform on schedule, improved both performance and SEO, and made the purchase flow much smoother and more reliable for users around the world.

It was a great experience showing how strong cross-team collaboration can make a complex rollout successful.

---

## 8. Mentoring and supporting teammates

Yeah, sure — I’ve actually mentored several new engineers who joined our Webex web client team.

Whenever someone new comes in, I usually start by walking them through how our team works — where to find our internal wikis and documents, what tools we use, and how our dev workflow is set up with Jira, GitHub, and etc.

I also help them get their local environment running, including Node, Yarn, and our internal build scripts.

Once they’re set up, I introduce our main tech stack — React, TypeScript, Redux— and explain the learning order so they can pick things up efficiently.

After that, I give them an overview of the project structure and how different modules connect.

When they get comfortable, I usually assign a few small bug-fix tickets first so they can practice making PRs and going through code reviews.

Then we gradually move to small new feature development, and later on, I start involving them in the meetings with cross functional team so they can see the bigger picture of how we operate.

I really enjoy seeing them grow from just learning the codebase to becoming fully independent contributors — and it’s also rewarding because mentoring new teammates makes the whole team stronger.

---

## 9. Mistake and what you learned

Yeah, sure — one time I made a small mistake was during a release for the Webex web client.

Everything passed tests locally, and because we were under time pressure, I decided to force merge and push it out quickly. It looked fine, so I didn’t run the full end-to-end tests.

After we deployed, a small issue showed up that we didn’t catch before. It wasn’t a big problem, but it made me realize that I should’ve added proper E2E test cases for that change — we already had E2E tests in place, I just didn’t update them.

Since then, I always make sure that any new feature or flow gets covered in our existing E2E suite before merging.

It was a small miss, but a good reminder to slow down a bit and always keep testing complete.

---

## 10. Handling conflict within a team

Yeah, sure — this happened when I was working with our UX and UI team on a new feature for the Webex web client.

Most of the Figma designs were made for the desktop app. There wasn’t a web-specific version, so some things looked great in design but didn’t really work on the web.

For example, they wanted a pop-out window just like on desktop. But browsers don’t really support that — if you open a new window, it’s a whole new context, not the same app.

Instead of just saying “we can’t do it,” I explained what browsers can and can’t handle. I showed a quick demo with a floating panel inside the same page, which looked almost the same.

After a few short discussions, we agreed to go with that approach. It kept the design idea but worked better for web. And after that, our communication with the design team got a lot smoother.

---

## 11. Managing multiple priorities

Yeah, sure — when we have multiple projects going on, I think the key is not to rush into coding, but to plan properly from the start.

We usually do a technical breakdown early, create Jira tickets for each task, and add clear estimates. That helps us see the overall scope and plan resources before we start.

I also make sure we leave enough buffer for testing and QA, so we don’t end up scrambling at the end.

When it comes to priorities, I normally look at three things. First, anything that directly impacts users comes first — for example, performance issues or user-facing bugs always take top priority.

Second, I try to unblock other teammates as early as possible. If someone’s waiting for an API or a shared component, I’ll prioritize that work so the team can move faster as a whole.

And third, I handle high-risk or complex tasks earlier in the cycle. Those are the things that can cause delays later, so it’s better to surface them early while there’s still time to adjust.

This approach has worked really well in our projects — it keeps the team aligned, reduces last-minute surprises, and helps us deliver on schedule without burning out.

---

## 12. Working with unclear requirements

Yeah, sure — one good example was when I worked with our UX and UI design team on a new feature for the Webex web client.

In many cases, the design team would give us the main user flow — the happy path — and maybe a few basic error cases. But on the engineering side, we knew the backend could return many different types of errors that weren’t covered in the design.

To avoid confusion later, I usually took a proactive approach. I built a small demo that simulated different error scenarios and showed how each message would appear in the UI. Then I brought that demo to the design team for quick feedback.

If they agreed with the behavior, we kept it; if not, I adjusted it right away based on their comments.

This process helped us clarify things much earlier in the cycle, so we didn’t have to make last-minute changes during rollout or fix things after going live.

It worked really well — we saved time, avoided rework, and the user experience stayed consistent across different edge cases.

---

## 13. Learning something new quickly

Yeah, sure — one good example was when I joined the project to rebuild our e-commerce site purchase.webex.com.

At that time, we decided to move from a traditional React setup to Next.js, so we could support server-side rendering and improve SEO. It was my first time working with Next.js at that scale, so I had to learn it pretty quickly.

I started by reading through the official docs and experimenting with small sample projects to understand the rendering flow — SSR, SSG, and client-side hydration.

Then I worked with another engineer who had used it before to review our setup and deployment process.

Within a couple of weeks, I built the first version of our product page using Next.js, and we later extended it to the whole checkout flow.

The migration went smoothly, and the new setup gave us much faster load times and much better SEO performance.

That experience really taught me how to learn fast under pressure — break things down, prototype quickly, and apply what I learn right away in real projects.

---

## 14. Why TikTok (Shop)?

Yeah, sure — I’m really interested in joining TikTok, especially the TikTok Shop team.

TikTok is a product with hundreds of millions of users. The scale and impact are huge.

And TikTok Shop is one of the key parts of that ecosystem — it’s not just a side feature, it’s a major driver for growth.

Since it launched in the U.S. in 2023, it’s still pretty new. That means a lot of room to grow, a lot of challenges to solve. I really enjoy that kind of environment — fast-moving, full of problems to fix and things to improve.

I have around ten years of front-end experience, mostly building large-scale web apps. I love optimizing performance and improving user experience.

What gives me the most satisfaction is when millions of users can actually see and use what I build. I want them to feel that the product is fast, easy to use, and enjoyable.

For me, TikTok Shop feels like the perfect mix — huge impact, fast growth, and tons of opportunities to make a difference.


## 15. Tech Lead Responsibilities

### Step 1: Requirement Clarification

**🎯 Goal:** Ensure complete understanding of business goals, user scenarios, and success metrics.

**Key Actions:**
- Attend **PRD (Product Requirement Document)** review with PM and Designer.  
- Identify both **functional** and **non-functional** requirements:
  - Core features, flows, edge cases.
  - Performance (e.g., LCP ≤ 2s, TTI ≤ 3s).
  - Accessibility, i18n/l10n, browser compatibility.
- Clarify API dependencies (GraphQL / REST / BFF).
- Identify potential risks or unclear acceptance criteria.
- ✅ **Deliverable:** Requirement Review Notes.

---

### Step 2: Technical Design & Scoping

**🎯 Goal:** Define architecture, data flow, and tech stack before development begins.

**Key Actions:**
- Choose rendering strategy: **CSR / SSR / SSG / ISR**.
- Define **state management** (Redux Toolkit, Recoil, Zustand, etc.).
- Plan performance optimizations: Code-splitting, Lazy-loading, Virtualization.
- Define API contract with backend (OpenAPI / GraphQL Schema / Mock server).
- Review security, auth flow, and caching strategy.
- Conduct a **Design Review** meeting with stakeholders.
- ✅ **Deliverables:**
  - Technical Design Doc (with architecture diagram)
  - Review approval or comments

---

### Step 3: Task Breakdown & Estimation

**🎯 Goal:** Translate design into clear, traceable development tasks.

**Key Actions:**
- Break down into feature modules: UI Components / Hooks / API / Routing / State.
- Estimate time & complexity (T-shirt sizing: S / M / L / XL).
- Create **Jira / Asana / Notion** task board with ownership.
- Sync with PM for sprint planning & milestones.
- ✅ **Deliverables:**
  - Sprint Board
  - Effort Estimation Document

---

### Step 4: Implementation & Code Review

**🎯 Goal:** Deliver high-quality, maintainable, and consistent code.

**Key Actions:**
- Follow Git branching model: `feature/<module-name>`.
- Enforce coding standards (ESLint, Prettier, TypeScript strict mode).
- Write unit tests (Jest, RTL) and E2E tests (Playwright, Cypress).
- Conduct thorough **Code Reviews**:
  - Maintainability and scalability
  - Performance and security
  - Consistency with design and UX
- ✅ **Deliverables:**
  - Merged PR through CI/CD pipeline
  - Review checklist completed

---

### Step 5: Testing & Quality Assurance

**🎯 Goal:** Validate stability, performance, and accessibility before release.

**Key Actions:**
- Run smoke tests, regression tests, and integration tests.
- Measure Lighthouse metrics and fix regressions.
- Perform **A11y testing** (Screen reader, Keyboard navigation).
- Conduct **UAT (User Acceptance Testing)** with PM/QA.
- ✅ **Deliverables:**
  - QA Report / Bug List
  - Accessibility Compliance Report

---

### Step 6: Release & Monitoring

**🎯 Goal:** Deploy safely and monitor stability post-launch.

**Key Actions:**
- Automate build & deploy (GitHub Actions / Jenkins / GitLab CI).
- Manage environments: Dev → Staging → Production.
- Use feature flags, A/B testing, or phased rollout.
- Monitor with Sentry, Datadog, Grafana, or custom dashboards.
- Track real-user performance (RUM) metrics post-deployment.
- ✅ **Deliverables:**
  - Release Notes
  - Monitoring Dashboard / Alert setup

---

### Step 7: Retrospective & Continuous Improvement

**🎯 Goal:** Identify lessons learned and process improvements.

**Key Actions:**
- Conduct a post-release retrospective with team.
- Discuss “What went well / What didn’t / What to improve”.
- Identify automation or documentation gaps.
- Update internal playbooks or templates for future reuse.
- ✅ **Deliverables:**
  - Retrospective Doc
  - Action Items List

---

### 🧠 Tech Lead Responsibilities

| Dimension | Responsibilities |
|------------|------------------|
| **Technical Direction** | Architecture decisions, performance budgets, component reusability |
| **Team Collaboration** | Mentoring, code review quality, conflict resolution |
| **Cross-Function Communication** | Sync with PM, Design, Backend, QA |
| **Quality Control** | Maintain test coverage, performance, and accessibility KPIs |
| **Risk Management** | Detect scope creep, dependency delays, or bottlenecks early |

---

## 15. How to Systematically Approach a Complex Technical Problem

> As a technical leader, you should **analyze before coding**, and **structure uncertainty into solvable components**.

---

### 🧭 Step 1: Problem Framing — Clarify and Define the Scope

**Goal:** Ensure the team is solving the *right problem* before exploring solutions.

**Key Actions:**
- Restate the problem in your own words.
- Identify **business goals** and **technical goals** separately.
- Define **success metrics**: performance, scalability, reliability, maintainability.
- Identify constraints (time, resources, dependencies, legacy systems).
- Clarify what is *in scope* vs *out of scope*.

**Deliverable:**  
✅ Problem Definition Doc / Clarified PRD / Assumption List

> 💡 Ask yourself: “What does success look like?” before asking “How do we build it?”

---

### 🧩 Step 2: Decomposition — Break Down into Subproblems

**Goal:** Turn a large, vague problem into smaller, analyzable units.

**Key Actions:**
- Separate **functional** and **non-functional** aspects.
- Identify major system boundaries: frontend / backend / data / infra / integrations.
- Apply **divide and conquer**:
  - Inputs → Processing → Outputs
  - Data flow vs Control flow
- Visualize with a diagram (sequence / architecture / data flow).

**Deliverable:**  
✅ High-Level Architecture Diagram or Problem Breakdown Tree

> 💡 “You can’t solve what you can’t name.” — Make complexity visible.

---

### 🧮 Step 3: Exploration — Generate Multiple Solution Paths

**Goal:** Don’t commit prematurely; explore trade-offs systematically.

**Key Actions:**
- Brainstorm 2–3 solution strategies (A/B/C).
- For each option, evaluate:
  - Complexity vs scalability
  - Performance vs maintainability
  - Development cost vs business impact
- Involve senior peers for design review / async feedback.

**Deliverable:**  
✅ Design Alternatives Table or RFC Document

> 💡 The best engineers don’t chase “the smartest solution,” but the one that best fits *the context.*

---

### ⚖️ Step 4: Evaluation & Decision — Make the Trade-Off Explicit

**Goal:** Choose and justify a direction with transparent reasoning.

**Key Actions:**
- Quantify: latency, throughput, cost, risk.
- Use decision matrices or scoring tables.
- Consider short-term feasibility vs long-term evolution.
- Record rejected options and why.

**Deliverable:**  
✅ Final Design Doc with Decision Rationale

> 💡 “Decision quality” > “Decision speed.”  
> The clarity of reasoning matters more than the specific choice.

---

### 🔧 Step 5: Implementation Planning — Reduce Unknowns Before Building

**Goal:** Translate design into an actionable, low-risk plan.

**Key Actions:**
- Identify **unknowns / spikes** and validate via quick prototypes.
- Define milestones and integration points.
- Assign owners and clarify dependencies.
- Setup observability: metrics, logs, alerts.

**Deliverable:**  
✅ Implementation Plan / Proof-of-Concept (POC) / Milestone Tracker

> 💡 “Build small, learn fast.” Prototype before full commitment.

---

### 🧪 Step 6: Validation & Feedback — Measure, Learn, Iterate

**Goal:** Verify assumptions early; validate results continuously.

**Key Actions:**
- Measure impact against baseline metrics.
- Conduct peer reviews, load testing, or dry runs.
- Collect feedback from users or stakeholders.
- Adjust the plan if the data disagrees with the hypothesis.

**Deliverable:**  
✅ Metrics Report / Iteration Log / Retrospective Summary

> 💡 “If it’s not measured, it’s just an opinion.”

---

### 🔁 Step 7: Reflection — Capture Lessons & Institutionalize Knowledge

**Goal:** Turn experience into reusable organizational knowledge.

**Key Actions:**
- Document what worked / what didn’t.
- Extract reusable patterns into a **playbook** or **internal wiki**.
- Share learnings in team sync or design-review meeting.
- Update standards or tooling if needed.

**Deliverable:**  
✅ Retrospective Doc / Knowledge Base Entry

> 💡 “A mature tech lead not only solves problems but builds systems that make the next problem easier to solve.”

---

### 🧠 Summary: The Thinking Loop

| Stage | Focus | Mindset |
|--------|--------|----------|
| **1. Frame** | Define the real problem | Think “Why” |
| **2. Decompose** | Expose structure | Think “What” |
| **3. Explore** | Generate possibilities | Think “How (options)” |
| **4. Evaluate** | Make trade-offs | Think “Compare” |
| **5. Implement** | Execute pragmatically | Think “Deliver” |
| **6. Validate** | Measure outcome | Think “Verify” |
| **7. Reflect** | Capture learnings | Think “Improve” |

---

### 💡 Guiding Principles for Technical Leaders

1. **Think in systems, not symptoms.**  
   Avoid fixing what’s visible — understand underlying causes.
2. **Bias to clarity, not speed.**  
   Unclear direction wastes more time than slow progress.
3. **Collaborate early.**  
   Use reviews and async RFCs to derisk decisions.
4. **Make decisions reversible when possible.**  
   Prefer low-cost experiments over high-cost bets.
5. **Document reasoning, not just results.**  
   Future teams inherit your context.

---

