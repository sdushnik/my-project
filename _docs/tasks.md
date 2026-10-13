# MVP task backlog

Product rules: [_docs/plan.md](./plan.md). Implementation rules: [_docs/spec.md](./spec.md). Stack: [_docs/tech-stack.md](./tech-stack.md).

Numbering is the suggested order. Each task is written so it can be handed to someone who has not read the other tasks. Do not add features that are out of the MVP. Do not start application code from this file until a task is explicitly taken.

## 1. Scaffold an empty project with a passing test
Goal: Create a Next.js TypeScript App Router repo whose test suite runs and passes with one dummy test.
Description: Use the locked stack from `_docs/tech-stack.md`: Next.js App Router and TypeScript only. Add a unit test runner (Vitest is fine) and a single test that asserts a trivial true condition so `npm test` (or the equivalent script) is green. No database, auth, UI kit, or product pages yet.

## 2. Add Tailwind, shadcn/ui, and a signed-out layout
Goal: Make the empty app render a minimal responsive shell with Tailwind and shadcn/ui.
Description: Install Tailwind and shadcn/ui as specified in `_docs/tech-stack.md`. Add a simple root layout that looks acceptable on a phone-width and desktop-width viewport, plus placeholder routes matching `_docs/spec.md` section 2 (`/`, `/habits`, `/challenges`, `/household`, `/settings`, `/sign-in`). No real data or authentication behavior.

## 3. Connect PostgreSQL with Drizzle and a first migration
Goal: Run the app against PostgreSQL through Drizzle with a documented migrate command.
Description: Add Drizzle, a `.env.example` for a Postgres URL (Neon or local), and a migration workflow. Ship one migration that can be as small as a `schema_migrations` proof or a health table; the point is a repeatable migrate + query path. Do not implement product tables beyond what you need to prove the connection.

## 4. Create Auth.js email/password and Google sign-in
Goal: Let a person create an account and sign in with email/password or Google.
Description: Implement Auth.js per `_docs/spec.md` story A1 and `_docs/tech-stack.md`. Email/password and Google must create or attach a single `users` row; duplicate email must not create a second account. Persist sessions with the Auth.js Drizzle adapter and protect later app routes by redirecting signed-out users to `/sign-in`.

## 5. Build sign-in, sign-up, and signed-out redirect pages
Goal: Provide usable `/sign-in` and `/sign-up` pages that complete Auth.js flows.
Description: Wire the UI for email/password and a Google button, including safe error copy for invalid credentials (do not leak extra account existence info beyond Auth.js defaults). After a successful sign-in, send users with no household to a prompt to create a household or use an invite, as in story A1. Keep styling on Tailwind/shadcn; no household APIs in this task unless a stub is required to redirect.

## 6. Let a user create a household and become owner
Goal: A signed-in user with no household can create one and become its only member and owner.
Description: Implement `_docs/spec.md` story A2: household name, required IANA timezone, `owner_user_id` = creator, membership row for the creator. Reject a second household for a user who already belongs to one. Cover this with tests around the unique membership rule and owner assignment.

## 7. Let the owner create and revoke invitation links
Goal: The household owner can mint a 7-day join link and revoke it; only a token hash is stored.
Description: Implement story A3: high-entropy token shown once, `token_hash` stored, expiry 7 days, owner-only create/revoke. Non-owners must not create, revoke, or read the raw token after creation. Creating a new link does not have to invalidate older ones unless they are revoked or expired.

## 8. Let a user join a household from an invite token
Goal: A signed-in user with no household can join via `/invite/[token]` when the invite is valid.
Description: Implement story A4. Expired, revoked, unknown, or already-used tokens fail with a clear error; a user already in a household cannot join another; the joiner becomes a member, not owner. Join must fail if the household has no owner. Hash the presented token and look it up; never store plaintext.

## 9. Let the owner manage membership and household settings
Goal: The owner can rename the household, change timezone, and remove members without leaving the household ownerless.
Description: Implement story A5. The owner cannot remove themselves if that would leave no owner; owner transfer is out of MVP. Removed members lose access immediately; keep any historical rows they already have. Timezone changes affect future day/week boundaries only; existing `occurrence_date` values stay as stored calendar dates.

## 10. Implement personal habit create and edit
Goal: A household member can create and edit their own personal habits with the MVP fields.
Description: Implement story B1 and the `habits` columns in `_docs/spec.md` section 4. Support name, optional description, check-off vs measurable (target > 0 and optional unit), schedule (daily, selected weekdays, or weekly frequency 1–7), visibility private or household, status active. Only the creator manages a personal habit. Validate with Zod on the server.

## 11. Implement shared habits, pause, and archive
Goal: Members can create shared household habits and pause or archive habits they are allowed to manage.
Description: Implement stories B2 and B4. Shared habits are always household-visible; any member may edit, pause, or archive them. Paused and archived habits create no new expected occurrences and keep history; archived habits are not hard-deleted and cannot be added to new challenges. Personal pause/archive remains limited to the creator.

## 12. Enforce habit privacy in server queries
Goal: Private habit data never appears in another member’s API responses or dashboard payloads.
Description: Implement story B3 and F3. Filter on the server: private names, values, check-ins, and points are visible only to the creator. Requests for another member’s private habit by id should return 404. UI hiding is not sufficient; add tests that a second household member cannot read private rows.

## 13. Implement occurrence scheduling helpers
Goal: Given a habit and a household-timezone calendar date, decide whether that date is an expected occurrence.
Description: Implement `_docs/spec.md` section 7 as pure, well-tested functions using Luxon and the household IANA timezone. Daily is due every active day; weekdays use ISO Monday=1 through Sunday=7; weekly-frequency is N completions per Monday–Sunday week, not extra invented due dates. Paused or archived habits are not due. No UI required beyond tests.

## 14. Record a member’s own check-in for an occurrence
Goal: A member can upsert their own check-in for a habit occurrence with a database unique constraint.
Description: Implement stories C1 (today) and C2. One row per `(habit_id, user_id, occurrence_date)`; repeats update the same row. Check-off stores completed or not; only the signed-in member can write their own check-in (no proxy). Occurrence dates are `date` values in the household timezone; timestamps stay UTC.

## 15. Support measurable targets and partial progress
Goal: Measurable habits store partial amounts and become completed only at or above the full target.
Description: Implement the measurable parts of C1 and plan.md scoring rules. Amounts above target are allowed and still count as a single completion. Partial progress must not be treated as complete. Keep this on the same unique check-in row; add tests for partial, exact, and exceeded target.

## 16. Limit check-in writes to today and the previous 7 days
Goal: Members can add or correct check-ins only for today and the previous 7 household-timezone calendar days.
Description: Implement story C3. Writes for day 8 and older must be rejected. “Today” uses the household IANA timezone, including midnight boundaries. Corrections overwrite the existing unique row. Do not yet implement points or challenges in this task unless a hook is already there; the window rule is the deliverable.

## 17. Show weekly-frequency progress without fake daily dues
Goal: Weekly-frequency habits track N completions per household week and can overshoot for recording.
Description: Implement story C4. A week is on track when completed count ≥ N; extra completions in the same week may be stored. Each completed occurrence remains a distinct check-in row. Home (or a small week control) must not pretend the habit is due every day if that misrepresents the frequency target.

## 18. Award and reverse points through a ledger
Goal: Each newly completed occurrence awards 10 points once, and corrections keep the ledger consistent.
Description: Implement story D1 and the `point_ledger` design in `_docs/spec.md` section 4. Insert 10 points when `is_completed` becomes true; remove or reverse the row when a correction makes it incomplete; do not leave two active awards for the same check-in (`check_in_id` unique). Copy privacy onto the ledger row so private points can be excluded later. Same 10 points for every habit; measurable awards only at full target.

## 19. Build the weekly household leaderboard
Goal: Show Monday–Sunday rankings from non-private points, including members with zero.
Description: Implement story D2. Week start is Monday 00:00 in the household timezone. Query only ledger rows that are not private (shared or household-visible completions). Every current member appears, even at zero. Rankings reset each Monday; do not show lifetime totals.

## 20. Assemble the signed-in home dashboard
Goal: After sign-in, Home shows today’s personal checklist plus household summary without leaking private details.
Description: Implement story F1 and spec section 2 item 1. Include today’s personal habits, today’s shared habits with only the current member’s progress, household summary, weekly leaderboard, badges placeholder or real badges if present, and active challenges placeholder or real data if present. Household blocks must omit private habit details. Server-compose the payload; do not fetch other members’ private habits on the client.

## 21. Seed and recalculate the four MVP badges
Goal: Award First Step, Getting Consistent, Habit Builder, and Family Win from valid completions, and keep `criteria_met` honest after corrections.
Description: Implement story D3. Count non-duplicate completed check-ins for the numeric badges (private completions count for the individual). If a correction means the count no longer meets the criterion, keep the `user_badges` row but set `criteria_met` to false. Family Win is granted to participating members when a cooperative challenge finalizes as a win; if challenges are not present yet, implement the numeric badges and a tested hook/function for Family Win.

## 22. Let members create challenges from existing shared habits
Goal: A member can create a competitive or cooperative challenge with name, date range, and selected shared habits.
Description: Implement story E1. End date ≥ start date; dates are household calendar dates. Only shared, non-archived habits at creation time may be selected; store them in `challenge_habits`. All current household members are eligible in the MVP. Listing pages should separate upcoming, active, and completed using start/end and status.

## 23. Compute competitive challenge rankings
Goal: Rank participants by completed check-ins on selected habits in the date range; ties share the win.
Description: Implement story E2. Count completions whose `occurrence_date` is in `[start_date, end_date]` for the selected shared habits only. Do not include private habits. Persist or return a result summary with ranks and winner ids (multiple winners on a tie). Add tests for a tie and for a clear single winner.

## 24. Compute cooperative challenge results including the empty-schedule case
Goal: A household wins a cooperative challenge at ≥ 80% of scheduled check-ins, or the result is invalid if none were scheduled.
Description: Implement story E3. Denominator is scheduled eligible occurrences in the period (use the scheduling rules in spec section 7); numerator is completed ones. Compare `completed / scheduled >= 0.8` (4/5 wins, 3/4 loses). If scheduled count is 0, status is `invalid` with an explanation — never 0% and never an automatic win.

## 25. Keep challenge results provisional until the correction window closes
Goal: Challenge results stay provisional until every occurrence date in the window is older than 7 days, then become final.
Description: Implement story E4 and the archive-during-challenge edge case. Recompute on read while provisional; persist when transitioning to `final`. If a selected habit is archived mid-challenge, keep history and stop creating future occurrences. Corrections inside the 7-day window, including after the end date, still change provisional totals.

## 26. Add one daily in-app reminder per member
Goal: A member can set one daily reminder time and see an in-app prompt for unfinished habits without push or email.
Description: Implement story F2 and `reminder_preferences`. Interpret the time in the household timezone. If the app is open at that time with unfinished habits, show the prompt; if it was closed, show the prompt on the next visit while unfinished habits remain. Copy must not claim a notification was delivered while the app was closed. No email, FCM, or web push.

## 27. Finish the five-item navigation and responsive polish
Goal: Ship Home, Habits, Challenges, Household, and Profile/settings as a usable phone and desktop web app.
Description: Complete the navigation in `_docs/spec.md` section 2 with real links, signed-in layout, and owner-only controls hidden but still enforced on the server. Household page holds members, invite, weekly leaderboard, and settings; settings page holds profile and reminder time. Verify the main flows at a mobile and desktop width; no native apps.

## 28. Cover the family journey and listed edge cases with tests
Goal: Automate the MVP completion checklist and the edge cases in plan.md section 10.
Description: Add end-to-end or integration tests so a family can sign up, create a household, invite a second account, manage habits, check in, score points, see the leaderboard, earn badges, and run both challenge types. Include duplicate check-in, correction reversing points, timezone midnight, private habit 404, expired invite, ownerless-household prevention, and invalid empty cooperative challenge. This task is test-only; fix product bugs you hit, but do not add features.
