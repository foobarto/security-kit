# Installed skills — one-line reference

Every skill the kit can install, with a short description and primary use case.

## Sec-review skills (Trail of Bits + OWASP)

| Skill | Purpose |
|---|---|
| `audit-context-building` | Line-by-line architectural context before hunting. Run first in any review. |
| `variant-analysis` | After finding a bug, check whether the same pattern recurs elsewhere. |
| `supply-chain-risk-auditor` | Audit dependencies + install scripts + runtime remote fetchers for supply-chain attack vectors. |
| `insecure-defaults` | Hunt hardcoded secrets, weak auth, fail-open configs, permissive defaults. |
| `fp-check` | Validate suspected bugs to eliminate false positives. Use as a post-pass with explicit evidence requirements. |
| `differential-review` | Security-focused review of PR diffs / commit ranges. Blast-radius + trust-boundary + new-surface classification. |
| `semgrep-rule-creator` | Write custom Semgrep rules when you need to hunt a specific pattern across large codebases. |
| `constant-time-analysis` | Timing-side-channel audits for cryptographic code. |
| `zeroize-audit` | Memory zeroization audits in Rust (for code handling secrets). |
| `codeql` | Run CodeQL interprocedural taint tracking. Requires `codeql` binary installed. |
| `semgrep` | Run Semgrep with parallel subagents. Requires `semgrep` binary installed. |
| `sarif-parsing` | Parse SARIF output from semgrep/codeql. Support skill. |
| `owasp-security` | OWASP Top 10 (2025-2026) + ASVS 5.0 lens. Good for web-facing code. |

## Threat-modeling skills (josemlopez)

| Skill | Purpose |
|---|---|
| `tm-init` | Bootstrap a new threat model from code. Creates `.threatmodel/` state. |
| `tm-threats` | Enumerate STRIDE threats per trust boundary. |
| `tm-drift` | Detect drift against a prior threat-model snapshot. Answer "does this change invalidate the TM?" |
| `tm-full` | Run the complete threat-modeling workflow (init → threats → verify → compliance → report). |
| `tm-report` | Generate the final markdown threat model report. |
| `tm-verify` | Cross-check threats against mitigations. |
| `tm-compliance` | Map threats to compliance frameworks (SOC2, ISO-27001, etc.). |
| `tm-status` | Show current state of the threat model in this project. |
| `tm-tests` | Generate test cases from threat model. |

## Cybersecurity skills (Anthropic-Cybersecurity-Skills)

53 curated skills from 754 total, spanning 15 security domains. All skills follow the agentskills.io standard with MITRE ATT&CK, NIST CSF 2.0, ATLAS, D3FEND, and NIST AI RMF mappings.

### Cloud Security
| Skill | Purpose |
|---|---|
| `auditing-aws-s3-bucket-permissions` | Audit S3 bucket ACLs, policies, encryption, public access |
| `auditing-azure-active-directory-configuration` | Azure AD identity and access configuration review |
| `auditing-gcp-iam-permissions` | GCP IAM role and permission audit |
| `detecting-aws-cloudtrail-anomalies` | AWS audit log anomaly detection |
| `detecting-azure-lateral-movement` | Azure-specific lateral movement detection |
| `performing-cloud-penetration-testing-with-pacu` | AWS pentesting with Pacu framework |
| `implementing-cloud-security-posture-management` | CSPM implementation and monitoring |
| `implementing-zero-trust-in-cloud` | Cloud zero-trust architecture |

### Threat Hunting
| Skill | Purpose |
|---|---|
| `hunting-advanced-persistent-threats` | APT detection methodology |
| `hunting-for-cobalt-strike-beacons` | C2 beacon detection via TLS/JA3/HTTP patterns |
| `hunting-for-lateral-movement-via-wmi` | WMI-based lateral movement hunting |
| `hunting-for-persistence-mechanisms-in-windows` | Windows persistence detection |
| `hunting-for-webshell-activity` | Web shell detection |
| `building-threat-hunt-hypothesis-framework` | Structured threat hunting methodology |
| `detecting-living-off-the-land-attacks` | LOTL/BYOB technique detection |

### Malware Analysis
| Skill | Purpose |
|---|---|
| `reverse-engineering-malware-with-ghidra` | Static/dynamic RE with Ghidra |
| `analyzing-malware-behavior-with-cuckoo-sandbox` | Automated dynamic analysis pipeline |
| `extracting-iocs-from-malware-samples` | IOC extraction workflow |
| `analyzing-cobalt-strike-beacon-configuration` | C2 config extraction and analysis |
| `performing-malware-triage-with-yara` | Yara-based malware triage |
| `deobfuscating-powershell-obfuscated-malware` | PowerShell deobfuscation techniques |

### Incident Response
| Skill | Purpose |
|---|---|
| `conducting-cloud-incident-response` | Cloud-specific IR procedures |
| `conducting-malware-incident-response` | Malware incident containment and analysis |
| `conducting-phishing-incident-response` | Phishing IR playbook |
| `building-incident-response-playbook` | IR playbook creation |
| `building-incident-timeline-with-timesketch` | Timeline reconstruction |
| `collecting-volatile-evidence-from-compromised-host` | Live evidence collection |

### Penetration Testing / Red Teaming
| Skill | Purpose |
|---|---|
| `conducting-full-scope-red-team-engagement` | End-to-end red team operations |
| `conducting-internal-network-penetration-test` | Internal network pentest |
| `conducting-network-penetration-test` | External network pentest |
| `exploiting-active-directory-with-bloodhound` | AD attack path mapping |
| `performing-active-directory-penetration-test` | AD-specific pentest |
| `performing-web-application-penetration-test` | Web app pentest |
| `executing-red-team-engagement-planning` | Red team scoping and planning |

### Web Application Security
| Skill | Purpose |
|---|---|
| `conducting-api-security-testing` | API security assessment |
| `exploiting-sql-injection-vulnerabilities` | SQLi exploitation and remediation |
| `testing-for-broken-access-control` | OWASP access control testing |
| `performing-web-application-firewall-bypass` | WAF evasion testing |

### Network Security
| Skill | Purpose |
|---|---|
| `analyzing-network-traffic-with-wireshark` | Packet analysis and protocol inspection |
| `configuring-suricata-for-network-monitoring` | NIDS deployment and rule management |
| `scanning-network-with-nmap-advanced` | Advanced network reconnaissance |

### Digital Forensics
| Skill | Purpose |
|---|---|
| `analyzing-memory-dumps-with-volatility` | Memory forensics with Volatility |
| `performing-disk-forensics-investigation` | Disk forensic investigation |
| `acquiring-disk-image-with-dd-and-dcfldd` | Forensic disk imaging |

### SOC Operations
| Skill | Purpose |
|---|---|
| `analyzing-security-logs-with-splunk` | SIEM log analysis and correlation |
| `building-detection-rules-with-sigma` | Sigma rule creation |
| `triaging-security-incident` | SOC triage workflow |

### Identity & Access Management
| Skill | Purpose |
|---|---|
| `analyzing-active-directory-acl-abuse` | AD ACL attack detection and remediation |

### Container Security
| Skill | Purpose |
|---|---|
| `hardening-docker-containers-for-production` | Container hardening |
| `performing-container-security-scanning-with-trivy` | Image vulnerability scanning |

### DevSecOps
| Skill | Purpose |
|---|---|
| `implementing-secret-scanning-with-gitleaks` | Secret detection in code repos |

### Vulnerability Management
| Skill | Purpose |
|---|---|
| `performing-vulnerability-scanning-with-nessus` | Vulnerability scanning and reporting |

### Phishing Defense
| Skill | Purpose |
|---|---|
| `analyzing-email-headers-for-phishing-investigation` | Email header forensics |
| `implementing-dmarc-dkim-spf-email-security` | Email authentication protocols |

## Opt-in frameworks

### PentAGI (via `install-pentagi`)

PentAGI is not a skill but a full autonomous penetration testing platform. Installing it adds:

- A wrapper skill in `.claude/skills/pentagi/` with API interaction instructions
- `.pentagi/` directory with docker-compose.yml, .env.example, and management scripts
- Three helper scripts: `pentagi-start.sh`, `pentagi-stop.sh`, `pentagi-status.sh`

PentAGI provides:
- AI-powered multi-agent penetration testing (Researcher, Developer, Executor)
- 20+ built-in security tools (nmap, metasploit, sqlmap, etc.)
- Web UI at https://localhost:8443
- REST + GraphQL API for automation
- Long-term memory and knowledge graph (Neo4j/Graphiti)
- Support for 10+ LLM providers (OpenAI, Anthropic, Gemini, Ollama, etc.)

Use when you want fully autonomous pentesting with AI agents, not just skill-guided manual analysis.

### HexStrike AI (via `install-hexstrike`)

HexStrike AI is an MCP server, not a skill. Installing it adds:

- A wrapper skill in `.claude/skills/hexstrike/` with API and tool usage instructions
- `.hexstrike/` directory with:
  - Python virtual environment (`venv/`)
  - Symlink to source repo (`source/` → `$SOURCES/hexstrike-ai`)
  - Management scripts (`hexstrike-start.sh`, `hexstrike-stop.sh`, `hexstrike-setup.sh`)

HexStrike provides:
- 150+ security tools accessible via MCP protocol (nmap, nuclei, sqlmap, gobuster, hashcat, etc.)
- 12+ autonomous AI agents (BugBounty, CTF Solver, CVE Intelligence, Exploit Generator)
- REST API with intelligent decision engine and tool selection
- Smart caching, process management, and vulnerability intelligence
- Support for Claude Desktop, VS Code Copilot, Cursor, Roo Code, and any MCP-compatible agent

Use when you want AI agents to directly execute real security tools against authorized targets.

## Opt-in hooks (via `install-hooks`)

| Hook | Purpose | Risk |
|---|---|---|
| `gh-cli` | Redirects github.com URLs to the `gh` CLI for better structured output | Low |
| `modern-python` | Suggests `uv` when `pip` is used | Low |

## Opt-in raptor framework (via `install-raptor`)

Raptor is not a skill but a full offensive-security workspace harness. Installing it adds:

- ~17 agents (crash analysis, OSS forensics, exploitability validation)
- ~20 slash commands (`/raptor-scan`, `/raptor-fuzz`, `/raptor-web`, `/agentic`, `/oss-forensics`, etc.)
- A SessionStart hook that prints a banner
- ~400 lines appended to project `CLAUDE.md`
- A Python CLI pipeline (`raptor.py agentic --repo <path>`) that orchestrates Semgrep + CodeQL + Claude

Use when you want runnable PoCs and unified-diff patches for disclosure.

## Deliberately excluded

These are available in the source repos but not installed by any variant:

| Skill / plugin | Reason |
|---|---|
| `entry-point-analyzer` | Smart-contract-specific (Solidity/Vyper/Solana/Move/TON/CosmWasm). Doesn't fit general-purpose codebases. |
| `firebase-apk-scanner` | Active internet scanner. Too invasive for a default install. |
| `skill-improver` | Aggressive self-continuing Stop-hook loop. Burns tokens. |
| `second-opinion` | Auto-launches Codex MCP if present. Third-party dependency. |
| fr33d3m0n threat-modeling | Ships a PostToolUse Write hook. Use raptor if you want a hook-bearing framework. |

If you want any of these, clone the upstream repo yourself and symlink manually. The kit intentionally doesn't automate their installation.
