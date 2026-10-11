# Aztecs Cricket Club · Nets Booking

Book cricket practice nets by the hour. React (Vite) frontend, Node/Express API, SQLite storage.

## Branding

The look follows [aztecscricketclub.com](https://www.aztecscricketclub.com/), and the club logo is used with the club's permission.

- **Logo:** `client/public/aztecs-logo.png`, used in the header, browser tab and emails.
- **Colours:** site blue `#2188C5` for the header, active tabs and selections. Logo navy `#232576` for title bands and email headings. Logo red `#EE1A1F` used sparingly. Light grey `#F1F1F1` page with `#333` text. All colours are tokens at the top of `client/src/styles.css`. The site is always light, like the club website, even when the browser or system is in dark mode.
- **Type:** Montserrat for text, headings and buttons. Anton for the club name and page titles. Both load from Google Fonts in `client/index.html`.
- **Components:**
  - a full-width blue header with the logo badge;
  - black rounded uppercase main buttons, matching the site's JOIN button, and white-outlined ones in the header (Login);
  - pill tabs, blue when active and grey otherwise;
  - heavy uppercase section titles.
- **Club sponsors:** the six sponsors from the club website's footer (Sandra Davidson, Ilford City Kitchen, Santosh Goyal Foundation, Connexion, Jai Solutions LTD, EasyFundraising) appear in a "Club sponsors" strip in the footer of every customer page. Each shows its role and links to the sponsor's site. To change them, edit `client/src/shared/sponsors.js` and the logos in `client/public/sponsors/`.
- **Emails** use the same header bar, logo and colours. The logo in emails loads from `APP_URL`, so set that to the live site address or recipients won't see it.

## Features

- **Home page** (`#home`, the default page): club-branded hero with Book a net / Create an account buttons, or the customer's next session when signed in; "Today at the nets" cards with each net's price, free hours left today and next free time (live); how it works; opening hours, booking window, cancelling and paying; a link to the club website.
- **Customer accounts**: register with name, email, phone and password, then sign in. If you pick slots while signed out and then choose to sign in or register, your selection is kept.
- **Passwords and email confirmation**:
  - **Forgot password** on Sign in emails a reset link valid for 1 hour. Using it signs the customer out on all other devices.
  - The form gives the same answer whether or not the email has an account, so it can't be used to find out who's registered.
  - New accounts get a "confirm your email" link (valid 7 days), and a banner with **Resend link** shows until they confirm. Using a reset or set-password link also confirms the address.
  - All links work once. Only a SHA-256 hash is stored, a newer link replaces older ones, and emails are limited to 3 per hour per address.
- **Guest booking**: book without an account by entering name, email and phone. The confirmation email contains a private "Manage your booking" link (`#manage/REF/TOKEN`) for viewing or cancelling it. Only a SHA-256 hash of the token is stored, and the token sits in the URL fragment, so it isn't sent to the server in request URLs or logs. One guest email can hold at most 3 upcoming bookings. Admin shows guest bookings with a "Guest" badge.
- **Book a net**: AllBooked-style date selection: a toolbar with **Day | Month** views, ‹ › arrows, a 📅 date button that opens a pop-up calendar, and **Today**. Day view lists times down the side with one column per net, so all four fit on a phone. Month view is a calendar showing free hours per day (with a bar); click a day to open it. Customers can pick dates up to 14 days ahead, staff up to a year. Select up to 4 back-to-back hours on one net. The price is shown before you confirm, and the booking uses the account's name and phone.
- **No double booking**: a unique index on `(net, date, hour)` covers active bookings. Multi-hour bookings are written in one transaction, so if two people race for the same slot, one gets a clear "just booked" error.
- **Customer portal**: once signed in, customers get the same left-sidebar layout as the admin portal:
  - **Bookings:** Book a net (`#book`), Upcoming bookings (`#mine`, soonest first, cancellable until they start), Past & cancelled (`#mine/past`).
  - **Account:** My details (`#account`): change name and phone, which also updates upcoming bookings. Email is read-only because it's the sign-in. Change password (`#account/password`): needs the current password, signs out other devices, and allows 5 wrong attempts per 15 minutes.
  - **Signed-out visitors** keep the public header (Book a net, My bookings, Register, Login), and guest booking is unchanged.
- **Admin**: separate staff sign-in (username and password) at `#admin`, linked from the footer. The admin portal has a left sidebar (Dashboard; Upcoming bookings, Bookings by day, New booking; Members). Each section has its own address (`#admin/dashboard`, `#admin/upcoming`, `#admin/day`, `#admin/new`, `#admin/members`), so links, bookmarks and the back button work. On phones the sidebar becomes a scrollable strip of links. Shows upcoming bookings or a single day, with booking count, hours and revenue, and can cancel any booking.
- **Dashboard** (first Admin tab): **Daily**, **Weekly** (Monday to Sunday) or **Monthly**, stepping back and forward with ◀ ▶.
  - Headline figures: booked revenue, bookings, hours booked, utilisation, cancellations and new members, each compared with the previous period.
  - Revenue or hours by hour (daily) or by day (weekly/monthly), with hover details and a table view.
  - Utilisation per net, members vs guests (and how many bookings staff made), and a weekday-by-hour peak-times heatmap for weekly and monthly.
  - Revenue is the value of active bookings by session date, since customers pay at the counter. Utilisation is booked hours out of all bookable hours (16 per net per day).
- **Staff roles (Admin and Coach)**: both sign in at the staff sign-in (`#admin`).

  | | Admin | Coach |
  |---|---|---|
  | Book nets for members/guests, up to a year ahead | ✓ | ✓ |
  | View upcoming bookings / bookings by day | ✓ (with revenue) | ✓ (no revenue total) |
  | Cancel bookings | any | only ones they made |
  | Blocked times (block or unblock nets) | ✓ | ✗ |
  | Members, Dashboard, Staff | ✓ | ✗ |

  Admins manage staff on the **Staff** page (`#admin/staff`): add an admin or coach with a starting password, change role, reset password (signs them out) or remove an account. Bookings and blocks the removed person made are kept. There must always be at least one admin, and you can't remove yourself. Rules are enforced by the server (403 for anything a coach isn't allowed); the screens also hide what a coach can't use. Existing staff accounts became admins when roles were added.
- **Staff booking window and blocked times**:
  - Staff can book up to 365 days ahead; customers stay at 14. The New booking grid pages two weeks at a time and has a date picker for jumping anywhere in the year.
  - **Blocked times** (`#admin/blocks`) take one net or all nets out of use for a date range and time range, optionally only on chosen weekdays (for example "Net 2, every Tue and Thu, 6–8 PM, April to September, Club training"), up to a year ahead.
  - Customers see blocked slots as "Unavailable" without the reason. Nobody can book them, staff included: remove the block first.
  - If a new block overlaps bookings, they're listed. Staff can adjust the block, or cancel those bookings in the same step, which emails the customers.
  - The dashboard's utilisation still counts blocked hours as bookable.
- **Member management**: Admin, then **Members**. Search members and see their upcoming and total bookings. **Add** a member and either email them a link to set their own password (the default, valid 7 days) or type a starting password to pass on. **Password link** emails an existing member a set-password link. The Status column shows whether each member's email is confirmed, and changing an email marks it unconfirmed again. **Edit** name, email or phone; name and phone changes also update their upcoming bookings. Setting a new password resets it and signs them out everywhere. **Delete** cancels their upcoming bookings (emailing them), keeps past bookings in the records under the name on each booking, and removes the account and its sign-ins. **Book** opens New booking with that member already chosen.
- **Staff booking on behalf of customers**: Admin, then **+ New booking**. Pick slots on the same grid customers use, then either search for a member (by name, email or phone) or enter guest details. The customer gets the usual confirmation email, saying the venue booked it for them; guests also get their manage link. Member bookings appear in that member's My bookings. These bookings record which admin made them and show a "by staff" badge. Staff bookings follow the same slot rules but aren't subject to the guest booking cap.
- **Emails**: customers get a confirmation email when they book and a cancellation email when a booking is cancelled (saying whether the venue cancelled it). Emails are sent after the response, so a mail problem never blocks a booking.
- **Security**: passwords are hashed with scrypt. Sessions are random tokens in `HttpOnly`, `SameSite=Lax` cookies; only a hash of each token is stored. Customer and admin sessions are separate. Failed logins are limited to 10 per 15 minutes per IP and account.
- Responsive layout. Always light (white) theme, including in browsers set to dark mode.

## Project structure

The customer ("user") side and the admin side are separate modules on both the server and the client. Code that both use lives in a shared folder.

```
server/
  index.js              entry point (opens the DB, starts Express)
  app.js                builds the app: shared services, then mounts each API module
  config.js             opening hours, booking limits, session lengths, cookie names
  lib/                  shared by every module
    db.js               schema, migrations, seed data
    auth.js             password hashing, session store, cookies, login rate limiter
    sessions.js         sign-in/out helpers, currentUser/currentAdmin, requireUser/requireAdmin
    bookings.js         createBooking (the one set of slot rules), grouping, email lookups
    mailer.js           SMTP / outbox transport and email templates
    http.js             HttpError, async wrapper, input validators, search helpers
    dates.js            local-date helpers
  modules/
    public/routes.js    /api/config, /api/nets, /api/availability
    user/               mounted at /api
      auth.routes.js      /api/auth/*            register, login, logout, me
      bookings.routes.js  /api/bookings*         book (member or guest), my bookings, cancel
      guest.routes.js     /api/guest-bookings/*  view/cancel via the emailed link
    admin/              mounted at /api/admin (login/logout/me are open; the rest need an admin session)
      auth.routes.js      login, logout, me
      bookings.routes.js  bookings list, staff booking, cancel, customer search
      members.routes.js   /members           add, edit, delete members
      reports.routes.js   /reports           dashboard endpoint
      reports.service.js  dashboard calculations
  tests/api.test.js

client/src/
  App.jsx               shell: header, routing, customer session
  shared/               used by both sides
    http.js             fetch wrapper (ApiError, request/post/patch/del)
    api.js              public endpoints (config, availability)
    format.js           date, time and money formatting
    nav.js              hash routing helpers
    components/SlotPicker.jsx
  user/
    api.js              customer endpoints
    pages/              BookPage, MyBookingsPage, AuthPages (login/register), ManageBookingPage
  admin/
    api.js              admin endpoints
    pages/              AdminPage (tabs shell), LoginPage, DashboardPage, NewBookingPage, MembersPage
```

API paths didn't change when the code was split into modules.

## Requirements

Node 16+ (Vite 4 is used so it runs on Node 16).

## Run it

```bash
npm run install:all
```

Then, in two terminals:

```bash
npm run server
```

```bash
npm run client
```

Open http://localhost:5173. The API runs on http://localhost:3001 and Vite proxies `/api` to it.

### Production

```bash
npm run build
```

```bash
npm start
```

Express serves the built client from `client/dist` on port 3001.

## Configuration

| Env var     | Default            | Purpose                       |
|-------------|--------------------|-------------------------------|
| `PORT`      | `3001`             | API port                      |
| `ADMIN_USERNAME` | `admin`       | Username for the first admin account |
| `ADMIN_PASSWORD` | `admin123`    | Password for the first admin account. **Change this.** |
| `DB_PATH`   | `server/nets.db`   | SQLite file                   |
| `SMTP_HOST` | *(unset)*          | Mail server. If unset, emails are saved to `server/outbox/*.eml` instead of sent |
| `SMTP_PORT` | `587`              | Use `465` with `SMTP_SECURE=true` |
| `SMTP_SECURE` | `false`          | `true` for implicit TLS (port 465) |
| `SMTP_USER` / `SMTP_PASS` | *(unset)* | SMTP login |
| `MAIL_FROM` | `Cricket Nets <no-reply@localhost>` | Sender shown to customers |
| `APP_URL`   | `http://localhost:5173` | Base URL used for links in emails |
| `VENUE_NAME` | `Aztecs Cricket Club` | Name shown in emails |

### Sending real email

Any SMTP provider works (Gmail with an app password, Microsoft 365, SendGrid, Mailgun, Amazon SES, etc.). For example, with Gmail in PowerShell:

```powershell
$env:SMTP_HOST="smtp.gmail.com"; $env:SMTP_PORT="465"; $env:SMTP_SECURE="true"
$env:SMTP_USER="you@gmail.com"; $env:SMTP_PASS="<16-character app password>"
$env:MAIL_FROM="Cricket Nets <you@gmail.com>"
npm run server
```

Without `SMTP_HOST`, open the `.eml` files in `server/outbox/` with Outlook, Thunderbird or Windows Mail to see exactly what would have been sent.

The admin account is created only when the `admins` table is empty, so the variables take effect on first run. To change an existing admin's password, delete its row from `admins` and restart with the new values.

Opening hours, the customer and staff booking windows (14 and 365 days), the max hours per booking and the guest booking cap are in `server/config.js`. Nets are seeded on first run in `server/lib/db.js`. Edit the `nets` table to change names, surfaces or prices, or set `active = 0` to hide a net.

## Tests

```bash
npm test
```

These run the API against an in-memory database with a fixed clock. They cover registration and login, password hashing, sessions, rate limiting, availability, past-slot blocking, double-booking, validation, customers only seeing and cancelling their own bookings, and admin sign-in and summary.

### Local test accounts

Created in the local dev database (`server/nets.db` only) while building the app: customer `demo@example.com` / `demo-pass-123`, and coach `coach.sam` / `coach-pass-123`. The admin is `admin` / `admin123` unless `ADMIN_PASSWORD` was set on first run.

## API

| Method | Path | Notes |
|---|---|---|
| GET  | `/api/config` | Opening hours, booking window |
| GET  | `/api/nets` | Active nets |
| GET  | `/api/availability?date=YYYY-MM-DD` | Slots per net: `free` / `booked` / `past` / `blocked` |
| GET  | `/api/availability/month?month=YYYY-MM` | Per day: `total`, `free`, `booked`, `blocked`, `past` hours (for the Month view) |
| POST | `/api/auth/register` | `{ name, email, phone, password }`, signs in |
| POST | `/api/auth/login` | `{ email, password }` |
| POST | `/api/auth/logout` | |
| GET  | `/api/auth/me` | `{ user }` (includes `emailVerified`) or `{ user: null }` |
| PATCH | `/api/auth/me` | Customer session. `{ name, phone }` |
| POST | `/api/auth/password` | Customer session. `{ currentPassword, newPassword }` |
| POST | `/api/auth/forgot` | `{ email }`. Always `{ ok: true }`; emails a reset link if the account exists |
| POST | `/api/auth/reset` | `{ token, password }` from a reset or set-password link. Sets the password, confirms the email, ends other sessions, signs in |
| POST | `/api/auth/verify` | `{ token }` from a confirm-email link |
| POST | `/api/auth/resend-verification` | Customer session. Sends a new confirm-email link |
| POST | `/api/bookings` | `{ netId, date, hours: [18,19] }` books for the signed-in customer. Add `guest: { name, email, phone }` to book as a guest (no session needed); the response then includes `manageToken` |
| GET  | `/api/bookings` | Customer session. Your bookings |
| POST | `/api/bookings/:ref/cancel` | Customer session. Only your own bookings |
| POST | `/api/guest-bookings/:ref/view` | `{ token }` from the manage link |
| POST | `/api/guest-bookings/:ref/cancel` | `{ token }` from the manage link |
| POST | `/api/admin/login` | `{ username, password }` |
| POST | `/api/admin/logout` | |
| GET  | `/api/admin/me` | `{ admin }` or `{ admin: null }` |
| GET  | `/api/admin/bookings?date=…` or `?from=…` | Admin session. One day, or that day onwards |
| POST | `/api/admin/bookings/:ref/cancel` | Admin session |
| GET  | `/api/admin/customers?q=…` | Admin session. Search members by name, email or phone (min 2 characters, max 10 results) |
| POST | `/api/admin/bookings` | Admin session. `{ netId, date, hours, userId }` for a member, or `{ netId, date, hours, guest: { name, email, phone } }` |
| GET  | `/api/admin/reports?period=day\|week\|month&date=…` | Admin session. Figures for the period containing `date` (default today), plus the previous period |
| GET  | `/api/admin/staff` | Admin only. Staff accounts with roles |
| POST | `/api/admin/staff` | Admin only. `{ username, role: 'admin'|'coach', password }` |
| PATCH | `/api/admin/staff/:id` | Admin only. `{ role }` and/or `{ password }` |
| DELETE | `/api/admin/staff/:id` | Admin only |
| GET  | `/api/admin/blocks` | Admin session. Current and upcoming blocks |
| POST | `/api/admin/blocks` | Admin session. `{ netId (null = all), startDate, endDate, startHour, endHour, weekdays: [0-6, Mon=0], reason, cancelClashes }`. 409 with `clashes` if bookings overlap and `cancelClashes` isn't true |
| DELETE | `/api/admin/blocks/:id` | Admin session |
| GET  | `/api/admin/members?q=…` | Admin session. Members with `upcoming` and `total` booking counts (max 200) |
| POST | `/api/admin/members` | Admin session. `{ name, email, phone, password }`, or `{ name, email, phone, sendInvite: true }` to email a set-password link |
| POST | `/api/admin/members/:id/password-link` | Admin session. Emails the member a set-password link |
| PATCH | `/api/admin/members/:id` | Admin session. Any of `{ name, email, phone, password }` |
| DELETE | `/api/admin/members/:id` | Admin session. Returns `{ cancelled }`, the number of upcoming bookings cancelled |
