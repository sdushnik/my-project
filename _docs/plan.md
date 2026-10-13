# Family Habit Tracker — MVP Scope

**Document status:** Proposed MVP scope, ready to use as the baseline for design and development  
**Product:** Responsive browser-based web app for families and households

## 1. Product summary

Build a simple family habit tracker where household members use individual accounts to track personal habits and shared routines. Members record their own progress, earn points and badges, and participate in cooperative or competitive challenges based on existing shared habits.

**Product principle:** prioritize habit tracking, household accountability, privacy, and lightweight gamification. Avoid turning the MVP into a complex productivity tool or social network.

## 2. Target users and core concept

- **Primary audience:** families and households.
- **Account model:** each person has their own account and joins a household.
- **Habit model:** supports both individual habits and shared household habits.
- **Platform:** responsive web app that works in desktop and mobile browsers.
- **Primary landing page after sign-in:** a combined dashboard showing today's personal habits alongside household progress, points, badges, and active challenges.

## 3. Decisions selected during brainstorming

| Area | Selected decision |
|---|---|
| Target audience | Family or household |
| Habit types | Both personal habits and shared household habits |
| Tracking methods | Simple check-offs and measurable numeric targets |
| Habit schedules | Flexible: daily, selected weekdays, or a frequency target such as 3 times per week |
| Progress visibility | Configurable for each habit |
| Account access | Individual accounts that join a household |
| Reminders | One daily reminder time, shown in-app only |
| Progress and motivation | Points, badges, and family challenges |
| Challenge types | Both cooperative goals and friendly competition |
| Habit management | Members manage personal habits and can create/manage shared habits and challenges |
| Platform | Browser-based web app for phones and computers |
| Shared-habit completion | Each member records their own completion |
| Past check-ins | Allow check-ins for the past 7 days |
| Points | Fixed points per completed habit occurrence |
| Measurable targets | Full target required to count as completed and earn points; partial progress may be recorded |
| Leaderboard | Weekly rankings reset every week |
| Household joining | Shared invitation link |
| Roles | One household owner manages membership; everyone can manage habits and challenges |
| Authentication | Email/password and Google sign-in |
| Home screen | Combined personal checklist and household dashboard |
| Challenge source | Challenges use existing habits |
| Challenge duration | Creator chooses start and end dates |

## 4. MVP feature scope

### 4.1 Accounts and household membership

- Support sign-up and sign-in with email/password and Google.
- A user can create a household and become its owner.
- Members join using an invitation link and their own account.
- The owner can create/revoke invitation links, manage household settings, and remove members.
- All members can create and manage their own personal habits, and can create and manage shared habits and challenges.
- In the MVP, each account belongs to one household at a time.
- Invitation links expire after 7 days. The owner can revoke a link before expiration.

**Why this choice:** individual accounts provide personal ownership and privacy, while a link makes inviting relatives simple. A short expiry limits the risk of old links staying usable indefinitely.

**Alternatives considered:** one shared household account, household PIN, permanent invite links, email-only invitations, or multiple households per account. These were not selected because they either reduce accountability/privacy or add extra membership complexity.

### 4.2 Habits

Each habit supports:

- Name and optional description.
- Owner/member association.
- Type: **check-off** or **measurable target**.
- Schedule: every day, selected weekdays, or a weekly frequency target.
- Visibility: private to the creator or visible to household members. Shared habits are visible to the household.
- Active, paused, or archived status.

Personal habits are controlled by the member who created them. Shared habits can be created and managed by household members.

**Why this choice:** the two habit types cover both binary routines (for example, read today) and quantity-based goals (for example, drink eight glasses of water) without needing a full habit-formula builder.

**Alternatives considered:** check-offs only, numbers only, or complex schedule/target rules. Both check-offs and measurable targets are included; more advanced scheduling remains out of scope.

### 4.3 Check-ins and progress recording

- Each scheduled habit occurrence can be checked off or have an amount recorded.
- For a measurable habit, the member may record partial progress during the day.
- A measurable habit is only completed once its full target is reached.
- One habit occurrence can award points only once.
- Members can add or correct check-ins for today and the previous 7 days.
- Corrections recalculate points and relevant challenge totals.
- Use the household timezone for day/week boundaries.
- Archived habits retain history so past dashboard and score records remain interpretable.

**Why this choice:** a 7-day correction window supports realistic catch-up while keeping the risk of old records changing current results manageable.

**Alternatives considered:** same-day-only check-ins or unrestricted edits to all past dates. The 7-day limit is a compromise between convenience and score integrity.

### 4.4 Privacy

- Each habit has an explicit visibility setting.
- Private habit names, values, check-ins, and completion history must not appear on household dashboards, leaderboards, or shared challenge results.
- Shared habit progress is visible to household members.
- Challenge participation and scoring use only the selected shared habits.
- Enforce permissions at the data-access/API layer, not merely by hiding elements in the user interface.

**Why this choice:** habit tracking can include sensitive personal routines. Per-habit privacy allows a family to coordinate without making every personal detail public.

**Alternatives considered:** all-progress-visible or a single privacy rule for all habits. Per-habit visibility offers useful flexibility without requiring a complex permissions system.

### 4.5 Points

- Every completed habit occurrence earns **10 points**.
- All habits use the same point value in the MVP.
- For measurable habits, points are earned only when the full target is reached.
- Partial progress can be recorded but does not earn points.
- A completion cannot earn points more than once. If a check-in is corrected, the score is recalculated.
- Private habits do not contribute to the shared household leaderboard or challenge results.

**Why this choice:** fixed points are transparent and easy to implement, explain, and test.

**Alternatives considered:** variable points based on difficulty, points proportional to target progress, or configurable values for each habit. These introduce scoring disputes and balancing work, so they are deferred.

### 4.6 Badges

Include four basic badges:

1. **First Step:** complete the first habit check-in.
2. **Getting Consistent:** complete 10 check-ins.
3. **Habit Builder:** complete 50 check-ins.
4. **Family Win:** the household completes a cooperative challenge.

Badge criteria should be calculated from valid, non-duplicate check-ins. If a check-in is later corrected, badge eligibility should be recalculated consistently; already-earned badges should not become misleading if their criteria are no longer met.

**Why this choice:** a small, understandable set provides immediate feedback without requiring a badge editor or elaborate achievement system.

**Alternatives considered:** streak levels, seasonal badges, custom badges, and progression trees. Those are deferred.

### 4.7 Weekly leaderboard

- Show a household leaderboard for the current Monday–Sunday week.
- Reset weekly rankings each Monday based on the household timezone.
- Use points earned from completed shared/visible-to-household habit occurrences; do not reveal points from private habits.
- A member with no eligible points can still appear with zero points.

**Why this choice:** weekly resets give members a fresh start and avoid permanently insurmountable lifetime rankings.

**Alternatives considered:** monthly or lifetime rankings, or showing weekly and lifetime totals together. The MVP uses weekly ranking only to keep the experience focused.

### 4.8 Challenges

Challenges use existing shared habits rather than creating a second habit system.

Each challenge has:
- A name.
- Type: competitive or cooperative.
- One or more selected shared habits.
- A start date and an end date.
- A creator and eligible household members.
- A results view.

**Competitive challenge rules**
- Rank participants by the number of completed check-ins for the selected habits during the challenge period.
- Ties share the win.

**Cooperative challenge rules**
- Calculate the number of scheduled check-ins for the selected habits over the challenge period.
- The household wins if it completes at least **80% of those scheduled check-ins**.

**Result finalization**
- Allow corrections within the existing 7-day backfill window after the relevant check-in date, including after the challenge end date.
- Finalize challenge results once all eligible check-ins are outside that correction window. Show that results are provisional before then.

**Why this choice:** competitive raw counts are simple to understand when participants work from the same selected shared habits. An 80% cooperative threshold encourages teamwork without requiring a custom target editor.

**Alternatives considered:** completion-rate rankings, custom numerical challenge targets, or open-ended challenges. These require more configuration and additional edge-case rules.

### 4.9 Dashboard

After sign-in, show a combined dashboard containing:

- Today's personal habit checklist.
- Today's visible shared habits and the current member's own progress on them.
- Household progress summary.
- Weekly points/leaderboard.
- Earned badges and recent badge progress.
- Active challenges and their progress.
- Reminder state or reminder prompt when relevant.

Do not expose private habit details in household sections.

**Why this choice:** a combined dashboard supports the main daily action—recording habits—while keeping family motivation visible without making users navigate through many separate pages.

**Alternative considered:** opening directly to the family dashboard or to only today's checklist. The combined view balances individual action and shared accountability.

### 4.10 Daily in-app reminders

- Each member can choose one daily reminder time.
- If the app is open at that time, show an in-app reminder for unfinished habits.
- If the app is closed, show the reminder the next time the member opens the app, provided unfinished habits remain.
- No email reminders or browser push notifications in the MVP.

**Important limitation:** an in-app-only reminder cannot reliably notify a person while the browser/app is closed.

**Why this choice:** it honors the preference for simple, non-intrusive reminders and avoids notification permission flows.

**Alternatives considered:** email and browser push notifications. Those reach users outside the app but add delivery infrastructure, permissions, and testing complexity.

## 5. Remaining implementation defaults selected for the MVP

| Topic | MVP default | Why | Alternatives considered |
|---|---|---|---|
| Habit points | 10 points per completed occurrence | Easy to explain and test | Difficulty-based, variable, or user-configured scoring |
| Week boundary | Monday to Sunday, household timezone | Common, consistent week definition | Sunday start or rolling 7-day windows |
| Cooperative challenge goal | 80% of scheduled check-ins | Encourages progress without requiring perfection | 100%, a fixed count, or a custom goal |
| Competitive challenge winner | Most completed check-ins; ties share the win | Simple and transparent | Completion percentage or weighted scoring |
| Challenge result status | Provisional until corrections are no longer possible | Avoids locking inaccurate scores | Immediate finalization or unlimited changes |
| Invitation expiry | 7 days; owner can revoke | Limits stale-link risk | Permanent link or email invitation only |
| Household membership | One household per account in MVP | Simplifies permissions and data model | Multiple household memberships |
| Archived habits | Keep history; stop future scheduling | Preserves historical results | Permanent deletion |
| Challenge eligible habits | Shared habits only | Keeps results visible and auditable to participants | Include private habits, which would compromise privacy |

## 6. Out of scope for the MVP

Do not build these in the first release:

- Dedicated iOS or Android apps.
- Email or browser push notifications.
- Advanced analytics, custom reports, or elaborate calendar visualizations.
- Difficulty multipliers or configurable points.
- Challenge goals unrelated to existing shared habits.
- Multiple households per account.
- Custom badge creation, levels, or complex streak systems.
- Detailed social feeds, messaging, comments, or public profiles.
- Habit recommendations or AI-generated routines.

These may be considered later based on feedback from real households.

## 7. MVP completion checklist

The MVP is ready for release when a family can complete this end-to-end journey:

- [ ] A person creates a household and becomes its owner.
- [ ] The owner creates an invitation link and another person joins with an individual account.
- [ ] Members can sign up and sign in with email/password or Google.
- [ ] Members can create personal habits and shared habits.
- [ ] Habits support check-off and measurable target modes.
- [ ] Habits support daily, selected-day, and weekly-frequency schedules.
- [ ] Members can choose habit visibility and private data is not exposed to household members.
- [ ] Each member records their own completion of shared habits.
- [ ] Measurable habits support partial progress but only earn points at the full target.
- [ ] Each valid completed occurrence awards 10 points once, with corrections recalculated correctly.
- [ ] Members can record or correct check-ins for today and the previous 7 days.
- [ ] The dashboard shows today's personal habits plus household progress, points, badges, and active challenges.
- [ ] Weekly rankings reset on Monday in the household timezone and exclude private habits.
- [ ] All four badges are awarded according to their criteria.
- [ ] Members can create competitive and cooperative challenges using existing shared habits with start/end dates.
- [ ] Competitive ties share the win; cooperative challenges use the 80% threshold.
- [ ] Challenge results remain provisional while eligible check-ins can still be corrected.
- [ ] A member can choose a daily in-app reminder time and understand that it will not notify them while the app is closed.
- [ ] The owner can manage membership and household settings; members can manage habits and challenges.
- [ ] Archived habits preserve historical records.
- [ ] The responsive web app works on desktop and mobile browsers.

## 8. Suggested user-facing navigation

Keep the first version's navigation small:

1. **Home** — today's checklist and household summary.
2. **Habits** — create, edit, pause, archive, and view habits.
3. **Challenges** — active, upcoming, and completed challenges.
4. **Household** — members, invitation link, weekly leaderboard, and household settings.
5. **Profile & settings** — sign-in account details, timezone/reminder settings, and personal preferences.

This navigation is a recommended implementation detail, not a requirement to add more feature scope.

## 9. Core data entities (initial proposal)

A developer can use this as a starting point for the data model:

- **User:** identity, authentication provider references, profile name, household membership.
- **Household:** name, owner, timezone, settings, created date.
- **Invitation:** household, token/hash, creator, expiry, revoked/used state.
- **Habit:** household, creator, name, description, scope (personal/shared), visibility, tracking type, target/unit if measurable, schedule, status.
- **Check-in:** habit, member, scheduled occurrence date, recorded value, completion state, created/updated timestamps.
- **Point ledger entry or computed score:** member, source check-in, points, scoring week; design this to prevent duplicate awards and support corrections.
- **Badge definition / user badge:** badge criterion and whether/how a member has earned it.
- **Challenge:** household, creator, type, selected shared habits, start/end dates, status.
- **Reminder preference:** member, selected daily time, timezone handling.

Use stable IDs and explicit timestamps. Keep authorization checks server-side. Exact schema and implementation technology remain to be selected during technical planning.

## 10. Important edge cases to test

- A member tries to check in the same habit occurrence more than once.
- A member edits a completed check-in to an incomplete state; points and challenge scores update.
- A measurable target is partly complete, exactly complete, and exceeded.
- A weekly-frequency habit is completed fewer, exactly, or more times than its target.
- A week or challenge boundary crosses midnight in the household timezone.
- A private habit is requested by another member through a dashboard or API.
- A habit is archived while a challenge or historical score references it.
- An invitation link is expired, revoked, or used by an already-joined account.
- A household owner changes or is removed; the system must prevent a household from being left without an owner.
- A challenge has no eligible scheduled check-ins; it should not produce a misleading 0% or automatic win. Mark it invalid/no result and explain why.
- A challenge's selected habit is archived during the challenge; preserve the existing challenge history and stop creating future occurrences as appropriate.
- A reminder time occurs while the app is closed; show the unfinished-habits reminder on the next app visit rather than claiming an alert was delivered at the chosen time.

## 11. Recommended build order

1. Authentication, household creation, invitations, roles, and data-access/privacy rules.
2. Habit CRUD, scheduling, check-ins, measurable progress, and 7-day corrections.
3. Points calculation, dashboard, and weekly leaderboard.
4. Badges and cooperative/competitive challenges.
5. In-app reminders, responsive polish, accessibility, and end-to-end testing.

Build and test the core tracking and privacy rules before adding gamification. Correct and trustworthy records matter more than decorative rewards.

## 12. Product scope freeze

Treat this document as the initial MVP baseline. Avoid adding new features until the release checklist passes with a small test household. After launch, prioritize changes using feedback from real families rather than assumptions.

**Follow-on artifacts:** [_docs/tech-stack.md](./tech-stack.md) (Option A locked) and [_docs/spec.md](./spec.md) (user stories, acceptance criteria, first schema/API).
