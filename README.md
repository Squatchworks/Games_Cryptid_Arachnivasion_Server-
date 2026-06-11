# Games Cryptid Arachnivasion Server

![Language](https://img.shields.io/badge/language-Python%203-blue)
![Framework](https://img.shields.io/badge/framework-Flask-lightgrey)
![Database](https://img.shields.io/badge/database-PostgreSQL%20via%20Supabase-green)
![Hosting](https://img.shields.io/badge/hosting-Render-purple)

The backend REST API powering the **Arachnivasion global leaderboard**. Built with **Python and Flask**, backed by a **Supabase PostgreSQL** database, and hosted on **Render**.

> Companion to [Games_Cryptid_Arachnivasion](https://github.com/Squatchworks/Games_Cryptid_Arachnivasion)

---

## Endpoints

### `GET /highscores`
Returns the top 10 scores ordered by score descending, then by submission time ascending for tiebreaking.

**Response:**
```json
{
  "highscores": [
    { "initials": "DEV", "score": 50000 },
    { "initials": "JDB", "score": 1360 }
  ]
}
```

### `POST /submit`
Submits a new score. Initials are automatically uppercased and clamped to 3 characters.

**Request body:**
```json
{ "initials": "JDB", "score": 1360 }
```

**Response:**
```json
{ "ok": true }
```

**Error cases:**
- Missing or malformed body → `400 Invalid payload`
- Non-integer score → `400 Score must be an integer`
- Negative score → `400 Score must be >= 0`

---

## Architecture
ArachnivasionServer/
├── server.py # Flask app — GET /highscores, POST /submit
├── requirements.txt # flask, gunicorn, psycopg[binary]
└── highscores.json # Legacy local fallback (pre-Supabase dev reference)


`get_conn()` reads `DATABASE_URL` from the environment — set by Render and pointed at the Supabase PostgreSQL instance. All DB operations use `psycopg` (v3) with SSL required. Gunicorn is used as the production WSGI server on Render.

---

## Why This Architecture

This server went through three iterations before landing on the current solution:

1. **Local Flask only** — worked on a shared LAN but inaccessible over the internet and reset on restart.
2. **Flask on Render (free tier, in-memory)** — internet-accessible but Render's free tier spins down on inactivity; leaderboard data was lost on each wake.
3. **Flask on Render + Supabase PostgreSQL** — the final solution. Supabase provides a persistent hosted PostgreSQL database that survives server sleep, restarts, and redeployments. Render handles the HTTP layer; Supabase handles persistence.

---

## Local Development

### Prerequisites
- Python 3.10+
- A Supabase project with a `public.highscores` table:

```sql
create table public.highscores (
  id serial primary key,
  initials text not null,
  score integer not null,
  created_at timestamptz default now()
);
```

### Setup

```bash
git clone https://github.com/Squatchworks/Games_Cryptid_Arachnivasion_Server.git
cd Games_Cryptid_Arachnivasion_Server/ArachnivasionServer
pip install -r requirements.txt
export DATABASE_URL="postgresql://..."   # your Supabase connection string
python server.py
```

Server runs on `http://localhost:5000` by default.

### Production (Render)

Set `DATABASE_URL` as an environment variable in the Render dashboard pointing to your Supabase database. Render uses `gunicorn` as the start command: gunicorn server:app


---

## Author

**Jacob Blackburn** — designed and built the full leaderboard pipeline including Flask server, Render deployment, and Supabase persistence integration.
- GitHub: [@Squatchworks](https://github.com/Squatchworks)
- LinkedIn: [linkedin.com/in/squatchworks](https://linkedin.com/in/squatchworks)
