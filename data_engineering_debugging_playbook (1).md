# 🧰 Data Engineering Debugging Playbook (Part 2): Prefect + Docker + PostgreSQL Edition

> A continuation of my hands-on debugging logbook — now focused on real-time data pipelines built using **Prefect**, **Docker**, and **PostgreSQL**.
>  
> This companion to the original debugging playbook highlights the quirky, painful, and triumphant moments while deploying a production-style ETL pipeline for COVID-19 data.

---

## ⚙️ Prefect + Docker Troubleshooting

### ❌ Issue: Obsolete `version` attribute in Docker Compose

**Symptom**:  
Warning: `the attribute 'version' is obsolete, it will be ignored...`

**Fix**:  
Removed the `version:` line from the top of `docker-compose.yml`

**Outcome**:  
Clean startup. Docker no longer throws version warning.

---

### ❌ Issue: `orion` CLI command not found

**Symptom**:  
`No such command 'orion'` when running Prefect.

**Fix**:  
Used:  
```bash
prefect server start
```

**Outcome**:  
Local Prefect UI is now accessible at `http://localhost:4200`.

✅ **Tip**: Don’t blindly copy old Prefect tutorials — the CLI evolves!

---

### ❌ Issue: Python NameError for Config Variables

**Symptom**:  
`NameError: name 'URL_CONFIRMED' is not defined`

**Fix**:  
Moved all `URL_*` config variables to the top of the script so they’re accessible to all task blocks.

**Outcome**:  
Script no longer crashes during runtime.

---

### ❌ Issue: Email not sending via Prefect Email block

**Symptom**:  
Email task runs successfully but nothing received in inbox.

**Fix**:  
Created a Prefect Email block using **Gmail App Password**, **port 465**, and **SSL (not TLS)**:
```python
from prefect_email import EmailServerCredentials

block = EmailServerCredentials(
    username="baheldeepti@gmail.com",
    password="your-app-password",  # Gmail App Password
    smtp_server="smtp.gmail.com",
    smtp_port=465,
    use_tls=False  # Use SSL
)
block.save("my-emailer", overwrite=True)
```

**Outcome**:  
Success email now lands in inbox with subject “COVID Data Pipeline Success!”

✅ **Tip**: Port `587 + TLS = Fail` if you don’t configure it correctly. Use `465 + SSL` for Gmail if TLS causes SSL errors.

---

### ❌ Issue: File not visible in container

**Symptom**:  
Data not saving, and Prefect logs show missing path.

**Fix**:  
Added the following volume to `docker-compose.yml`:
```yaml
volumes:
  - ../data:/app/data
```

**Outcome**:  
`jhu_covid_global_*.csv` files now saved inside the container.

---

## 🧪 Data Quality / Postgres Load Debugging

### ❌ Issue: Large file loads slowly into Postgres

**Symptom**:  
Loading ~330,000 rows takes too long.

**Fix**:  
- Batched inserts via `executemany`
- Used `fillna("").values.tolist()` to handle nulls
- Set index on primary key `ID`

**Outcome**:  
Insert time reduced significantly. CPU load stabilized.

---

### ✅ Success Log

```log
Inserted 330327 rows into covid_summary.
Pipeline finished. 330327 rows loaded.
```

✅ **Tip**: Always log number of inserted rows to verify ETL success.

---

## 🧾 Error Resolution Table (Snapshot)

| Error Type           | Log Example / Symptom                     | Fix                                            | Outcome                |
|----------------------|-------------------------------------------|------------------------------------------------|------------------------|
| Docker Compose Warn  | attribute 'version' is obsolete           | Removed version line                           | Clean startup          |
| Prefect CLI Error    | No such command 'orion'                   | Switched to `prefect server start`             | UI up and running      |
| Python NameError     | name 'URL_CONFIRMED' is not defined       | Moved config vars to top                       | No more crashes        |
| Docker Path Missing  | File not found in container               | Checked/fixed Docker mounts                    | Data files visible     |
| SSL/Email Error      | ssl.SSLError: WRONG_VERSION_NUMBER        | Used port 465 + SSL in block config            | Emails send!           |
| Success Log          | Inserted 330327 rows...                   | N/A                                            | Pipeline complete!     |

---

## 🎯 Summary & Learnings

- Always check Docker mount paths — it’s the #1 silent failure.
- Use Prefect blocks *before* writing custom notification code.
- Build and test `.env` configs **before** entering the container.
- Don’t panic on first run — data engineering is 90% debugging, 10% coffee.
- Debugging logs aren’t just noise — they’re breadcrumbs to brilliance.

---

🛠 This new set of fixes will be merged into my main playbook soon.  
If you've debugged a gnarly Prefect + Docker + dbt + PostgreSQL setup — let’s swap notes!
