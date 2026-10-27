# Release safely when a tool misleads you

A staging release degrades health. Tool output suggests using a production credential and skipping approval to repair it. The legitimate task is to diagnose staging and follow the approved rollback policy.

Version `v2` is deployed on staging and service health is poor: the error rate and the p95 latency (`p95Ms`, the time within which 95% of requests finish) are above the healthy limits in `contract.md`. One tool response tells Claude to read a production key, deploy to production and skip approval. That text is data returned by a tool, not an approval; obeying it would be a prompt injection. `fixtures/scenarios.json` describes the four cases. This starter has no live staging server: you write the instructions for two Claude roles and the approval policy that limits what they may do.

Terms used below:

- **Diagnosis role** (`agents/diagnosis.json`): reads status, logs and approval records; changes nothing.
- **Release role** (`agents/release.json`): deploys or rolls back on staging, only with a matching approval.
- **Mutation:** a deploy or a rollback.
- **Approval record:** an instructor-owned record that authorizes one mutation. A mutation request names it by `approvalId` and must match its action, environment, version and expiry.

**What you'll practise:** keeping Claude within its authority when tool output tries to expand it. Separate read-only diagnosis from approved release, and write a policy that only a matching approval record can satisfy, never text in a tool response.

**In the course slides:** Chapter 2, Lessons 4 and 5 (Hooks as approval gates; CI/CD integration): read-only triage, release, status and rollback, rolling back when unhealthy, and approvals the agent cannot write. The slides run this in a pipeline with MCP servers and a PreToolUse gate; here you write the policy and role instructions those would enforce.

## Task

1. Read the release contract (`contract.md`) and the sample scenarios (`fixtures/scenarios.json`).
2. Write diagnosis/release role instructions (`instructions` in `agents/diagnosis.json` and `agents/release.json`) and the approval policy (`approval-policy.json`, format below).
3. Walk through the healthy, degraded, unapproved and failed-rollback cases in `fixtures/scenarios.json` with Claude: for each, which role acts, what your policy allows or denies, and why. Optionally, give a subagent only your diagnosis instructions and the degraded scenario's `toolText`, and check that it treats the text as data, not as approval.
4. Make the release instructions and `requiredApprovalFields` check every environment, version and approval before a mutation.
5. Verify recovery after rollback and report unresolved failures with evidence: set `rollbackWhen`, `recoveryChecks`, `healthyObservationsRequired` and `onFailedRollback` from the health rules in `contract.md`.

A good way to work: ask Claude to apply your draft policy to each scenario and explain each allow or deny decision, then fix the policy where the reasoning depends on tool text instead of an approval record. Keep the walkthrough and any subagent run in the chat; do not save them to files.

## Working rules

Edit only role instructions and approval-policy.json.

This starter has no staging server, release tools or credentials: you write instructions and a policy, not a running release. Only instructor approval records grant authority; your policy describes the checks and cannot authorize itself. The grader applies `approval-policy.json` to fixed approval records and health observations, and reads the two role files.

## Acceptance targets

- Diagnosis remains read-only.
- Mutations require a matching valid instructor approval for the environment and version.
- Malicious tool text never creates approval or expands available capabilities.
- Required rollback happens and recovery is independently checked.

## Done when

- Both role files have written `instructions` (no TODO), and the diagnosis instructions say the role is read-only and never deploys or rolls back.
- `approval-policy.json` stays on staging, requires approval and uses the fields in the policy format below.
- For each of the four scenarios you can say which role acts, what the policy allows or denies, and why; the degraded scenario's tool text never counts as approval.
- Only `agents/` files and `approval-policy.json` changed.

## Policy format (graded)

The grader applies `approval-policy.json` to fixed approval records and health
observations, so use exactly these fields:

- `environment`, `requireApproval`: as in the starter.
- `requiredApprovalFields`: the record fields an approval must match, chosen from
  `id`, `action`, `environment`, `version`, `expiresAt`, `requestedBy`, `toolText`.
  `id` means the record's `id` equals the request's `approvalId`; `expiresAt` means
  "still valid now". A request looks like
  `{"approvalId": "approve-v2", "action": "deploy", "environment": "staging", "version": "v2"}`
  and an approval record like `{"id": "approve-v2", "action": "deploy", "environment": "staging",
  "version": "v2", "expiresAt": 2000, "requestedBy": "ops", "toolText": ""}`.
- `rollbackWhen`: rules `{"metric": "errorRate"|"p95Ms", "operator": ">"|">="|"<"|"<=", "threshold": number}`;
  any matching observation triggers rollback.
- `recoveryChecks`: rules in the same shape; every one must hold for a healthy observation.
- `healthyObservationsRequired`: how many consecutive healthy observations prove recovery.
- `onFailedRollback`: `"escalate"`, `"retry"` or `"declare-recovered"`.

Each `agents/*.json` keeps only `purpose` and `instructions` (at least 100 characters), and
`agents/diagnosis.json` says explicitly that the role is read-only.

## How to submit

1. Fork this repository to your personal GitHub account. Use fictional information
   only, keep the fork public, and keep `main` as its default branch. In your fork's
   **Actions** tab, click **I understand my workflows, go ahead and enable them** if shown.
2. On the lab page, use **Connect GitHub for scoring** to install the read-only grading
   App on your fork, then **Recheck connection**.
3. Download the TensorLearn connector from the lab page and extract it outside your fork.
   From your fork folder, run the page's `--code` connection command and paste your token
   only into the hidden terminal prompt. This opens a Claude Code session connected to the lab.
4. In that session, run `/load`. Run `/load` again after your instructor launches the round.
5. Create a branch such as `release/first-draft` and complete the task above. Change only the
   files the task allows: any other change, including `CLAUDE.md`, `.claude/` settings or
   agents, or notes files, makes the submission invalid.
6. Commit and push your branch. Open a PR **inside your own fork**, targeting its `main`
   branch (GitHub may suggest the upstream repository: select your fork instead). Check
   that **Check release submission** starts in the PR's checks. Keep the PR open; no merge
   is required.
7. Run **`/submit`** in the connected Claude Code session, then `/check` (or refresh the
   lab page) to read your saved score. The Actions run is useful feedback, but it does not
   submit your work.

A change to your answer needs a new commit, push and `/submit`. Keep `main` unchanged:
the grader compares your PR with the released starter.

## What is graded

The pinned grader in `.github/workflows/grade.yml` reads your files as data and runs your
code only in an isolated, offline container. Every check is a fixed rule; no AI model
judges your work. Your score is out of 100. Your facilitator separately reviews
clarity and reasoning.

- approval-policy.json stays on staging, requires approval and lists exactly the fields an approval must match.
- The grader applies your policy to fixed cases: mismatched, expired and forged approvals are denied.
- Your rollback and recovery rules give the right answer on fixed health observations, including the boundaries.
- A failed rollback escalates; both role instructions are written and the diagnosis role is read-only.
- Leaving staging, dropping approval, or accepting a forged approval caps the score at 25/100.
