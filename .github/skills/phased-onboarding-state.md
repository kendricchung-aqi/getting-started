# Skill: Phased Onboarding State Management

This skill enables the AQUARIUS Implementation Specialist agent to support **phased onboarding engagements** — where work is spread across multiple sessions. A state file is created and maintained so the agent can pick up exactly where it left off.

---

## When to Use This Skill

Use this skill when:
- The customer onboarding spans multiple work sessions or days
- Not all 8 phases will be completed in a single sitting
- You want to track progress across phases, locations, or time series
- You need to hand off mid-engagement to another specialist

---

## State File

### Location

Create the state file in the customer's working directory or project folder:

```
<CustomerName>/onboarding-state.json
```

Example: `AcmeWater/onboarding-state.json`

### Schema

```json
{
  "customer": "<CustomerName>",
  "server": "<AQTS server hostname or IP>",
  "engagementStartDate": "YYYY-MM-DD",
  "lastUpdated": "YYYY-MM-DD",
  "currentPhase": <1–8>,
  "phases": {
    "1": { "status": "complete|in-progress|not-started", "notes": "" },
    "2": { "status": "complete|in-progress|not-started", "notes": "" },
    "3": { "status": "complete|in-progress|not-started", "notes": "" },
    "4": { "status": "complete|in-progress|not-started", "notes": "" },
    "5": { "status": "complete|in-progress|not-started", "notes": "" },
    "6": { "status": "complete|in-progress|not-started", "notes": "" },
    "7": { "status": "complete|in-progress|not-started", "notes": "" },
    "8": { "status": "complete|in-progress|not-started", "notes": "" }
  },
  "locations": [
    {
      "locationIdentifier": "<ID>",
      "displayName": "<Name>",
      "provisioningStatus": "complete|in-progress|not-started",
      "extractionStatus": "complete|in-progress|not-started",
      "transformationStatus": "complete|in-progress|not-started",
      "importStatus": "complete|in-progress|not-started",
      "validationStatus": "complete|in-progress|not-started",
      "notes": ""
    }
  ],
  "timeSeries": [
    {
      "locationIdentifier": "<ID>",
      "parameterId": "<e.g. HG>",
      "label": "<e.g. Telemetry>",
      "utcOffset": "<e.g. -07:00>",
      "historicalDateRange": { "start": "YYYY-MM-DD", "end": "YYYY-MM-DD" },
      "sourceRecordCount": null,
      "aqtsPointCount": null,
      "importStatus": "complete|in-progress|not-started",
      "validationStatus": "complete|in-progress|not-started",
      "deltaRequired": false,
      "deltaStatus": "complete|in-progress|not-started|not-applicable",
      "notes": ""
    }
  ],
  "openIssues": [
    {
      "id": 1,
      "description": "",
      "severity": "blocking|warning|info",
      "status": "open|resolved",
      "resolution": ""
    }
  ],
  "nextSteps": []
}
```

---

## Agent Behaviour — Session Start

When beginning a session for an existing engagement, the agent **must**:

1. Ask the user: *"Do you have an existing onboarding state file for this customer? If so, please provide the path or paste the contents."*
2. If a state file is provided:
   - Read and summarise the current progress (current phase, completed locations, open issues)
   - Identify the `currentPhase` and resume from there
   - List any `nextSteps` recorded from the previous session
   - Flag any `openIssues` with `status: "open"` before proceeding
3. If no state file exists:
   - Offer to create one: *"I'll create a state file to track progress across sessions."*
   - Collect the minimum required fields: `customer`, `server`, `engagementStartDate`
   - Set all phases to `not-started` and `currentPhase` to `1`

---

## Agent Behaviour — During a Session

As work progresses, the agent **must** keep the state file current:

- Update `phases[n].status` when a phase is entered (`in-progress`) or finished (`complete`)
- Update `phases[n].notes` with any important decisions, deviations, or client confirmations
- Update per-location and per-time-series statuses as each step completes
- Append new `openIssues` as they are discovered
- Resolve `openIssues` (set `status: "resolved"`) as they are closed
- Set `lastUpdated` to today's date at the end of every session

---

## Agent Behaviour — Session End

At the end of each session, the agent **must**:

1. Update `currentPhase` to reflect where work stopped
2. Populate `nextSteps` with a clear, ordered list of what to do next session. Example:
   ```json
   "nextSteps": [
     "Complete transformation for location 05JJ010 — deduplication script still running",
     "Import RawPoints.zip for HG.Telemetry@05JJ010",
     "Validate point counts for all 05JJ009 series against AQUARIUS API",
     "Follow up with client on confirmed data gap 2018-07-03 to 2018-07-10"
   ]
   ```
3. Output the full updated state file and instruct the user to save it for next session

---

## Phase Status Transitions

| Phase | Enters `in-progress` when… | Enters `complete` when… |
|-------|---------------------------|------------------------|
| 1 — Discovery & Planning | BA review or discovery session starts | Migration plan + field mapping confirmed with client |
| 2 — AQTS Configuration | ProvisioningTool tasks begin | All locations and time series provisioned and verified |
| 3 — Data Extraction | Legacy export scripts run | All required files extracted and organised |
| 4 — Transformation & Cleansing | Transformation scripts run | All EXIM archives built and checklist passed |
| 5 — Import Execution | First EXIM import runs | All imports complete, no unresolved log errors |
| 6 — Validation & QA | Post-import checks begin | Validation report accepted internally |
| 7 — Client Review | Client walkthrough scheduled | Written sign-off received from client |
| 8 — Documentation & Handover | Handover doc drafting begins | All deliverables handed over; handover meeting complete |

---

## Example: Resuming a Session

**User:** Here is my state file: `AcmeWater/onboarding-state.json`

**Agent response pattern:**

> **Resuming onboarding for AcmeWater** (last updated: 2026-05-07)
>
> **Current phase:** Phase 4 — Data Transformation & Cleansing (in progress)
>
> **Location progress:**
> - `05JJ009` — Extraction ✓, Transformation ✓, Import ✓, Validation ✓
> - `05JJ010` — Extraction ✓, Transformation ⏳ in progress
> - `05JJ011` — Not started
>
> **Open issues (1):**
> - ⚠️ `05JJ010`: Deduplication script found 47 duplicate timestamps — pending resolution
>
> **Next steps from last session:**
> 1. Complete transformation for `05JJ010` — resolve 47 duplicate timestamps
> 2. Build `RawPoints.zip` for `HG.Telemetry@05JJ010`
> 3. Begin extraction for `05JJ011`
>
> Ready to continue. Which item would you like to start with?

---

## State File Tips

- Store the state file alongside your migration working files (not inside the EXIM folder hierarchy)
- Commit the state file to source control (Git) if the team uses a shared repository
- One state file per customer engagement — do not reuse across different customers
- If a phase must be re-done (e.g., re-import after a fix), set its status back to `in-progress` and add a note explaining why

