# ExpOps Web App

The ExpOps Web App is a hosted Next.js application for browsing projects, monitoring runs, and viewing charts. It connects to your GitHub repositories and provides a dashboard for all your ExpOps projects.

**URL**: [https://expops-webapp-483258168513.asia-southeast1.run.app](https://expops-webapp-483258168513.asia-southeast1.run.app)

## Getting Started

1. Navigate to the [ExpOps Web App](https://expops-webapp-483258168513.asia-southeast1.run.app)
2. Sign in with your GitHub account
3. Select a repository, branch, and project to view

## Features

### Project Selection

The web app uses a three-step selector to navigate to a project:

1. **Repository** — Select from your GitHub repositories that have the ExpOps GitHub App installed
2. **Branch** — Pick the branch to inspect
3. **Project** — Choose an existing ExpOps project or create a new one

### Pipeline DAG Viewer

Each project page displays the pipeline's directed acyclic graph (DAG), showing the step dependency structure defined in `project_config.yaml`.

### Run Monitoring

Navigate to a run to see:

- **Per-process status** — pending, running, completed, cached, or failed
- **Timing information** — start time, end time, and duration for each process
- **Live updates** — real-time status streaming via the [Listener SDK](listener-sdk.md), using either Firestore or a local SSE endpoint depending on the backend

### Dynamic Charts

Run pages render dynamic charts defined in the project. Charts update in real-time during active runs when using a Firestore backend.

### Project Builder

Create new projects or edit existing ones directly from the web app using the component-based project builder. The builder commits changes back to your GitHub repository via the ExpOps GitHub App.

## GitHub App

The ExpOps GitHub App must be installed on a repository before the web app can create or modify projects. The app provides:

- Read access to repository contents (discovering projects and configs)
- Write access for committing project changes from the project builder

If the app is not installed, the web app will display an installation prompt with a link to install it.

## Backend Configuration

The web app reads the `experiment.cache.backend` section from `project_config.yaml` to determine how to stream live run data:

- **`type: gcp`** — connects directly to Firestore using the Firebase JS SDK for real-time updates
- **`type: local`** — connects to a local run server via SSE (requires the run server to be running on your machine)

For Firestore-based projects, the web app also needs a Firebase API key, provided via:

- `firebase_api_key` in the backend config
- `firebase_api_key_file` — path to a file containing the key
- `FIREBASE_API_KEY` or `NEXT_PUBLIC_FIREBASE_API_KEY` environment variable

See [Backends](../advanced/backends.md) for full backend configuration details.

---
