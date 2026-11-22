<div align="center">

# InAmigos Foundation — Operations ERP

**Volunteers, funding, initiatives and audit trail in one system instead of four spreadsheets and a WhatsApp group.**

[![Live Demo](https://img.shields.io/badge/live-demo-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://in-amigos-foundation-ai-application-ecru.vercel.app)
[![React](https://img.shields.io/badge/React-Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://vitejs.dev)
[![Express](https://img.shields.io/badge/Express-Node-5FA04E?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://expressjs.com)
[![Supabase](https://img.shields.io/badge/Postgres-Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)](https://supabase.com)
[![License: MIT](https://img.shields.io/badge/license-MIT-2ea44f?style=for-the-badge)](LICENSE)

[Live app](https://in-amigos-foundation-ai-application-ecru.vercel.app) · [Report a bug](https://github.com/99Ayush/inamigos-erp/issues)

</div>

---

Small NGOs run on spreadsheets. Volunteer applications arrive across three channels, CSR proposals live in email threads, and nobody can answer *"how much did we actually spend on this initiative?"* without a week of digging.

This is an operations platform for that gap: one place for applications, initiatives, funding and accountability, with role-based access and an audit trail that records who changed what.

## What it covers

| Area | What it does |
| :--- | :--- |
| **Applicants directory** | Searchable, sortable table of volunteer applications with a detail view per applicant |
| **AI triage** | Each application is scored against the foundation's six initiatives and routed by skill fit, with a confidence score and written reasoning |
| **Volunteer portal** | Self-service view where volunteers track their own application and assignment |
| **Initiative tracking** | Kanban board for running initiatives plus a live monitor for progress and staffing |
| **CSR / corporate funding** | Funding pipeline with proposal detail views and a partner-facing panel |
| **Analytics** | Recharts visualisations across applications, initiatives and funding sources |
| **Audit trail** | Every state change recorded with actor, action and timestamp |
| **Roles** | Admin, partner and volunteer views with different reach |

### How the AI triage is built

`backend/services/aiTriageService.js` calls **Gemini 2.5 Flash** with a strict JSON response schema:

```text
application text ──► Gemini (structured output) ──► { recommended_project,
                                                      extracted_skills[],
                                                      match_confidence,
                                                      summary_reasoning }
                          │
                          └── 5s timeout / error / no key ──► FALLBACK_TRIAGE
                                                             (manual review, is_fallback: true)
```

Three deliberate choices:

- **The project list is an enum**, not free text — the model can only route to an initiative that exists.
- **A timeout is treated as a failure**, not a slow answer. Better to flag for manual review than block the coordinator's screen.
- **Fallback is explicit** (`is_fallback: true`) and surfaces in the UI, so nobody mistakes a fallback for a confident recommendation.

## Architecture

```text
inamigos-erp/
├── backend/
│   ├── server.js                  Express app, route mounting, error handler
│   ├── config/supabase.js         Supabase client
│   ├── models/schema.sql          Postgres schema — the DB is reproducible from the repo
│   ├── routes/
│   │   ├── auth.js                POST /api/v1/auth (login + signup, role resolution)
│   │   ├── volunteers.js          POST /api/v1/volunteers/apply
│   │   └── admin.js               Applications, CSR proposals, audit logs
│   ├── services/aiTriageService.js  Gemini triage + fallback contract
│   └── data/mockData.js           Seed data for a demo instance
├── frontend/                      React + Vite
│   └── src/
│       ├── pages/                 Applicants, application detail, initiatives, CSR,
│       │                          audit trail, analytics, volunteer portal, auth
│       ├── components/            Kanban board, high-density table, analytics bars,
│       │                          funding panel, sidebar, settings, volunteer form
│       └── services/api.js        API client
└── vercel.json
```

**Data layer:** Supabase (Postgres). The schema ships as `models/schema.sql`, so a fresh project can be stood up from the repository alone — no undocumented migrations, no "ask someone for the SQL".

## API

| Method | Endpoint | Purpose |
| :--- | :--- | :--- |
| `GET` | `/health` | Liveness |
| `POST` | `/api/v1/auth` | Login or signup (`action: "login" \| "signup"`) |
| `POST` | `/api/v1/volunteers/apply` | Submit a volunteer application |
| `GET` | `/api/v1/admin/applications` | All applications |
| `GET` | `/api/v1/admin/csr-proposals` | CSR funding proposals |
| `GET` | `/api/v1/admin/audit-logs` | Recorded state changes |
| `POST` | `/api/v1/admin/audit-logs` | Append an audit entry |

## Run locally

**Prerequisites:** Node.js 18+, a Supabase project, a Gemini API key (optional — triage falls back without one).

```bash
# backend
cd backend
npm install
# .env: SUPABASE_URL, SUPABASE_KEY, GEMINI_API_KEY
npm run dev          # http://localhost:5000

# frontend (separate terminal)
cd frontend
npm install
npm run dev          # http://localhost:5173
```

Then run `backend/models/schema.sql` against your Supabase instance and load the seed data from `backend/data/mockData.js`.

### Demo accounts

The live deployment runs on demo auth so reviewers can log in without provisioning anything:

| Role | Email | Password |
| :--- | :--- | :--- |
| Admin | `admin@inamigos.org` | `admin123` |
| Partner | `partner@inamigos.org` | `partner123` |

Volunteer signup is open. These are **hardcoded demo credentials, not production auth** — see the notes below.

## Honest scope

- **Auth is demo-grade.** Admin and partner credentials are hardcoded in `routes/auth.js`; volunteer sessions are resolved by email lookup. Real deployment needs Supabase Auth or an equivalent with hashed credentials, signed sessions and server-side authorisation on every admin route.
- **Seed data is shipped in the repo.** `mockData.js` exists so the demo is populated on first load; a production instance would migrate to tables only.
- **No payments.** The funding pipeline tracks proposals and commitments; it does not move money.
- **Single-tenant.** Built for one foundation, not multi-organisation hosting.

## Stack

**Frontend** — React, Vite, React Router, Recharts, Framer Motion, Lucide, custom CSS design system
**Backend** — Node.js, Express, Supabase (Postgres), `@google/genai`
**Deploy** — Vercel (frontend + serverless API via `vercel.json`)

## License

[MIT](LICENSE) · built by [Ayush Kumar](https://github.com/99Ayush)
