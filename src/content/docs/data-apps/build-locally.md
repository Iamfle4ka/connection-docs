---
title: Build locally
slug: 'data-apps/build-locally'
description: Connect a Python/JS app to Keboola data - Input Mapping, the Storage API, and real-time read/write with Storage Access.
---



You can develop and test a Python/JS app on your own machine and then deploy it to Keboola. This guide covers how to connect your app to Keboola data: loading tables via Input Mapping, fetching data at runtime through the Storage API, and reading and writing in real time with Storage Access. For the build flow itself (repository structure, configuration files, deployment), see [build with Kai](/data-apps/build-with-kai/).

## Working with Keboola data

### Reading data loaded via Input Mapping

If you configured **Input Mapping** in your app settings, Keboola loads selected tables from Storage into the container before your app starts. Your app can then read them as CSV files:

```python
import pandas as pd

# File path follows this pattern: /data/in/tables/<table-name>.csv
df = pd.read_csv("/data/in/tables/my_table.csv")
print(df.head())
```

**Note:** Input Mapping data is loaded once when the app starts. To get fresh data, you need to redeploy the app or use the Storage API to fetch data at runtime.

### Reading data at runtime using the Storage API

For apps that need to fetch up-to-date data without redeploying, use the Keboola Storage API. Store your Storage token as a secret (see [secrets and environment variables](/data-apps/reference/#secrets-and-environment-variables)) and load data programmatically:

```python
import requests
import pandas as pd
from io import StringIO
import os

KBC_URL = os.environ.get("KBC_URL")          # e.g. https://connection.keboola.com
KBC_TOKEN = os.environ.get("KBC_TOKEN")       # your Storage API token

def load_table(table_id: str) -> pd.DataFrame:
    """Load a table from Keboola Storage into a pandas DataFrame."""
    url = f"{KBC_URL}/v2/storage/tables/{table_id}/export-async"
    headers = {"X-StorageApi-Token": KBC_TOKEN}
    response = requests.post(url, headers=headers)
    response.raise_for_status()
    # Follow the async export job...
    return pd.read_csv(StringIO(response.text))
```

For a complete example using the official Python client library, see the [Keboola Storage Python Client documentation](https://developers.keboola.com/integrate/storage/python-client/).

## Storage Access

Storage Access allows your app to read data from and write data back to Keboola Storage tables in real-time. Your app connects directly to Keboola's storage through Query Service via SQL, enabling:

- **Real-time data access**: Always work with the latest data, no redeployment needed
- **Write-back capability**: Update, insert into **existing** Storage tables directly from your app
- **Interactive applications**: Build data entry forms, approval workflows, and collaborative tools

This feature is available for both **Streamlit** and **Python/JS** apps. Code examples on this page use Python; the same concepts apply when calling the Query Service API from JavaScript.

:::caution
**Snowflake only.** Storage Access currently works only on projects using the Snowflake storage backend. BigQuery support is coming soon.
:::

For how Storage Access works internally (the workspace architecture and lifecycle), the environment variables it sets, how it compares to Input Mapping, and its limitations, see the [reference](/data-apps/reference/#storage-access-reference).

### When to use Storage Access

**Use Storage Access when you need to:**

- Build interactive data entry or editing applications
- Work with large datasets more efficiently - no need to load them via input mapping
- Enable business users to update data directly from the app

**Stick with Input Mapping when:**

- You don't need write-back capability — the app only reads and displays data.
- Your dataset is small and changes infrequently (e.g., static reference data loaded at deploy time).
- You want the simplest possible setup with no additional configuration.

### Setting up Storage Access

#### Step 1: Enable Storage Access

1. Go to the **Project Settings**.
2. Go to the **Features**.
3. Find the **Storage Access** feature and activate it.

#### Step 2: Configure writable tables

1. Open your app configuration in Keboola.
2. Go to the **Advanced Settings** tab.
3. Find the **Storage Access** section.
4. Click **+ Add Writable Table**.
5. Select a bucket and table from Storage.
6. For each table, the app will have **SELECT**, **INSERT**, **UPDATE**, **DELETE**, and **TRUNCATE** permissions.

**Notes:**

- You can add multiple tables from different buckets.
- All selected tables must exist before deploying.

##### Configuring writable tables programmatically

If you manage app configurations through the Storage API (or via automation/agents) rather than the UI, the same writable-table selection is expressed in the configuration JSON under `storage.output.tables`. Each entry is a table the app gets read/write permissions on:

```json
{
  "storage": {
    "output": {
      "tables": [
        {
          "destination": "out.c-data-app.mvc-crashes",
          "unload_strategy": "direct-grant"
        }
      ]
    }
  }
}
```

- **`destination`** — the full Storage table ID (`<stage>.<bucket>.<table>`) the app should be able to read and write. The table must exist before the app is deployed.
- **`unload_strategy: "direct-grant"`** — required marker that tells the platform "grant the app's workspace direct SELECT/INSERT/UPDATE/DELETE/TRUNCATE on this table." Tables without this strategy in `storage.output.tables` are not exposed via Storage Access.

To add or remove writable tables programmatically, update the app's configuration via the Storage API ([Component Configurations endpoint](https://api.keboola.com/?service=storage#tag--Component-Configurations)) and redeploy the app for the new permissions to take effect.

#### Step 3: Deploy your app

Click **Deploy** (or **Redeploy** for existing apps). During deployment:

1. Keboola creates a new workspace with your selected table permissions.
2. The workspace ID is passed to your app as an environment variable.
3. Your app code can now use the Query Service to read and write data.

### Reading data from Storage

#### Using the Query Service client

Install the Keboola Query Service client:

**In `pyproject.toml` (Python):**

```toml
dependencies = [
    "keboola-query-service>=0.2.0",
]
```

**In your Python code:**

```python
import json
import os
from keboola_query_service import Client

# Storage Access config is set by the platform when the feature is enabled.
# workspace_id is read from the manifest file (recommended); the other values
# are plain env vars.
try:
    branch_id = os.environ["BRANCH_ID"]
    query_service_url = os.environ["QUERY_SERVICE_URL"]
    with open(os.environ["KBC_WORKSPACE_MANIFEST_PATH"]) as f:
        workspace_id = json.load(f)["workspaceId"]
except (KeyError, FileNotFoundError) as e:
    raise RuntimeError(
        "Storage Access is not enabled for this app. "
        "Enable it in Advanced Settings and redeploy."
    ) from e

# Initialize the Query Service client
client = Client(
    base_url=query_service_url,
    token=os.environ["KBC_TOKEN"],
)
```

#### Reading selected tables

To read a table you've selected in the UI:

```python
import pandas as pd

# Query a table - use the full table ID (bucket.table)
results = client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=['SELECT * FROM "in.c-main"."customers" LIMIT 1000'],
)

# One QueryResult per statement — we sent one statement, so take the first.
result = results[0]

# Convert to DataFrame
df = pd.DataFrame(result.data, columns=[c.name for c in result.columns])
print(df.head())
```

**Table naming convention:**

- Use the full table ID in quotes: `"bucket_stage.bucket_name"."table_name"`
- Example: `"in.c-sales"."orders"` for a table `orders` in bucket `in.c-sales`

#### Running custom queries

You can run any SELECT query against your permitted tables:

```python
# Join multiple tables
query = """
    SELECT
        c.customer_name,
        SUM(o.amount) as total_spent
    FROM "in.c-main"."customers" c
    JOIN "in.c-main"."orders" o ON c.id = o.customer_id
    GROUP BY c.customer_name
    ORDER BY total_spent DESC
    LIMIT 10
"""

results = client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=[query],
)
result = results[0]
```

### Writing data back to Storage

Storage Access allows your app to modify data in Storage tables using standard SQL statements via the Query Service. This is useful for:

- Data entry forms
- Approval workflows
- Data correction interfaces
- Collaborative editing

:::caution
The Query Service accepts raw SQL and does not support parameterized queries or server-side bind variables. Your application is responsible for validating every untrusted value before interpolating it into a statement. See the [Validate and sanitize user input](#best-practices) guidance for patterns.
:::

#### Inserting and updating data

You can use standard SQL INSERT and UPDATE statements directly via the Query Service. Pass `statements` as a list — the SDK will execute them (transactionally by default) and return one result per statement:

```python
# INSERT new records
client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=['''
        INSERT INTO "in.c-main"."approvals" ("id", "name", "status", "updated_at")
        VALUES (1, 'New Record', 'pending', CURRENT_TIMESTAMP)
    '''],
)

# UPDATE existing records
client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=['''
        UPDATE "in.c-main"."approvals"
        SET status = 'approved', updated_at = CURRENT_TIMESTAMP
        WHERE id = 123
    '''],
)

# DELETE records
client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=['''
        DELETE FROM "in.c-main"."approvals"
        WHERE status = 'cancelled'
    '''],
)
```

The Query Service automatically handles metadata refresh in Storage after write operations, so row counts and table statistics stay current without any additional calls.

#### Truncating tables

To remove all data from a table:

```python
client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=['TRUNCATE TABLE "in.c-main"."temp_data"'],
)
```

:::caution
Truncation removes every row in the target table immediately and cannot be undone through the Query Service. Use with caution.
:::

#### Important considerations

**Metadata refresh:** After any write operation, Keboola automatically refreshes the table metadata. This ensures:

- Row counts are accurate in the Storage UI
- Other components see the updated data
- Table statistics are current

**Concurrency:** Multiple users of your app may write simultaneously. If you need to prevent conflicts, you must handle this in your application logic:

```python
# Example: Optimistic locking with version column
query = """
    UPDATE "in.c-main"."records"
    SET status = 'approved', version = version + 1
    WHERE id = 123 AND version = 5
"""
results = client.execute_query(
    branch_id=branch_id,
    workspace_id=workspace_id,
    statements=[query],
)

if results[0].rows_affected == 0:
    raise Exception("Record was modified by another user. Please refresh and try again.")
```

### Example: read-write app

This example shows a simple Flask app that reads records from Storage and allows users to update their status.

**`app.py`:**

```python
from flask import Flask, request, render_template_string
import json
import os
from keboola_query_service import Client

app = Flask(__name__)

# Read Storage Access config once at startup.
# workspace_id is read from the manifest (recommended); other values are env vars.
BRANCH_ID = os.environ["BRANCH_ID"]
with open(os.environ["KBC_WORKSPACE_MANIFEST_PATH"]) as f:
    WORKSPACE_ID = json.load(f)["workspaceId"]

qs_client = Client(
    base_url=os.environ["QUERY_SERVICE_URL"],
    token=os.environ["KBC_TOKEN"],
)

ALLOWED_STATUSES = {"pending", "approved", "rejected"}


@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        # Validate and sanitize user input BEFORE it reaches SQL.
        # int() guarantees record_id is a number; the allowlist guarantees
        # new_status is one of three exact strings. This is the only reason
        # the f-string below is safe — do not add other form fields to the
        # query without analogous validation.
        record_id = int(request.form["record_id"])
        new_status = request.form["status"]
        if new_status not in ALLOWED_STATUSES:
            return "Invalid status", 400

        qs_client.execute_query(
            branch_id=BRANCH_ID,
            workspace_id=WORKSPACE_ID,
            statements=[f'''
                UPDATE "in.c-main"."approvals"
                SET status = '{new_status}', updated_at = CURRENT_TIMESTAMP
                WHERE id = {record_id}
            '''],
        )

    # Load current records
    results = qs_client.execute_query(
        branch_id=BRANCH_ID,
        workspace_id=WORKSPACE_ID,
        statements=['SELECT id, name, status, updated_at FROM "in.c-main"."approvals" ORDER BY id'],
    )
    result = results[0]
    column_names = [c.name for c in result.columns]
    records = [dict(zip(column_names, row)) for row in result.data]

    return render_template_string(TEMPLATE, records=records)


TEMPLATE = """
<!DOCTYPE html>
<html>
<head><title>Approval Manager</title></head>
<body>
    <h1>Pending Approvals</h1>
    <table border="1">
        <tr><th>ID</th><th>Name</th><th>Status</th><th>Action</th></tr>
        {% for r in records %}
        <tr>
            <td>{{ r.id }}</td>
            <td>{{ r.name }}</td>
            <td>{{ r.status }}</td>
            <td>
                <form method="POST" style="display:inline">
                    <input type="hidden" name="record_id" value="{{ r.id }}">
                    <select name="status">
                        <option value="pending">Pending</option>
                        <option value="approved">Approved</option>
                        <option value="rejected">Rejected</option>
                    </select>
                    <button type="submit">Update</button>
                </form>
            </td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
"""

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**`pyproject.toml`:**

```toml
[project]
name = "approval-app"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "flask>=3.0.0",
    "keboola-query-service>=0.2.0",
]

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

### Best practices

**1. Handle missing workspace gracefully**

```python
import json
import os

try:
    branch_id = os.environ["BRANCH_ID"]
    query_service_url = os.environ["QUERY_SERVICE_URL"]
    with open(os.environ["KBC_WORKSPACE_MANIFEST_PATH"]) as f:
        workspace_id = json.load(f)["workspaceId"]
except (KeyError, FileNotFoundError):
    # Storage Access is not enabled — show a user-friendly error
    import streamlit as st  # or use your framework's error handling
    st.error("Storage Access is not enabled for this app. Enable it in Advanced Settings and redeploy.")
    st.stop()
```

**2. Validate and sanitize user input to prevent SQL injection**

Since the Query Service accepts raw SQL strings, you must validate all user input before including it in queries:

```python
# ❌ DANGEROUS - never do this
query = f"SELECT * FROM table WHERE id = {user_input}"

# ✅ SAFE - validate types and use allowlists
safe_id = int(user_input)  # Ensure it's actually a number
query = f"SELECT * FROM table WHERE id = {safe_id}"

# ✅ For string values, use an allowlist of permitted values
ALLOWED_STATUSES = {"pending", "approved", "rejected"}
if status not in ALLOWED_STATUSES:
    raise ValueError(f"Invalid status: {status}")
query = f"UPDATE table SET status = '{status}' WHERE id = {safe_id}"
```

:::tip[Safer interpolation helpers are coming]
First-class `SQL.literal()` / `SQL.ident()` / `sql.format()` helpers (with dialect-aware escaping and a `SafeSql` trust marker) are in development in the [Keboola Query Service Python SDK](https://github.com/keboola/query-service-api-python-sdk) and [JavaScript SDK](https://github.com/keboola/query-service-api-js-sdk) and will replace the allowlist/type-coercion patterns above once a release ships. Until then, the validation approach shown here is the recommended interim solution — especially for arbitrary string input, which is genuinely hard to sanitize by hand.
:::

**3. Implement keyset pagination for large datasets**

Use keyset (cursor-based) pagination instead of OFFSET, which can produce duplicates or gaps on live data:

```python
page_size = 1000
last_id = 0  # Start from the beginning

while True:
    results = client.execute_query(
        branch_id=branch_id,
        workspace_id=workspace_id,
        statements=[f'''
            SELECT * FROM "in.c-main"."my_table"
            WHERE id > {last_id}
            ORDER BY id ASC
            LIMIT {page_size}
        '''],
    )
    rows = results[0].data
    if not rows:
        break
    process_batch(rows)
    last_id = rows[-1][0]  # Update cursor to last row's id
```

**4. Cache frequently-used data**

For **Streamlit** apps, use `st.cache_data`:

```python
import streamlit as st

@st.cache_data(ttl=300)  # Cache for 5 minutes
def load_reference_data():
    results = client.execute_query(
        branch_id=branch_id,
        workspace_id=workspace_id,
        statements=['SELECT * FROM "in.c-main"."reference_data"'],
    )
    result = results[0]
    return pd.DataFrame(result.data, columns=[c.name for c in result.columns])
```

For **Python/JS** (non-Streamlit) apps, use a simple in-memory cache:

```python
import time

_cache = {}
_cache_ttl = 300  # seconds

def get_cached_data(key, query_fn):
    now = time.time()
    if key in _cache and now - _cache[key]["ts"] < _cache_ttl:
        return _cache[key]["data"]
    data = query_fn()
    _cache[key] = {"data": data, "ts": now}
    return data
```

**5. Track write operations**

Write operations are automatically tracked by the Query Service for billing purposes. For additional application-level auditing, log to stdout (visible in app container logs):

```python
import logging
logging.basicConfig(level=logging.INFO)

logging.info(f"User {current_user} updated record {record_id} to status {new_status}")
# Output goes to stdout → visible in the Terminal Log tab of your app
```
