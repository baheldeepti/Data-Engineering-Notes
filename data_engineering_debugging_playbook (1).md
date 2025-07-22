

```markdown
# Data Engineering Debugging Playbook

A curated playbook documenting common errors encountered during data engineering projects — especially with Docker, Prefect workflows, Python dependencies, and email notifications — along with practical resolutions and best practices.  
This guide is intended to save you time and frustration by helping you quickly identify root causes and fixes.

---

## Table of Contents

1. [Docker & Docker-Compose Issues](#docker--docker-compose-issues)  
2. [Python Dependency & Version Compatibility](#python-dependency--version-compatibility)  
3. [Prefect Workflow & Script Errors](#prefect-workflow--script-errors)  
4. [File Handling & Volume Mounting](#file-handling--volume-mounting)  
5. [Python Configuration & NameError](#python-configuration--nameerror)  
6. [Email SSL/TLS Connection Issues](#email-ssltls-connection-issues)  
7. [Best Practices & Tips](#best-practices--tips)  

---

## Docker & Docker-Compose Issues

### Issue: Obsolete `version` attribute warning  
**Symptom:**  
```

Warning: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion

```
**Cause:**  
Docker Compose v2 ignores the `version` attribute in `docker-compose.yml`.  

**Resolution:**  
- Remove the `version` line from your `docker-compose.yml` file to prevent confusion and suppress warnings.

---

### Issue: Missing `requirements.txt` during Docker build  
**Symptom:**  
```

failed to calculate checksum ... "/requirements.txt": not found

```
**Cause:**  
Dockerfile expects `requirements.txt` in a path relative to build context but file is missing or incorrectly placed.

**Resolution:**  
- Verify `requirements.txt` exists in your Docker build context directory (usually where Dockerfile resides).  
- Update `COPY` statements in Dockerfile if necessary (e.g., `COPY docker/requirements.txt .` means `requirements.txt` should be inside the `docker` folder).  
- Use relative paths carefully and confirm with `docker build` context.

---

### Issue: Pip install failures for specific package versions  
**Symptom:**  
```

ERROR: No matching distribution found for prefect==2.15.8
ERROR: No matching distribution found for great\_expectations==0.17.24
ERROR: No matching distribution found for numpy==1.25.6

```
**Cause:**  
- Package versions incompatible with Python version in Docker image or virtual environment.  
- Yanked or unavailable versions in PyPI.

**Resolution:**  
- Check Python version compatibility (e.g., Prefect >=2.1.0 requires Python 3.10+).  
- Update `requirements.txt` to specify package versions compatible with your Python version.  
- Use a supported Python base image (e.g., `python:3.10-slim`) if possible.  
- Regularly update dependencies to avoid deprecated/yanked versions.

---

## Python Dependency & Version Compatibility

### Issue: Pandas and NumPy binary incompatibility  
**Symptom:**  
```

ValueError: numpy.dtype size changed, may indicate binary incompatibility. Expected 96 from C header, got 88 from PyObject

````
**Cause:**  
Mismatched numpy and pandas versions or corrupted installation.

**Resolution:**  
- Reinstall numpy and pandas ensuring compatible versions.  
- Clear caches:  
  ```bash
  pip uninstall numpy pandas
  pip install numpy==<compatible_version> pandas==<compatible_version>
````

* Rebuild your Docker image or virtual environment cleanly.

---

## Prefect Workflow & Script Errors

### Issue: Prefect CLI command error: No such command 'orion'

**Symptom:**

```
prefect_server | No such command 'orion'
```

**Cause:**
Prefect CLI has changed between versions (Orion is the new name for Prefect 2+).

**Resolution:**

* Verify Prefect version installed.
* Use commands compatible with your Prefect version.
* Consult official Prefect docs for CLI changes.

---

### Issue: NameError: 'task' or config vars not defined

**Symptom:**

```
NameError: name 'task' is not defined
NameError: name 'URL_CONFIRMED' is not defined
```

**Cause:**

* Missing imports (e.g., `from prefect import task`)
* Constants/config variables referenced before definition.

**Resolution:**

* Import `task` and other decorators at the top of your script.
* Define all config constants at the beginning of the script, before usage.

---

### Issue: Flow run fails due to missing CSV file inside container

**Symptom:**

```
FileNotFoundError: No such file or directory: '/app/data/covid_data.csv'
```

**Cause:**

* Data file not present or volume not mounted inside container.

**Resolution:**

* Correctly mount data volume in `docker-compose.yml` or Docker run command, e.g.:

  ```yaml
  volumes:
    - ../data:/app/data
  ```
* Check presence of file inside container using `docker exec -it <container> ls /app/data`.
* Ensure data files exist on host machine path.

---

## File Handling & Volume Mounting

### Issue: Docker volume mount causes missing data inside container

**Symptom:**
Files expected inside container are missing or inaccessible.

**Resolution:**

* Validate volume mount paths in `docker-compose.yml`.
* Make sure host directories exist and contain files.
* Restart Docker Compose with `docker compose down --volumes` and `docker compose up` to refresh mounts.

---

## Python Configuration & NameError

### Issue: Config variables referenced before declaration

**Symptom:**

```
NameError: name 'URL_CONFIRMED' is not defined
```

**Resolution:**

* Define all constants and config variables at the top of your scripts.
* Keep config in a dedicated config file or environment variables loaded early.

---

## Email SSL/TLS Connection Issues

### Issue: SSL error when sending emails

**Symptom:**

```
ssl.SSLError: WRONG_VERSION_NUMBER
```

**Cause:**
Incorrect port or TLS settings for SMTP server (especially Gmail).

**Resolution:**

* For Gmail SMTP:

  * Use port **465** with `use_tls=False` (SSL).
  * Use port **587** with `use_tls=True` (STARTTLS).
* Always use an app-specific password, **never your regular Gmail password**.
* Confirm SMTP settings carefully in your email sending code.

---

## Best Practices & Tips

* **Docker:** Always check container logs (`docker logs <container>`) and statuses (`docker ps -a`) when troubleshooting.
* **Python Dependencies:** Use virtual environments or Docker images with fixed Python versions for consistency.
* **Prefect:** Use `prefect --help` and test DAGs locally with `prefect deployment run` or `prefect flow run` commands.
* **File Paths:** Verify volume mounts and data file presence inside containers early in your debugging workflow.
* **Config & Constants:** Define configuration constants at the top of your scripts to avoid NameErrors.
* **Email Notifications:** Test email sending separately before integrating with your pipeline alerts.
* **Keep Calm & Iterate:** Data engineering is often 80% fixing file paths and environment issues — patience is key!

---



