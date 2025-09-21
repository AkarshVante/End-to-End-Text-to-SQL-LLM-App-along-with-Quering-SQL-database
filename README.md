# End-to-End Text-to-SQL LLM App (Gemini Pro)

Convert natural-language questions into SQL, run them safely against a database, and return human-friendly answers — powered by Google’s Gemini models using the official `google-genai` Python SDK.

This repository is a compact, practical scaffold for building a Text→SQL pipeline with safe execution and easy integration into a CLI, API, or lightweight web UI.

---

## Highlights

* Use Gemini (via `google-genai`) to generate SQL from plain English.
* Support for SQLite (local dev) and adapters for PostgreSQL/MySQL.
* Safety & validation layer to reduce destructive model-generated queries.
* Pluggable design: LLM client, prompt templates, DB access, and execution are separated for clarity.

---

## Tech snapshot

* Language: **Python**
* Primary SDK: **google-genai** (Gemini models)
* DB access: **SQLAlchemy** (compatible with SQLite/Postgres/MySQL)
* Optional: **LangChain** SQL utilities for schema introspection and richer prompt composition
* License: **MIT**

---

## Quickstart — get running locally

1. Clone the repo and enter the folder.

2. Create & activate a virtual environment, then install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate   # macOS/Linux
# .\.venv\Scripts\activate  # Windows PowerShell
pip install -r requirements.txt
```

3. Configure environment variables (recommended):

```bash
export GEMINI_API_KEY="YOUR_API_KEY"         # macOS / Linux
export DATABASE_URL="sqlite:///data/example.db"
```

4. Run the example CLI or start the API (see `src/app.py` for examples).

---

## Minimal Gemini usage (example)

```python
from google import genai

client = genai.Client()  # reads GEMINI_API_KEY from environment by default

resp = client.models.generate_content(
    model="gemini-2.5-flash",
    contents="Write a 1-line explanation of how attention works in transformers"
)
print(resp.text)
```

---

## How it works (pipeline)

1. **Input** — user asks a question in plain language.
2. **Context** — fetch schema and (optionally) column samples from the target DB.
3. **Prompting** — compose a prompt that includes schema + instructions for safe SQL generation.
4. **Generation** — Gemini produces a candidate SQL string.
5. **Validation** — static safety checks (deny `DROP/DELETE`), ensure only allowed tables/columns used, parameterize if needed.
6. **Execution** — run the SQL in a controlled session/transaction and collect results.
7. **Summarization** — optionally ask Gemini to summarize or convert results into a natural-language answer.

---

## Important patterns & safety

* **Allow-listing**: explicitly allow which tables and columns the app can query.
* **Deny-listing**: block destructive or administrative SQL (`DROP`, `DELETE`, `ALTER`, `CREATE` except read-only metadata queries).
* **Test queries**: run generated queries against a read-only replica or inside a transaction that gets rolled back during development.
* **Parameterization**: prefer parameterized queries; avoid direct string interpolation of user inputs into SQL.

---

## Suggested project layout

```
.
├── README.md
├── requirements.txt
├── .env.example
├── src/
│   ├── app.py                # CLI or FastAPI entrypoint
│   ├── llm/
│   │   ├── client.py         # genai client wrapper
│   │   └── prompts.py        # prompt templates & helpers
│   ├── db/
│   │   ├── connection.py     # create engine/session from DATABASE_URL
│   │   └── schema.py         # fetch and format schema context
│   └── pipeline/
│       ├── generate_sql.py   # NL -> SQL
│       ├── validate_sql.py   # safety + allow-list checks
│       ├── execute_sql.py    # run query and return results
│       └── summarize.py      # optional NL summarization of results
└── tests/
```

---

## Example prompt template

```
# Provide the DB schema first (table names, columns, and types).
# Instruct: "Produce a single, valid SQL SELECT query using only the listed tables and columns. Do not use DELETE/UPDATE/CREATE/DROP/ALTER. Use LIMIT when appropriate."

Schema:
<schema_here>

Question:
{user_question}

SQL:
```

---

## Starter `requirements.txt`

```
google-genai>=0.3.0
python-dotenv>=1.0.1
SQLAlchemy>=2.0.0
langchain>=0.2.0              # optional
```

---

## Roadmap (next small wins)

* Add a small FastAPI demo endpoint (`/ask`) and a minimal web UI.
* Scaffold an evaluation harness: test prompts → expected SQL → diffing for regressions.
* Implement cached schema summaries for large DBs to keep prompts small.

---

## Contributing

PRs welcome. Aim for small, testable changes. Include schema samples and example prompts for any SQL-related behavior you add.

---

## License

This project is released under the MIT License.

---
