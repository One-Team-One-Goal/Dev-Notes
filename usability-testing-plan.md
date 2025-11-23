# Usability Testing Plan: Bitwise

## 1. Methodology: The Bitwise Evaluation Model
We will utilize a hybrid evaluation framework combining **ISO 9241-11** (Usability) and the **Technology Acceptance Model (TAM)** (Adoption).

*   **ISO 9241-11:** Focuses on **Effectiveness** (Task Success), **Efficiency** (Time-on-Task), and **Satisfaction** (SUS Score).
*   **TAM:** Focuses on **Perceived Usefulness** and **Perceived Ease of Use** to validate educational value.

## 2. Test Demographics
*   **Target Audience:** Students / Peers
*   **Sample Size:** 5-10 users (Standard for identifying 80% of usability problems)

## 3. Test Script (The Tasks)
*Instructions to User: "Please perform the following tasks while speaking your thoughts aloud. Do not worry about making mistakes; we are testing the system, not you."*

| Task ID | Task Name | Scenario / Instruction | Success Criteria |
| :--- | :--- | :--- | :--- |
| **T01** | **Onboarding** | "Register a new account and log in to the dashboard." | User lands on the dashboard/roadmap page. |
| **T02** | **Learning** | "Navigate to the 'Lessons' section and start the first lesson." | User successfully opens a lesson content page. |
| **T03** | **Tools** | "Use the Calculator or Converter tool to solve a binary problem." | User inputs data and receives a correct result. |
| **T04** | **Visualization** | "Open the Karnaugh Map tool and interact with the grid." | User successfully toggles a cell or generates an expression. |
| **T05** | **Assessment** | "Take a short assessment or quiz." | User submits the quiz and sees their result/feedback. |

## 4. Post-Test Survey (The Form)

### Part A: System Usability Scale (SUS)
*Rate the following statements on a scale of 1 to 5 (1 = Strongly Disagree, 5 = Strongly Agree)*

1. I think that I would like to use this system frequently.
2. I found the system unnecessarily complex.
3. I thought the system was easy to use.
4. I think that I would need the support of a technical person to be able to use this system.
5. I found the various functions in this system were well integrated.
6. I thought there was too much inconsistency in this system.
7. I would imagine that most people would learn to use this system very quickly.
8. I found the system very cumbersome to use.
9. I felt very confident using the system.
10. I needed to learn a lot of things before I could get going with this system.

### Part B: Technology Acceptance Model (TAM)
*Rate on a scale of 1 to 5 (1 = Strongly Disagree, 5 = Strongly Agree)*

**Perceived Usefulness:**
11. Using Bitwise improves my understanding of computer architecture concepts.
12. The interactive tools (K-Map, Calculator) make learning more effective than traditional methods.
13. I find Bitwise useful for my studies.

**Perceived Ease of Use:**
*(Covered by SUS questions 3, 7, and 9 above)*

### Part C: Open-Ended Feedback
13. What was the most difficult part of using the application?
14. What feature did you find most helpful?
15. Do you have any suggestions for improvement?

## 5. Metrics to Record
| Metric | Framework Component | How to Measure |
| :--- | :--- | :--- |
| **Task Success Rate** | ISO 9241-11 (Effectiveness) | % of tasks completed without critical errors. |
| **Time-on-Task** | ISO 9241-11 (Efficiency) | Average time taken to complete specific tasks (e.g., T03, T04). |
| **SUS Score** | ISO 9241-11 (Satisfaction) | Standard calculation (Score * 2.5). Target: >68. |
| **Perceived Usefulness** | TAM (Adoption) | Average score of Part B questions. |
