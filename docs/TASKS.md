# Coach client manager: tasks, in order

Tick a task only when its "Done when" is true, then commit.

## 0. Planning (this commit)
- [x] Specification (with data model), acceptance criteria, task list.
- [ ] **Needs Kidd:** review decisions and criteria.

## 1. Skeleton
- [ ] Next.js, TypeScript, Tailwind CSS, ESLint, Vitest, Playwright; `supabase init`.
  - Done when: build, lint and tests pass.

## 2. Database (**needs Docker** to run)
- [ ] Migrations: businesses, staff, clients, assignments, health notes; row-level security; explicit grants.
- [ ] Migrations: packages, client packages, availability, sessions with exclusion constraints, `book_session` and `cancel_session` functions, balance view.
- [ ] Migrations: exercises, templates, programmes, logs, check-in forms, check-ins, photos (private bucket policies).
- [ ] pgTAP: criteria 1 to 13.

## 3. Screens
- [ ] Coach: dashboard, clients, client page, programme builder, calendar, packages.
- [ ] Client: home, workout logger, check-in, progress, bookings.
- [ ] Charts and "needs attention" (criteria 14, 15).

## 4. Sample data and demo
- [ ] Seed per SPEC section 8; nightly reset; demo sign-in; criteria 16 and 17.

## 5. Continuous integration and documentation
- [ ] CI workflow; README with the reasons for each database rule; case study.
