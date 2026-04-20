# Listener SDK

`@expops/listener-sdk` is a TypeScript library for subscribing to real-time ExpOps run data from browser and Node.js applications. It provides React hooks and a backend abstraction that works with both Firestore (GCP) and local SQLite backends.

## Installation

```bash
npm install @expops/listener-sdk
```

**Peer dependencies** (must be installed separately):

```bash
npm install react@>=18 firebase@>=10 @tanstack/react-query@>=5
```

## Quick Start

Wrap your application in `LiveDataProvider` and use the provided hooks to subscribe to run data:

```tsx
import { LiveDataProvider, useLiveRunStatus } from "@expops/listener-sdk";

function App() {
  return (
    <LiveDataProvider config={{ backend: "local", serverUrl: "http://127.0.0.1:8765" }}>
      <RunDashboard projectId="my-project" runId="run-abc123" />
    </LiveDataProvider>
  );
}

function RunDashboard({ projectId, runId }) {
  const { data } = useLiveRunStatus(projectId, runId);

  return (
    <div>
      {Object.entries(data?.process_info ?? {}).map(([name, info]) => (
        <div key={name}>{name}: {info.status}</div>
      ))}
    </div>
  );
}
```

## Backend Configuration

`LiveDataProvider` accepts a `config` prop that selects between two backends:

### Local SQLite (development)

Connects to the ExpOps run server via Server-Sent Events. Use this with the local filesystem backend.

```tsx
<LiveDataProvider config={{ backend: "local", serverUrl: "http://127.0.0.1:8765" }}>
  {children}
</LiveDataProvider>
```

Start the local run server first:

```bash
python -m expops.web.run_server  # deprecated — used only for local SSE streaming
```

### Firestore (GCP)

Connects directly to Firestore using the Firebase JS SDK. Use this with the `gcp` KV backend.

```tsx
import { initializeApp } from "firebase/app";

const firebaseApp = initializeApp({
  apiKey: "...",
  projectId: "...",
  // other Firebase config
});

<LiveDataProvider config={{ backend: "firestore", firebaseApp }}>
  {children}
</LiveDataProvider>
```

## React Hooks

All hooks must be called inside a `LiveDataProvider`. They return `{ data, error, isLoading }` shapes from `@tanstack/react-query`.

### `useLiveRunsList(projectId)`

Subscribes to the list of run IDs for a project. Updates whenever a new run is created.

```tsx
import { useLiveRunsList } from "@expops/listener-sdk";

function RunList({ projectId }: { projectId: string }) {
  const { data } = useLiveRunsList(projectId);

  return (
    <ul>
      {(data?.runs ?? []).map(runId => (
        <li key={runId}>{runId}</li>
      ))}
    </ul>
  );
}
```

---

### `useLiveRunStatus(projectId, runId)`

Subscribes to the full status snapshot for a specific run. Includes per-step status, process-level aggregations, and timing information.

```tsx
import { useLiveRunStatus } from "@expops/listener-sdk";
import type { RunStatusResponse } from "@expops/listener-sdk";

function ProcessStatus({ projectId, runId }) {
  const { data } = useLiveRunStatus(projectId, runId);

  return (
    <table>
      {Object.entries(data?.process_info ?? {}).map(([name, info]) => (
        <tr key={name}>
          <td>{name}</td>
          <td>{info.status}</td>
          <td>{info.duration_sec?.toFixed(1)}s</td>
        </tr>
      ))}
    </table>
  );
}
```

The `process_info` object has the shape:

```typescript
type ProcessInfo = {
  status: "pending" | "running" | "completed" | "cached" | "failed";
  started_at: number | null;   // Unix timestamp
  ended_at: number | null;
  duration_sec: number | null;
};
```

---

### `useLiveProbeMetrics(projectId, runId, probePath)`

Subscribes to metrics for a specific probe path within a run. Useful for live-updating charts during training.

```tsx
import { useLiveProbeMetrics } from "@expops/listener-sdk";

function MetricsDisplay({ projectId, runId }) {
  const { data } = useLiveProbeMetrics(
    projectId,
    runId,
    "//*[@name='train_model']"
  );

  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

## Server-Side: SSE Endpoint (Node.js only)

For the local SQLite backend, the SDK includes a Node.js helper to mount an SSE route in Next.js or Express. Import from the `/server` subpath:

```typescript
import { createLiveDataSseResponse } from "@expops/listener-sdk/server";
```

**Next.js App Router example**:

```typescript
// app/api/live-data/[projectId]/route.ts
import { createLiveDataSseResponse } from "@expops/listener-sdk/server";

export async function GET(
  request: Request,
  { params }: { params: { projectId: string } }
) {
  return createLiveDataSseResponse(request, { projectId: params.projectId });
}
```

The handler watches the project's SQLite database for changes and emits SSE events to connected clients. It emits a `listener.unavailable` event if the database file cannot be found.

**Important**: Do not import `@expops/listener-sdk/server` in browser bundles or Edge middleware — it uses Node.js `fs` APIs.

## Type Reference

Key types exported from `@expops/listener-sdk`:

```typescript
type BackendType = "firestore" | "local";

type LiveDataConfig =
  | { backend: "firestore"; firebaseApp: FirebaseApp }
  | { backend: "local"; serverUrl: string };

type ProcessRunStatus = "pending" | "running" | "completed" | "cached" | "failed";

type ProcessInfo = {
  status: ProcessRunStatus;
  started_at: number | null;
  ended_at: number | null;
  duration_sec: number | null;
};

type RunStatusResponse = {
  status: string;
  steps: Record<string, unknown>;
  process_status: Record<string, ProcessRunStatus>;
  process_info: Record<string, ProcessInfo>;
};

type RunListResponse = {
  runs: string[];
};
```
