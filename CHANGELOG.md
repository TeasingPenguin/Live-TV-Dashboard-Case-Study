# Changelog

All notable changes to the Live TV Dashboard. Dates are when each change went live.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.3.0] – 2026-11-05

Portfolio clean-up.

### Added

- **Demo mode** (`npm run demo`, or `DEMO=1`): made-up bookings, invoices and to-dos, built
  through the same normalisers as live data, so the dashboard can be shown and developed
  without real customer data or credentials. Paying, voiding and hand-offs are refused.
- Real systemd unit files in `deploy/systemd/`.
- This changelog, and a rewritten README covering setup, architecture, security and recovery.

### Removed

- The server-sent-events stream (`/api/events`) left over from v0.1. Screens have used the
  WebSocket since v1.0.
- Settings nothing reads any more (`DASHBOARD_POLLING_ONLY`, `GHL_API_VERSION`,
  `GHL_BUSINESS_NAME`) and an unused icon import.

### Changed

- `.env.example` now lists exactly the settings the code reads, grouped and explained.
- Requires Node 24.

## [1.2.0] – 2026-11-03

Moved from a temporary preview address to a permanent domain, then hardened sign-in and
backups.

### Added

- **Permanent address** through a Cloudflare Tunnel run as a systemd service: survives
  reboots, needs no open ports, and the dashboard only accepts changes from that origin.
- **Signed-in devices** in Settings: every screen with its name, browser, and when it signed
  in and was last active. Anyone signed in can view the list; only the server itself can name
  or sign out devices (`npm run devices`), and those requests are refused if they arrive
  through the tunnel.
- **10-minute encrypted snapshots** with restic, storing only what changed (about 20 MB a
  run), mirrored to Google Drive with rclone. Kept: everything from the last day, then
  hourly for a week and daily for a month.
- **Disaster recovery:** `scripts/restore.sh` rebuilds a fresh Ubuntu server from the Drive
  snapshots, and `scripts/emergency-kit.mjs` produces a single file, kept off the server,
  that holds everything the restore needs. Tested end to end: 2.7 GB restored in about a
  minute.

### Security

- **Three wrong access keys block a network** from key sign-ins, using the visitor's real
  address from Cloudflare (the whole /64 for IPv6). Previously the per-minute limit counted
  every visitor as the tunnel's own address. Blocks survive restarts and are lifted only
  from the server.
- Anti-framing and content-type headers (`frame-ancestors 'none'`, `X-Frame-Options`,
  `nosniff`, `Referrer-Policy`).
- **One-time sign-in links need a tap:** opening one shows a button, and only the button
  press (a POST) uses it. Message-app link previews were using links up before the person
  could open them.
- The snapshot mirror refuses to overwrite Drive if it holds a different repository, so a
  freshly rebuilt server can't wipe the off-site copy.

### Fixed

- Every address other than `/` returned "Not Found" because the project's path contains a
  hidden folder, which Express treats as a dotfile.
- The Google Drive permission message mentioned the calendar.

## [1.1.0] – 2026-11-01

### Added

- **Native iPhone layouts.** Portrait: a bottom tab bar, the calendar week as a day-by-day
  list, and Database rows as cards with a sort picker. Landscape: compact metrics, a
  calendar that fits the screen, and two columns of cards clear of the camera cut-out.
- Add to Home Screen support (web app manifest and icons).

### Fixed

- The hand-off menu could be hidden behind other content. It is now drawn above everything,
  as a bottom sheet on phones.

## [1.0.0] – 2026-10-30

A rebuild around two businesses: Sleak Property Services and Sandbox Automations.

### Added

- **TV layout for 1920 × 1080:** each business gets six metric bubbles (quotes, jobs and
  money for today and tomorrow) and one to-do list; a week-view calendar and a personal
  to-do list fill the bottom half.
- **Metrics from GoHighLevel bookings:** quotation services count as quotes, every other
  service as a job, and money is the summed opportunity value of that day's job contacts.
- **Unified calendar:** both GoHighLevel sub-accounts and Google Calendar, colour-coded.
  Google copies of GoHighLevel bookings are dropped, and blocked slots only show when they
  aren't duplicates.
- **Unpaid invoices as to-dos**, with _mark paid_ (records a bank transfer) and _void_.
  Both re-read the invoice first and ask for confirmation.
- **Hand-offs:** pass an invoice or task to a team member, who gets an SMS from the
  business's GoHighLevel number, or to an AI agent through a webhook. The AI asks before
  changing anything.
- **Task history:** every task and invoice event is recorded permanently, downloadable as
  CSV and browsable in a sortable, filterable **Database view**.
- **Hourly AI to-do scan:** a scheduled agent reads new email and GoHighLevel messages
  through a local-only agent API and adds to-dos. A to-do Jordan deletes never comes back.
- **Live updates** to every screen over a WebSocket, falling back to polling.
- **Sign-ins survive restarts**, and one-time sign-in links let a TV sign in without typing.
- **Nightly encrypted backups** (zstd + AES-256) kept on the server and in Google Drive.

## [0.2.0] – 2026-10-06

### Added

- "Needs a reply" email sidebar with mark-as-done.

## [0.1.0] – 2026-10-02

### Added

- First version: a React dashboard for a Chrome TV with today / week / month task lists,
  a Gmail priority inbox, a GoHighLevel unread-message count and a live clock. Changes
  pushed to screens with server-sent events.
