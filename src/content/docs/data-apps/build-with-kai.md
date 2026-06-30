---
title: Build with Kai
slug: 'data-apps/build-with-kai'
description: Build a Python/JS app with the help of an AI assistant - from an empty GitHub repository to a deployed, secure URL.
---



This guide walks you through building a Python/JS app with the help of an AI assistant. Every step is explained in plain language - no deep technical knowledge required. You bring the code; Keboola handles the hosting, access control, and data connectivity. For how Python/JS apps run under the hood, see [what are apps?](/data-apps/what-are-apps/#how-pythonjs-apps-work).

## What you need before starting

* A **GitHub account** (free). Your app code lives here.
* A **Keboola project** with Apps available.
* Basic comfort with creating files and folders - an AI assistant can generate all the code for you.

You do **not** need to install Python, Node.js, or any development tools on your computer. Everything runs inside Keboola's infrastructure. (To develop and test on your own machine instead, see [build locally](/data-apps/build-locally/).)

## Repository structure - the golden rule

Every Python/JS app repository **must** follow this structure. Missing any piece will cause the deployment to fail.

```
your-repo/
├── keboola-config/             <- Required. Keboola reads this folder on startup.
│   ├── nginx/
│   │   └── sites/
│   │       └── default.conf    <- Required. Tells the web server how to reach your app.
│   ├── supervisord/
│   │   └── services/
│   │       └── app.conf        <- Required. Tells Keboola how to start your app.
│   └── setup.sh                <- Required if your app has dependencies. Installs them on startup.
├── pyproject.toml              <- Required for Python apps. Lists your Python packages.
└── app.py                      <- Your application code (name and structure are up to you).
```

:::caution
**Important:** The `keboola-config` folder name and the subfolder paths inside it are exact - do not rename them or change the folder hierarchy.
:::

## Step 1 - Create your GitHub repository

1. Go to [github.com](https://github.com/) and sign in.
2. Click **New repository**.
3. Give it a name (e.g., `my-keboola-app`).
4. Set it to **Public** or **Private** (both work; private requires extra credentials in Keboola - see Step 5).
5. Click **Create repository**.

You now have an empty repository. Next, you will add your app code and configuration files.

## Step 2 - Write your application code

Your app can be written in Python or JavaScript (Node.js). The only firm rules are:

* Your app must listen on an **internal port** (common choices: `5000` for Flask/FastAPI, `3000` for Node.js, `8050` for Dash).
* Your app must respond to both **GET and POST requests** on the root path (`/`). Keboola sends a POST request to `/` when it starts your app to check that it is running.

### Python example (Flask)

```python
# app.py
from flask import Flask
import os

app = Flask(__name__)
PORT = int(os.environ.get("PORT", 5000))

@app.route("/", methods=["GET", "POST"])   # POST is required - Keboola checks this on startup
def index():
    return "<h1>Hello from Keboola!</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=PORT)
```

### Node.js example (Express)

```javascript
// server.js
import express from 'express';
const app = express();
const PORT = process.env.PORT || 3000;

app.all('/', (req, res) => {          // app.all handles both GET and POST
    res.send('<h1>Hello from Keboola!</h1>');
});

app.listen(PORT, '0.0.0.0', () => {
    console.log(`App running on port ${PORT}`);
});
```

**Common mistake:** If your root route only handles GET (`@app.route("/")` in Flask or `app.get('/')` in Express), Keboola's startup check will receive a "Method Not Allowed" error and your app will appear broken even though it works locally. Always allow POST on `/`.

## Step 3 - Create the `keboola-config` folder

This folder is the bridge between your code and Keboola's infrastructure. You need three files inside it.

### File 1: `keboola-config/nginx/sites/default.conf`

This file configures Nginx - the web server that sits in front of your app. The only thing you need to change is the **port number** to match what your app listens on.

```
server {
    listen 8888;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;    # Change 5000 to your app's port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

If your app uses **WebSockets** (for example, Dash or any app with live-updating content), add these lines inside the `location /` block:

```
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
```

**What is Nginx?** Nginx (pronounced "engine-x") is a web server that handles incoming traffic and forwards it to your application. Think of it as a reception desk - visitors arrive at port 8888, and Nginx sends them to wherever your app is actually running.

### File 2: `keboola-config/supervisord/services/app.conf`

This file tells Keboola's process manager (Supervisord) how to start your application.

**For Python (Flask/FastAPI):**

```
[program:app]
command=uv run python /app/app.py
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

**For Python (FastAPI with uvicorn):**

```
[program:app]
command=uv run uvicorn app:app --host 127.0.0.1 --port 5000
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

**For Node.js:**

```
[program:app]
command=node /app/server.js
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

Rules to follow:

* Always use **absolute paths** starting with `/app/` (e.g., `/app/app.py`, not `./app.py`).
* For Python commands, always prefix with `uv run` (see [key terms](/data-apps/reference/#key-terms) for why).
* Do **not** add a `[program:nginx]` section - Nginx is managed by Keboola automatically.

**What is Supervisord?** Supervisord is a process manager - software that starts your application and automatically restarts it if it crashes. Think of it as a watchdog for your app.

### File 3: `keboola-config/setup.sh`

This script runs once when the container starts, before your app launches. Its job is to install all dependencies.

**For Python apps:**

```bash
#!/bin/bash
set -Eeuo pipefail
cd /app && uv sync
```

**For Node.js apps:**

```bash
#!/bin/bash
set -Eeuo pipefail
cd /app && npm install
```

**For apps with both Python and Node.js:**

```bash
#!/bin/bash
set -Eeuo pipefail
cd /app && uv sync &
cd /app/frontend && npm install &
wait
```

**Note:** Keboola automatically makes `setup.sh` executable before running it. If you are testing locally outside of Keboola, you may need to run `chmod +x keboola-config/setup.sh` yourself.

## Step 4 - Define your dependencies

### Python: `pyproject.toml`

Python apps must list their dependencies in a `pyproject.toml` file at the root of your repository. A plain `requirements.txt` file is **not sufficient** - Keboola uses a modern Python tool called `uv` that requires `pyproject.toml`.

```toml
[project]
name = "my-keboola-app"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "flask>=3.0.0",
    "pandas>=2.0.0",
    "requests>=2.31.0",
]

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

Replace the packages in `dependencies` with whatever your app needs. Add one package per line, using `>=` to specify the minimum version.

**What is `uv`?** `uv` is a fast Python package manager - it is the tool Keboola uses to install your Python libraries. Instead of the older `pip install` command, Keboola uses `uv sync`, which reads your `pyproject.toml` and installs everything listed there. You don't need to call `uv` directly from your application code. The `uv sync` command in your `setup.sh` runs automatically before your app starts, installing everything listed in `pyproject.toml`.

**What is `pip`?** `pip` is the traditional Python package installer. You may have seen commands like `pip install flask` in tutorials. In Keboola's Python/JS image, direct `pip` usage is blocked for system stability reasons - `uv sync` replaces it.

### Node.js: `package.json`

For Node.js apps, dependencies are defined in `package.json` as usual. The `setup.sh` script runs `npm install` to install them.

```json
{
  "name": "my-keboola-app",
  "version": "1.0.0",
  "type": "module",
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

## Step 5 - Configure and deploy in Keboola

1. In your Keboola project, go to **Apps** and click **Create App**.
2. Choose **Python/JS** as the type.
3. Under **Repository**, enter your GitHub repository URL.
4. Select the **branch** you want to deploy from (usually `main`).
5. If your repository is private, enable **Private repository** and authenticate using either:
   - **Personal Access Token**: Enter your GitHub username and a [Personal Access Token](https://github.com/settings/tokens), or
   - **SSH Private Key**: Paste your SSH private key for key-based authentication.
6. Add any [secrets](/data-apps/reference/#secrets-and-environment-variables) your app needs.
7. Click **Deploy**.

Keboola will clone your repository, run `setup.sh`, and start your app. The first deployment may take a few minutes. Once complete, a URL will appear - click it to open your app.

**To update your app:** Push changes to your GitHub repository, then click **Redeploy** in the Keboola app configuration. Keboola will pull the latest code and restart the app.

## Example: Hello World app

This is the simplest possible Python/JS app. It displays "Hello from Keboola!" in a browser.

You can clone the complete example from **[keboola/example-python-js-hello-world](https://github.com/keboola/example-python-js-hello-world)** and deploy it directly.

**Repository structure:**

```
helloworld/
├── keboola-config/
│   ├── nginx/
│   │   └── sites/
│   │       └── default.conf
│   ├── supervisord/
│   │   └── services/
│   │       └── app.conf
│   └── setup.sh
├── pyproject.toml
└── app.py
```

`app.py`:

```python
from flask import Flask
import os

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    return """
    <html>
      <body style="font-family: sans-serif; padding: 2rem;">
        <h1>Hello from Keboola!</h1>
        <p>Your Python/JS App is running.</p>
      </body>
    </html>
    """

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

`pyproject.toml`:

```toml
[project]
name = "helloworld"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "flask>=3.0.0",
]

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

`keboola-config/nginx/sites/default.conf`:

```
server {
    listen 8888;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

`keboola-config/supervisord/services/app.conf`:

```
[program:app]
command=uv run python /app/app.py
directory=/app
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

`keboola-config/setup.sh`:

```bash
#!/bin/bash
set -Eeuo pipefail
cd /app && uv sync
```

## Next steps

- Connect your app to Keboola data: [build locally](/data-apps/build-locally/) covers Input Mapping, the Storage API, and Storage Access.
- Store API keys and tokens safely: see [secrets and environment variables](/data-apps/reference/#secrets-and-environment-variables).
- If your app fails to start, see [troubleshooting](/data-apps/reference/#troubleshooting).
