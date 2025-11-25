# Resume Insight AI

AI-powered resume parsing, coding-profile analytics, and guided career planning for job seekers, trainers, and placement teams. The platform combines a FastAPI backend, a React dashboard, and multiple AI services to turn a single resume upload into actionable insights.

---

## Highlights

- Automated PDF ingestion with ATS scoring, section coverage, keyword density, and word-count heuristics.
- One-click aggregation of GitHub, LeetCode, and CodeChef profiles (with activity graphs and PR analytics).
- AI services for personalized guidance, certificate/project evaluation, and fresher-role prediction (OpenRouter + Hugging Face).
- Role-based dashboard (admin, trainer, user) with protected routes and history tracking.
- Portfolio generator that turns structured resume data into a ready-to-share profile.

---

## Tech Stack

| Layer      | Technologies                                                                 |
|------------|------------------------------------------------------------------------------|
| Frontend   | React 19, React Router, Chart.js, Tailwind utilities, ProtectedRoute wrapper |
| Backend    | FastAPI, Uvicorn, MongoDB (Atlas/local), pdfplumber, requests, bcrypt        |
| AI/3rd Party | OpenRouter (LLM), Hugging Face Inference, GitHub GraphQL & REST APIs       |
| Tooling    | npm / React Scripts, pip, virtualenv                                         |

---

## Repository Layout

```
FINAL-1/
├── backend1/
│   ├── app/
│   │   ├── main.py                # FastAPI entrypoint & router wiring
│   │   ├── config.py              # Env + external client bootstrap
│   │   ├── helpers/               # Resume, GitHub, LeetCode, CodeChef, AI helpers
│   │   └── routes/                # Feature routers (auth, resume, analyze_all, guidance, etc.)
│   ├── requirements.txt
│   └── run.py                     # Optional script alias (use `uvicorn app.main:app`)
├── frontend-react/                # React SPA (Home, Admin, Trainer, User dashboards)
│   ├── src/
│   │   ├── pages/                 # Route-level screens
│   │   ├── components/            # ProtectedRoute + UI primitives
│   │   └── hooks/useAuth.js       # JWT verification hook
├── package.json                   # Root utilities (jspdf helper)
└── README.md                      # This guide
```

---

## Quick Start

### Prerequisites

- **Python 3.10+** with `pip`
- **Node.js 18+** with `npm` (or `yarn`)
- **MongoDB** cluster/instance (connection string via `MONGO_URI`)
- API keys: OpenRouter (optional but recommended), Hugging Face (optional), GitHub PAT (required for GitHub metrics)

### 1. Backend (FastAPI)

```bash
cd backend1
python -m venv .venv
.venv\Scripts\activate         # Windows
# source .venv/bin/activate    # macOS/Linux
pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env           # create if the sample does not exist
# update .env with your secrets
uvicorn app.main:app --reload --port 8000
```

Key routers mount under `http://127.0.0.1:8000` and CORS already allows the React dev server (`http://localhost:3000`).

### 2. Frontend (React)

```bash
cd frontend-react
npm install
npm start
```

The SPA expects the backend on `http://127.0.0.1:8000`. For production, configure environment-specific API URLs (e.g., via `.env` files and proxy settings).

---

## Environment Variables (backend `.env`)

| Variable | Required | Description |
|----------|----------|-------------|
| `SECRET_KEY` | ✅ | JWT signing key for auth tokens. |
| `JWT_ALGORITHM` | ⛔ (default `HS256`) | Signing algorithm. |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | ⛔ (default `60`) | JWT expiry window. |
| `MONGO_URI` | ✅ | Connection string used by all routes (`users`, `portfolio`, etc.). |
| `MONGO_DB_NAME` | ⛔ (default `resume_analyzer`) | Database name. |
| `OPENROUTER_API_KEY` | ⚠️ | Enables certificate, project, and guidance AI features. |
| `GITHUB_TOKEN` | ⚠️ | Required for GraphQL + PR metrics; PAT needs `repo` or `public_repo` scope. |
| `HUGGINGFACE_API_KEY` | ⚠️ | Optional, improves role prediction latency/limits. |
| `HUGGINGFACE_MODEL` | ⛔ | Defaults to `bhadresh-savani/distilbert-base-uncased-emotion`. |
| `FRONTEND_URL` | ⛔ | Use if you extend CORS beyond `http://localhost:3000`. |

> Tip: Keep `.env` inside `backend1/` (next to `run.py`). `app/config.py` automatically loads it.

Example snippet:

```
SECRET_KEY=change-me
MONGO_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/?retryWrites=true&w=majority
MONGO_DB_NAME=resume_insight_ai
OPENROUTER_API_KEY=sk-or-...
GITHUB_TOKEN=ghp_...
HUGGINGFACE_API_KEY=hf_...
```

---

## Feature Overview

- **Resume Parsing (`resume_routes`)** — pdfplumber extraction, ATS scoring (`calculate_ats_score`), language normalization, and skill extraction.
- **Platform Analytics** — GitHub repo counts, contributions, and PR stats; LeetCode GraphQL profile + submission heatmap; CodeChef scraping for rankings, badges, and learning paths.
- **`/analyze_all` Orchestrator** — single multipart request combining resume upload with GitHub/LeetCode/CodeChef handles for a consolidated dashboard payload.
- **Guidance + Evaluations** — OpenRouter-powered certificate/project evaluations and detailed learning paths (`guidance_routes`, `guidance_helper`).
- **AI Role Predictor** — Hugging Face Inference + keyword fallback to recommend fresher job titles (`role_predictor.py`).
- **User/Trainer/Admin Dashboards** — Protected routes in React with `useAuth` verifying JWTs via `/auth/verify_token`. Admin screens show history, charts, and messaging to users.
- **Portfolio Generator** — Converts stored resume data into a portfolio document saved in MongoDB (`/user/portfolio`).

---

## API Reference (selected)

| Method | Endpoint | Description | Notes |
|--------|----------|-------------|-------|
| `POST` | `/resume/upload_resume` | Upload PDF resume → structured data + ATS metrics. | multipart/form-data (`file`). |
| `POST` | `/analyze_all` | Resume upload + optional `github`, `leetcode`, `codechef` query params. | Returns combined JSON response. |
| `GET`  | `/github/analyze_github/{user_input}?token=...` | GitHub GraphQL + PR metrics. | `token` query or `GITHUB_TOKEN`. |
| `GET`  | `/leetcode/analyze_leetcode/{username}` | LeetCode stats + activity graph. | No auth needed. |
| `GET`  | `/codechef/analyze_codechef/{username}` | Scrapes rating, ranks, badges. | No auth needed. |
| `POST` | `/guidance/generate` | Returns LLM guidance object. | Body `{ "resume_data": {...} }`. |
| `POST` | `/predict-role` | Hugging Face role prediction. | Body `{ "inputs": "<skills + summary>" }`. |
| `POST` | `/auth/login` | Email/password login (supports plain + bcrypt). | Returns JWT + sanitized user. |
| `POST` | `/auth/register` | Create user with hashed password and role. | Roles: `admin`, `trainer`, `user`. |
| `POST` | `/auth/set_password` | Upsert password + optional role change. | Useful for seeding trainers/admins. |
| `GET`  | `/auth/verify_token` | Validates JWT (used by `useAuth`). | Send `Authorization: Bearer <token>`. |
| `GET`  | `/user/info/{email}` | Fetches user profile sans password. | Admin/trainer dashboards. |
| `POST` | `/user/upload_resume` | Stores resume results against user email. | Form data with `email` + `file`. |
| `GET`  | `/user/history/{email}` | Read past ATS results and suggestions. | |

> Explore the remaining routers inside `backend1/app/routes/` for admin filters, AI chat helpers, and guidance extensions.

---

## Frontend Routes

- `/` Home landing page with CTA to log in.
- `/login` Authentication form (JWT stored in `localStorage`).
- `/admin` Resume analytics dashboard (charts, upload history, coding-platform fetchers).
- `/admin_resume_filter` Advanced filtering UI for resumes.
- `/trainer`, `/user` Role-specific portals using the same backend data.
- `/features`, `/about`, `/contact`, `/portfolio` Marketing/supporting pages.

Routing is powered by React Router v7 with a reusable `<ProtectedRoute allowedRoles={[]}>` wrapper that redirects unauthorized users.

---

## Scripts & Tooling

| Location | Command | Purpose |
|----------|---------|---------|
| `frontend-react/` | `npm start` | React dev server on `localhost:3000`. |
| `frontend-react/` | `npm run build` | Production build. |
| `frontend-react/` | `npm test` | CRA test runner placeholder. |
| `backend1/` | `uvicorn app.main:app --reload` | FastAPI dev server. |
| `backend1/` | `pytest` *(if added)* | Testing hook (no tests bundled yet). |

Root-level `package.json` currently exposes `jspdf` for potential resume downloads; extend as needed.

---

## Troubleshooting

- **MongoDB connection fails** — ensure IP allowlist is updated for Atlas clusters; verify `MONGO_URI` and `MONGO_DB_NAME` in `.env`.
- **OpenRouter / Hugging Face unavailable** — helpers automatically fall back to deterministic guidance or heuristics, but you will lose AI-enhanced messaging.
- **GitHub API errors** — supply a Personal Access Token with scope `public_repo`. Rate limit handling truncates PR metrics when `X-RateLimit-Remaining < 5`.
- **CORS issues** — update `origins` inside `app/main.py` or set `FRONTEND_URL` to match your deployment hostnames.
- **LeetCode/CodeChef scraping** — subject to HTML changes. Monitor helper modules if responses start failing.

---

## Contributing

1. Fork & clone the repo.
2. Create a branch: `git checkout -b feat/<short-name>`.
3. Run backend + frontend locally and add tests/docs where relevant.
4. Submit a PR describing the feature, setup impact, and testing performed.

Issues and feature requests are welcome—please include environment details, reproduction steps, and screenshots/logs when available.

---

## License

Specify the license (e.g., MIT) in a `LICENSE` file. Update this section once finalized.

---

## Contact

Built by **Sri Varsha, Yuvashri, Arjun Hareesh & Manoj**. For inquiries, reach out via the contact info embedded in the frontend footer or create an issue in this repository.
