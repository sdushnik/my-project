# MVP development specification

**Status:** Implementation baseline for Option A  
**Product scope:** [_docs/plan.md](./plan.md)  
**Stack:** [_docs/tech-stack.md](./tech-stack.md)

This document does not add features. It turns the product plan into user stories, acceptance criteria, a first schema, and a server API shape.

## 1. Architecture rules

- Next.js App Router. All reads and writes that touch habits, check-ins, points, badges, challenges, or membership run on the server.
- PostgreSQL is the source of truth. Drizzle owns the schema and queries.
- Authorization is checked in every server action / route handler. The UI may hide controls; it is not the security boundary.
- Day and week boundaries use the **household IANA timezone**. Weeks are Monday 00:00 through Sunday 23:59:59.999 in that timezone.
- Timestamps in the database are UTC. Occurrence dates are calendar dates in the household timezone (`date`, not `timestamptz`).
- One account belongs to at most one household in the MVP.

## 2. Navigation (implementation)

1. `/` (signed in) — Home: today's checklist + household summary  
2. `/habits` — create, edit, pause, archive  
3. `/challenges` — active, upcoming, completed  
4. `/household` — members, invite link, weekly leaderboard, settings  
5. `/settings` — profile, reminder time, account  
6. `/sign-in`, `/sign-up`, `/invite/[token]` — unauthenticated / join flows  

Signed-out visitors hitting protected routes go to sign-in.

## 3. User stories and acceptance criteria

IDs are stable for tests. Given/when/then is implied by the criteria.

### A. Accounts and household

**A1. Sign up and sign in**  
As a person, I can create an account with email/password or Google and sign in again.

- Email/password and Google both create or attach a `users` row.
- Duplicate email cannot create a second account.
- Invalid credentials show a safe error; they do not leak whether the email exists beyond Auth.js defaults.
- After sign-in with no household, the user is prompted to create a household or wait for an invite.

**A2. Create household**  
As a signed-in user with no household, I can create one and become owner.

- Household has name, owner = creator, IANA timezone (required).
- Creator is the only member and the owner.
- A user who already belongs to a household cannot create another.

**A3. Invite link**  
As owner, I can create a join link.

- Token is a high-entropy value; only a hash is stored.
- Link expires 7 days after creation.
- Owner can revoke an unused, unexpired link.
- Creating a new link does not have to invalidate old ones unless revoked or expired; the owner can revoke explicitly.
- Non-owners cannot create, revoke, or view the raw token after creation (they may see that invites exist).

**A4. Join household**  
As a signed-in user with no household, I can join via a valid invite.

- Expired, revoked, or unknown tokens fail with a clear error.
- A user already in a household cannot join another.
- Join succeeds only while the household still has an owner.
- Joiner becomes a member, not owner.

**A5. Membership management**  
As owner, I can remove members and update household name/timezone.

- Owner cannot remove themselves if that would leave the household without an owner.
- Owner transfer is **out of MVP** unless needed to unblock tests; if implemented, it must be explicit and leave exactly one owner.
- Removing a member keeps their historical check-ins and ledger rows; they lose access immediately.
- Timezone changes apply to future day/week boundaries; existing occurrence dates stay as stored calendar dates.

### B. Habits

**B1. Create personal habit**  
As a member, I can create a personal habit I control.

- Fields: name (required), description (optional), type check-off or measurable, schedule, visibility private or household-visible, status active.
- Measurable habits require a numeric target > 0 and an optional unit label.
- Schedule is one of: every day; selected weekdays (at least one); weekly frequency N (1–7) times per week.

**B2. Create shared habit**  
As a member, I can create a shared household habit.

- Shared habits are always visible to household members.
- Any member can later edit, pause, or archive a shared habit.
- Each member records their own completions on shared habits.

**B3. Visibility**  
As a member, I can mark a personal habit private.

- Private habit name, description, values, check-ins, points, and history never appear in another member’s dashboard, leaderboard, challenge results, or API responses.
- Household-visible personal habits show completion to household members without exposing other private habits of the same person.

**B4. Pause and archive**  
As someone allowed to manage the habit, I can pause or archive it.

- Paused: no new expected occurrences; history remains; can be reactivated.
- Archived: no new expected occurrences; cannot be used on **new** challenges; existing challenge history remains interpretable.
- Archived habits are not hard-deleted.

### C. Check-ins

**C1. Record today**  
As a member, I can record my own progress for a habit occurrence due today (household timezone).

- Check-off: complete or not complete.
- Measurable: record an amount; partial is allowed; completed only when amount ≥ target.
- Amount above target is allowed; it still counts as a single completion.
- Only the member can write their own check-in. There is no proxy check-in in MVP.

**C2. Uniqueness**  
As the system, I allow at most one check-in row per (habit, member, occurrence date).

- Repeating a check-off does not create a second row or a second point award.
- Updates overwrite the same row.

**C3. Seven-day window**  
As a member, I can add or correct check-ins for today and the previous 7 calendar days in the household timezone, and not older.

- Day 8 and earlier reject writes.
- Corrections recalculate completion, points, badges, and affected challenge totals.

**C4. Weekly-frequency habits**  
As a member, I can complete a weekly-frequency habit up to its N times that week, and also overshoot for tracking, but scoring treats each completed occurrence independently.

- The habit is “on track” when completed count for the week ≥ N.
- Extra completions in the same week may be recorded; each completed occurrence still awards 10 points once (fixed points per occurrence, per plan).
- Under-N shows incomplete week progress on the dashboard.

### D. Points, leaderboard, badges

**D1. Points**  
As a member, I earn 10 points the first time an occurrence becomes completed.

- Same value for all habits.
- Measurable: points only when full target is reached.
- If a later correction makes it incomplete, the ledger entry is reversed/removed; net points drop.
- Completing again after a reversal awards 10 points once more (still one active award per occurrence).

**D2. Weekly leaderboard**  
As a household member, I can see Monday–Sunday rankings for **shared and household-visible** completions only.

- Private habit points never appear.
- Members with zero eligible points still appear.
- Rankings reset each Monday 00:00 household time.

**D3. Badges**

| Badge | Criterion |
|---|---|
| First Step | First valid completed check-in (any habit of that member, including private) |
| Getting Consistent | 10 valid completed check-ins |
| Habit Builder | 50 valid completed check-ins |
| Family Win | Household completes a cooperative challenge (result finalized as win) |

- Counts use non-duplicate completed occurrences. Corrections can make a member no longer meet a numeric criterion; the earned badge row remains but is marked inactive/`criteria_met = false` so it is not misleading.
- Family Win is household-level; each participating member receives it when the cooperative challenge finalizes as a win.

### E. Challenges

**E1. Create challenge**  
As a member, I can create a competitive or cooperative challenge from one or more **existing shared** habits, with start and end dates.

- Dates are calendar dates in the household timezone; end ≥ start.
- Only shared, non-archived habits at creation time are eligible.
- All current household members are eligible participants in MVP.

**E2. Competitive**  
Rank participants by completed check-ins on selected habits whose occurrence dates fall in [start, end]. Ties share the win.

**E3. Cooperative**  
Household wins if completed eligible check-ins ≥ 80% of **scheduled** eligible check-ins in the period.

- If scheduled count is 0, result is `invalid` (not 0%, not automatic win), with an explanation.

**E4. Provisional vs final**  
Results stay `provisional` until every occurrence date in the challenge window is older than the 7-day correction window. Then they become `final`.

- Archiving a selected habit mid-challenge keeps history; no new scheduled occurrences are generated after archive.

### F. Dashboard, reminders, access

**F1. Home**  
After sign-in, Home shows: today’s personal checklist; today’s shared habits with **my** progress; household summary; weekly leaderboard; badges; active challenges; reminder prompt if relevant. Private details never appear in household blocks.

**F2. In-app reminder**  
As a member, I set one daily reminder time.

- If the app is open at that time and unfinished habits remain, show an in-app reminder.
- If the app was closed, show the reminder on the next visit that day (or the next visit while unfinished habits remain from that reminder slot), without claiming a notification was delivered while closed.
- No email or push.

**F3. Privacy API**  
A request for another member’s private habit, check-in, or private points returns 404 or 403 (same treatment for existence hiding is acceptable; pick **404** for habit-by-id that the caller cannot see). Never 200 with empty-but-secret fields.

## 4. First database design

All primary keys are UUIDs. `created_at` / `updated_at` are `timestamptz` UTC.

### `users`

- `id`
- `name`
- `email` unique
- `email_verified` (nullable timestamptz)
- `image` (nullable)
- Auth.js account/session tables as required by the adapter

### `households`

- `id`, `name`, `owner_user_id` (FK users, restrict delete)
- `timezone` (IANA text, e.g. `Asia/Almaty`)
- `created_at`

### `household_members`

- `household_id`, `user_id`
- `joined_at`
- Unique `(user_id)` — one household per account
- Unique `(household_id, user_id)`
- Role is implied: `users.id === households.owner_user_id` is owner; otherwise member

### `invitations`

- `id`, `household_id`, `created_by_user_id`
- `token_hash` unique
- `expires_at`, `revoked_at` nullable, `used_at` nullable, `used_by_user_id` nullable
- Usable iff `revoked_at` is null, `used_at` is null, and `expires_at` > now()

### `habits`

- `id`, `household_id`, `creator_user_id`
- `name`, `description` nullable
- `scope`: `personal` | `shared`
- `visibility`: `private` | `household`  
  Constraint: `scope = shared` ⇒ `visibility = household`
- `tracking_type`: `check_off` | `measurable`
- `target_value` numeric nullable; required when measurable
- `unit` nullable
- `schedule_kind`: `daily` | `weekdays` | `weekly_frequency`
- `schedule_weekdays` (smallint[] 0–6 or 1–7; document ISO: 1=Monday … 7=Sunday)
- `weekly_frequency` smallint nullable (1–7)
- `status`: `active` | `paused` | `archived`
- `created_at`, `updated_at`

### `check_ins`

- `id`, `habit_id`, `user_id`
- `occurrence_date` date (household calendar date)
- `value` numeric nullable (measurable)
- `is_completed` boolean
- `created_at`, `updated_at`
- Unique `(habit_id, user_id, occurrence_date)`

### `point_ledger`

- `id`, `user_id`, `household_id`, `check_in_id` unique
- `points` integer (10 or 0; prefer insert 10 and delete/reverse on correction)
- `occurrence_date` date
- `scoring_week_start` date (Monday of that week in household TZ)
- `is_private` boolean (copied from habit visibility at award time; private rows never selected for leaderboard)
- `created_at`

Do not leave an active +10 row when `is_completed` becomes false.

### `badge_definitions`

Seed four rows: `first_step`, `getting_consistent`, `habit_builder`, `family_win`.

### `user_badges`

- `user_id`, `badge_key`
- `earned_at`, `criteria_met` boolean
- Unique `(user_id, badge_key)`

### `challenges`

- `id`, `household_id`, `creator_user_id`
- `name`
- `type`: `competitive` | `cooperative`
- `start_date`, `end_date`
- `status`: `upcoming` | `active` | `provisional` | `final` | `invalid`
- `result_summary` jsonb nullable (ranks, percentages, winners, invalid reason)
- `created_at`

### `challenge_habits`

- `challenge_id`, `habit_id`
- Unique pair. Habits must be `scope = shared` at insert.

### `reminder_preferences`

- `user_id` PK
- `local_time` (minutes from midnight or `time` without TZ)
- Interpreted in the **household** timezone for “is it that time of day”

## 5. Authorization matrix

| Action | Owner | Member | Other household | Signed-out |
|---|---|---|---|---|
| Household settings, invites, remove member | yes | no | no | no |
| Create/manage own personal habits | yes | yes | no | no |
| Create/manage shared habits | yes | yes | no | no |
| Check in for self | yes | yes | no | no |
| Check in for someone else | no | no | no | no |
| Read private habit of another member | no | no | no | no |
| Read shared / household-visible habits | yes | yes | no | no |
| Create challenges | yes | yes | no | no |

Server queries for “household dashboard” must filter `visibility = household` or `scope = shared`, plus `habits.user_id = current` for the personal checklist.

## 6. Server API shape (Next.js)

Prefer Server Actions for form mutations and Route Handlers only where the client needs JSON. Names below are logical; exact file placement can follow `src/server/*`.

**Auth:** Auth.js handlers.

**Household:** `createHousehold`, `updateHousehold`, `createInvitation`, `revokeInvitation`, `joinHousehold(token)`, `removeMember`.

**Habits:** `createHabit`, `updateHabit`, `setHabitStatus`.

**Check-ins:** `upsertCheckIn({ habitId, occurrenceDate, value? | completed? })` — rejects dates outside the 7-day window.

**Reads:** `getHomeDashboard`, `listHabits`, `getWeeklyLeaderboard`, `listChallenges`, `getChallenge(id)`.

Every mutation that changes completion must:

1. Upsert `check_ins`  
2. Sync `point_ledger`  
3. Recompute `user_badges.criteria_met` for the member  
4. Leave challenge `result_summary` to be recomputed on read while `provisional`, persisted when transitioning to `final`

## 7. Scheduling notes

For a given `occurrence_date` D in the household timezone:

- `daily`: due if habit is active that day (not paused/archived as of D).
- `weekdays`: due if D’s weekday is in `schedule_weekdays`.
- `weekly_frequency`: the habit appears on the daily checklist every day of the week until N completions exist, or as a week-scoped item on Home (implementation may show a week progress control rather than a false “due every day”). Product requirement is tracking N times per week, not inventing extra due dates.

Paused/archived as of D: not due; existing check-ins for earlier dates remain.

## 8. Build order (unchanged)

1. Auth, household, invites, roles, privacy  
2. Habits, schedules, check-ins, 7-day corrections  
3. Points, Home, weekly leaderboard  
4. Badges, challenges  
5. In-app reminders, responsive polish, a11y, e2e  

Do not start gamification until privacy and check-in uniqueness have tests.

## 9. Test cases (minimum)

Cover every bullet in plan.md §10, plus:

- Unique `(habit_id, user_id, occurrence_date)` at the database  
- Leaderboard query excludes `point_ledger.is_private = true`  
- Invite token plaintext is never stored  
- Cooperative 80% rounding: define **floor not used**; use completed / scheduled with standard comparison `>= 0.8` (e.g. 4/5 = 80% win; 3/4 = 75% lose)
