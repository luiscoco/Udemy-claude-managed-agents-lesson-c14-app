# C14 · Instructions and Explicit Context

Composes local instructions from a host-owned registry. User project text cannot select system authority. Each run starts from a fresh local seed. The run journal stores results, not production workflow state.

## Concepts students will learn

- Trusted style registry
- Project facts remain data
- Hash the effective instructions

## Configure and run

Use Node.js 24 or later. In PowerShell, from this repository's folder:

```powershell
node --version
npm ci
Copy-Item .env.example .env
notepad .env
```

No API key is required for the local experiment. Optional live commentary requires your own API key, agent ID and environment ID. It explains the local result; it does not activate the integration discussed in the lesson.

For the optional live path, set your API key, then run npm run setup if you do not already have resource IDs. The script creates the environment before a text-only agent, saves each ID immediately, and reuses IDs already configured. It uses CLAUDE_MODEL or the source default; choose a model available to your account. An uncertain network failure may have created a resource: inspect your account before retrying.

```powershell
npm start
```

Open `http://127.0.0.1:3014`. Navigation leads to `/guide.html#learning`, `/guide.html#run-guide`, `/architecture.html`, and the application lab. If you change `PORT`, use the configured port. Keep the terminal running; stop with Ctrl+C. `npm run dev` also starts the web application.

Select the first scenario, inspect its editable fields, press Run experiment, and compare the outcome and evidence with the table above. Select another scenario to investigate the boundary condition.

## Create your .env file

Copy `.env.example` to `.env` and enter your own values. Do not commit the real file. Restart the server after editing it. The example contains safe placeholders; optional settings may be left blank to use source defaults. Resource IDs and keys stay on the server.

```dotenv
ANTHROPIC_API_KEY=your_api_key
CLAUDE_MANAGED_AGENT_ID=your_agent_id
CLAUDE_MANAGED_ENVIRONMENT_ID=your_environment_id
PORT=3014
```

## How the app was build

### 1. Resolve instructions from a trusted registry

The style key must exist in the host-owned registry. Unreviewed keys are rejected instead of being treated as executable instruction templates.

Source: [src/lesson.ts](src/lesson.ts#L13) · selected statements

```typescript
const registry: Record<string, string> = {
  "review-v2": "Explain the change, then provide a concise review checklist.",
};

const key = text(input, "style");
```

### 2. Compose the effective system text

The base reviewer instruction and reviewed style are joined explicitly. This preserves the base instruction rather than accidentally replacing it with a style fragment.

Source: [src/lesson.ts](src/lesson.ts#L15)

```typescript
const system = ["You are a project reviewer.", registry[key]].join("\n\n");
```

### 3. Keep project material in the user-data channel

Hash the effective instruction bytes and inspect projectMessage separately. User-supplied facts are placed in a user message; they are not concatenated into the trusted system text.

Source: [src/lesson.ts](src/lesson.ts#L16)

```typescript
return report(
  "passed",
  "Composed trusted instructions while keeping project facts separate",
  {
    styleSource: "host-registry",
    system,
    systemHash: hash(system),
    projectMessage: { role: "user", text: text(input, "facts") },
  },
  [
    "Resolve a reviewed style",
    "Compose rather than silently discard the base system",
    "Hash the effective instruction bytes",
    "Send project material as data",
  ],
);
```

### Check your understanding

Run **Compose the reviewed style** in the application. Compare the result with these scenario expectations:

```json
{
  "input": {
    "style": "review-v2",
    "facts": "Aurora uses TypeScript."
  },
  "expectedOutcome": "passed",
  "checkEvidence": {
    "styleSource": "host-registry"
  }
}
```

Then try **Reject an unknown style**. Its expected outcome is `rejected`; identify the condition or evidence field that explains the difference.

## Validation

```powershell
npm test
python scripts/verify-walkthrough.py
```

The source hash check detects when walkthrough excerpts need refreshing. Browser verification covers page navigation, readable text, mobile layout, dark mode, diagram and image routes. Local scenarios and live provider calls are recorded separately in the workspace verification report; local success is not evidence of a live provider operation.

## Repository contents

Application sources, student web pages, safe configuration example and documentation images are included. `node_modules`, `presentation`, `output`, `reviewed`, `v3`, `teaching-material`, build output, local journals, credentials and local resource files are ignored. Existing teaching videos and presentations remain in the original workspace.
