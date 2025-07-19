
````markdown
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
````

* Ensured `webserver` and `scheduler` depend on `airflow-init` container.

---

### ❌ Issue: Port 8080 not accessible on localhost

**Symptom**: Airflow webserver does not load despite container running

**Fix**:

```bash
lsof -i :8080          # Find process using port
kill -9 <PID>          # Kill conflicting process
docker compose down --volumes --remove-orphans
docker compose up --build
```

✅ **Tip**: Always check container logs and status:

```bash
docker ps -a
docker logs airflow_webserver
```

---

## 📋 Environment Variables & .env

### ❌ Issue: Environment variables not available inside containers

**Symptom**: Python or dbt scripts fail due to missing variables like `SNOWFLAKE_WAREHOUSE`

**Fixes**:

* Add `.env` file to Docker compose:

  ```yaml
  env_file: ../.env
  ```
* Verify variables inside container:

  ```bash
  env | grep SNOWFLAKE
  ```
* Load environment variables in Python scripts using `python-dotenv`:

  ```python
  from dotenv import load_dotenv
  load_dotenv()
  ```

---

## ❄️ Snowflake Connectivity

### ❌ Issue: SAML or SSL connection errors with `snowsql`

**Symptom**: Errors related to SAML Identity Provider or invalid account string

**Fix**:

* Ensure `SNOWFLAKE_ACCOUNT` is the **full account identifier** (not URL).
* Use command line with proper account string:

  ```bash
  snowsql -a orgname-accountname -u username --authenticator externalbrowser
  ```

---

### ❌ Issue: Schema does not exist error

**Symptom**: SQLAlchemy error:
`Schema 'PUBLIC' does not exist or not authorized.`

**Fix**:

* Set `SNOWFLAKE_ROLE`, `SNOWFLAKE_DB`, and `SNOWFLAKE_SCHEMA` explicitly in `.env`:

  ```env
  SNOWFLAKE_ROLE=ACCOUNTADMIN
  SNOWFLAKE_DB=MY_DATABASE
  SNOWFLAKE_SCHEMA=PUBLIC
  ```
* Use safe table existence checks in code:

  ```python
  if inspector.has_table(table_name, schema=schema):
      ...
  ```

---

## ⚙️ Airflow DAG Debugging

### ❌ Issue: DAG not triggering or failing silently

**Fixes**:

* Added retry logic and detailed task logs.
* Test DAG execution locally with:

  ```bash
  airflow dags test <dag_id> <execution_date>
  ```
* Inspect logs inside container:

  ```bash
  cd /opt/airflow/logs
  tail -f <dag_id>/<task_id>/<execution_date>/1.log
  ```

---

## 🧪 Great Expectations

### ❌ Issue: Initialization errors like `pyspark not installed` or missing dependencies

**Fixes**:

* Install required packages in Dockerfile or container environment:

  ```bash
  pip install pyspark
  ```
* Ensure GE config folder and files are correctly mounted and accessible in the container.

---

### ❌ Issue: Great Expectations datasource connection failure

**Symptom**: Checkpoint errors with PostgreSQL or Snowflake

**Fixes**:

* Correct `datasource_name` in `checkpoint.yml`
* Validate SQL queries and table names in batch requests
* Ensure datasource exists and is properly registered before checkpoint runs

---

### ❌ Issue: Great Expectations server down or port conflicts (default port 8888)

**Fix**:

* Stop all running Docker containers:

  ```bash
  docker compose down --volumes --remove-orphans
  ```
* Restart Docker Desktop or kill conflicting processes on port 8888:

  * On Windows CMD:

    ```bash
    taskkill /PID <PID> /F
    ```
* Ensure Dockerfile exposes the required port(s).

---

### ❌ Issue: Permission denied error writing to `data_docs/local_site`

**Symptoms**:

* GE validation fails with:
  `PermissionError: [Errno 13] Permission denied: '/opt/airflow/great_expectations/uncommitted/data_docs/local_site/static'`

**Root Cause**: Mounted folder owned by root, but Airflow runs as `airflow` user inside container.

**Fix / Recommended Solution**:

* On Windows (CMD as Admin):

  ```cmd
  icacls "path\to\great_expectations\uncommitted\data_docs\local_site" /grant Everyone:(OI)(CI)F /T
  ```

* On Linux/WSL:

  ```bash
  sudo chown -R 50000:0 path/to/great_expectations/uncommitted/data_docs/local_site
  sudo chmod -R 775 path/to/great_expectations/uncommitted/data_docs/local_site
  ```

* Or fix inside container as root user:

  ```bash
  docker exec -it --user root airflow_webserver bash
  rm -rf /opt/airflow/great_expectations/uncommitted/data_docs/local_site
  mkdir -p /opt/airflow/great_expectations/uncommitted/data_docs/local_site
  chown -R airflow:root /opt/airflow/great_expectations/uncommitted/data_docs/local_site
  chmod -R 775 /opt/airflow/great_expectations/uncommitted/data_docs/local_site
  ```

* Restart Airflow containers after fixing permissions.

---

## 📦 dbt Task Failures

### ❌ Issue: `dbt: command not found` or Permission errors on `/opt/dbt`

**Symptoms**:

* Airflow logs show `/usr/bin/bash: line 1: dbt: command not found`
* Permission denied errors on `/opt/dbt/logs/dbt.log` or `/opt/dbt/target/manifest.json`

**Root Cause**:

* dbt CLI not installed or PATH not set inside container
* Permission mismatch on mounted `/opt/dbt` volume

**Fixes**:

* Exec into airflow container as root:

  ```bash
  docker exec -it --user root airflow_webserver bash
  ```

* Fix ownership and permissions:

  ```bash
  chown -R airflow:root /opt/dbt
  chmod -R 775 /opt/dbt
  ```

* Optionally, modify Dockerfile to include entrypoint script fixing permissions automatically:

  ```bash
  # entrypoint.sh
  chown -R airflow:root /opt/dbt
  chmod -R 775 /opt/dbt
  exec /entrypoint "$@"
  ```

* Update Dockerfile:

  ```dockerfile
  COPY entrypoint.sh /entrypoint.sh
  RUN chmod +x /entrypoint.sh
  ENTRYPOINT ["/entrypoint.sh"]
  CMD ["webserver"]
  ```

* Or override command in `docker-compose.yml` to fix permissions before start:

  ```yaml
  command: >
    bash -c "chown -R airflow:root /opt/dbt && chmod -R 775 /opt/dbt && airflow webserver"
  ```

* Restart Airflow containers after permission fixes.

---

## ✉️ SMTP / Email Setup in Airflow

### ❌ Issue: SMTP Authentication errors (e.g., Gmail `Application-specific password required`)

**Cause**: Gmail requires app-specific password when 2FA is enabled.

**Fixes**:

* Generate an **App Password** in your Google Account security settings:

  * Go to [https://myaccount.google.com/security](https://myaccount.google.com/security)
  * Under "Signing in to Google", select "App passwords"
  * Generate a password for "Mail" and device "Other" (e.g., "Airflow")

* Update Airflow SMTP connection to use this app password instead of your normal Gmail password.

### Update Airflow SMTP config (in `airflow.cfg` or connection settings)

```ini
[smtp]
smtp_host = smtp.gmail.com
smtp_user = your_email@gmail.com
smtp_password = your_app_password_here  # Use the app password generated above
smtp_port = 587
smtp_starttls = True
smtp_ssl = False
```

* Alternatively, configure SMTP connection in Airflow UI (`Admin > Connections`) with this info.

---

## 🛠️ Summary of Best Practices for Bug Fixes

| Category           | Best Practice                                                                                          |
| ------------------ | ------------------------------------------------------------------------------------------------------ |
| Docker             | Use `docker-compose down --volumes --remove-orphans` before restart; fix volume mounts and permissions |
| Airflow            | Use `airflow dags test` for local DAG testing; check logs frequently                                   |
| dbt                | Fix permissions on mounted volumes; ensure dbt CLI is installed and PATH set                           |
| Great Expectations | Correct datasource config and batch requests; fix permission errors on docs output                     |
| Snowflake          | Use correct account identifiers; explicitly set role, database, schema                                 |
| Email              | Use Gmail app password for SMTP authentication with 2FA enabled                                        |

---

## 🎯 Want to Contribute?

If you’ve debugged your own pipeline issues and found fixes, **you’re welcome to contribute**:

* Fork the repo
* Add your issue + fix in Markdown format
* Submit a pull request

Let’s build a better data engineering troubleshooting guide — together. 🛠️

---

✉️ **Ping me** if you’d like to co-author""
