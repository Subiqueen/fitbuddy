# FitBuddy - AI Fitness Plan Generator (FastAPI + Gemini)

Generates a personalized 7-day workout plan (Gemini Pro), a nutrition/recovery tip (Gemini Flash),
lets users refine the plan with feedback, and gives coaches an admin view. Data is stored in SQLite.

## Project structure
```
fitbuddy/
├── app/
│   ├── main.py                  # FastAPI app + startup (creates DB tables)
│   ├── routes.py                # HTML pages + JSON API
│   ├── config.py                # .env / settings
│   ├── database.py              # SQLAlchemy engine + CRUD (save_user, save_plan, update_plan ...)
│   ├── models.py                # User, Plan tables
│   ├── schemas.py               # UserInput, FeedbackRequest (Pydantic)
│   ├── gemini_client.py         # Google Gen AI SDK wrapper (retries, cleanup)
│   ├── gemini_generator.py      # generate_workout_gemini()            -> Gemini Pro
│   ├── gemini_flash_generator.py# generate_nutrition_tip_with_flash()  -> Gemini Flash
│   ├── updated_plan.py          # update_workout_plan()                -> Gemini Pro
│   └── mock_ai.py               # offline fallback (no API key / MOCK_AI=true)
├── templates/                   # base, index, result, feedback, all_users (Jinja2)
├── static/css, static/images    # styling + background
├── tests/                       # pytest suite (runs offline)
├── .env.example  requirements.txt  requirements-dev.txt  pytest.ini
└── .vscode/launch.json          # press F5 to run/debug
```

## Setup in VS Code
1. Install **Python 3.10+** and **VS Code** with the *Python* extension.
2. `File > Open Folder...` and choose the `fitbuddy` folder. Open a terminal: `` Ctrl+` ``.
3. Create and activate a virtual environment:
   - Windows (PowerShell): `python -m venv venv` then `venv\Scripts\activate`
   - macOS/Linux: `python3 -m venv venv` then `source venv/bin/activate`
   - If PowerShell blocks activation: `Set-ExecutionPolicy -Scope Process Bypass`
4. `Ctrl+Shift+P` > **Python: Select Interpreter** > pick the one inside `venv`.
5. Install dependencies: `pip install -r requirements.txt`
6. Create your `.env`: copy `.env.example` to `.env` and set `GOOGLE_API_KEY=` (free key: https://aistudio.google.com/app/apikey).
   No key? Leave it as is, or set `MOCK_AI=true`, and the app runs in offline demo mode.

## Run
```
uvicorn app.main:app --reload
```
(or press **F5** in VS Code). Always run from the `fitbuddy` folder.

- App: http://127.0.0.1:8000
- Interactive API docs: http://127.0.0.1:8000/docs
- Admin dashboard: http://127.0.0.1:8000/view-all-users

## Test
Automated (offline, uses mock AI and a temporary database):
```
pip install -r requirements-dev.txt
pytest -v
```
Manual:
1. Open http://127.0.0.1:8000, fill the form, click **Generate Plan** -> user info, 7-day plan, nutrition tip.
2. Type feedback (e.g. "include more rest days") -> **Update My Plan** -> green confirmation; original plan stays under "Show original plan".
3. Open **All Users** -> your row shows original and updated plans; **Delete** removes it.
4. In `/docs`, try `POST /api/generate-workout` with:
   `{"username":"Ravi","user_id":"ravi_9","age":30,"weight":80,"goal":"weight loss","intensity":"high"}`
5. Check `http://127.0.0.1:8000/health` - `"mock_mode": false` means real Gemini is active.

## Troubleshooting
- **"Gemini request failed ... NOT_FOUND"**: the model name was retired/renamed. Set `GEMINI_PRO_MODEL` / `GEMINI_FLASH_MODEL` in `.env` to a current model (default `gemini-2.5-pro` / `gemini-2.5-flash`; see https://ai.google.dev/gemini-api/docs/models).
- **"API key not valid"**: re-check `GOOGLE_API_KEY` in `.env`, then restart the server.
- **`ModuleNotFoundError: app`**: run commands from the project root, not inside `app/`.
- **Reset data**: stop the server and delete `fitbuddy.db`.
- The admin pages have no login; keep this app local, or add authentication before deploying.
