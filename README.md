<p align="center">
  <img src="assets/logo.png" width="128" alt="hundred. app icon">
</p>

<h1 align="center">hundred.</h1>

<p align="center">
  100 days. 2 people. no excuses.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Next.js-16-black" alt="Next.js 16">
  <img src="https://img.shields.io/badge/TypeScript-5-blue" alt="TypeScript 5">
  <img src="https://img.shields.io/badge/Supabase-Postgres-3ECF8E" alt="Supabase">
  <img src="https://img.shields.io/badge/PWA-installable-purple" alt="PWA">
  <img src="https://img.shields.io/badge/Vercel-hosted-black" alt="Hosted on Vercel">
  <img src="https://img.shields.io/badge/source-private-lightgrey" alt="Source: private">
  <img src="https://img.shields.io/badge/case%20study-showcase-brightgreen" alt="Case study">
</p>

---

## About

**hundred.** is a private, two-person 100-day challenge, built as a PWA for me and my partner. Every day you both tap your goals; miss one and the run resets to day 1. It sends real push notifications, updates live between the two of you, and uses a design system that is pure black and bold Helvetica.

> **Case study.** This repository is a public showcase, not the source code. The app's source is private, and the live app is invite-only, with exactly two seats.

## Screenshots

<table align="center">
  <tr>
    <td align="center">
      <img src="screenshots/01-login.png" width="240" alt="Login screen">
      <br><sub><b>Login</b></sub>
    </td>
    <td align="center">
      <img src="screenshots/02-today.png" width="240" alt="Today: shared goal checklist">
      <br><sub><b>Today</b>: shared goal checklist, updating live</sub>
    </td>
    <td align="center">
      <img src="screenshots/03-wall.png" width="240" alt="The wall: 100-dot streak grid">
      <br><sub><b>The wall</b>: 100-dot streak grid</sub>
    </td>
  </tr>
</table>

## Features

- **Today.** A shared checklist with one column per person, updating live as your partner ticks things off. React or nudge them from the same screen.
- **The wall.** A 100-dot grid per person showing the current unbroken run.
- **Strict resets.** A missed day sends the run back to day 1. A scheduled job decides that, not the client.
- **Progress photos and reactions.** Emoji, short text and selfies between the two of you.
- **Push notifications.** About three a day per person, rotated from message pools so they don't turn into noise.
- **Installable.** Works as a PWA on iOS and Android.

## Stack

| Layer | Choice |
|---|---|
| App | Next.js 16 (App Router, Turbopack), TypeScript, Tailwind v4, shadcn/ui |
| Data and auth | Supabase: Postgres, Google sign-in, Row Level Security, Realtime |
| Push | Web Push (VAPID), sent server-side |
| Scheduling | Supabase Edge Function invoked every 15 minutes by `pg_cron` + `pg_net` |
| Hosting | Vercel |

## Architecture

```
client (PWA) ──► Supabase Postgres (RLS, Realtime)
                        ▲
                        │ writes day_results (service role)
pg_cron + pg_net ──► Edge Function (every 15 min) ──► Web Push (VAPID)
```

Clients only write plain check-ins. Pass/fail results are written by the scheduled job, never by the client.

## Engineering decisions

### Security lives in the database, not the UI

Every table is protected by Row Level Security. A third Google account can sign in, but it isn't bound to either seat, so every policy rejects it. "This app is full" is enforced by Postgres, not by hiding a button. Check-ins can only be written to your own slot, for today, while the challenge is active.

### The server owns the truth about a day

Clients only ever write plain check-ins. Pass/fail results (`day_results`) are written by a scheduled job with the service role, so nobody can edit their own streak.

### Streak semantics

"Strict reset" and "one 100-dot wall per person" only reconcile one way: each person's day number is their own current streak. In team mode, one miss resets both.

### No secrets in git

The scheduled job needs the service-role key to authorise its own call. Instead of committing it in a migration, the key lives in Supabase Vault and is referenced by name from SQL.

### Notifications designed against habituation

All copy comes from pools, picked deterministically from the Madrid date. It varies day to day, but retrying the same tick produces the same message, which keeps the job idempotent. Intermediate check-ins don't push at all, because the partner already sees them live. That cut volume from about 8 to about 3 a day.

### Shared code across two runtimes

The notification copy and scheduling rules are pure TypeScript, imported unchanged by both the Deno edge function and the Node server actions.

## Status

In daily use. Source available to discuss in an interview.

---

<p align="center">
  <sub>© 2026 Pol Cabezas Agustí. All rights reserved. Screenshots and text shared for portfolio purposes.</sub>
</p>
