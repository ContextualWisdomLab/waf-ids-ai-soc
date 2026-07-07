# Agent Instructions

- Keep the project Rust-first for gateway, DNSBL, and high-throughput control-plane code.
- Prefer proven security engines over fake in-house detections. Integrate OWASP CRS/Coraza, Suricata, STIX/TAXII, MISP, or OpenCTI before inventing equivalent engines.
- Do not use Figma Code Connect for this project unless explicitly requested later.
- Keep MVP work narrow: web management, gateway decisions, event/KPI visibility, and DNSBL publishing before broader SIEM/SOAR scope.
- Run `cargo fmt --check` and `cargo test` before claiming code is ready.

<!-- BEGIN cwl-agent-guidance -->
## Agent guidance (CWL governance)

Cross-agent conventions for any agent (Claude, Codex, Cursor, opencode, …) working in this repo.

### Security & review gate

- Every PR runs a central **Security Scan** required gate: `osv-scan` + `dependency-review` (diff-scoped) and `trivy-fs` (repo-wide, CRITICAL/HIGH, fixable). It runs against every PR base, **including stacked PRs**.
- A failing **`trivy-fs` is a REAL finding, not a flake.** Read the job log — it prints each finding's rule id / severity / file — or the run's SARIF results, then **remediate**:
  - Rust dependency CVE → bump the crate (`cargo update -p <crate>`, adjust `Cargo.toml`) and commit the updated `Cargo.lock`.
  - Container/OS finding → fix the base image or package in the `Dockerfile`.
  - k8s/IaC misconfig → fix `deploy/kubernetes/waf-ids-ai-soc.yaml` or `deploy/docker-compose.yml`.
  - Genuine false positive only → add a narrow, commented entry to `.trivyignore` (see the existing `AVD-KSV-0125` note for the expected style). Never broaden it to silence a real vuln.
- Do **not** weaken or disable the gate. A local scan with a stale DB misses findings: run `trivy --download-db-only` first, then scan the **merge ref**, not just the PR head (e.g. `trivy fs --scanners vuln,misconfig --severity CRITICAL,HIGH --ignore-unfixed .`).
- Gating is by the Security Scan **job result**, not the `code_scanning` rule. That org ruleset is intentionally **CodeQL-only** (multiple code-scanning tools can't converge on one PR ref) — do **not** add tools to it.

### Code exploration

- There is no `.codegraph/` index in this repo, so use normal search (grep/ripgrep, `cargo` tooling, editor navigation). If a `.codegraph/` index is added later, prefer CodeGraph (`codegraph explore "<query>"` or the code-review-graph MCP tools) before grep/find — it surfaces callers/callees/impact that text search misses.
<!-- END cwl-agent-guidance -->

