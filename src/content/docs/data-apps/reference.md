---
title: Reference
slug: 'data-apps/reference'
description: Reference for Keboola apps - environment variables, secrets, backend versions, sleep and resume, app management, the terminal log, troubleshooting, key terms, and limits.
redirect_from:
  - /components/data-apps/backend-versions/
  - /components/data-apps/terminal-log-tab/
---



## Secrets and environment variables

Sensitive values - API keys, tokens, passwords - should never be written directly into your code. Store them as **secrets** in the app configuration instead.

The platform also provides several environment variables automatically: `BRANCH_ID` (always set), `KBC_TOKEN` and `DATA_LOADER_API_URL` (with Data Loader), and `WORKSPACE_ID` / `QUERY_SERVICE_URL` / `KBC_WORKSPACE_MANIFEST_PATH` (with [Storage Access](#storage-access-reference)). See the [runtime README](https://github.com/keboola/data-app-python-js/blob/main/README.md#environment-variables) for the full list.

### Adding a secret in Keboola

In your app configuration, go to the **Secrets** section and add key-value pairs:

| Key | Value |
|---|---|
| `#KBC_TOKEN` | `your-storage-api-token` |
| `#ANTHROPIC_API_KEY` | `your-api-key` |
| `#DB_PASSWORD` | `your-database-password` |

The `#` prefix marks the value as a secret (encrypted at rest).

### Accessing secrets in your code

Keboola automatically makes secrets available as environment variables when your app starts. The `#` prefix is stripped, and the name is uppercased:

```python
# Python
import os

kbc_token = os.environ.get("KBC_TOKEN")
api_key = os.environ.get("ANTHROPIC_API_KEY")
```

```javascript
// Node.js
const kbcToken = process.env.KBC_TOKEN;
const apiKey = process.env.ANTHROPIC_API_KEY;
```

**How the name is transformed:** `#my-custom-var` becomes `MY_CUSTOM_VAR`. Dashes become underscores, the value is uppercased, and the `#` is removed.

## Backend versions and runtime

When deploying an app, you can select a **backend version** from a dropdown in the deployment wizard. Each backend version defines the runtime environment your app runs in, including the Python version, Streamlit version, and a set of pre-installed packages.

### Version format

Each backend version is displayed in the following format:

```
<backend_version> - Python <python_version> + Streamlit <streamlit_version>
```

For example: `1.15.2 - Python 3.13 + Streamlit 1.51`

This means:

- **Backend version** (`1.15.2`): The release version of the base Docker image that powers your app. Newer backend versions may include updated pre-installed packages, bug fixes, and infrastructure improvements.
- **Python version** (`3.13`): The version of the Python interpreter. Different Python versions offer different language features and performance characteristics.
- **Streamlit version** (`1.51`): The version of the [Streamlit](https://streamlit.io/) framework used to run your app.

### Choosing a version

Starting with backend version **1.15.0**, each release is available in multiple Python variants. Currently, the following Python versions are offered:

| Python Version | Notes |
|---|---|
| **3.10** | Default. Most widely compatible with third-party packages. Recommended if you are unsure. |
| **3.11** | Faster execution in many workloads (10–60% speedup over 3.10). Good balance of compatibility and performance. |
| **3.13** | Latest stable Python release. Best performance and newest language features, but some packages may not yet support it. |

**Recommendations:**

- **Stick with the default (Python 3.10)** if your app relies on packages that may not yet support newer Python versions, or if you want maximum compatibility.
- **Choose Python 3.11 or 3.13** if you need better performance or want to use newer Python language features, and you have verified that your dependencies support the chosen version.
- **Use an older backend version** only if you need to match a previously tested environment for stability. Older versions remain available in the dropdown.

### Pre-installed packages

All backend versions ship with the same set of pre-installed packages regardless of the Python variant. These packages are available immediately without adding them to the `Packages` field or `requirements.txt`:

- `streamlit`
- `pandas`
- `numpy`
- `matplotlib`
- `plotly`
- `scikit-learn`
- `seaborn`
- `graphviz`
- `deepmerge`
- `python-dotenv`
- `keboola.component`
- `streamlit-aggrid`
- `streamlit-keboola-api`
- `streamlit_authenticator` (pinned to 0.3.1)
- `toml`

To add packages beyond this list, specify them in the **Packages** field (for code deployment) or in a `requirements.txt` file (for Git repository deployment). Custom packages are installed on top of the pre-installed set and will not conflict with it.

:::tip
If you need to pin a specific version of a pre-installed package, you can override it by specifying the desired version in the Packages field or in your requirements.txt.
:::

## Sleep and resume

Our Suspend/Resume feature helps you save resources by automatically putting your app to sleep after a period of inactivity. Here is how it works:

**Activity monitoring**: The app monitors for HTTP requests and active Websocket connections. If no activity is detected for the configured period, the app automatically suspends. Please note that an inactive browser tab where your app is open may still cause background activity, potentially preventing your app from sleeping. If you are using Google Chrome, you may want to enable Memory Saver in the settings, which can help prevent such background activity.

**Automatic resumption**: As soon as a new request is made to the app, it wakes up and resumes operation. While the resume process is designed to be smooth, the first request upon waking may take slightly longer to process.

**Cost efficiency**: For example, if your app is active for two hours and then becomes inactive, it will go to sleep after the configured inactivity timeout. You will only be billed for the time when the app was active or waiting to suspend.

This feature ensures you pay only for what you use, while keeping the app ready for when you need it next.

If you enter the URL of a sleeping app, it will trigger its wakeup, and you will see a **waking up** page.

![Waking up](/data-apps/proxy-wakeup.png)

Should anything unexpected occur, a **wakeup error** page will appear, and you can click **Show More** to view the error details.

![Wakeup error](/data-apps/proxy-error-wakeing-up.png)

### How to set up inactivity timeout

When you click **Deploy** or **Redeploy** for your app, a wizard will appear, prompting you to specify the backend size and the auto-sleep timeout. You can set the duration of inactivity after which the app will go to sleep, with options ranging from five minutes to 24 hours. The default is set to five minutes.

![Deploy timeout and backend size](/data-apps/deploy-timeout-backedsize.png)

## Deployment and app management

### Actions menu

![Actions menu](/data-apps/manage-redeploy.png)

- **Deploy App** - starts the app. Once the deployment job is finished, you can go to the app public URL by clicking **Open App**.
- **Open App** - opens a new window with your app.
- **Redeploy** - if you made changes in the app configuration, you have to redeploy it for the changes to take effect.
- **Suspend App** - stops the app. The container in which the application is running will be stopped, and the app URL will no longer be available. The configuration of the app will remain intact.
- **Delete App** - stops the app deployment and deletes its configuration.

### Debugging app deployment

If the app deployment job fails, you can see the logs from its container in the event log of the deployment job. For example, there may be a conflict with the specified packages:

![Job error log](/data-apps/job-error-log.png)

## Terminal log tab

The Terminal Log Tab in Keboola provides users with an **almost real-time** view of the application's terminal logs. While there is a slight delay of a few seconds, this feature is designed to offer valuable insights into the application's runtime environment, helping with monitoring and troubleshooting.

![Screenshot - Hello World App](/data-apps/terminal-log.png)

### Key features

The Terminal Log Tab includes the following features:

- **Near real-time log display:**
    - Displays terminal output from the running application with a slight delay of a few seconds.
    - Enables users to monitor logs as they are generated, providing visibility into the application's current state.
- **Full log download:**
    - Allows users to download a complete log of the application from its start by selecting the **Download Logs** button located on the right side of the tab.
- **Log availability:**
    - Logs are accessible only while the application is running.
    - To maintain system efficiency, logs are automatically deleted when the application is stopped or paused.

### Key benefits

The Terminal Log Tab offers several benefits:

- **Near real-time monitoring:**
    - Provides users with a clear view of the application's performance during runtime.
    - Enables prompt detection of anomalies or bottlenecks.
- **Troubleshooting:**
    - Helps developers and operators identify issues by observing log outputs, such as error messages or warnings.
    - Simplifies debugging by offering direct insights from the runtime environment.
- **Transparency:**
    - Offers a detailed view of the application's internal operations, ensuring visibility and understanding.
- **Full log access for analysis:**
    - Supports in-depth analysis by allowing users to download the full log file, which contains all outputs from the start of the application's execution.

## Troubleshooting

### App shows "Method Not Allowed" or a blank page on first open

Your root route (`/`) likely only accepts GET requests. Keboola sends a POST request to `/` to verify the app is running. Add `methods=["GET", "POST"]` in Flask, or use `app.all('/')` in Express.

### App fails to start / keeps restarting

Check the **Terminal Log** tab in your app configuration in Keboola - it shows stdout and stderr output from your app. Common causes:

* A path in `app.conf` is relative (`app.py`) instead of absolute (`/app/app.py`).
* A Python command in `app.conf` is missing the `uv run` prefix.
* A package listed in `pyproject.toml` has a typo or does not exist.

### "externally-managed-environment" error

You have `pip install` somewhere in `setup.sh` or your code. Replace it with `uv sync` in `setup.sh` and make sure all dependencies are listed in `pyproject.toml`.

### App works locally but not in Keboola

* **Port mismatch:** The port in `default.conf` (the `proxy_pass` line) must match the port your app listens on.
* **Missing secrets:** A required environment variable is not defined in the Secrets section.
* **Missing `uv run`:** Python commands in `app.conf` must be prefixed with `uv run`.

### Environment variable is undefined

Add it as a secret in the Keboola app configuration (see [secrets and environment variables](#secrets-and-environment-variables)). Secrets are available to both `setup.sh` and your running app.

### Streaming responses arrive all at once instead of in real time

Add `proxy_buffering off;` to the relevant `location` block in `default.conf`. By default, Nginx collects the full response before forwarding it - this breaks Server-Sent Events and other streaming patterns.

## Key terms

**uv** - A fast Python package manager. Keboola uses it to install your Python dependencies from `pyproject.toml`. You interact with it via `uv sync` (in `setup.sh`) and `uv run` (in your Supervisord config). Think of it as a modern replacement for `pip`.

**pip** - The traditional Python package installer (e.g., `pip install flask`). Keboola's Python/JS image blocks direct `pip` usage to keep the environment stable - use `uv sync` instead.

**pyproject.toml** - A configuration file that defines your Python project: its name, required Python version, and list of dependencies. It is the modern standard for Python projects and is required by `uv`.

**Nginx** - A web server that handles incoming internet traffic and forwards it to your application. You configure it with `default.conf`. The most important setting is the `proxy_pass` port, which must match your app's port.

**Supervisord** - A process manager that starts and monitors your application. You configure it with `app.conf`. If your app crashes, Supervisord automatically restarts it.

**Port** - A number that identifies a specific communication channel on a computer. Your app listens on an internal port (e.g., 5000), while Nginx listens on port 8888 (the public-facing port required by Keboola). You never need to change 8888; only change your app's internal port.

**Environment variable** - A named value available to a running program, set outside the code itself. In Keboola, secrets become environment variables accessible via `os.environ.get("MY_KEY")` in Python or `process.env.MY_KEY` in Node.js.

**Input Mapping** - A Keboola feature that copies selected Storage tables into your app container as CSV files before the app starts. Useful for apps that need a snapshot of your data at startup.

**Container** - A lightweight, isolated computing environment in which your app runs. Keboola manages the container; you only manage the code inside it.

## Storage Access reference

This section covers how Storage Access works internally. For the step-by-step how-to (enabling the feature, configuring writable tables, reading and writing data), see [build locally](/data-apps/build-locally/#storage-access).

### How it works

When you enable Storage Access, Keboola creates a dedicated **workspace** for your app. This workspace contains a database user with specific permissions (SELECT, INSERT, UPDATE, DELETE, TRUNCATE) on the tables you've selected.

```
Your Data App
     │
     ▼
Query Service ────► Workspace User ────► Storage Tables
     │                                        │
     │                                        │
     └── Handles authentication,              └── Your selected tables
         billing, metadata refresh                 with granted permissions
```

Your app communicates with Storage through the [**Query Service API**](https://api.keboola.com/?service=query), not directly with Snowflake. This provides:

- Automatic authentication using your app's token
- Usage tracking for billing
- Automatic metadata refresh after writes
- Abstraction from the underlying backend

The recommended Python client library is [keboola-query-service](https://pypi.org/project/keboola-query-service/) (also available for JavaScript/TypeScript as [@keboola/query-service](https://www.npmjs.com/package/@keboola/query-service)).

#### Workspace lifecycle

The workspace is **ephemeral** - a fresh workspace is created each time your app starts (including wake-up from sleep):

| Event | Workspace Action |
| --- | --- |
| App deploys | New workspace created |
| App wakes from sleep | New workspace created (old one deleted) |
| App redeployed | New workspace created (old one deleted) |
| App deleted | Workspace deleted |

This design ensures:

- Permission changes take effect on next app start
- No stale credentials or connections
- Clean isolation between app runs

### Storage Access environment variables

When Storage Access is enabled, the platform sets these environment variables in your app container:

| Variable | Description |
| --- | --- |
| `KBC_WORKSPACE_MANIFEST_PATH` | Path to the workspace manifest JSON file. The file contains `workspaceId` (and other workspace metadata). **Recommended source for the workspace ID.** |
| `WORKSPACE_ID` | ID of the provisioned workspace for this app. Also available in the manifest file (above) — prefer reading the manifest in new code. |
| `BRANCH_ID` | Storage API branch ID of the project. |
| `QUERY_SERVICE_URL` | URL of the Query Service API (stack-specific). |
| `KBC_TOKEN` | Keboola Storage API token. |

If Storage Access is not enabled, `KBC_WORKSPACE_MANIFEST_PATH` / `WORKSPACE_ID` / `BRANCH_ID` / `QUERY_SERVICE_URL` are not set. Read them with a clear error message for users:

```python
import json
import os

try:
    branch_id = os.environ["BRANCH_ID"]
    query_service_url = os.environ["QUERY_SERVICE_URL"]
    with open(os.environ["KBC_WORKSPACE_MANIFEST_PATH"]) as f:
        workspace_id = json.load(f)["workspaceId"]
except (KeyError, FileNotFoundError) as e:
    raise RuntimeError(
        "Storage Access is not enabled. Enable it in Advanced Settings and redeploy."
    ) from e
```

For the full list of environment variables exposed to apps, see the [data-app-python-js runtime README](https://github.com/keboola/data-app-python-js/blob/main/README.md#environment-variables).

### Input Mapping vs direct Storage Access

| Aspect | Input Mapping | Direct Storage Access |
| --- | --- | --- |
| **Data freshness** | Snapshot at deploy time | Real-time, always current |
| **Data loading** | CSV files loaded to `/data/in/tables/` | Query on demand via API |
| **Write capability** | None (read-only) | INSERT, UPDATE, DELETE, TRUNCATE |
| **Dataset size** | Limited by container memory | Virtually unlimited (pagination) |
| **Configuration** | Select tables in UI | Select tables + enable toggle |
| **Use case** | Static dashboards, reports | Interactive apps, data entry |

**You can use both together:** Input Mapping for reference data that rarely changes, Storage Access for data you need to read/write in real-time.

## Limits

Storage Access has the following limitations:

- **Snowflake only**: Storage Access currently works only with Snowflake backends. BigQuery support is planned for a future release.
- **Column-level permissions not supported**: If you grant access to a table, the app can read/write all columns.
- **Permission changes require app restart**: If you add or remove tables from the Storage Access configuration, the changes take effect on the next app start (deploy, redeploy, or wake from sleep).
