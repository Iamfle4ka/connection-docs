---
title: What are apps?
slug: 'data-apps/what-are-apps'
description: What Keboola apps are, why you would build one, the two technology stacks, and the features every app shares.
---



Apps are simple, interactive web applications that use data to deliver insights or automatically take action. They are custom-tailored to tackle specific business problems and enable dynamic, purpose-built user experiences.

As web applications deployed within your Keboola project, apps can be publicly accessed from outside the project. This means users accessing your app do not need access to a Keboola project - they simply visit a URL.

## Why build apps?

Apps transform how your organization interacts with data by providing purpose-built interfaces that solve specific business challenges.

### Replace traditional BI tools

Move beyond generic dashboards to create custom visualizations that match your exact needs:

* **Brand consistency**: Design dashboards in your company's color palette and visual identity.
* **Real-time updates**: Pull fresh data on demand without waiting for scheduled refreshes.
* **Cost efficiency**: Eliminate per-seat licensing costs of traditional BI platforms.

### Empower business users

Build interactive tools that put data in the hands of non-technical users:

* **Self-service analytics**: Enable business teams to explore data without SQL knowledge.
* **Data editing interfaces**: Replace Excel workflows with controlled, collaborative editing tools.
* **Automated workflows**: Trigger actions based on data insights without manual intervention.
* **Custom reporting**: Generate tailored reports that match your team's specific needs.

### Solve specific business problems

Create targeted solutions for unique challenges:

* **Recommendation engines**: Build personalized product or content recommendations.
* **Interactive segmentation tools**: Allow marketing teams to define and visualize customer segments.
* **AI integration**: Connect machine learning models directly to business workflows.
* **Financial analysis apps**: Provide real-time insights into spending patterns and forecasts.
* **Campaign management**: Design and execute data-driven marketing campaigns.

### Benefits of building in Keboola

* **Integrated platform**: Access all your transformed data directly from Storage.
* **No infrastructure management**: Focus on building features, not managing servers.
* **Secure by design**: Built-in authentication and authorization options.
* **Automatic scaling**: Resources scale based on usage.
* **Version control**: Git-based deployment for production-grade development.

## Choose your technology

Keboola Apps support two technology stacks, each designed for different use cases and skill sets.

### Streamlit (Python)

**Best for**: Data scientists, analysts, and Python developers who want rapid prototyping.

Streamlit is a Python framework that turns Python scripts into interactive web applications with minimal code.

**Key features:**

* Write apps entirely in Python.
* Built-in widgets and charts.
* Rapid development cycle.
* Extensive Python data science ecosystem.

**Ideal use cases:**

* Quick dashboards and prototypes.
* Data exploration tools.
* Model demos and visualizations.
* Internal reporting tools.

[Learn more about Streamlit apps -->](/data-apps/streamlit/)

### Python/JS (custom frameworks)

**Best for**: Development teams building production-grade, interactive applications.

Python/JavaScript apps give you full control over what you build and how you build it. Unlike Streamlit apps - which use a ready-made Python environment - Python/JS apps let you use **any Python web framework** (Flask, FastAPI, Dash), serve a **JavaScript frontend**, or combine both. You bring the code; Keboola handles the hosting, access control, and data connectivity.

**Key features:**

* Any Python or Node.js web framework.
* Full control over UI/UX.
* Custom Nginx and process configuration.
* Modern dependency management with `uv`.

**Ideal use cases:**

* Complex interactive applications.
* Customer-facing products.
* Applications requiring advanced UI features.
* Tools needing multiple data processing backends.

[Build a Python/JS app with Kai -->](/data-apps/build-with-kai/)

### How Python/JS apps work

When you deploy a Python/JS app, Keboola:

1. Clones your GitHub repository into the app container.
2. Runs your `setup.sh` script to install dependencies.
3. Starts your application using the process configuration you provide.
4. Routes all traffic through an internal web server (Nginx) that listens on port 8888.
5. Makes your app available at a secure URL.

You do not manage servers, ports, or Docker. You only manage your code and a small configuration folder called `keboola-config`.

```
Your browser -> Keboola -> Nginx (port 8888) -> Your app (internal port, e.g. 5000)
```

## Creating your first app

Getting started with apps is straightforward:

1. **Navigate to Apps**: In your Keboola project, go to the **Apps** section.
2. **Create new app**: Click the **+** button to create a new app.
3. **Choose type**: Choose your technology stack (Streamlit or Python/JS).

![Choose type](/data-apps/app-modal.png)

4. **Configure basic settings**: Enter a custom URL prefix for your app and select a deployment method (Code or Git repository).
5. **Deploy your app**: Click **Deploy** and your app will be available at its public URL.

For detailed setup instructions, see the technology-specific guides:

- [Build with Kai](/data-apps/build-with-kai/) or [build locally](/data-apps/build-locally/) (Python/JS)
- [Streamlit apps](/data-apps/streamlit/)

## Common features

Regardless of which technology you choose, all apps share these capabilities.

### Authentication and security

Keboola provides built-in authentication methods to protect your apps:

* **None**: No authentication - the app is publicly accessible. You may implement your own auth logic within the app.
* **Basic authentication**: Simple password protection using a Keboola-generated password.
* **OIDC/SSO integration**: Enterprise single sign-on support (Auth0, Google, Microsoft Entra ID, Okta).
* **GitHub authentication**: Restrict access using GitHub OAuth - by organization, team, repository, or allowed users.
* **GitLab authentication**: Restrict access using GitLab OAuth - by groups, projects, or allowed roles.
* **JumpCloud authentication**: Restrict access using JumpCloud OIDC - with optional role-based filtering.

[Learn more about authentication -->](/data-apps/authentication/)

### Data integration

* **Input Mapping**: Automatically load specific tables into your app.
* **Storage API client**: Programmatic access to all Storage features.
* **Environment variables**: Platform-provided env vars include `BRANCH_ID` (always set), `KBC_TOKEN` and `DATA_LOADER_API_URL` (with Data Loader), and `WORKSPACE_ID` / `QUERY_SERVICE_URL` / `KBC_WORKSPACE_MANIFEST_PATH` (with [Storage Access](/data-apps/build-locally/#storage-access)). See the [runtime README](https://github.com/keboola/data-app-python-js/blob/main/README.md#environment-variables) for the full list.

### Configuration and secrets

* **Environment variables**: Pass configuration to your apps.
* **Secrets management**: Securely store API keys and credentials.
* **Theming**: Customize appearance with predefined or custom themes (Streamlit apps).

### Resource management

* **Auto-sleep/resume**: Automatically suspend inactive apps to save costs.
* **Configurable timeouts**: Set inactivity periods from 5 minutes to 24 hours.
* **Scalable backend**: Choose appropriate compute resources for your needs.

### Development workflow

* **Code deployment**: Paste code directly for simple apps (Streamlit only).
* **Git integration**: Connect GitHub repositories for version control.
* **Private repository support**: Authenticate with personal access tokens or SSH keys.
* **Multiple branches**: Deploy from any branch for testing.

For how apps sleep, resume, deploy, and report errors, see the [reference](/data-apps/reference/).

## Example apps

### Hello World

Simple demonstration of Streamlit code deployment.

- [Live App](https://hello-world-75299519.hub.north-europe.azure.keboola.com)

### Titanic Demo App

Interactive data exploration with visualizations and filters.

- [Source Code](https://github.com/keboola/titanic-data-app)
- [Live App](https://titanic-demo-app-deployed-from-a-github-repository-49752295.hub.north-europe.azure.keboola.com/)

### Interactive Keboola Sheets

Collaborative data editing without exporting to external tools.

- [Live App](https://interactive-keboola-sheets-keboola-sheets-app-51814820.hub.north-europe.azure.keboola.com)
- [Source Code](https://github.com/keboola/planning-sheets-data-app/)

### eCommerce KPI Dashboard

Business metrics visualization with Slack integration.

- [Live App](https://interactive-kpi-report-kpi-app-71250158.hub.north-europe.azure.keboola.com)
- [Source Code](https://github.com/keboola/interactive-kpi-reporting)

### AI SQL Bot

Natural language interface for Snowflake queries.

- [Source Code](https://github.com/keboola/Kai-SQL-bot)

### Online Marketing Dashboard

Multi-channel campaign cost overview.

- [Live App](https://online-marketing-dashboard-49569899.hub.north-europe.azure.keboola.com)
- [Source Code](https://github.com/keboola/marketing-dashboard-data-app)
