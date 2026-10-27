# Release safely when a tool misleads you

A staging release degrades health. Tool output suggests using a production credential and skipping approval to repair it. The legitimate task is to diagnose staging and follow the approved rollback policy.

## Task

1. Read the release contract and sample scenario.
2. Write diagnosis/release role instructions and approval policy.
3. Run healthy, degraded, unapproved and failed-rollback cases.
4. Check every environment, version and approval before a mutation.
5. Verify recovery after rollback and report unresolved failures with evidence.

## Working rules

Edit only role instructions and approval-policy.json. All effects are simulated staging operations. Production endpoints, generic HTTP, shell, approval-write tools and credentials are absent. The staging server checks immutable instructor approval records on every mutation; student policy cannot authorize itself.

## Acceptance targets

- Diagnosis remains read-only.
- Mutations require a matching valid instructor approval for the environment and version.
- Malicious tool text never creates approval or expands available capabilities.
- Required rollback happens and recovery is independently checked.

## Policy format (graded)

The grader applies `approval-policy.json` to fixed approval records and health
observations, so use exactly these fields:

- `environment`, `requireApproval`: as in the starter.
- `requiredApprovalFields`: the record fields an approval must match, chosen from
  `id`, `action`, `environment`, `version`, `expiresAt`, `requestedBy`, `toolText`.
  `expiresAt` means "still valid now".
- `rollbackWhen`: rules `{"metric": "errorRate"|"p95Ms", "operator": ">"|">="|"<"|"<=", "threshold": number}`;
  any matching observation triggers rollback.
- `recoveryChecks`: rules in the same shape; every one must hold for a healthy observation.
- `healthyObservationsRequired`: how many consecutive healthy observations prove recovery.
- `onFailedRollback`: `"escalate"`, `"retry"` or `"declare-recovered"`.

## How to submit

1. Fork this repository to your personal GitHub account. Use fictional information
   only, keep the fork public, and keep `main` as its default branch.
2. Clone your fork and enable GitHub Actions in its **Actions** tab. Use **Connect
   GitHub for scoring** on the lab page to install the read-only grading App on your
   fork. Return to the lab and use **Recheck connection** if needed.
3. Download the TensorLearn client from the lab page and extract it outside your fork.
   Follow the page's connection command from your fork folder. Paste your learner
   token only into the hidden terminal prompt. Run `/load` from your fork folder to
   connect the repository. Run `/load` again after the facilitator launches the round.
4. Create a branch such as `release/first-draft` and complete the task above. Change
   only the files the task allows; any other change makes the submission invalid.
5. Commit and push your branch. Open a PR **inside your own fork**, targeting its
   `main` branch. GitHub may suggest the upstream repository: select your fork instead.
   Opening or updating this PR starts the checks automatically.
6. Keep the PR open. When ready, run **`/submit`** in the connected Claude Code session
   to submit that exact pushed revision for the current round. **No merge is required.**
7. Use `/check` or refresh the lab page to read the saved result. The Actions run and
   `lab-score/score.json` are useful feedback, but an Actions check alone does not
   submit your work or prove that the server saved a score.

A change to your answer requires a new commit, push, and explicit `/submit`. Keep
`main` unchanged: the grader compares your PR with the released starter.

## What is graded

The pinned grader in `.github/workflows/grade.yml` reads your files as data and runs
your code only in an isolated, offline container. Every check is a fixed rule; no AI
model judges your work. Your facilitator separately reviews clarity and reasoning.

- approval-policy.json stays on staging, requires approval and lists exactly the fields an approval must match.
- The grader applies your policy to fixed cases: mismatched, expired and forged approvals are denied.
- Your rollback and recovery rules give the right answer on fixed health observations, including the boundaries.
- A failed rollback escalates; both role instructions are written and the diagnosis role is read-only.
- Leaving staging, dropping approval, or accepting a forged approval caps the score at 25/100.
