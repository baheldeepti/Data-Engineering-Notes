
```markdown
# Data Engineering Debugging Playbook

A practical guide capturing errors encountered during data engineering projects with Docker, Prefect workflows, Python dependencies, and related areas — along with clear resolutions to save your time and headaches.

---

## Summary of Errors & Resolutions

| Error / Issue                                         | Symptom / Log Example                                          | Resolution Summary                                         |
|-----------------------------------------------------|---------------------------------------------------------------|------------------------------------------------------------|
| 1. Obsolete `version` attribute in docker-compose.yml | Warning: `the attribute version is obsolete, it will be ignored` | Remove `version` line from `docker-compose.yml`             |
| 2. Missing `requirements.txt` during Docker build    | `failed to calculate checksum ... "/requirements.txt": not found` | Ensure `requirements.txt` is in Docker build context; fix COPY paths |
| 3. Pip install failures due to incompatible versions | `ERROR: No matching distribution found for prefect==2.15.8` and similar | Match package versions to Python version; update Python base image if needed |
| 4. Prefect CLI command error (`No such command 'orion'`) | `No such command 'orion'`                                      | Use Prefect CLI commands compatible with installed version  |
| 5. Python NameError: missing imports (`task` not defined) | `NameError: name 'task' is not defined`                        | Add missing imports (`from prefect import task`)             |
| 6. Python NameError: config vars undefined (`URL_CONFIRMED`) | `NameError: name 'URL_CONFIRMED' is not defined`                | Define constants/config vars at the top before usage        |
| 7. Pandas / NumPy binary incompatibility             | `ValueError: numpy.dtype size changed...`                      | Reinstall compatible numpy and pandas versions; clean env   |
| 8. FileNotFoundError: missing CSV file inside container | `No such file or directory: '/app/data/covid_data.csv'`        | Verify Docker volume mounts; check file existence inside container |
| 9. Docker volume mount causes missing data            | Data files expected but missing inside container                | Fix volume mounts in docker-compose.yml; restart containers |
| 10. Email SSL error (`ssl.SSLError: WRONG_VERSION_NUMBER`) | Email sending fails with SSL error                              | Use Gmail SMTP port 465 with `use_tls=False`; use app password |

---

## Detailed Playbook

### 1. Obsolete `version` attribute warning in Docker Compose

**Symptom:**  
```

the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion

```

**Cause:** Docker Compose v2 ignores the `version` attribute.

**Fix:**  
Remove the `version` line from your `docker-compose.yml` file.

---

### 2. Missing `requirements.txt` in Docker build

**Symptom:**  
```

failed to calculate checksum ... "/requirements.txt": not found

```

**Cause:** `requirements.txt` missing or COPY paths incorrect.

**Fix:**  
- Place `requirements.txt` in correct directory relative to Dockerfile.  
- Adjust Dockerfile `COPY` statements (e.g., `COPY docker/requirements.txt .` requires file inside `docker` folder).

---

### 3. Pip install failures (Prefect, Great Expectations, NumPy)

**Symptom:**  
```

ERROR: No matching distribution found for prefect==2.15.8
ERROR: No matching distribution found for great\_expectations==0.17.24
ERROR: No matching distribution found for numpy==1.25.6

```

**Cause:** Package version incompatibility with Python base version.

**Fix:**  
- Check Python version compatibility (Prefect 2.1+ requires Python 3.10+).  
- Update `requirements.txt` to versions compatible with your Python version or upgrade Python base image.  
- Consider `python:3.10-slim` or later base image.

---

### 4. Prefect CLI command error: No such command 'orion'

**Symptom:**  
```

prefect\_server | No such command 'orion'

```

**Cause:** Prefect CLI commands changed between versions.

**Fix:**  
Use CLI commands matching your installed Prefect version; check official Prefect documentation.

---

### 5. Python NameError: 'task' not defined

**Symptom:**  
```

NameError: name 'task' is not defined

````

**Cause:** Missing import of Prefect's `task` decorator.

**Fix:**  
Add at the top of your script:  
```python
from prefect import task
````

---

### 6. Python NameError: config variable undefined (`URL_CONFIRMED`)

**Symptom:**

```
NameError: name 'URL_CONFIRMED' is not defined
```

**Cause:** Config constants referenced before definition.

**Fix:**
Define all constants/config variables at the top of your scripts before usage.

---

### 7. Pandas and NumPy binary incompatibility error

**Symptom:**

```
ValueError: numpy.dtype size changed, may indicate binary incompatibility
```

**Cause:** Version mismatch or corrupt installs.

**Fix:**
Reinstall compatible versions:

```bash
pip uninstall numpy pandas
pip install numpy==<compatible_version> pandas==<compatible_version>
```

Or rebuild Docker images cleanly.

---

### 8. FileNotFoundError for CSV data inside container

**Symptom:**

```
FileNotFoundError: No such file or directory: '/app/data/covid_data.csv'
```

**Cause:** Data file missing or volume mount incorrect.

**Fix:**

* Map host data directory to container path via Docker volumes:

  ```yaml
  volumes:
    - ../data:/app/data
  ```
* Verify file exists inside container:

  ```bash
  docker exec -it <container_name> ls /app/data
  ```

---

### 9. Docker volume mount issues causing missing data inside container

**Symptom:**
Data files not found or empty directories inside container.

**Fix:**

* Check and fix volume mount paths in `docker-compose.yml`.
* Restart containers after fixing mounts:

  ```bash
  docker compose down --volumes --remove-orphans
  docker compose up --build -d
  ```

---

### 10. Email SSL/TLS errors (`ssl.SSLError: WRONG_VERSION_NUMBER`)

**Symptom:**
SSL errors sending emails.

**Cause:** Wrong SMTP port or TLS settings.

**Fix:**
For Gmail SMTP:

* Use port **465** with `use_tls=False` for SSL connection.
* Use port **587** with `use_tls=True` for STARTTLS.
* Always use app-specific password, **not your regular Gmail password**.

---

## Additional Tips

* Always check container logs:

  ```bash
  docker logs <container_name>
  ```
* Validate all environment variables are loaded and available inside containers.
* Test small pieces (e.g., email sending) standalone before integration.
* Data engineering troubleshooting is mostly fixing paths and environment setups — patience is key!

