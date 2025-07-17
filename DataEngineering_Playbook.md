
# 🧰 Data Engineering Debugging Playbook

> A growing playbook of real-world data engineering issues and how I solved them — covering Docker, Airflow, dbt, Snowflake, and more.  
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
- Added `airflow-init` container in `docker-compose.yml`
- Created persistent entrypoint for dbt:
  ```dockerfile
  ENTRYPOINT ["tail", "-f", "/dev/null"]
````

* Ensured `webserver/scheduler` depend on `airflow-init`

---

### ❌ Issue: Port 8080 not accessible

**Symptom**: `localhost:8080` not loading despite container running

**Fix**:

```bash
lsof -i :8080         # Find conflicting process
kill -9 <PID>         # Kill it
docker compose down --volumes --remove-orphans
docker compose up --build
```

✅ **Tip**: Always run `docker ps -a` and `docker logs <container>` when things go wrong.

---

## 📋 Environment Variables (.env)

### ❌ Issue: Env vars not available in container

**Symptom**: Python script throws `Missing env vars: SNOWFLAKE_WAREHOUSE`

**Fixes**:

* Added `.env` file to Docker:

  ```yaml
  env_file: ../.env
  ```
* Verified inside container:

  ```bash
  env | grep SNOWFLAKE
  ```
* Added Python support:

  ```python
  from dotenv import load_dotenv
  load_dotenv()
  ```

✅ **Tip**: Double-check `.env` filename and path before restarting.

---

## ❄️ Snowflake Connectivity

### ❌ Issue: SAML/SSL connection error with `snowsql`

**Symptom**:

> There was an error related to the SAML Identity Provider account parameter.

**Fix**:

* Your web URL is **not** your account string!
* Use:

  ```env
  SNOWFLAKE_ACCOUNT=orgname-accountname
  snowsql -a orgname-accountname -u username --authenticator externalbrowser
  ```

---

### ❌ Issue: Schema does not exist

**Symptom**: SQLAlchemy throws:

> `Schema 'PUBLIC' does not exist or not authorized.`

**Fix**:

* Set these in `.env`:

  ```env
  SNOWFLAKE_ROLE=ACCOUNTADMIN
  SNOWFLAKE_DB=MY_DATABASE
  SNOWFLAKE_SCHEMA=PUBLIC
  ```
* Use safe table checks:

  ```python
  if inspector.has_table(table_name, schema=schema):
  ```

---

## ⚙️ Airflow DAG Debugging

### ❌ Issue: DAG not triggering or failing silently

**Fixes**:

* Added task logs and retries
* Used:

  ```bash
  airflow dags test <dag_id> <execution_date>
  ```
* Checked logs in container:

  ```bash
  cd /opt/airflow/logs
  ```

---

## 🧪 Great Expectations

### ❌ Issue: Checkpoint fails with PostgreSQL

**Symptom**: Errors loading `data_asset` or executing query

**Fixes**:

* Updated `checkpoint.yml` with:

  * Correct `datasource_name`
  * Valid `batch_request` or SQL `query`
* Verified table name matches expectations

✅ **Tip**: Most checkpoint failures come from misconfigured queries or identifiers.

---

## 📁 File Handling

### ❌ Issue: CSV not found inside container

**Symptom**: `FileNotFoundError: /path/to/file.csv`

**Fix**:

* Map host folder to container:

  ```yaml
  volumes:
    - ../data:/opt/airflow/data
  ```
* Then verify inside container:

  ```bash
  ls /opt/airflow/data
  ```

---

## ✅ Summary: Best Practices

| Category      | Best Practice                                        |
| ------------- | ---------------------------------------------------- |
| Docker        | Always check logs and container status               |
| Airflow       | Set proper dependencies, include `airflow-init`      |
| dbt           | Add `tail -f` entrypoint to keep container alive     |
| Snowflake     | Validate account string, role, and schema explicitly |
| Env Variables | Load via `dotenv`, verify with `env`                 |
| Debugging     | Use `lsof`, `kill`, and container logs proactively   |

---

## 🎯 Want to Contribute?

If you’ve debugged your own pipeline issues and found fixes, **you’re welcome to contribute**:

* Fork the repo
* Add your issue + fix in Markdown format
* Submit a pull request

Let’s build a better data engineering troubleshooting guide — together. 🛠️

---

✉️ **Ping me** if you’d like to co-author or share ideas: *\[your-email-or-link]*



