
# 🧰 Data Engineering Debugging Playbook: CKD Vitals Pipeline Errors & Fixes

This section documents real errors encountered while building an end-to-end CKD patient vitals data pipeline using Kafka, PostgreSQL, Great Expectations, Prefect, Docker, and Hugging Face models — along with solutions and lessons learned.

---

## 📦 Docker Build & Context Issues

**Issue:** Docker build fails with errors like:

```

failed to compute cache key: failed to calculate checksum of ref ... "/requirements.txt": not found
failed to compute cache key: failed to calculate checksum of ref ... "/streamlit\_app": not found

````

**Symptom:** Build logs show `COPY` commands failing because files/directories are missing.

**Cause:** Dockerfile `COPY` references files outside the build context (e.g., `COPY ../../requirements.txt .`), but Docker can only access files inside the build context folder.

**Fix:**  
- Set the build context in `docker-compose.yml` to the project root folder containing all files (e.g., `context: ../`).
- Use relative `COPY` paths inside the build context:

  ```dockerfile
  COPY requirements.txt .
  COPY streamlit_app/ .
````

* Confirm your directory structure and relative paths before building.

**Tip:**
Always verify your Docker build context and file paths to avoid this common pitfall.

---

## 📋 Dependency Conflicts & Version Resolution

**Issue:** Pip install fails due to conflicting dependencies:

```
ERROR: ResolutionImpossible:
The user requested pandas==2.2.2
streamlit 1.35.0 depends on pandas<3 and >=1.3.0
great-expectations 0.16.16 depends on pandas<2.0.0 and >=1.3.0
```

Also:

```
ERROR: No matching distribution found for prefect-email==0.2.6
ERROR: No matching distribution found for huggingface-hub==0.18.1
```

**Cause:** Hard version pins cause incompatibilities between libraries and missing versions on PyPI.

**Fix:**

* Use compatible version ranges in `requirements.txt`:

  ```
  pandas>=1.5,<2.0
  streamlit==1.35.0
  great_expectations==0.16.16
  prefect==2.16.5
  prefect-email==0.4.2  # Valid version
  huggingface-hub>=0.23,<1.0
  transformers==4.41.2
  ```

* Check dependency trees locally with `pipdeptree`.

* Avoid pinning to nonexistent versions.

**Tip:**
Test dependency installs locally before building Docker images to catch issues early.

---

## ⚙️ Prefect Task Execution RuntimeError

**Issue:** Runtime error during Prefect flow execution:

```
RuntimeError: Tasks cannot be run from within tasks. Did you mean to call this task in a flow?
```

**Cause:** Calling a Prefect task inside another task instead of only from a flow.

**Incorrect example:**

```python
@task
def task_a():
    pass

@task
def task_b():
    task_a()  # Causes RuntimeError
```

**Fix:**

* Call tasks only from flows.
* Use `.submit()` to schedule async calls if needed.

```python
@flow
def main_flow():
    task_a.submit()
    task_b()
```

**Tip:**
Understand Prefect’s execution model to avoid nested task calls.

---

## 🤖 Hugging Face Model & Prompt Tuning

**Challenge:** AI model responses were generic or irrelevant initially.

**Fix:**

* Iteratively refined prompts with more context and clearer instructions.

```python
prompt = (
    "You are a caring kidney health assistant. "
    "Given these vitals, provide 2-3 sentences of advice in simple English. "
    f"Vitals: {vitals}"
)
```

**Tip:**
Prompt engineering is essential for getting useful AI-generated outputs.

---

## 🔍 Summary of Best Practices

| Category       | Best Practice                                 |
| -------------- | --------------------------------------------- |
| Docker         | Verify build context and `COPY` paths         |
| Dependencies   | Use flexible compatible version ranges        |
| Prefect        | Call tasks only within flows, avoid nesting   |
| AI Integration | Spend time tuning prompts for quality output  |
| Debugging      | Check container logs and error messages early |

---

This playbook will continue growing as I encounter new challenges. I hope it helps you save time and frustration on your data engineering projects!

---

✉️ Feel free to reach out if you want help with Dockerfiles, Prefect flows, or dependency management!

---

*Deepti Bahel*

```

---

