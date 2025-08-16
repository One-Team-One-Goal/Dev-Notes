# Bitwise v2 — Requirements, Roadmap, and Tracking
This document consolidates SMART goals, functional requirements, acceptance criteria, and non-functional metrics for Bitwise v2. Use it as the single source of truth for scope, progress, and ownership.

## Legend
- Type: Functional = feature behavior; NFR = non-functional requirement; CST = constraint
- Priority: 1 = Highest, 2 = High, 3 = Medium
- Status %: Rough completion across design, implementation, and validation
---

## Requirements Matrix
| Goal ID | SMART Goal | Requirement ID | Requirement Statement | Transaction (Input → Process → Output) | Type | Priority | Status % | Notes / Issues | Owner | Acceptance Criteria (Given-When-Then) | Metrics / Constraints |
|---|---|---|---|---|---|---:|---:|---|---|---|---|
| G1 | Rewrite the User Module to make authentication less strict and optional for non-critical functionalities by the end of September 2025. | REQ-101 | The system shall provide user authentication using Supabase. | User clicks “Sign Up” → System uses Supabase authentication → User account is created in the database. | Functional | 1 | 60% | Restructured authentication ready; waiting for extra functionalities to test. | Infra Lead | Given a user provides valid credentials, when they sign up, then an account is created and stored in the database using Supabase. | NFR-001: 95% of authentication attempts complete in ≤ 1s during load testing. CST-001: Must use Supabase for authentication. |
| G1 | — | REQ-102 | The system shall allow access to predefined non-critical functionalities without authentication (e.g., using calculator, using circuit maker tool). | User accesses non-critical functionality → System allows interaction without requiring authentication. | Functional | 1 | 50% | Non-critical features list to be finalized. | Full Stack Dev | Given a user is not logged in, when they open a non-critical feature, then the system allows full use without requesting authentication. | NFR-002: Zero guest-user access attempts trigger login prompts during QA testing. |
| G2 | Improve Lessons to be more accessible, easier to understand, and simulate adaptive behavior by the end of September 2025. | REQ-201 | The system shall allow users to take a Boolean algebra practice test consisting of 10–15 multiple-choice questions. | User takes practice test → Dynamically generates 10–15 multiple-choice questions → Practice test presented to the user. | Functional | 1 | 10% | Dynamic MCQ generation logic pending implementation and testing. | Full Stack Dev | Given a user starts the pre-test, when they answer and submit, then the system stores responses in the database for scoring and retains them for analysis. | NFR-003: Each generated pre-test must be unique with ≥80% new questions and cover all required topics. |
| G2 | — | REQ-202 | The system shall provide automated feedback based on the user’s practice-test score. | System receives submitted answers → System calculates score → System generates feedback message. | Functional | 1 | 30% | Feedback generation logic pending integration. | Full Stack Dev | Given a score is calculated, when feedback is generated, then it must match the scoring rubric with 100% accuracy. | NFR-004: 95% of redirects complete within ≤ 2s in user simulation tests. |
| G2 | — | REQ-203 | The system shall redirect users to a tailored Boolean algebra lesson path based on the practice-test scoring rubric. | System receives feedback → System selects lesson path from rubric → System redirects user to that path. | Functional | 1 | 30% | Finalization of lesson and lesson path mapping needs final review. | Full Stack Dev | Given a user has completed the pre-test, when their score is matched to a rubric, then they are redirected to the correct lesson path. | NFR-005: 100% of redirections must match the lesson path associated with the correct tags. |
| G3 | Enhance the Boolean Algebra Calculator to handle more complex expressions by the 2nd week of September 2025. | REQ-301 | The system shall process and solve complex Boolean expressions (≤ 5 variables) and provide a step-by-step visual explanation. | User inputs Boolean equation → System computes solution → System returns result with step-by-step visual explanation. | Functional | 1 | 20% | Algorithm shows inconsistencies. | Backend Dev | Given a valid Boolean expression with ≤ 5 variables, when it is submitted, then the correct result and visual explanation are displayed. | NFR-005: Pass rate ≥ 90% in functional test cases for supported expressions. CST-005: Maximum Boolean expression size is ≤ 5 variables. |
| G4 | Add a Karnaugh Maps Visualizer by the end of September 2025. | REQ-401 | The system shall generate the Karnaugh map equivalent of a given Boolean expression. | User inputs Boolean algebra equation → System processes and simplifies expression → System generates Karnaugh Map. | Functional | 3 | Blocked | Implementation deferred due to lower priority in current sprint. | Frontend Dev | Given a valid Boolean expression, when it is submitted, then a Karnaugh map is generated that matches the simplified form; invalid inputs trigger an error message. | NFR-006: 90% of render events ≤ 2s during benchmark testing. CST-006: Feature cannot exceed priority 3 until next sprint. |
| G5 | Integrate a third-party circuit maker app via API call by the end of September 2025. | REQ-501 | The system shall display the logic circuit equivalent of a given Boolean expression using a third-party circuit maker API. | User inputs Boolean expression → System calls third-party API → Circuit diagram is displayed. | Functional | 3 | Blocked | Implementation deferred due to lower priority in current sprint. | Infra Lead | Given a Boolean expression, when it is sent to the third-party API, then the returned diagram matches the logical output; API failure returns a fallback error message. | NFR-006: 90% of render events ≤ 2s during benchmark testing. CST-003: Must comply with third-party licensing terms. CST-006: Feature cannot exceed priority 3 until next sprint. |
| G5 | — | REQ-502 | The system shall allow users to manually create and edit logic circuits using the third-party circuit maker API interface. | User opens circuit maker tool → System displays blank circuit canvas → User edits and saves circuit. | Functional | 2 | Blocked | Implementation deferred due to lower priority. | Infra Lead | Given a blank circuit canvas, when a user creates/edits and saves, then the system must store and retrieve the saved design accurately. | CST-003: Must comply with third-party licensing terms. |
---

## Goal Snapshots
- G1: Auth flexibility and Supabase integration — In progress (REQ-101: 60%, REQ-102: 50%).
- G2: Adaptive lessons — Early stage (REQ-201/202/203: 10–30%).
- G3: Boolean calculator — Working API; improving consistency and step-by-step output (20%).
- G4: Karnaugh Maps — Blocked this sprint; revisit next sprint.
- G5: Circuit maker integration — Blocked pending licensing and prioritization.
---

## Updating This Document
- Add new requirements or changes by appending rows to the matrix.
- Keep Status % conservative and justify in Notes / Issues.
- Link related PRs or Issue IDs in Notes / Issues when available.
# Dev Notes:

# To Do's:
- fix build issues ui
- finish calculator AST (let claude do it eheh)
- discuss final reqs for the app.


## Keiru 
- BATI KAG NAWNG


## Mars


## Val
### Reports:

### To Do's:
- finish calculator AST.
- connect backend + frontend (talleco people should do this btw :3 )
- finish auth

### Finished
- [x] auth for backend + frontend.
- [x] finished calculator api (smart goal #2)


## Angela