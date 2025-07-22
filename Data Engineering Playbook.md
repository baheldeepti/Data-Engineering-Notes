## Debugging Playbook: Real-time Anomaly Pipeline Issues & Resolutions

This playbook documents the issues encountered while building and running a real-time anomaly detection pipeline using Docker Compose, Python, Kafka, and Postgres, along with the corresponding resolutions.

---

### 1. Docker Compose Build Context Error

**Log**:

```
target producer: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

**Cause**: The `build.context` and `dockerfile` paths in `docker-compose.yml` did not point to the correct `scripts/` directory containing the Dockerfile.

**Resolution**:

* In `docker-compose.yml`, set:

  ```yaml
  producer:
    build:
      context: ../scripts
      dockerfile: Dockerfile
  ```
* Ensure a `Dockerfile` exists in the `scripts/` folder.

---

### 2. Obsolete `version` Warning & Missing Build Context

**Log**:

```
time="..." level=warning msg="...docker-compose.yml: the attribute `version` is obsolete..."
unable to prepare context: path ".../docker/scripts" not found
```

**Cause**: The top-level `version:` key is ignored, and a relative `./scripts` path didn’t exist under `docker/`.

**Resolution**:

* Remove the obsolete `version` attribute or upgrade Compose file schema to `version: '3.8'`.
* Update service `build.context` from `./scripts` to `../scripts`.

---

### 3. Virtualenv Activation Error

**Log**:

```
venv\Scripts\activatevenv\Scripts\activate : The module 'venv' could not be loaded.
```

**Cause**: Typo in activation command: `activatevenv\Scripts\activate` doesn’t exist.

**Resolution**:

* Activate with the correct path:

  ```powershell
  .\venv\Scripts\Activate.ps1
  ```

---

### 4. Missing `requirements.txt`

**Log**:

```
ERROR: Could not open requirements file: ... 'requirements.txt'
```

**Cause**: No `requirements.txt` in the project root or wrong working directory.

**Resolution**:

* Create a `requirements.txt` listing dependencies, e.g.:

  ```text
  confluent-kafka
  psycopg2-binary
  python-dotenv
  pandas
  ```
* Then install:

  ```bash
  pip install -r requirements.txt
  ```

---

### 5. Postgres Connection Auth Error

**Log**:

```
DB Insert Error: connection to server at "localhost" ... failed: fe_sendauth: no password supplied
```

**Cause**: `.env` variables were not loaded, so `PG_PASSWORD` was empty.

**Resolution**:

* Ensure `.env` in project root contains:

  ```dotenv
  PG_HOST=localhost
  PG_PORT=5432
  PG_DB=vitalsdb
  PG_USER=admin
  PG_PASSWORD=admin123
  ```
* Mount `.env` into containers or load it locally:

  ```python
  load_dotenv(path_to_env)
  ```

---

### 6. Missing Table `patient_vitals`

**Log**:

```
DB Insert Error: relation "patient_vitals" does not exist
```

**Cause**: The table wasn’t created in Postgres.

**Resolution**:

* Connect to Postgres and create the table:

  ```sql
  CREATE TABLE IF NOT EXISTS patient_vitals (
    patient_id VARCHAR(50),
    timestamp TIMESTAMP,
    temperature REAL,
    heart_rate REAL,
    is_anomaly BOOLEAN
  );
  ```

---

### 7. Kafka Bootstrap Connect Refused

**Log**:

```
Connect to ipv4#127.0.0.1:9092 failed: Connection refused
```

**Cause**: Inside Docker, `localhost:9092` points to the container itself, not the Kafka broker.

**Resolution**:

* Inject the correct broker address via env var in Compose:

  ```yaml
  environment:
    KAFKA_BOOTSTRAP_SERVERS: kafka:9092
  ```
* In code, use:

  ```python
  os.getenv('KAFKA_BOOTSTRAP_SERVERS')
  ```

---

### 8. Undefined Volumes/Services in Compose

**Logs**:

```
service "kafka" refers to undefined volume kafka_logs
service "consumer" depends on undefined service "dz_postgres"
```

**Cause**: Top-level `volumes:` missing or incorrect service names used in `depends_on`.

**Resolution**:

* Add top-level definitions:

  ```yaml
  volumes:
    postgres_data:
    kafka_logs:
  ```
* Use service names (`kafka`, `postgres`), not container names, in `depends_on`.

---

### 9. SMTP Connection Closed Error

**Log**:

```
smtplib.SMTPServerDisconnected: Connection unexpectedly closed
```

**Cause**: Using plain SMTP on SSL port (465) without SSL wrapper.

**Resolution**:

* For port 465, use `smtplib.SMTP_SSL`:

  ```python
  server = smtplib.SMTP_SSL(EMAIL_HOST, EMAIL_PORT)
  ```
* Or switch to STARTTLS on port 587:

  ```dotenv
  EMAIL_PORT=587
  EMAIL_USE_TLS=true
  ```

---

## Conclusion

Keep this playbook handy as your single reference when troubleshooting common data engineering pipeline issues — especially around Docker Compose, env vars, DB schema, Kafka connectivity, and email delivery.
