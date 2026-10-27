# Match Scoring Genie: Workato Genie

An AI-powered resume scoring solution for Workato. Recruiters chat with the **Match Scoring Genie** in Slack, send one job description and any number of resumes, and get back a ranked match score report. Scoring is done by OpenAI.

- **Owner:** Harbinger Group
- **Guide version:** 1.0 (2026)
- **Audience:** Client administrators and operations teams

---

## 1. Project Overview

### Purpose

The Match Scoring Genie reads each resume against a job description and returns a match tier, a final score and key reasoning for every candidate. It then adds a candidate ranking summary. Recruiters work entirely inside Slack and never need to open the Workato recipe build view.

### Business problem it solves

Screening a stack of resumes against one role by hand is slow and inconsistent. This package lets a recruiter upload a job description and several resumes in a single Slack message and get a consistent, ranked report in return. 

### Systems integrated

| System | Role |
| --- | --- |
| Workato | Hosts the skill recipe, the Genie and the audit data table |
| OpenAI | Reads each resume against the job description and returns the match score, tier and reasoning (model: `gpt-5.6-luna`) |
| Slack | Conversational chat interface for the Match Scoring Genie |

### How it works

1. A recruiter sends one job description file and one or more resume files to the Match Scoring Genie in Slack, asking for a match score report.
2. The Genie calls the Match Score generation skill.
3. The Genie replies in Slack with:
   - a match score table (candidate, email, match tier, final score, key reasoning), and
   - a candidate ranking summary.

The skill runs whenever a recruiter asks the Genie for generating a match score, so it does not need to be run manually.

---

## 2. Features and Workflows

### Features

- Conversational match score reporting through Slack
- One job description scored against any number of resumes in a single request
- Match tier, final score and key reasoning for each candidate
- Candidate ranking summary
- Audit trail of every OpenAI scoring call
- Credentials kept in Workato Connections only, never in recipe steps
- Access to the Genie limited to chosen Slack user groups

### Recipes

| Order | Asset | What it does |
| --- | --- | --- |
| 1 | `[MSE] │ Skill │ 001 Match Score generation` | Reads one job description file and multiple resume files, scores each candidate against the job description using OpenAI. |
| 2 | Match Scoring Genie | Receives the request in Slack, calls the skill, and shows the match score report and candidate ranking. |

### Callable recipes

None. The package contains one skill recipe, registered as an Enterprise skill under the Genie.

### Connections

- OpenAI account (AI resume scoring)
- Slack connection (chat interface for the Match Scoring Genie)

### Data tables

- `matchscore_openai_audit`: records each OpenAI scoring call.

### Lookup tables and environment properties

The package has no lookup tables and no environment properties today. If a customer-specific value is added later, keep it in an environment property, not inside a recipe step.

---

## 3. Package Contents

- **Package name:** Match Scoring Genie
- **Package format:** zip file imported through Workato Recipe Lifecycle Management (RLC)
- **Root folder:** `Match Scoring Genie`

| Asset type | Name | Count | Purpose |
| --- | --- | --- | --- |
| Skill recipe | `[MSE] │ Skill │ 001 Match Score generation` | 1 | Generates match scores for each resume against a job description. |
| Genie | Match Scoring Genie | 1 | Slack entry point for recruiters. It ships with no skills attached, so you must attach the skill after import. |
| Data table | `matchscore_openai_audit` | 1 | Audit log of each OpenAI scoring call. |
| Connections | OpenAI account, Slack connection | 2 | Connection shells. Credentials never travel with the package. |

The import should show exactly these assets.

**Skill "Use when" text** (confirm it reads exactly like this after attaching the skill):

> Use when the user asks to generate a match score report. Accepts one job description file and any number of resume files, and returns a match score result for each resume.

---

## 4. Installation and Deployment

### 4.1 Import into Workato

1. Log into the customer's Workato workspace.
2. Create a new folder named **Match Scoring Genie** to hold all imported assets.
3. Go to **Tools → Recipe Lifecycle Management**.
4. Select **Import Package**, choose the **Match Scoring Genie** folder as the destination, and upload the zip file.
5. Confirm the import completes without errors and shows these assets:

   | Asset type | Count |
   | --- | --- |
   | Skill recipe | 1 |
   | Genies | 1 |
   | Data tables | 1 |
   | Connections | 2 |

### 4.2 Configure connections

Recipes are imported inactive and unconfigured. Open the imported **Match Scoring Genie** project and re-authorize both connections with the customer's own credentials.

| Connection | What to re-authorize |
| --- | --- |
| OpenAI account | Customer's OpenAI API key (model `gpt-5.6-luna`) |
| Slack connection | Customer's Slack workspace and channel |

Double-check every connection before moving on. A missed value here is the most common cause of failure in the next step.

### 4.3 Connect Slack and grant access

1. In the Genie, open **Triggers** and add **Slack** under "Chat with this genie in". Confirm it shows **Active**.
2. Under **End user access**, add the user group that should be able to use the Genie. For testing, create a group (for example, "Test") under **Genie → End user access** and add the intended users.
3. Confirm the Match Scoring Genie name appears in the Slack workspace.
4. Confirm only the intended users or groups have access, not the entire "Everyone" system group, unless that is intentional.

### 4.4 Activate and verify

Activate the skill recipe and the Genie. No manual run is needed.

For the first run, send this prompt in Slack with one job description and two or three resumes attached:

```text
Generate a match score report for the attached job description and resumes.
```

Confirm:

- The skill's **Job history** shows success, and any failed steps are inspected.
- The Genie returns a match score table (candidate, email, match tier, final score, key reasoning) followed by a ranking summary.
- New rows appear in `matchscore_openai_audit`.
- A score is returned for every resume submitted, and the number of results matches the number of resume files attached.

Treat the deployment as complete only after a clean run in both the Workato job history and Slack.

---

## 5. Configuration and Prerequisites

### Access and workspace

- Recipe Lifecycle Management is available in the customer workspace (**Tools → Recipe Lifecycle Management**).
- The package zip and its version number are available from the release location.
- The workspace plan and task capacity are assigned. The skill makes **one OpenAI call per resume**, so capacity must cover the expected resume volume.

### Customer accounts and credentials

| Application | Requirement |
| --- | --- |
| OpenAI | A customer API key with access to the `gpt-5.6-luna` model and enough quota for the expected volume. |
| Slack | A Slack Admin available to approve the OAuth connection. The account must be allowed to post and read messages in the channel(s) the Genie will use. |

### Slack and users

- A list of the recruiters who will use the Genie, with their Slack accounts, so they can be added to the end-user group.
- A Slack channel or direct-message arrangement agreed for testing before go-live.

### Test data for the first run

- One sample job description file.
- Two or three sample resume files matched to that job description.

### Environment properties and dependencies

- No environment properties are used today.
- The solution depends on Workato (Recipe Lifecycle Management, Genies, Data Tables), OpenAI and Slack.

---

## 6. Security, Testing and Support

### Credential handling

- Never hard-code secrets in recipe steps. Always use Connections for OpenAI API keys and Slack tokens.
- Credentials never travel with a package. Every connection must be re-authorized in each workspace after import.
- Keep customer-specific values out of recipe steps. Use environment properties if any are added later.
- Use separate credentials per environment (for example, OpenAI Dev and Prod keys, and a test Slack channel versus the production channel).
- Document who owns credential rotation for OpenAI and Slack.
- Turn on SSO if the customer requires it.

### Access control and least privilege

Each customer gets their own workspace, so recipes, connections and data stay isolated from other customers. Invite users by email into only that customer's workspace.

| Role | Give to | Allowed | Not allowed |
| --- | --- | --- | --- |
| Administrator (customer-side, limited count) | Customer IT/HR owner | Manage users, connections and recipes as permitted by governance | Other customer workspaces |
| Operator (or similar run-focused role) | Day-to-day recruitment ops | Start/stop jobs, view job history, update connection credentials | Redesigning the core skill or scoring logic |
| Analyst / Viewer | Auditors | Read job reports and the audit data | Change credentials or recipes |
| Recruiter (Genie end user) | Recruiters, hiring managers | Use the Genie in Slack to upload a job description and resumes and receive the report | Access the recipe build view or connections |

Do not add customer users to the parent Development workspace unless they are co-builders under NDA.

**Customers may safely change:**

- Connection credentials and certificates for their own systems (OpenAI, Slack)
- Whether the skill recipe and the Genie are started or stopped
- The Genie's Enterprise skill active/inactive toggle
- The Slack end-user group

**Customers should not change:**

- The core scoring logic of the `[MSE] | Skill | 001 Match Score generation` recipe
- The scoring method, including how the match tier and final score are calculated
- Anything in another customer's workspace

### Go-live checklist

**Package import**

- [ ] Package version noted and zip stored in the release location
- [ ] Skill recipe present (`[MSE] | Skill | 001 Match Score generation`)
- [ ] Data table present (`matchscore_openai_audit`)
- [ ] Connections connected (OpenAI account, Slack)
- [ ] Genie present and configured (Match Scoring Genie)
- [ ] Skill attached to the Genie with the correct "Use when" text and shown as Active
- [ ] Skill recipe and Genie activated
- [ ] Slack tested with one job description and two or three resumes, returning the match score table and ranking summary
- [ ] New rows populated in `matchscore_openai_audit` after the test run
- [ ] End user confirms the Match Scoring Genie appears in the Slack workspace
- [ ] Match Scoring Genie name is present under the group name inside the Slack workspace

**Genie verification**

- [ ] Genie responds correctly to a request with one job description and several resumes
- [ ] Genie returns a score for every resume submitted, and the total matches the number of files attached
- [ ] Candidates cannot access the Genie unless their user group is added (boundary test)
- [ ] The user group is present under **End user access** for each Genie

### Troubleshooting

| Symptom | What to check |
| --- | --- |
| Genie does not respond in Slack | The Slack trigger shows Active, the Slack connection is authorized, and the user is in the End user access group. |
| Genie replies but no report is produced | The skill is attached under Enterprise skills, shows Active, and the "Use when" text is correct. |
| Skill job fails | The skill's Job history for the failed step. Then check the OpenAI key, model access (`gpt-5.6-luna`) and quota. |
| Fewer results than resumes attached | Job history for failed OpenAI calls, and workspace task capacity versus resume volume. |
| No new audit rows | The `matchscore_openai_audit` table was not cleared or replaced during an upgrade. Restore from backup if it was. |
| Genie missing after an upgrade | Re-check the skill attachment, Slack trigger and end-user access, as these must be verified after every import. |

### Ownership and support

- **Package owner:** Harbinger Group
---

Harbinger Group © 2026
