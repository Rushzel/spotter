# Coach client manager: specification

**Status: draft for Kidd's review.** Working name "Coachbook", to be renamed.

A web app that lets one coach, or a small coaching business, run many clients: profiles, training programmes, weekly check-ins and progress, session scheduling and session packages. Clients get their own portal to see their programme, log workouts, submit check-ins and book sessions.

## 1. Problem

A coach with 20 to 40 clients usually juggles a spreadsheet of programmes, chat messages for check-ins, a calendar app for sessions and notes on who has paid for how many sessions. Things slip: a client trains on an old programme, a check-in gets buried in chat, a session is booked after the package ran out. This app puts each client's programme, check-ins, sessions and remaining paid sessions in one place, and the database refuses the mistakes (double bookings, sessions with no package balance).

## 2. Stack

Same as the workforce system: Next.js with TypeScript and Tailwind CSS, Supabase (Postgres, Auth, row-level security, Storage, `pg_cron`), Playwright, Vitest, pgTAP, GitHub Actions, Vercel. Charts with Recharts.

## 3. Roles

| Role | Can |
|---|---|
| Head coach (owner) | Everything in their coaching business: clients, programmes, packages, prices, schedule, other coaches |
| Coach | Their assigned clients only: programmes, check-in reviews, sessions |
| Client | Their own profile, programme, workout logs, check-ins, bookings and package balance |

Each coaching business is isolated with row-level security, the same pattern as the workforce system.

## 4. Features

1. **Client profiles:** goals, training history, availability, emergency contact, injury and health notes (visible to the client's coaches only), consent to store health notes recorded with a date.
2. **Programmes:** reusable templates (weeks, days, exercises with sets, reps, load or effort, rest and notes) built from an exercise library. Assigning a template copies it to the client, so later template edits don't silently change an active programme.
3. **Workout logging:** clients log what they actually did per set. Coaches see planned against actual.
4. **Weekly check-ins:** a form the coach configures (body weight, measurements, sleep, energy, adherence from 1 to 5, wins, struggles, optional progress photos). One per client per week. Coaches reply with feedback; unreviewed check-ins are listed first.
5. **Progress:** charts for body weight trend (7-day average), measurements, training volume per muscle group, personal bests, adherence; a "needs attention" list for clients who missed two check-ins or whose adherence dropped.
6. **Packages:** products like "10 sessions, valid 60 days". Selling a package to a client records the price paid and the expiry. The remaining balance is always calculated from sessions, never typed.
7. **Scheduling:** coach availability, client bookings in open slots, recurring sessions. A booking needs an active package with balance. Cancelling inside the business's cancellation window (default 12 hours) still uses a session; earlier cancellations don't.
8. **Reminders:** in-app notifications for upcoming sessions, check-ins due and packages running low or about to expire. Email is optional (needs an email service).

## 5. Data model

| Table | Key columns | Notes |
|---|---|---|
| `businesses` | `id`, `name`, `timezone`, `currency`, `cancellation_window_hours`, `is_demo` | |
| `staff` | `id`, `business_id`, `user_id`, `role` (`head_coach` / `coach`), `active` | |
| `clients` | `id`, `business_id`, `user_id` (nullable until invited), `full_name`, `goals`, `availability`, `status` | |
| `client_coaches` | `client_id`, `staff_id` | Which coaches see which client |
| `health_notes` | `client_id`, `notes`, `consent_at`, `updated_by` | Separate table so its policy is stricter |
| `exercises` | `id`, `business_id` (null for the shared library), `name`, `muscle_groups`, `equipment`, `video_url` | |
| `programme_templates`, `template_days`, `template_items` | weeks, days, exercise, sets, reps, load, rest, order | |
| `programmes`, `programme_days`, `programme_items` | The client's copy; `client_id`, `starts_on`, `status` (`active` / `completed` / `archived`) | Partial unique index: one active programme per client |
| `workout_logs`, `set_logs` | `client_id`, `programme_day_id`, `performed_on`; per set: reps, load, effort | |
| `check_in_forms` | `business_id`, `fields` (JSON) | |
| `check_ins` | `id`, `client_id`, `week_start`, `answers` (JSON), `submitted_at`, `reviewed_by`, `reviewed_at`, `feedback` | Unique on client and week |
| `progress_photos` | `check_in_id`, `storage_path` | Private bucket; only the client and their coaches |
| `packages` | `id`, `business_id`, `name`, `sessions`, `valid_days`, `price_minor`, `active` | |
| `client_packages` | `id`, `client_id`, `package_id`, `sessions_total`, `price_paid_minor`, `purchased_at`, `expires_at` | Snapshot of the package at sale |
| `availability` | `staff_id`, `weekday`, `starts`, `ends` | |
| `sessions` | `id`, `business_id`, `staff_id`, `client_id`, `client_package_id`, `starts_at`, `ends_at`, `status` (`booked` / `completed` / `cancelled_late` / `cancelled` / `no_show`) | Exclusion constraints: no overlap per coach, no overlap per client |
| `notifications` | `user_id`, `kind`, `payload`, `read_at` | |
| `audit_log` | as in the workforce system | Append-only |

**View:** `client_package_balance` = sessions total minus sessions with status `booked`, `completed`, `cancelled_late` or `no_show`.

## 6. Rules enforced in the database

1. Row-level security on every table: businesses are isolated; coaches see only their assigned clients; clients see only their own rows.
2. A coach can't be booked twice for overlapping times; neither can a client (exclusion constraints).
3. A booking needs an active, unexpired client package with a balance above zero; the balance can never go below zero. Booking runs in a function that locks the package row, so two bookings at the same moment can't both take the last session.
4. Cancellation inside the window becomes `cancelled_late` and still counts against the package; the time comes from the server.
5. One check-in per client per week; a check-in can be edited by the client only until a coach reviews it.
6. One active programme per client.
7. Health notes and progress photos are readable only by the client and their assigned coaches, not by other coaches in the business.
8. Package prices and session counts are copied at sale, so changing a package later doesn't change what a client bought.
9. Demo accounts can't change email or password.

## 7. Screens

- **Coach:** dashboard (today's sessions, check-ins to review, needs attention), clients list, client page (profile, programme, logs, check-ins, progress, packages, sessions), programme builder, calendar, packages, settings.
- **Client:** home (next session, today's workout, check-in due), workout logger (built for phones), check-in form, progress, bookings, package balance.

Design follows Kidd's preferences: clean, Apple-like, one accent colour, dates day-first.

## 8. Sample data

One made-up coaching business, **Ridgeway Strength**, with a head coach, one coach and 16 made-up clients with 12 weeks of programmes, logs and check-ins, a mix of packages including one nearly used up and one expired. Progress photos are illustrations or plain placeholders, never photos of real people. Demo accounts: head coach, coach, client.

## 9. Out of scope for the first version

Taking payments (packages are recorded as sold, not charged), nutrition and meal plans, video calls, native phone apps.

## 10. Decisions to review

1. Name.
2. Should one person be able to be a client of two coaching businesses? Proposed: yes, with a switcher, same as the workforce system.
3. Progress photos in the demo: placeholders (recommended) or generated illustrations.
4. Default cancellation window: 12 hours (proposed) or 24.
