## Databricks MCP Code Execution Template via Command Execution API

**Goal:** Enable end-to-end development in Databricks using VibeCoding. Test code directly on clusters via MCP, then deploy with Databricks Asset Bundles (DABs).

---

### 🚀 Before Starting — User Configuration Required

When a user asks to build a pipeline, the AI assistant must **first collect:**

#### 1. Databricks Workspace Details

- **Workspace URL** (e.g., `https://dbc-xxxxx.cloud.databricks.com`)
- **Personal Access Token (PAT)**
- **Catalog name** (Unity Catalog; underscores only)
- **Cluster ID** (used for MCP command execution)

#### 2. MCP Server (Required)

```json
{
  "mcpServers": {
    "databricks-dev-mcp": {
      "type": "http",
      "url": "http://localhost:8000/message"
    }
  }
}
```

---

### 🧠 Rule Precedence (Critical)

1. **MCP commands must always run automatically on the Databricks cluster.**
   - No confirmation required
   - Never simulate execution locally
2. **`databricks bundle validate` and `databricks bundle deploy` must run automatically.**
3. **`databricks bundle run <job>` must NEVER run automatically.**
   - Always ask user **explicitly** before running
4. When unsure whether to execute something, **ask the user**.

These rules override anything else in this file.

---

### 📦 Packaging & Deployment Standards

#### 🔹 Phase 1: Test & Iterate (MCP Execution)

- Always use `databricks-dev-mcp` to run code on a Databricks cluster.
- MCP executions should **run immediately** without asking.
- Return full cluster output to the user.
- Iterate until the code works.

#### 🔹 Phase 2: Package & Deploy (DABs)

Once MCP testing is complete:

1. Create a Databricks Asset Bundle project.
2. Generate all necessary files inside a dedicated project folder.
3. Populate the `databricks.yml` bundle configuration.
4. Add job definitions under `resources/`.

#### ⚠️ Automatic Steps (assistant must run these immediately)

After generating the bundle:

1. `databricks bundle validate -t dev`
2. `databricks bundle deploy -t dev`

These do **not** require user confirmation.

#### ❗ Manual Step (requires explicit user confirmation)

Before running the deployed job, the assistant **must ask:**

> "Do you want to run `databricks bundle run <job_name> -t dev` now?"

Run the job **only if the user says yes**.

---

### 📁 Standard Project Structure

```
project/
├── databricks.yml
├── resources/
│   └── job.yml
├── src/<project>/
│   └── notebooks/
│       ├── 01_data_prep.py
│       ├── 02_training.py
│       └── 03_validation.py
└── tests/
```

#### ❗ Important Path Rule

Paths in `resources/*.yml` resolve **relative to the resource file**.

Example:

```yaml
notebook_path: ../src/notebooks/01_data_prep.py
```

---

### 🔧 Parameterization Requirements

- Never hard-code values.
- Use:
  - `${var.catalog}`
  - `${var.schema}`
  - `${bundle.target}`
  - `${workspace.current_user.userName}`

Notebooks must use:

```python
dbutils.widgets.get()
```

---

### 🌍 Multi-Environment Targets

Support at least:

- `dev`
- `staging` (optional)
- `prod` (optional)

---

### 🛠️ Serverless Compute Guidelines

- Do NOT define `new_cluster` in tasks.
- Use `%pip install` for library installation.
- Wrap widget access in try/except:

```python
try:
    x = dbutils.widgets.get("param")
except:
    x = default
```

---

### 🧠 AI Assistant Workflow (Final)

When the user asks to build or deploy:

1. Collect workspace details.
2. Use MCP to run notebook code directly on Databricks.
   - **Runs automatically**
3. Debug with real cluster output.
4. Generate full DAB project.
5. Automatically run:
   - `databricks bundle validate -t dev`
   - `databricks bundle deploy -t dev`
6. Confirm deployment.
7. Ask the user:
   > "Would you like to run the deployed job now?"
8. If yes → execute `databricks bundle run <job_name> -t dev`.
9. Report:
   - Success/failure
   - Run status
   - Result state
   - Any error message
10. Never remain silent after a job execution.

---

### 🔒 Security

- Never store or print access tokens.
- Use environment variables for authentication.
- Never embed secrets in code or notebooks.
- Always use secure API calls.

---

### 📚 References

- [Databricks Asset Bundles](https://docs.databricks.com/dev-tools/bundles/index.html)
- [Databricks MLOps Patterns](https://docs.databricks.com/machine-learning/mlops/mlops-workflow.html)
