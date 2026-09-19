# Incident Template

---

## Situation

_Describe the production scenario. What system is involved? What is happening?_

## Known Facts

- _Observable symptoms_
- _Available metrics and logs_
- _System configuration_
- _Recent changes_

## Unknowns

- _What information is missing?_
- _What can't be directly observed?_

## Constraints

- _Time pressure_
- _Resource limitations_
- _What you cannot change_

## SLO

- _Specific service level objectives that are at risk_

## Task

Produce:

1. **Hypotheses** — list all plausible explanations
2. **Prioritized investigation** — order hypotheses by information gain / cost of checking
3. **Missing instrumentation** — what telemetry would you add?
4. **Quantitative estimate** — predict the magnitude of the issue
5. **Discriminating experiments** — tests that distinguish between hypotheses
6. **Findings** — what did the investigation reveal?
7. **Architecture decision** — what change do you recommend?
8. **Trade-offs** — what does the decision cost?
9. **Rollback criteria** — when should the decision be reversed?

---

> **Note:** The solution is intentionally NOT included in the incident. The learner must work through the reasoning independently.
