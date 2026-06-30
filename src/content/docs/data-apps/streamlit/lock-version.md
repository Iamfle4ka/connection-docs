---
title: Lock package versions
slug: 'data-apps/streamlit/lock-version'
description: Lock Streamlit package versions in Keboola for both deployment methods - from code and from a Git repository.
redirect_from:
  - /components/data-apps/lock-streamlit-version/
  - /components/data-apps/lock-streamlit-version/code-deployment/
  - /components/data-apps/lock-streamlit-version/git-deployment/
---



This guide explains how to lock package versions in Streamlit applications in Keboola, covering both deployment methods: from code and from a Git repository.

## Why lock package versions?

Locking package versions is essential for:

- **Stability and compatibility:** Avoid disruptions and ensure seamless app functionality by preventing issues with unexpected updates or incompatible package versions.
- **Reproducibility:** Guarantee consistent app behavior across development, testing, and production environments.
- **Ease of debugging:** Simplify troubleshooting and streamline future updates by maintaining a predictable environment.

Choose the deployment method that matches how you deploy your app:

- **[Code deployment](#code-deployment)** — upload or create `requirements.txt` in the Keboola UI, then enable the "Freeze versions" toggle.
- **[Git repository deployment](#git-repository-deployment)** — set up a virtual environment, install packages, and generate `requirements.txt` with `pip freeze`.

## Code deployment

This path explains how to lock package versions in Streamlit applications deployed from code in the Keboola platform.

### Development best practices

The recommended approach is to develop your Streamlit app locally first, as this significantly speeds up the development process. Once your app is working as expected, you can incorporate it into the Keboola environment.

### Locking package versions in Keboola

#### 1. Create and upload requirements.txt

When your app is ready for deployment, you have two options to create your requirements.txt file:

- **Manual creation:** Write the requirements.txt file manually, specifying exact versions for each package
- **Using pip freeze:** Generate the file automatically using `pip freeze > requirements.txt` in your local environment

You can then upload this file directly through the Keboola UI:

![Upload Requirements](/data-apps/streamlit/lock-upload-requirements.png)

You can also update requirements.txt in the UI:

![Update Requirements](/data-apps/streamlit/lock-update-requirements.png)

**Important notes:**
- If a package has a specific version defined, that exact version will be installed
- If no version is specified, the latest version will be installed
- You can edit the requirements.txt content directly in the UI before uploading if needed

#### 2. Enable version freezing

After uploading your requirements.txt:

1. Toggle the "Freeze versions"  option in the UI
2. Click "Start App" to propagate this change

![Freeze Version Toggle](/data-apps/streamlit/lock-freeze-version-toggle.png)

This action will:
- Execute a pip freeze command in the app
- Store the frozen dependencies in the app state
- Use these frozen versions for future redeploys and app wake-ups

![Start Data App](/data-apps/streamlit/lock-start-data-app.png)

#### 3. Updating package dependencies

To update the requirements for your app:

1. Start the app
2. Enable the "Update packages dependencies" toggle

![Update Dependencies](/data-apps/streamlit/lock-update-dependencies.png)

When this option is enabled:
- All package dependencies will be updated to their latest versions (if no version is explicitly defined)
- The dependencies will be resolved automatically
- The new package versions will be frozen in the app state

### Best practices for package management

- **Regular updates:** Periodically review and update your dependencies to ensure security and performance improvements
- **Version testing:** Always test your app with new package versions in a development environment before updating production
- **Documentation:** Keep track of why specific package versions are required to help with future maintenance
- **Minimal dependencies:** Only include packages that are actually needed by your application

## Git repository deployment

This path explains how to lock package versions in Streamlit applications deployed from a Git repository in the Keboola platform.

Follow this [tutorial](https://packaging.python.org/en/latest/tutorials/packaging-projects/) to package a simple Python project, set up the required files and structure, build the package, and upload it to the Python Package Index. For information on installing dependencies, refer to the [Streamlit documentation](https://docs.streamlit.io/knowledge-base/dependencies).

### How to lock package versions
To ensure your Streamlit app runs reliably in Keboola, follow these steps to lock package versions:

- [Set up a virtual environment](#set-up-a-virtual-environment)
- [Install required packages](#install-required-packages)
- [Generate a requirements.txt file](#generate-requirementstxt)

#### Set up a virtual environment
Create and activate a Python virtual environment to isolate your app's dependencies:

`python3 -m venv venv`

`source venv/bin/activate  # On Linux/Mac`

`venv\Scripts\activate     # On Windows`

#### Install required packages
Install the necessary packages for your Streamlit app:

`pip install streamlit pandas numpy`

#### Generate requirements.txt
Generate a `requirements.txt` file to lock the versions of all installed packages:

`pip freeze > requirements.txt`

This file lists all installed packages along with their specific versions. It ensures consistent package versions across environments (development and production) and prevents dependency conflicts.

***Note:** After updating or installing a new dependency, remember to run `pip freeze` again to update your `requirements.txt` file.*

### Example requirements.txt for a Streamlit app
Below is an example optimized for a typical Streamlit app running in Keboola:

`streamlit==1.24.0`

`pandas==1.3.5`

`numpy==1.21.6`

`plotly==5.20.0`

`matplotlib==3.7.2`

`requests==2.31.0`

`protobuf==3.19.6`

`validators==0.20.1`

`pydeck==0.8.1b0`

`watchdog==3.0.0`

### Best practices for managing dependencies
Following these best practices will help you maintain a stable, efficient, and consistent environment for your Streamlit applications in Keboola.

- **Keep it clean:** Include only the necessary packages to keep the environment lightweight.
- **Update regularly:** Periodically update and test dependencies in a development environment before generating a new `requirements.txt` file.
- **Test locally:** Verify that your app works correctly with the locked package versions in a local environment before deploying it in Keboola.
- **Match environments:** Use the same Python version in your local setup with Keboola's version (Python 3.10) to prevent version mismatches.
