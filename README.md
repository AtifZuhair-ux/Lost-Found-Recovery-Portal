# 🎒 VIT Campus Lost & Found Recovery Portal

Replaces messy WhatsApp groups with a private, verified lost-and-found workflow for the VIT community.

**Stack:** Node.js (≥18) · Express · SQLite (`better-sqlite3`) · vanilla JS single-page frontend (no build step).

## Features
- Separate **Lost** / **Found** feeds, search + category + venue filters
- Locations restricted to official landmarks: SJT, TT, PRP, SMV, MB, GDN, CDMM · MH-A…MH-T · LH-A…LH-J · Gazebo, Food Mall, DC · Central Library · Sports Complex
- Categories: ID Cards, Room Keys, Calculators, Lab Equipment, Earphones, Wallets
- **Privacy:** the public API never returns a poster's name, registration number, or email; phone numbers are never collected. Posters and claimants appear only as anonymous roles (e.g. "Anonymous Finder", "Claimant #4").
- **Ownership verification:** the finder sets a custom challenge question → claimant submits a Claim Verification Request → finder approves/rejects in a private dashboard
- **Handoff chat:** opens only for approved claims, visible only to the two parties; propose a meetup checkpoint (SJT Ground Floor Reception, Central Library Security Desk, …)
- **Lifecycle:** either party can mark an item **Resolved**, removing it from active boards and closing the chat
- Server-side validation with field-level errors; UI loading skeletons, empty states, and submit/disabled states; login throttling; bcrypt password hashing; httpOnly JWT cookie; all output HTML-escaped

## Local setup
```bash
git clone <your-repo-url> && cd vit-lost-found
npm install
cp .env.example .env        # optional; or export JWT_SECRET=... in your shell
npm run db:init             # creates data/lostfound.db and tables
npm run db:seed             # optional demo users + listings
npm start                   # http://localhost:3000
```
Demo logins (after seeding): `25BIT0001` and `25BIT0002`, password `password123`.

Environment variables: `PORT` (default 3000), `JWT_SECRET` (**set this outside local dev**), `DB_FILE` (default `./data/lostfound.db`).
The server also auto-creates tables on start, so `db:init` is optional.

> `.env` isn't auto-loaded; use `node --env-file=.env server.js` (Node 20+) or export variables.

## Try the full flow
1. Log in as `25BIT0001`, open the "Student ID card" listing → Dashboard.
2. In a private window log in as `25BIT0002`, open that listing, submit an answer.
3. Back as `25BIT0001`, approve it in Dashboard → open chat, propose a checkpoint.
4. Either side clicks **Mark resolved**; the item disappears from the feed.

## API summary
| Method | Route | Notes |
|---|---|---|
| POST | `/api/auth/register`, `/login`, `/logout` · GET `/me` | VIT regno + `@vitstudent.ac.in` email |
| GET/POST | `/api/items` | public feed (active only) / create |
| GET | `/api/items/:id` | public view, no poster identity |
| POST | `/api/items/:id/claims` | claim verification request |
| POST | `/api/items/:id/resolve` | poster or approved claimant |
| GET | `/api/dashboard` | my listings (with claims) + my requests |
| POST | `/api/claims/:id/decision` | poster only: `approved`/`rejected` |
| GET/POST | `/api/claims/:id/thread`, `/messages`, `/checkpoint` | approved parties only |

## Project layout
`server.js` API · `db.js` + `db/schema.sql` database · `constants.js` venues/categories/checkpoints · `scripts/init-db.js` bootstrap · `public/` frontend.

## Notes / limitations
Registration is not email-verified (add OTP via SMTP for production). Chat uses 6 s polling rather than WebSockets to stay dependency-light.
