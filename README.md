# Security Lab: Reconnaissance, Testing, and Report

Work for the OWASP Juice Shop local lab, covering reconnaissance, vulnerability testing, and the final penetration test report. Target: OWASP Juice Shop v20.3.0 at http://127.0.0.1:3000, run locally in Docker and isolated to localhost. Scope and authorization are defined in `rules-of-engagement.md` in the `security-lab` repository.

## Contents
- `pentest-report.md`: full penetration test report (summary, findings, remediation and retest, recommendations) - start here
- `findings.md`: detailed findings table from testing (SQL injection, XSS, access control, headers)
- `command-log.md`: sanitized reconnaissance commands and outputs
- `service-inventory.md`: observations mapped to risk hypotheses
- `attack-surface-diagram.md`: diagram of inputs and routes
