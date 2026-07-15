---
---

## Track Structure (13 sections, 124 files)

| # | Section | Scope |
|---|---------|-------|
| 01 | Introduction | EM vs TL vs IC, People/Product/Process pillars |
| 02 | Key Focus Areas | Quality/process, technical strategy, foundational knowledge |
| 03 | Team Development | Hiring, structure, evaluations, career, mentoring |
| 04 | Leadership Skills | Delegation, conflict, feedback, EQ, communication, stakeholders |
| 05 | Execution | Agile, milestones, scope, planning, project management |
| 06 | Strategic Thinking | Product alignment, KPIs, culture, business acumen, ROI |
| 07 | Organizational Awareness | Company culture, change mgmt, structure, politics |
| 08 | Team Culture | Values, inclusion, rituals, recognition, bias |
| 09 | Engineering Culture | Innovation, learning, knowledge sharing, excellence, postmortems |
| 10 | Crisis Management | Team support, risk mitigation, incident response |
| 11 | Stakeholder Mgmt | Documentation, knowledge transfer, partners |
| 12 | Change Management | Technical, customer, executive, team, organizational |
| 13 | Monitoring | System monitoring & performance |

## Roles

| Dimension | IC | Tech Lead | EM |
|-----------|-----|-----------|-----|
| Output | Code, design docs | Architecture, alignment | Teams, careers, process |
| Tech Depth | Deepest | High (guiding) | Strategic (broad) |
| Time Horizon | Sprint | Quarter | Year/Career |
| Maker Time | ~80% | ~50% | ~10% |

## The 3 Pillars

| Pillar | Essence |
|--------|---------|
| People | Build/grow high-performing teams — hiring, mentoring, psych safety, 1:1s |
| Product | Bridge tech to business — EM-PM partnership, WSJF, tech debt framing |
| Process | Organize work — Agile (Scrum/Kanban), CI/CD, DORA metrics, QA |

## Core Frameworks

| Domain | Models & Tools |
|--------|---------------|
| Monitoring | Golden Signals (Latency/Traffic/Errors/Saturation), SLI/SLO/SLA, error budgets |
| CI/CD | Build Once Deploy Many, Blue/Green, Canary, DORA metrics |
| Branching | Trunk-Based (preferred), GitHub Flow, Gitflow (legacy/scheduled releases) |
| Testing | Pyramid: 70% unit + 20% integration + 10% E2E; flaky tests = kill immediately |
| Security | Shift left, SAST/DAST, PoLP, OWASP top 10, secrets mgmt (Vault) |
| Incident | War rooms, blameless postmortems, MTTR/MTTD/MTTA, Incident Commander role |
| Roadmapping | RICE score, MoSCoW, Now/Next/Later horizons |
| Architecture | ADRs, decision matrix, consensus vs consultative, CAP theorem |
| Build vs Buy | TCO analysis, core (build) vs context (buy), opportunity cost |
| Risk | Probability × Impact matrix, bus factor, SPOF, Strangler Fig for legacy |
| Scaling | Vertical (up) vs horizontal (out), cattle vs pets, caching, sharding |
| Hiring | Structured interviews, rubrics, bar raiser, culture add over fit |
| Teams | Conway's Law, Team Topologies (stream-aligned/platform/enabling/complicated), Two-Pizza |
| Perf Eval | 9-box grid, 360 feedback, PIPs, self-review vs manager calibration |
| Career Dev | Dual track (IC/Manager), IDPs, 70-20-10, sponsorship > mentorship |
| Mentoring | GROW model, situational leadership (S1:Directing → S4:Delegating) |
| Delegation | Match skill to challenge, outcomes not methods, Eisenhower matrix |
| Conflict | Thomas-Kilmann (collaborating/compromising/accommodating/competing/avoiding) |
| Feedback | SBI (Situation-Behavior-Impact), Radical Candor |
| Motivation | Autonomy + Mastery + Purpose (Pink), Herzberg hygiene factors |
| Trust | (Credibility + Reliability + Intimacy) / Self-Orientation |
| EQ | Self-awareness, self-regulation, empathy, relationship management |
| 1:1 Meetings | Employee's time (not status updates), weekly, never cancel |
| Meetings | Cost = Σ(hourly rate × duration), no-agenda-no-attendance rule |
| Stakeholder | Power/Interest grid: manage closely / keep satisfied / keep informed / monitor |
| Agile | Scrum (time-boxed sprints), Kanban (flow + WIP limits), XP (TDD + pairing) |
| Status | RAG (Red/Amber/Green), audience-tailored, no watermelon reports |
| KPIs | DORA (DF, Lead Time, CFR, MTTR), SPACE, team health |
| Culture | Psychological safety (#1 predictor), blameless postmortems, innovation fostering |
| Crisis | Contingency → DR → business continuity → incident response → post-incident analysis |
| Change | Organizational (ADKAR), team (reorgs/mergers), technical (migration/strangler fig) |

## Key Comparisons

| Trade-off | Option A | Option B |
|-----------|----------|----------|
| Delivery model | Scrum (time-boxed, predictable) | Kanban (flow-based, continuous) |
| Architecture | Monolith (early-stage, fast to build) | Microservices (scale, independent deploy) |
| Scaling | Vertical (simpler, ceiling exists) | Horizontal (complex, infinite) |
| Tech debt | Prudent (deliberate, tracked) | Reckless (untracked, crippling) |
| Team org | Cross-functional (speed, T-shape) | Functional/siloed (efficiency, handoffs) |
| Decision | Consensus (slow, high buy-in) | Consultative (fast, clear accountability) |
| Feedback | SBI: specific, timely, actionable | Vague: ignored, rejected |
| Motivation | Intrinsic (autonomy, mastery, purpose) | Extrinsic (bonuses, titles) — short-lived |

## Crisis & Change Management

| Phase | Crisis | Change |
|-------|--------|--------|
| Prepare | Risk assessment, contingency, DR plans | Impact assessment, stakeholder mapping |
| Execute | Incident command, war room, stakeholder comms | Communication plan, training, adoption support |
| Recover | Service restoration, post-incident analysis | Reinforcement, resistance management, sustainment |

## Rules

1. 1:1s belong to employee — never cancel, never status-update
2. No surprises — bad news delivered early and personally
3. Hiring for culture add, not culture fit — structured rubrics over gut
4. Feedback must be SBI: Situation, Behavior, Impact
5. Toxic high-performers destroy team output — values are non-negotiable
6. Tech debt is business risk — frame in dollars, latency, retention
7. Pipelines fail fast — lint → unit → integration → security scan
8. Incidents are inevitable — blameless postmortems mandatory
9. Delegation = matching skill to challenge; delegate outcomes, not methods
10. Trust = (Credibility + Reliability + Intimacy) / Self-Orientation
