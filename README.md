# InAmigos Foundation — Operations ERP

**An operations platform for an NGO: volunteers, CSR funding, initiatives and applications in one place.**

> 🔗 **Live:** [in-amigos-foundation-ai-application-ecru.vercel.app](https://in-amigos-foundation-ai-application-ecru.vercel.app)

Small NGOs run on spreadsheets and WhatsApp. Volunteer applications arrive in three places, CSR proposals live in email threads, and nobody can answer "how much did we actually spend on this initiative?" without a week of digging. This project centralises that into a single operations system with role-based access, an audit trail, and AI-assisted triage for incoming applications.

---

## What it covers

| Area | What it does |
| --- | --- |
| **Applicants directory** | Searchable, sortable table of volunteer applications with detail views |
| **AI triage** | Incoming applications are scored and routed by skill fit, so coordinators review the right ones first |
| **Volunteer portal** | Self-service view for volunteers to track their applications and assignments |
| **Initiative tracking** | Kanban board for running initiatives, with a live monitor for progress and staffing |
| **CSR / corporate funding** | Funding pipeline with proposal detail views and a corporate panel |
| **Analytics** | Aggregated bars across applications, initiatives and funding sources |
| **Audit trail** | Every state change recorded, so decisions are traceable |
| **Auth** | Session-based authentication with an admin/coordinator split |

---

## Architecture

```
inAmigos_foundation_ai-application_ERP/
├── backend/
│   ├── config/supabase.js          Supabase client
│   ├── models/schema.sql           Postgres schema
│   ├── routes/
│   │   ├── auth.js                 Login, session
│   │   ├── volunteers.js           Applications, assignments
│   │   └── admin.js                Admin-only operations
│   ├── services/aiTriageService.js Application scoring and routing
│   ├── data/mockData.js            Seed data
│   └── server.js
├── frontend/                       React + Vite
│   └── src/
│       ├── pages/                  Applicant, initiative, CSR, audit, analytics views
│       ├── components/             Kanban board, tables, analytics bars, sidebar, forms
│       └── services/api.js         API client
└── vercel.json                     Deployment config
```

**Data layer:** Supabase (Postgres) with the schema versioned in `models/schema.sql`, so the database can be stood up from the repository alone.

---

## Running locally

**Prerequisites:** Node.js 18+, a Supabase project

```bash
# backend
cd backend
npm install
# add SUPABASE_URL and SUPABASE_KEY to .env
npm run dev

# frontend
cd frontend
npm install
npm run dev
```

Run `backend/models/schema.sql` against your Supabase instance, then load the seed data.

---

## Stack

**Frontend** — React, Vite, custom CSS design system
**Backend** — Node.js, Express, Supabase (Postgres), AI triage service
**Deploy** — Vercel

## License

MIT
