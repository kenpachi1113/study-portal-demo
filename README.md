# Study Portal — LMS with an electronic gradebook

A closed learning platform for a teacher and their student groups: lectures and practical assignments released per subgroup on a schedule, private file storage, and a gradebook for attendance, grades and retakes.

[Русская версия](README.ru.md)

**Stack:** Next.js 16 (App Router, Server Actions) · React 19 · TypeScript · PostgreSQL · Prisma 7 · Tailwind CSS 4 · Zod · S3-compatible storage · JWT (jose) · Vercel

<!-- Screenshots: add images to docs/screenshots/ and uncomment
![Gradebook](docs/screenshots/journal.png)
![Topic page](docs/screenshots/topic.png)
-->

---

## Features

**Access model**
- Every topic is hidden by default. It becomes visible only when released to a specific subgroup for a specific date range (`TopicRelease`).
- A closed topic returns **404, not 403**, so topic IDs can't be enumerated.
- All access checks live in one module (`src/lib/access.ts`) and run on every request, including file downloads. Server Actions are treated as public endpoints — nothing relies on hiding buttons in the UI.

**Files**
- Stored in a **private** S3-compatible bucket and served only via short-lived (5 min) signed URLs.
- Uploads go straight from the browser to storage using presigned URLs — bypassing Vercel's 4.5 MB request limit. The server verifies the object with `HEAD` before creating the database record. Upload progress bar via XHR.

**Authentication (no email)**
- The teacher creates student accounts; students sign in with a login + password or a one-time first-login code, then must set their own password.
- Passwords hashed with bcrypt; one-time codes with HMAC-SHA256 (bulk import of 30 codes stays fast). Codes are shown once and never stored in readable form.
- Lockout for 15 minutes after 5 failed attempts. Sessions are JWT in an httpOnly cookie; role checks in `proxy.ts` (Next 16 middleware).
- Audit log of teacher actions.

**Teacher**
- Groups and subgroups; bulk student import from a list of full names with transliterated logins and a printable sheet of sign-in codes.
- Topics of two types (lecture / practical) with rich text content and file materials.
- Release topics to subgroups with open/close dates.

**Gradebook**
- Grades are tied to a class session (date), not to a topic — the same topic runs for different subgroups on different days.
- Attendance with 4 statuses (present / absent / late / excused).
- Grades are never overwritten: a retake is a new record with an attempt number. The final grade follows a per-course rule — best or last attempt.
- Mobile-friendly layout.

**Student**
- List of open topics with statuses and deadlines; topic page with materials.

**Planned:** student file submissions (schema is ready), auto-graded quizzes.

---

## Getting started

```bash
npm install          # postinstall runs prisma generate
npm run setup:env    # creates .env.local with a random secret and password
npm run db:up        # local PostgreSQL (keep this terminal open)
```

In a second terminal:

```bash
npm run db:migrate   # create tables
npm run db:seed      # teacher account and default course
npm run dev
```

The teacher's login and password are printed by `setup:env` and saved in `.env.local`.
Environment status (database, storage, session secret) is shown at `/setup` until the first user exists.

**Local PostgreSQL** runs from `node_modules` via `embedded-postgres` — nothing is installed system-wide. Alternatives: Docker (`npm run db:up:docker`) or any installed Postgres.

---

## Project structure

```
prisma/schema.prisma     full data model, with comments on design decisions
prisma/seed.ts           teacher account and default course
src/proxy.ts             role-based route guard (Next 16 middleware)
src/app/admin/           teacher: sessions, topics, groups, students, gradebook
src/app/study/           student: open topics and materials
src/app/api/materials/   downloads: permission check → signed URL
src/lib/access.ts        ★ the single place where visibility is decided
src/lib/storage.ts       ★ signed URLs, direct-to-storage uploads
src/lib/auth.ts          current user, requireTeacher / requireStudent
src/lib/password.ts      password and code hashing
src/lib/translit.ts      full name → login
```

---

## Author

**Kyrylo Bozhko** — Junior Fullstack Developer · [GitHub](https://github.com/kenpachi1113) · bozhkokd@gmail.com
