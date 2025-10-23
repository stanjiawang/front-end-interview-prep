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
