# Behavioral Stories & Tech Lead Responsibilities
---

## Table of Contents

1. [Ownership Beyond Your Responsibility](#1-ownership-beyond-your-responsibility)
2. [High-Impact Project – Performance & Scalability](#2-high-impact-project--performance--scalability)
3. [Disagreeing & Influencing Up (Accessibility Strategy)](#3-disagreeing--influencing-up-accessibility-strategy)
4. [Influencing Without Authority – Improving CI Process](#4-influencing-without-authority--improving-ci-process)
5. [Balancing Speed vs Quality & Scope](#5-balancing-speed-vs-quality--scope)
6. [Difficult Decision Under Pressure – Rollback](#6-difficult-decision-under-pressure--rollback)
7. [Cross-Team Collaboration – E-commerce Platform](#7-cross-team-collaboration--e-commerce-platform)
8. [Mentoring & Supporting Teammates](#8-mentoring--supporting-teammates)
9. [Failure & Learning – Missing E2E Coverage](#9-failure--learning--missing-e2e-coverage)
10. [Handling Conflict with UX / Design](#10-handling-conflict-with-ux--design)
11. [Handling Multiple Priorities & Tight Deadlines](#11-handling-multiple-priorities--tight-deadlines)
12. [Working with Limited Information / Unclear Requirements](#12-working-with-limited-information--unclear-requirements)
13. [Learning Quickly Under Pressure – Next.js Migration](#13-learning-quickly-under-pressure--nextjs-migration)
14. [Biggest Personal Growth Area – Communication](#14-biggest-personal-growth-area--communication)
15. [Project You’re Most Proud Of – Webex Web Client](#15-project-youre-most-proud-of--webex-web-client)
16. [Tech Lead Responsibilities – Playbook](#16-tech-lead-responsibilities--playbook)

---

## 1. Ownership Beyond Your Responsibility

**Question:** Tell me about a time you took ownership of a problem that wasn’t your responsibility.

### Quick Talking Points (STAR)

- **Situation:** On-call rotation at Cisco; internal tickets via BEMS, problem initially looked like backend issue.
- **Task:** Ensure user experience was not affected even though it wasn’t a frontend-owned issue.
- **Action:**
  - Collected logs and traces; identified a backend corner case.
  - Read through backend code to locate the issue and confirm root cause.
  - Synced quickly with backend team, aligned on fix plan and logic gap.
  - Updated frontend to handle the error gracefully while backend fix was in progress.
- **Result:**
  - Users were shielded from a bad experience.
  - Backend team had a clear, confirmed root cause and fix path.
  - Demonstrated ownership beyond team boundaries and reinforced cross-team collaboration.

### Story (Long Form)

During our on-call rotation at Cisco, we often receive internal tickets through BEMS — it’s our internal monitoring and support system. Normally, the issues are split between frontend and backend, but even when it’s not on the frontend side, I still try to take ownership from the user-experience point of view.

One time, after collecting logs and checking the traces, I realized it was actually a backend corner case that hadn’t been handled properly. Since the backend team was running out of bandwidth and couldn’t respond quickly, I went through their code myself to locate the issue and confirm the root cause.

I had a quick sync-up afterward, verified the logic gap, and aligned on the fix plan. Meanwhile, I made sure our frontend handled the error gracefully so users weren’t affected.

For me, ownership isn’t about blaming which side caused the problem — it’s about stepping up, collaborating across teams, and doing whatever it takes to make the product and user experience better.

---

## 2. High-Impact Project – Performance & Scalability

**Questions:**  
- Tell me about a project where your contribution had a significant impact on results or users.  
- Tell me about a time you improved performance or scalability.

### Quick Talking Points (STAR)

- **Situation:** Webex web client had grown heavy over time; ~10s load time, especially painful for large enterprise users.
- **Task:** Improve perceived and actual performance without sacrificing functionality.
- **Action:**
  - Proposed staged, lazy-loading data strategy instead of loading all conversations/messages upfront.
  - Loaded only the most recent 50 conversations first; fetched more on demand.
  - Implemented list virtualization with React Virtualized to render only visible items.
  - Cleaned up Redux store to avoid redundant calls.
  - Partnered with backend to add pagination and delta APIs.
  - Measured improvements using Lighthouse and performance tooling.
- **Result:**
  - Reduced load time from ~10 seconds to under 3 seconds.
  - Overall UI felt smoother and more responsive.
  - Improved user satisfaction and perception of Webex performance.

### Story (Long Form)

One project that really stands out to me was a big performance refactor we did for the Webex web client.

I’d already been on the team for a few years, and over time the app had gotten heavier — it was taking almost 10 seconds to load, which was painful for users, especially those in large enterprise accounts.

My proposal was to take a step back and rethink how we should load the data. Instead of fetching every conversation and message upfront, I proposed a staged, lazy-loading approach — just load the most recent 50 conversations first, and then pull more on demand when users open them. On the frontend, I added virtualization using React Virtualized so the browser only rendered what’s actually visible, and also cleaned up our Redux store to avoid redundant calls.

I worked closely with the backend team to add pagination and delta APIs, and tested the improvements locally using Lighthouse to measure the gains.

After rollout, the load time dropped from roughly 10 seconds to under 3. But more importantly, the whole app just felt faster and smoother. That project really reminded me how much performance impacts user perception — it completely changed how people felt about the product.

---

## 3. Disagreeing & Influencing Up (Accessibility Strategy)

**Questions:**  
- Describe a time you disagreed with a teammate or manager. How did you handle it?  
- Tell me about a time you had to influence a decision or change someone’s mind.

### Quick Talking Points (STAR)

- **Situation:** Webex web client faced government accessibility compliance deadline (~6 months, 2000+ QA tickets).
- **Task:** Decide the most effective strategy to close a large number of accessibility issues on time.
- **Action:**
  - Manager wanted to fix issues directly in the UI layers to reduce visible ticket counts fast.
  - Proposed focusing first on the shared component library (Momentum Design) where most issues originated.
  - Demonstrated leverage: fixing one button component could close ~50 tickets at once.
  - Communicated clearly with data and concrete examples rather than just opinion.
- **Result:**
  - Strategy changed to focus on the shared component library first.
  - Ticket count decreased much faster than expected.
  - Met the compliance deadline and avoided fines.
  - Built trust with management by proposing a scalable, long-term solution.

### Story (Long Form)

We suddenly got pressure from government compliance — they gave us around six months to make everything accessible, or we’d have to pay a big fine. At that time, there were more than two thousand tickets raised by the QA team.

Our senior manager wanted us to start fixing issues right away in the UI layer to make the ticket count go down. I totally understood the urgency, but I didn’t think that was the best long-term solution. So I suggested we focus first on our shared component library, Momentum Design, because most of the problems came from those base components. If we fix them once, the same fix applies everywhere.

I showed my manager a quick example — like fixing one button component could close maybe fifty tickets at once. That helped him see the benefit, and he agreed to try that approach.

After we cleaned up the shared library, the tickets started dropping much faster than we expected, and we finished everything on time. It really showed me that sometimes the fastest way to move forward is to fix the foundation first — and when you communicate clearly, people usually get on board pretty quickly.

---

## 4. Influencing Without Authority – Improving CI Process

**Questions:**  
- Tell me about a time you had to influence others without formal authority.  
- Tell me about a time you improved a process or system.

### Quick Talking Points (STAR)

- **Situation:** CI pipeline running ESLint and Stylelint on every commit on the build server; long queues, slow merges.
- **Task:** Reduce wasted CI time and speed up developer feedback loop.
- **Action:**
  - Not a tech lead, but identified opportunity to “shift left” linting.
  - Implemented Husky + lint-staged to run ESLint and Stylelint locally before commit.
  - Prepared a short demo and presented in team meeting.
- **Result:**
  - Lint errors caught locally before hitting CI.
  - Reduced CI queue time and faster merges for everyone.
  - Simple change, but significantly improved daily developer workflow.

### Story (Long Form)

Back then, the pipeline ran ESLint and Stylelint on the build server after every commit. Since everyone was committing multiple times a day, the queue kept growing and it really slowed down merges for the whole team.

I wasn’t a tech lead or anything, but I thought we could move those checks earlier. So I wrote a small script using Husky and lint-staged to run ESLint and Stylelint locally before each commit — basically shifting the lint step left to the developer’s machine instead of the CI server.

I showed it in our team meeting, did a quick demo, and people immediately saw the difference — their commits were faster and no one had to wait for lint errors from the server anymore. The change was super simple, but it made everyone’s workflow smoother and saved a ton of build time overall.

---

## 5. Balancing Speed vs Quality & Scope

**Question:** Tell me about a time you had to balance speed of delivery with quality or maintainability.

### Quick Talking Points (STAR)

- **Situation:** Building lobby experience feature for Webex meetings; special complexity with breakout sessions using different APIs from a team in China.
- **Task:** Deliver a cross-platform feature on a tight deadline without breaking the experience.
- **Action:**
  - Analyzed complexity of supporting lobby inside breakout sessions with external dependencies.
  - Decided to temporarily cut the scope and hide the lobby feature inside breakout sessions for first launch.
  - Coordinated with the team in China to plan full support in a follow-up release.
- **Result:**
  - Delivered on time across platforms (web, desktop, mobile).
  - Kept experience stable and avoided risky half-baked behavior.
  - Later extended feature fully without rushing a fragile solution.

### Story (Long Form)

The idea was that participants could wait in a lobby before joining, and hosts could move them in or out during the meeting. It worked perfectly for regular meetings, but when it came to breakout sessions, things got tricky because that part of the system used a completely different set of APIs built by another team in China.

We were under a tight deadline since this feature had to roll out across all platforms — web, desktop, and mobile — at the same time. After looking into the dependency, I realized it would take much longer to fully support breakout sessions. So, to keep the overall release on track, I decided to cut the scope temporarily and hide the lobby feature inside breakout sessions for the first launch.

That decision helped us deliver on time without breaking anything, and later I worked with the China team to extend full support in the following release. It turned out to be the right call — we hit the release date and kept the user experience stable, which was the most important part for me.

---

## 6. Difficult Decision Under Pressure – Rollback

**Question:** Tell me about a situation where you had to make a difficult decision under pressure.

### Quick Talking Points (STAR)

- **Situation:** New feature rollout for Webex web client; within minutes users couldn’t join meetings; spike in failed API calls.
- **Task:** Quickly restore stability and decide between hot-fix vs rollback.
- **Action:**
  - Analyzed impact and risk; join flow was affected and new feature touched critical path.
  - Chose to perform a full rollback rather than a risky live hot-fix.
  - Coordinated rollback within ~30 minutes and restored service.
  - Later investigated and discovered browser compatibility issue with specific versions.
- **Result:**
  - Minimized outage window and restored normal join behavior.
  - Root cause identified and fixed properly, with improved compatibility checks.
  - Reinforced principle that stability and user experience come first.

### Story (Long Form)

We had just rolled out a new feature to production, and within minutes we started getting error alerts — people couldn’t join meetings. The dashboard showed a spike in failed API calls right after the deployment.

At that moment, we had two options: try a quick hot-fix or roll back the release. The new feature was fairly isolated, but it touched part of the join flow, which made a live fix risky. After a quick check and analysis, I decided to do a full rollback to restore stability first and investigate the root cause afterward.

We rolled back within about 30 minutes, the service went back to normal. Later we found it was a browser compatibility issue — the new feature wasn’t fully supported on certain versions, which caused the failure for some users.

It was a tough call under pressure, but it reminds me that stability always comes first — it’s better to roll back quickly, fix it properly, and protect the user experience rather than risk a bigger outage with a live patch.

---

## 7. Cross-Team Collaboration – E-commerce Platform

**Question:** Tell me about a time you collaborated across teams or functions to achieve a goal.

### Quick Talking Points (STAR)

- **Situation:** Rebuilding purchase.webex.com around 2020 to improve UI and performance for SMB users buying Webex online.
- **Task:** Modernize the platform using Next.js and coordinate across multiple teams and vendors.
- **Action:**
  - Migrated to Next.js for faster rendering and better SEO.
  - Implemented geolocation-based support for different payment vendors.
  - Collaborated with UX/UI, backend, and third-party payment providers.
  - Created a shared wiki with API specs and UI designs.
  - Used design reviews and recorded demos to keep everyone aligned.
- **Result:**
  - Launched new platform on schedule.
  - Improved performance, SEO, and reliability of the purchase flow.
  - Enabled smoother global purchasing experience for users.

### Story (Long Form)

One project that really stands out was when I worked as a senior engineer on rebuilding our e-commerce platform purchase.webex.com around 2020.

The goal was to redesign the UI and improve performance for SMB users buying Webex products online. We decided to move to Next.js, which gave us faster rendering, better SEO, and more flexibility to support different payment vendors based on the users’ geolocation.

The project involved multiple groups — UX/UI design team, backend team, and several third-party payment vendors — so collaboration was key. Besides regular weekly syncs, I set up a shared document wiki to document all API specs and UI designs, and used design reviews and recorded demos to keep both the design and backend teams aligned. 

In the end, we launched the new platform on schedule, improved both performance and SEO, and made the purchase flow much smoother and more reliable for users around the world. It was a great experience showing how strong cross-team collaboration can make a complex rollout successful.

---

## 8. Mentoring & Supporting Teammates

**Question:** Tell me about a time you mentored or supported a teammate’s growth.

### Quick Talking Points (STAR)

- **Situation:** Multiple new engineers joining the Webex web client team.
- **Task:** Help them onboard, ramp up on stack, and grow into independent contributors.
- **Action:**
  - Walked them through team workflow, internal wikis, tools, and dev setup (Node, Yarn, internal scripts).
  - Introduced main tech stack (React, TypeScript, Redux) and recommended learning order.
  - Explained project structure and how modules connect.
  - Started them with small bug fixes → then small features → then cross-functional meetings.
- **Result:**
  - New teammates ramped up faster and became independent contributors.
  - Strengthened team capacity and reduced bus factor.
  - Built a culture of knowledge sharing and support.

### Story (Long Form)

I’ve actually mentored several new engineers who joined our Webex web client team.

Whenever someone new comes in, I usually start by walking them through how our team works — where to find our internal wikis and documents, what tools we use, and how our dev workflow is set up with Jira, GitHub, and etc. I also help them get their local environment running, including Node, Yarn, and our internal build scripts.

Once they’re set up, I introduce our main tech stack — React, TypeScript, Redux— and explain the learning order so they can pick things up efficiently. After that, I give them an overview of the project structure and how different modules connect.

When they get comfortable, I usually assign a few small bug-fix tickets first so they can practice making PRs and going through code reviews. Then we move to small new feature development, and later on, I start involving them in the meetings with cross functional team so they can see the bigger picture of how we operate.

I really enjoy seeing them grow from just learning the codebase to becoming fully independent contributors — and it’s also rewarding because mentoring new teammates makes the whole team stronger.

---

## 9. Failure & Learning – Missing E2E Coverage

**Question:** Tell me about a time you failed or made a mistake. What did you learn from it?

### Quick Talking Points (STAR)

- **Situation:** Release for Webex web client under time pressure.
- **Task:** Ship a change quickly while maintaining quality.
- **Action:**
  - Forced a merge after local tests passed, skipping full end-to-end test run.
  - A small issue appeared in production that E2E tests would have caught.
- **Result & Learning:**
  - Issue was not critical but highlighted a gap in test coverage.
  - Added proper E2E test cases for that flow.
  - Since then, always ensures new features/flows are covered by E2E before merging, even under pressure.

### Story (Long Form)

One time I made a small mistake was during a release for the Webex web client.

Everything passed tests locally, and because we were under time pressure, I decided to force merge and push it out quickly. It looked fine, so I didn’t run the full end-to-end tests.

After we deployed, a small issue showed up that we didn’t catch before. It wasn’t a big problem, but it made me realize that I should’ve added proper E2E test cases for that change.

Since then, I always make sure that any new feature or flow gets covered in our existing E2E suite before merging.

It was a small miss, but a good reminder to slow down a bit and always keep testing complete.

---

## 10. Handling Conflict with UX / Design

**Question:** Tell me about a time you handled conflict or disagreement within a team.

### Quick Talking Points (STAR)

- **Situation:** UX/UI designs were primarily targeting desktop app; web-specific constraints were not considered.
- **Task:** Implement a pop-out window feature that worked for web, despite browser limitations.
- **Action:**
  - Clarified browser constraints: a new window is a separate context, not the same app instance.
  - Instead of just saying “no,” explained limitations and proposed a floating panel within the page.
  - Built a quick demo showing a floating panel that visually matched the design intent.
- **Result:**
  - Agreed on using the in-page floating panel instead of a true pop-out window.
  - Maintained design intent while staying realistic for the web platform.
  - Improved mutual understanding and smoother communication with design team going forward.

### Story (Long Form)

This happened when I was working with our UX and UI team on a new feature for the Webex web client.

Most of the Figma designs were made for the desktop app. There wasn’t a web-specific version, so some things looked great in design but didn’t really work on the web.

For example, they wanted a pop-out window just like on desktop. But browsers don’t really support that — if you open a new window, it’s a whole new context, not the same app.

Instead of just saying “we can’t do it,” I explained what browsers can and can’t handle. I showed a quick demo with a floating panel inside the same page, which looked almost the same.

After a few short discussions, we agreed to go with that approach. It kept the design idea but worked better for web. And after that, our communication with the design team got a lot smoother.

---

## 11. Handling Multiple Priorities & Tight Deadlines

**Question:** Tell me about a time you had to handle multiple priorities or tight deadlines.

### Quick Talking Points (STAR)

- **Situation:** Multiple parallel projects and features in flight.
- **Task:** Deliver on schedule without burning out the team or sacrificing quality.
- **Action:**
  - Performed early technical breakdown and created Jira tickets with clear estimates.
  - Ensured buffer for testing and QA.
  - Prioritized:
    - User-impacting issues (performance, user-facing bugs) first.
    - Unblocking work (shared components, APIs) early to unblock others.
    - High-risk/complex tasks earlier in the cycle to surface problems while time remained.
- **Result:**
  - Teams stayed aligned and focused on the right work.
  - Reduced last-minute surprises and crunch.
  - Delivered projects on schedule with stable quality.

### Story (Long Form)

When we have multiple projects going on, I think the key is not to rush into coding, but to plan properly from the start.

We usually do a technical breakdown early, create Jira tickets for each task, and add clear estimates. That helps us see the overall scope and plan resources before we start. I also make sure we leave enough buffer for testing and QA, so we don’t end up scrambling at the end.

When it comes to priorities, I normally look at three things. First, anything that directly impacts users comes first — for example, performance issues or user-facing bugs always take top priority.

Second, I try to unblock other teammates as early as possible. If someone’s waiting for an API or a shared component, I’ll prioritize that work so the team can move faster as a whole.

And third, I handle high-risk or complex tasks earlier in the cycle. Those are the things that can cause delays later, so it’s better to surface them early while there’s still time to adjust.

This approach has worked really well in our projects — it keeps the team aligned, reduces last-minute surprises, and helps us deliver on schedule without burning out.

---

## 12. Working with Limited Information / Unclear Requirements

**Question:** Tell me about a time you had to work with limited information or unclear requirements.

### Quick Talking Points (STAR)

- **Situation:** UX/UI team provided main flows and a few error cases, but backend could return many more error types.
- **Task:** Ensure consistent, user-friendly error handling despite incomplete specs.
- **Action:**
  - Built a small demo simulating different error scenarios and UI messages.
  - Brought demo to design team for quick feedback and decisions.
  - Iterated on behavior live based on their input.
- **Result:**
  - Clarified error flows much earlier in the cycle.
  - Avoided last-minute changes or follow-up fixes post-release.
  - Achieved consistent and well-thought-out error experience for users.

### Story (Long Form)

One good example was when I worked with our UX and UI design team on a new feature for the Webex web client.

In many cases, the design team would give us the main user flow — the happy path — and maybe a few basic error cases. But on the engineering side, we knew the backend could return many different types of errors that weren’t covered in the design.

To avoid confusion later, I usually took a proactive approach. I built a small demo that simulated different error scenarios and showed how each message would appear in the UI. Then I brought that demo to the design team for quick feedback.

If they agreed with the behavior, we kept it; if not, I adjusted it right away based on their comments. This process helped us clarify things much earlier in the cycle, so we didn’t have to make last-minute changes during rollout or fix things after going live.

It worked really well — we saved time, avoided rework, and the user experience stayed consistent across different edge cases.

---

## 13. Learning Quickly Under Pressure – Next.js Migration

**Question:** Tell me about a time you had to quickly learn something new to complete a task or project.

### Quick Talking Points (STAR)

- **Situation:** Rebuild pricing.webex.com and purchase.webex.com, move from traditional SPA to Next.js for SSR and SEO.
- **Task:** Learn Next.js quickly enough to lead implementation at scale.
- **Action:**
  - Read Next.js docs and built small sample projects to understand SSR/SSG and hydration.
  - Paired with an engineer who had prior Next.js experience to validate setup and deployment.
  - Built first product page in Next.js within a couple of weeks, then extended to checkout flow.
- **Result:**
  - Smooth migration to Next.js.
  - Achieved faster load times and significantly better SEO.
  - Built confidence in learning and applying new frameworks rapidly under real deadlines.

### Story (Long Form)

One good example was when I joined the project to rebuild our e-commerce site pricing.webex.com and purchase.webex.com.

At that time, we decided to move from a traditional SPA to Next.js, so we could support server-side rendering and improve SEO. It was my first time working with Next.js at that scale, so I had to learn it pretty quickly.

I started by reading through the official docs and experimenting with small sample projects to understand the rendering flow — SSR, SSG, and client-side hydration. Then I worked with another engineer who had used it before to review our setup and deployment process.

Within a couple of weeks, I built the first version of our product page using Next.js, and we later extended it to the whole checkout flow. The migration went smoothly, and the new setup gave us much faster load times and much better SEO performance.

That experience really taught me how to learn fast under pressure — break things down, prototype quickly, and apply what I learn right away in real projects.

---

## 14. Biggest Personal Growth Area – Communication

**Question:** Biggest failure / weakness and how you grew from it.

### Quick Talking Points (STAR)

- **Situation:** Early days on Webex team; still new to product, English not first language, quiet in meetings.
- **Task:** Become an effective senior engineer who can represent frontend perspective and influence decisions.
- **Action:**
  - Initially held back opinions during design and API discussions.
  - Over time, gained deeper product and technical understanding.
  - Consciously started speaking up: challenging designs, suggesting alternatives, and explaining trade-offs.
- **Result:**
  - Became more proactive and confident in cross-functional discussions.
  - Built stronger trust with designers and backend engineers.
  - Now regularly leads discussions, guides design decisions, and represents the frontend perspective clearly and constructively.

### Story (Long Form)

If I look back, one of my early challenges — or you could say a failure — was around communication.

When I first joined the Webex team, I didn’t have much experience yet, and I was still getting to know the product and how everything worked. English isn’t my first language, so at the beginning I was a bit quiet in meetings. I wasn’t as confident pushing back on ideas or speaking up, especially when I worked with designers or backend teams on API or feature discussions. I often had opinions but kept them to myself because I didn’t feel ready.

Over time, as I gained more technical depth and a better understanding of the product, I realized that staying quiet doesn’t help the team. When I grew into a senior role, I started to see how important it was to speak up — not to argue, but to collaborate and clarify. In the last few years, I’ve become much more proactive. I now regularly challenge designs or suggest alternative approaches, and I do it in a constructive way — explaining the technical impact and proposing better solutions. I’ve found that being open and direct actually builds trust with cross-functional partners, because people see you care about the quality of the product, not just the code.

So while it started as a weakness, it really helped me grow into someone who can lead discussions, guide design decisions, and represent the front-end perspective more confidently.

---

## 15. Project You’re Most Proud Of – Webex Web Client

**Question:** Tell me about a project you’re most proud of.

### Quick Talking Points (STAR)

- **Situation:** Joined Webex web client from early days when it was a basic messaging app.
- **Task:** Help evolve it into a full collaboration platform with rich features, performance, compliance, and AI capabilities.
- **Action (Highlights):**
  - Built/led core messaging features: reactions, forward, flag, quote, thread replies.
  - Enhanced message composer: attachments, markdown, GIFs, emojis, mentions, AI-powered message rewrite.
  - Expanded meeting capabilities: from 1:1 calls to calendar meetings, PSTN, webinars/webcasts.
  - Helped integrate AI features: real-time translation, meeting summaries, action items.
  - Led work on accessibility and security compliance.
- **Result:**
  - Webex web client grew from small prototype to full enterprise platform used by millions.
  - Strengthened front-end skills in data-heavy interfaces, performance, and UX.
  - A long-term, end-to-end impact story showing growth and ownership over many years.

### Story (Long Form)

One of the projects I’m most proud of is my work on the Webex web client, which I’ve been part of since we started building it.

When we first began, it was a basic messaging app — just simple chat between users. Over the years, I lead and build many of the core features that turned it into a full collaboration platform.

On the messaging side, we added features like message reactions, forward, flag, quote, and thread replies. For the message composer, we built support for attachments, markdown, GIFs, emojis, mentions, and more recently, we introduced AI-powered message rewrite, which helps users rephrase their messages in different tones.

On the meeting side, we started small with 1-on-1 calls and ad-hoc meetings, and later expanded to calendar-integrated meetings, PSTN calling, and large meetings with over 200 participants called webinars and webcasts.

In the last year, we also started integrating AI capabilities into the experience — things like real-time translation, meeting summaries, and action item to improve the after meeting experience.

Beyond features, I also lead the work for compliance, like accessibility, and security — align our UI with accessibility standards, align with security policies.

It’s been a really rewarding project because I see the Webex web client grow from a small prototype into a full-scale enterprise platform used by millions of people every day. And I think working on these data-heavy interfaces with a strong focus on user experience has really sharpened my front-end skills.

---

## 16. Tech Lead Responsibilities – Playbook

Below is a structured playbook describing how you approach tech lead responsibilities. This can be used directly in interviews as “how I lead projects end-to-end.”

### Step 1: Requirement Clarification

🎯 **Goal:** Ensure complete understanding of business goals, user scenarios, and success metrics.

**Key Actions:**

- Attend PRD (Product Requirement Document) review with PM and Designer.
- Identify both **functional** and **non-functional** requirements:
  - Core features, flows, edge cases.
  - Performance (e.g., LCP ≤ 2s, TTI ≤ 3s).
  - Accessibility, i18n/l10n, browser compatibility.
- Clarify API dependencies (GraphQL / REST / BFF).
- Identify potential risks or unclear acceptance criteria.

✅ **Deliverable:** Requirement Review Notes.

---

### Step 2: Technical Design & Scoping

🎯 **Goal:** Define architecture, data flow, and tech stack before development begins.

**Key Actions:**

- Choose rendering strategy: CSR / SSR / SSG / ISR.
- Define state management (Redux Toolkit, Recoil, Zustand, etc.).
- Plan performance optimizations: code-splitting, lazy-loading, virtualization.
- Define API contract with backend (OpenAPI / GraphQL Schema / Mock server).
- Review security, auth flow, and caching strategy.
- Conduct a Design Review meeting with stakeholders.

✅ **Deliverables:**

- Technical Design Doc (with architecture diagram).
- Review approval or comments.

---

### Step 3: Task Breakdown & Estimation

🎯 **Goal:** Translate design into clear, traceable development tasks.

**Key Actions:**

- Break down into feature modules: UI Components / Hooks / API / Routing / State.
- Estimate time & complexity (T-shirt sizing: S / M / L / XL).
- Create Jira / Asana / Notion task board with ownership.
- Sync with PM for sprint planning & milestones.

✅ **Deliverables:**

- Sprint Board.
- Effort Estimation Document.

---

### Step 4: Implementation & Code Review

🎯 **Goal:** Deliver high-quality, maintainable, and consistent code.

**Key Actions:**

- Follow Git branching model: `feature/<module-name>`.
- Enforce coding standards (ESLint, Prettier, TypeScript strict mode).
- Write unit tests (Jest, RTL) and E2E tests (Playwright, Cypress).
- Conduct thorough Code Reviews focusing on:
  - Maintainability and scalability.
  - Performance and security.
  - Consistency with design and UX.

✅ **Deliverables:**

- Merged PRs through CI/CD pipeline.
- Review checklist completed.

---

### Step 5: Testing & Quality Assurance

🎯 **Goal:** Validate stability, performance, and accessibility before release.

**Key Actions:**

- Run smoke tests, regression tests, and integration tests.
- Measure Lighthouse metrics and fix regressions.
- Perform A11y testing (screen reader, keyboard navigation).
- Conduct UAT (User Acceptance Testing) with PM/QA.

✅ **Deliverables:**

- QA Report / Bug List.
- Accessibility Compliance Report.

---

### Step 6: Release & Monitoring

🎯 **Goal:** Deploy safely and monitor stability post-launch.

**Key Actions:**

- Automate build & deploy (GitHub Actions / Jenkins / GitLab CI).
- Manage environments: Dev → Staging → Production.
- Use feature flags, A/B testing, or phased rollout.
- Monitor with Sentry, Datadog, Grafana, or custom dashboards.
- Track real-user performance (RUM) metrics post-deployment.

✅ **Deliverables:**

- Release Notes.
- Monitoring Dashboard / Alert setup.

---

### Step 7: Retrospective & Continuous Improvement

🎯 **Goal:** Identify lessons learned and process improvements.

**Key Actions:**

- Conduct a post-release retrospective with the team.
- Discuss “What went well / What didn’t / What to improve.”
- Identify automation or documentation gaps.
- Update internal playbooks or templates for future reuse.

✅ **Deliverables:**

- Retrospective Doc.
- Action Items List.

---

### 🧠 Tech Lead Responsibilities – Guiding Principles

- **Think in systems, not symptoms.**  
  Avoid only fixing what’s visible — get to underlying causes.

- **Bias to clarity, not just speed.**  
  Unclear direction wastes more time than slow progress.

- **Collaborate early.**  
  Use reviews and async RFCs to derisk decisions.

- **Prefer reversible decisions.**  
  Use low-cost experiments instead of high-cost bets when possible.

- **Document reasoning, not just results.**  
  Future teams inherit your context; good documentation multiplies impact.
