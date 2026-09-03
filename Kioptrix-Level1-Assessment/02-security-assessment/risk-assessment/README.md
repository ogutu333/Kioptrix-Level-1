# Risk Assessment – Kioptrix Level 1

**Objective:** Evaluate the security impact of the identified vulnerabilities and provide actionable remediation guidance.

**Methodology:** CVSS v3 scoring, business impact analysis, and prioritization based on exploitability.

**Key Findings Summary:**
- **Critical (1):** Samba trans2open – allows immediate root access.
- **High (2):** Apache/mod_ssl vulnerabilities – remote code execution.
- **Medium (2):** SSHv1 and outdated kernel – reduced security posture.

**Overall Risk Rating:** **CRITICAL**

---

## Risk Assessment Files

| File | Description |
|------|-------------|
| `vulnerability-severity-matrix.md` | CVSS scores and priority levels |
| `risk-analysis.md` | Detailed breakdown of impact, likelihood, and risk |
| `remediation-recommendations.md` | Prioritized action plan with timelines |
| `attack-tree.md` | Visual representation of potential attack paths |