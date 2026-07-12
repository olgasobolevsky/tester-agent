---
name: test-plan-creator
description: >
  Create a comprehensive test plan or detailed test cases from a plain-text or Markdown spec file located
  in the specs/ folder. Produces a Requirements Digest, proposes structured test cases, and scores
  coverage against quality criteria. Use when: test plan, test cases, requirements analysis,
  coverage mapping, test strategy, what to test, scope, entry/exit criteria.
---

# Test Plan Creator

## Purpose

Produce a structured, reviewable test plan covering scope, acceptance criteria, gaps, test cases, and coverage scoring. Each phase ends with a checklist check, a formatted output, and a **human approval gate** — do not advance to the next phase until the user explicitly confirms.

Do not write implementation code — that belongs to `api-test-implementor` or `ui-test-implementor`.

## Input

The spec must be a `.txt` or `.md` file located in the `specs/` folder at the root of the workspace.
- If the user provides a filename (e.g. `checkout.txt` or `checkout.md`), read `specs/<filename>`.
- If the user provides no filename, list the files in `specs/` and ask the user which one to use.
- Do **not** accept Jira ticket keys, URLs, or inline pasted text as input.

---

## Phase 1 — Requirements Analysis & Gap Detection

### Steps
1. Read `specs/<filename>` in full.
2. Determine the **platform type** (web | mobile | both). If the spec does not explicitly state it, stop and ask:
   _"Is this a web app, mobile app, or both?"_ — wait for the answer before continuing.
3. Separate the content into two buckets:
   - **Feature Description** — narrative/context sections
   - **Requirements** — explicit functional and non-functional requirements
4. Cross-reference the two buckets and identify gaps:
   - Described behaviour that has no corresponding requirement
   - Requirements that lack a clear description or definition of done
   - Missing coverage of: positive behaviour, negative behaviour, edge cases
5. Build the **Acceptance Criteria List** — one unambiguous criterion per line, derived from the spec file requirements only (do not include gap additions yet).
6. For every gap found, produce a **Suggested Addition** entry with:
   - Category: `positive` | `negative` | `edge case`
   - Severity: 🔴 `Critical` (blocks release) | ⚠️ `Medium` (moderate impact) | ⚠️ `Low` (minor impact)
   - Missing point description
   - Suggested requirement or definition wording
7. If any **Critical** severity gaps exist, prepend a warning message before the output table

### Phase 1 Checklist
Before producing output, verify every item:
- [ ] Platform type (web | mobile | both) is confirmed
- [ ] Feature description and requirements are clearly separated
- [ ] Every requirement maps to at least one acceptance criterion
- [ ] Each gap has a severity rating (🔴 Critical | ⚠️ Medium | ⚠️ Low)
- [ ] Each gap has a concrete suggested addition
- [ ] If critical gaps exist, warning message is included

### Phase 1 Output

```markdown
## Requirements Analysis

### Platform
web | mobile | both

### Feature Description Summary
<1–3 sentence summary of the described feature>

### Acceptance Criteria
- AC-01: <criterion>
- AC-02: <criterion>
...

_Note: After the human gate, if the user approves any gap additions, add them to this list with a 📌 marker:_
- AC-13: 📌 <criterion from approved gap addition>
- AC-14: 📌 <criterion from approved gap addition>

### Gaps & Suggested Additions

> 🔴 **CRITICAL GAPS DETECTED**: X critical gap(s) found that may block release. Review and confirm before proceeding.

| # | Severity | Category | Missing Point | Suggested Wording |
|---|---|---|---|---|
| 1 | 🔴 Critical | negative | No requirement for data loss prevention | "The system shall..." |
| 2 | ⚠️ Medium | edge case | No requirement for concurrent access | "The system shall..." |
| 3 | ⚠️ Low | positive | No explicit rollback requirement | "The system shall..." |
...

_Note: Only show the critical gaps warning line if one or more 🔴 Critical severity gaps exist._
```

> **Human gate**: Present the output above and ask:
> _"Does the acceptance criteria list and gap analysis look correct? Should any suggested additions be included as confirmed requirements before I proceed to test case design? Reply **proceed** to continue or provide corrections."_
>
> If the user approves any gap additions, update the Acceptance Criteria list by adding them with a 📌 marker to indicate they came from gap analysis (not the original spec file).

---

## Phase 2 — Test Case Design

**Focus: E2E Testing Only**  
Create test cases for **end-to-end scenarios only** — complete user journeys from start to finish, not isolated unit or component tests. Each test case should represent a full workflow (e.g., user logs in → performs action → sees result), covering multiple system components and integrations.

### Steps
1. Use the confirmed acceptance criteria (including any additions approved in Phase 1).
2. Before creating test cases, perform a risk matrix evaluation for each acceptance criterion and relevant scenario candidate:
   - Assign **Probability**: likelihood the feature will fail here — 1 (low) | 2 (medium) | 3 (high)
   - Assign **Severity**: impact on the user/system if it fails — 1 (low) | 2 (medium) | 3 (high)
   - Derive **Priority** from Probability × Severity — P1 (critical, score 6–9) | P2 (high, score 4–5) | P3 (medium, score 2–3) | P4 (low, score 1)
   - Treat P1/P2 scenario candidates as high-risk and eligible for negative or edge-case test coverage
3. For each criterion, create test cases:
   - **Positive**: expected happy-path behaviour (always required)
   - **Negative**: invalid input, missing data, unauthorised access, error responses (only for P1/P2 scenario candidates from the risk matrix evaluation)
   - **Edge case**: boundary values, empty states, concurrent actions, max/min limits (only for P1/P2 scenario candidates from the risk matrix evaluation)
4. For each test case define:
   - **ID**: `TC-NNN`
   - **Title**: `should <expected behaviour> when <condition>`
   - **Description**: list of acceptance criteria this test case covers — one line per criterion (e.g. `AC-01: user can log in with valid credentials`)
   - **Type**: manual | api-automation | ui-automation. If `api-automation` is selected, the test must still cover a full API-level E2E workflow across integrated services, not an isolated endpoint check.
   - **Probability**: likelihood the feature will fail here — 1 (low) | 2 (medium) | 3 (high)
   - **Severity**: impact on the user/system if it fails — 1 (low) | 2 (medium) | 3 (high)
   - **Priority**: derived from Probability × Severity — P1 (critical, score 6–9) | P2 (high, score 4–5) | P3 (medium, score 2–3) | P4 (low, score 1)
   - **Category**: positive | negative | edge case
   - **Pre-conditions**: concrete, reproducible system state or data
   - **Steps**: numbered list where each step has THREE SEPARATE LINES (do not combine on one line):
     - Line 1 — **Step title**: one-line description of what this step does
     - Line 2 — **Action**: exact action to perform — e.g. click a specific button, navigate to a URL, send `POST /api/orders` with body `{"item": "X", "qty": 2}`
     - Line 3 — **Expected result**: concrete, assertion-level outcome — e.g. response status is `201`, element `#confirmation` is visible, error toast reads "Invalid email"

### Phase 2 Checklist
- [ ] Every acceptance criterion has at least one test case
- [ ] Each P1/P2 acceptance criterion has at least one negative test case
- [ ] Each P1/P2 acceptance criterion has at least one edge-case test case where boundary/concurrent/edge scenarios exist
- [ ] Boundary values are explicitly covered when risk matrix evaluation identifies boundary risk
- [ ] Pre-conditions are concrete and reproducible
- [ ] Expected results are specific (not "it works" or "no error")
- [ ] Positive tests exist for all ACs; negative and edge-case tests exist only where justified by P1/P2 risk evaluation
- [ ] No two test cases cover the exact same AC combination, steps, and conditions — if duplicates exist, merge or remove them and update the test case list before proceeding

### Phase 2 Output

The output has two parts: **1) Test Cases** (summary table + full detail per case) and **2) Coverage Matrix**.

```markdown
## Test Cases

### Summary

| ID | Title | Type | Priority | Category | Covers |
|---|---|---|---|---|---|
| TC-001 | ... | manual | P1 | positive | AC-01, AC-02 |
...

### Detail

#### TC-001: <Title>
**Type**: manual  
**Priority**: P1  
**Probability**: 3  
**Severity**: 3  
**Category**: positive  
**Description**:
- AC-01: <one-line summary of covered criterion>
- AC-02: <one-line summary of covered criterion>
**Pre-conditions**: ...  
**Steps**:
1. Step title: <what this step does>
   Action: <exact action — e.g. click "Submit" button / send `POST /api/login` with `{"email": "user@example.com", "password": "secret"}`>
   Expected result: <concrete assertion — e.g. response is `200 OK` with `{"token": "..."}` / user is redirected to `/dashboard`>
2. Step title: ...
   Action: ...
   Expected result: ...

## Coverage Matrix

Icons: 🖐 manual · 🤖 api-automation · 🌐 ui-automation

| Acceptance Criterion | TC-001 | TC-002 | TC-003 | ... |
|---|---|---|---|---|
| AC-01: <criterion summary> | 🤖 | 🌐 | | |
| AC-02: <criterion summary> | | | 🖐 | |
...

_Each cell shows the test type icon if that test case covers that acceptance criterion. Empty cells mean no coverage._
```

> **Human gate**: Present the output above and ask:
> _"Do the test cases look complete and correct? Reply **proceed** to continue to coverage scoring, or provide corrections."_

---

## Phase 3 — Coverage Scoring

### Steps
Evaluate the complete test plan (Phase 1 output + Phase 2 output) against the weighted categories below. For each category, assign a score from 0 to its maximum weight, then compute the total.

### Evaluation Categories

| # | Category | Max Weight | What to evaluate |
|---|---|---|---|
| 1 | **Requirements Coverage** | 25 | Every AC has at least one test case; no confirmed requirement is untested |
| 2 | **Scenario Completeness** | 20 | Positive scenarios cover all ACs; negative, edge-case, and boundary scenarios are included where P1/P2 risk evaluation justifies them |
| 3 | **Test Case Quality** | 20 | Steps are concrete and actionable; expected behaviors are specific and assertion-level; pre-conditions are reproducible |
| 4 | **Risk-Based Prioritisation** | 15 | Priority reflects Probability × Severity; P1/P2 scenario candidates have negative or edge-case coverage when applicable; highest-risk areas have the most coverage |
| 5 | **Clarity & Traceability** | 10 | Each test case links back to AC IDs; titles follow `should … when …` convention; no ambiguous language |
| 6 | **No Duplications** | 10 | No two test cases cover the identical AC combination, steps, and conditions; redundant or overlapping cases have been merged or removed |

**Total: 100 points. Minimum passing score: 75.**

For any category scoring below 60 % of its weight, identify the specific gaps and revise before presenting the final output.

### Phase 3 Checklist
- [ ] Scores assigned for all 6 categories with justification
- [ ] No category scores below 60 % of its max weight (otherwise revise)
- [ ] Total score ≥ 75

### Phase 3 Output

```markdown
## Test Plan Evaluation

| # | Category | Max | Score | % | Notes |
|---|---|---|---|---|---|
| 1 | Requirements Coverage | 25 | XX | XX% | <justification> |
| 2 | Scenario Completeness | 20 | XX | XX% | <justification> |
| 3 | Test Case Quality | 20 | XX | XX% | <justification> |
| 4 | Risk-Based Prioritisation | 15 | XX | XX% | <justification> |
| 5 | Clarity & Traceability | 10 | XX | XX% | <justification> |
| 6 | No Duplications | 10 | XX | XX% | <justification> |
| | **Total** | **100** | **XX** | **XX%** | |

### Gaps & Required Revisions
- <category>: <specific gap and what needs to change>
...
```

> **Human gate**: Present the evaluation and ask:
> _"Test plan evaluation complete with a score of XX/100. Reply **approve** to finalise the test plan, or provide feedback to revise."_

---

## Phase 4 — Summary File

### Steps
1. Collect the approved output from all three phases.
2. Create a file named `specs/<spec-base-name>-summary.md`, where `<spec-base-name>` is the spec filename without `.txt` or `.md` (e.g. `checkout.txt` or `checkout.md` becomes `specs/checkout-summary.md`).
3. Copy the "Gaps & Required Revisions" details from Phase 3 evaluation into the "Gaps & Revisions Applied" section of the summary file.
4. Write the full structured summary into the file using the template below.

### Phase 4 Output

File: `specs/<spec-base-name>-summary.md`

```markdown
# Test Plan Summary: <Feature Name>

**Spec file**: specs/<spec-filename>.<txt|md>  
**Platform**: web | mobile | both  
**Date**: <YYYY-MM-DD>  
**Evaluation score**: XX / 100

---

## Phase 1 — Requirements Analysis

### Feature Description Summary
<summary>

### Acceptance Criteria
- AC-01: ...
- AC-02: ...

### Gaps & Suggested Additions
| # | Severity | Category | Missing Point | Suggested Wording |
|---|---|---|---|---|

---

## Phase 2 — Test Cases

### Summary

| ID | Title | Type | Priority | Category | Covers |
|---|---|---|---|---|---|

### Detail

#### TC-001: <Title>
**Type**: ...  
**Priority**: ...  
**Probability**: ...  
**Severity**: ...  
**Category**: ...  
**Description**:
- AC-XX: ...
**Pre-conditions**: ...  
**Steps**:
1. Step title: ...
   Action: ...
   Expected result: ...

### Coverage Matrix

Icons: 🖐 manual · 🤖 api-automation · 🌐 ui-automation

| Acceptance Criterion | TC-001 | TC-002 | TC-003 | ... |
|---|---|---|---|---|

---

## Phase 3 — Evaluation

| # | Category | Max | Score | % | Notes |
|---|---|---|---|---|---|
| 1 | Requirements Coverage | 25 | | | |
| 2 | Scenario Completeness | 20 | | | |
| 3 | Test Case Quality | 20 | | | |
| 4 | Risk-Based Prioritisation | 15 | | | |
| 5 | Clarity & Traceability | 10 | | | |
| 6 | No Duplications | 10 | | | |
| | **Total** | **100** | | | |

### Gaps & Revisions Applied

_Copy the "Gaps & Required Revisions" list from Phase 3 evaluation here. If no gaps were identified (score 75+), state:_

No critical revisions required.

_Then list any suggestions from Phase 3 evaluation:_
- <Category>: <specific gap description and suggested improvement>
- <Category>: <specific gap description and suggested improvement>
...

_If there were no gaps at all, state:_

No gaps identified. All evaluation categories met quality thresholds.
```

> Confirm to the user: _"Summary saved to `specs/<spec-base-name>-summary.md`."_

