
# 🧰 Data Engineering Debugging Playbook

> A growing playbook of real-world data engineering issues and how I solved them — covering Docker, Airflow, dbt, Snowflake, Great Expectations, and more.  
> Built from hands-on experience with hospital pipelines, orchestrations, and production-level workflows.

This is intended as both a personal reference and a collaborative guide.  
**Feel free to contribute!** Add your own issues and fixes by submitting a pull request. ✨

---

## 📦 Docker & Containers

### ❌ Issue: Airflow or dbt container not starting

**Symptoms**:  
- `airflow_webserver` missing in `docker ps -a`  
- `dbt` container exits immediately

**Fixes**:  
- Added `airflow-init` container in `docker-compose.yml` for initialization.  
- Created persistent entrypoint for dbt containers:  
  ```dockerfile
  ENTRYPOINT ["tail", "-f", "/dev/null"]
  ```

---

## ⚙️ Prefect + Docker + PostgreSQL Pipeline Issues

These logs were generated while running a COVID-19 ETL pipeline using Prefect, Docker, and PostgreSQL locally.

### 🔧 Error Resolution Table

| Error Type           | Log Example / Symptom                     | Fix                                  | Outcome                |
|----------------------|-------------------------------------------|--------------------------------------|------------------------|
| Docker Compose Warn  | attribute 'version' is obsolete           | Removed version line                 | Clean startup          |
| Prefect CLI Error    | No such command 'orion'                   | Switched to `prefect server start`   | UI up and running      |
| Python NameError     | name 'URL_CONFIRMED' is not defined       | Moved config vars to top             | No more crashes        |
| Docker Path Missing  | File not found in container               | Checked/fixed Docker mounts          | Data files visible     |
| SSL/Email Error      | ssl.SSLError: WRONG_VERSION_NUMBER        | Used port 587 + TLS in block config  | Emails send!           |
| Success Log          | Inserted 330327 rows...                   | N/A                                  | Pipeline complete!     |

---

### 🪵 Additional Learnings

- Always check container logs using `docker compose logs -f [service]`.
- Prefect logs are your best friend — look for `"task run"` and `"flow run"` markers.
- Not all errors are fatal — some are deprecation warnings (e.g., Docker `version` field).

### 📫 Email Notifications

- Prefect email blocks can silently fail if misconfigured — double-check ports (`587` for TLS) and credentials.
- Test email delivery separately before adding to pipeline logic.

---

## 💡 Tips for New Data Engineers

- Don’t fear the logs — read them line-by-line like a mystery novel.
- Most bugs are either: a) missing paths, b) wrong configs, or c) ordering issues in Docker.
- Pipelines fail silently if notifications aren't in place — always configure alerting before deploying.

---

## 🧠 Final Note

This playbook is just the beginning. Debugging is where you truly become a Data Engineer — not just writing perfect code, but learning how to **fix** broken systems.

Stay caffeinated ☕, keep calm, and follow the logs.  
