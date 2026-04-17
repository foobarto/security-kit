# /security-kit install-hexstrike

Opt-in installation of the HexStrike AI MCP server (by 0x4m4).

## Why this is a separate command

HexStrike AI is not a skill — it's an MCP server that exposes 150+ security tools to AI agents:
- Python FastMCP server (`hexstrike_server.py`, `hexstrike_mcp.py`)
- 12+ autonomous AI agents (BugBounty, CTF Solver, CVE Intelligence, Exploit Generator, etc.)
- 150+ integrated security tools (nmap, nuclei, sqlmap, gobuster, hashcat, etc.)
- REST API with intelligent decision engine
- Requires Python venv, security tool installations, and MCP client configuration

The kit creates a wrapper skill, a deployment helper script, and an MCP config snippet.

## Prerequisites

- Python 3.8+
- `pip3`, `venv`
- Security tools installed (nmap, nuclei, gobuster, ffuf, sqlmap, etc.)
- Chromium/Chrome for browser agent
- An MCP-compatible AI client (Claude Desktop, VS Code Copilot, Cursor, Roo Code, etc.)

## What to do

### Step 1 — Ensure hexstrike source is cloned

If `$SOURCES/hexstrike-ai/` is missing:

```bash
git clone --depth 1 https://github.com/0x4m4/hexstrike-ai.git "$SOURCES/hexstrike-ai"
```

### Step 2 — Warn the user explicitly

```
Installing hexstrike will:

  1. Create a wrapper skill in .claude/skills/hexstrike/ with instructions
     for using the HexStrike AI MCP server and its 150+ security tools.

  2. Set up a Python virtual environment at .hexstrike/venv/ and install
     requirements from the hexstrike repo.

  3. Create helper scripts in .hexstrike/scripts/ for starting/stopping
     the MCP server.

  4. Provide an MCP config snippet for your AI client (Claude Desktop,
     VS Code Copilot, Cursor, etc.) — you must add it manually.

HexStrike gives AI agents the ability to run real security tools. Only install
if you have authorization to test the targets you plan to use.

Continue? (yes/no)
```

Wait for explicit "yes".

### Step 3 — Create the wrapper skill

```bash
mkdir -p .claude/skills/hexstrike .hexstrike/scripts
```

Write `.claude/skills/hexstrike/SKILL.md`:

```markdown
---
name: hexstrike
description: Use the HexStrike AI MCP server to run 150+ cybersecurity tools through an intelligent AI agent framework. Provides network scanning, web app testing, binary analysis, cloud security assessment, CTF solving, and bug bounty automation. Requires HexStrike MCP server running at localhost:8888.
license: MIT
metadata:
  author: 0x4m4
  source: https://github.com/0x4m4/hexstrike-ai
  version: "6.0"
---

# HexStrike AI — MCP Cybersecurity Automation Platform

## When to Use

- User asks to run security tools (nmap, nuclei, sqlmap, gobuster, etc.)
- User wants automated penetration testing or vulnerability assessment
- User needs CTF challenge solving assistance
- User wants bug bounty reconnaissance or analysis
- User asks to analyze a target for security vulnerabilities

## Prerequisites

- HexStrike MCP server running at http://localhost:8888
- Security tools installed and in PATH
- Target system ownership or explicit authorization

## Server Management

```bash
# Start HexStrike MCP server
../../.hexstrike/scripts/hexstrike-start.sh

# Stop HexStrike MCP server
../../.hexstrike/scripts/hexstrike-stop.sh

# Check server health
curl http://localhost:8888/health
```

## MCP Tools Available

### Network Reconnaissance (25+ tools)
- `nmap_scan()` — Advanced port scanning with NSE scripts
- `rustscan_scan()` — Ultra-fast port scanning
- `masscan_scan()` — Internet-scale port scanning
- `autorecon_scan()` — Comprehensive automated reconnaissance
- `amass_enum()` — Subdomain enumeration and OSINT

### Web Application Security (40+ tools)
- `gobuster_scan()` — Directory/file enumeration
- `feroxbuster_scan()` — Recursive content discovery
- `ffuf_scan()` — Fast web fuzzing
- `nuclei_scan()` — Vulnerability scanning (4000+ templates)
- `sqlmap_scan()` — SQL injection testing
- `wpscan_scan()` — WordPress security assessment
- `nikto_scan()` — Web server vulnerability scanning

### Binary Analysis & RE (25+ tools)
- `ghidra_analyze()` — Software reverse engineering
- `radare2_analyze()` — Advanced reverse engineering
- `gdb_debug()` — GNU debugger with exploit dev
- `pwntools_exploit()` — CTF framework and exploit dev

### Cloud Security (20+ tools)
- `prowler_assess()` — AWS/Azure/GCP security assessment
- `trivy_scan()` — Container vulnerability scanning
- `kube_hunter_scan()` — Kubernetes penetration testing

### Authentication & Password (12+ tools)
- `hydra_bruteforce()` — Network login cracking (50+ protocols)
- `hashcat_crack()` — GPU-accelerated password recovery
- `john_crack()` — Advanced password hash cracking

## API Endpoints

### Execute a command
```bash
curl -X POST http://localhost:8888/api/command \
  -H "Content-Type: application/json" \
  -d '{"command": "nmap -sV -sC target.com"}'
```

### AI-powered target analysis
```bash
curl -X POST http://localhost:8888/api/intelligence/analyze-target \
  -H "Content-Type: application/json" \
  -d '{"target": "example.com", "analysis_type": "comprehensive"}'
```

### Intelligent tool selection
```bash
curl -X POST http://localhost:8888/api/intelligence/select-tools \
  -H "Content-Type: application/json" \
  -d '{"target": "example.com", "test_type": "web"}'
```

## Workflow

1. Verify server is running: `curl http://localhost:8888/health`
2. Analyze target: POST to `/api/intelligence/analyze-target`
3. Execute selected tools via API or MCP tool calls
4. Correlate findings and generate report
5. Present results with severity ratings and remediation steps

## MCP Client Integration

### Claude Desktop
Add to `~/.config/Claude/claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "hexstrike-ai": {
      "command": "python3",
      "args": ["/path/to/hexstrike-ai/hexstrike_mcp.py", "--server", "http://localhost:8888"],
      "timeout": 300
    }
  }
}
```

### VS Code Copilot
Add to `.vscode/settings.json`:
```json
{
  "servers": {
    "hexstrike": {
      "type": "stdio",
      "command": "python3",
      "args": ["/path/to/hexstrike-ai/hexstrike_mcp.py", "--server", "http://localhost:8888"]
    }
  }
}
```

## Safety Notes

- HexStrike executes REAL security tools against targets
- Only test systems you own or have written authorization for
- Some tools (hydra, sqlmap, etc.) can cause service disruption
- Run in isolated VMs for safety
- Monitor AI agent activities through the dashboard
```

Write `.hexstrike/scripts/hexstrike-start.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
VENV="$SCRIPT_DIR/../venv"
SOURCE_DIR="$SCRIPT_DIR/../source"

if [ ! -d "$VENV" ]; then
  echo "ERROR: Virtual environment not found at $VENV"
  echo "Run: /security-kit install-hexstrike to set up"
  exit 1
fi

source "$VENV/bin/activate"

echo "Starting HexStrike AI MCP server..."
echo "Server: http://localhost:8888"
echo ""

cd "$SOURCE_DIR"
python3 hexstrike_server.py "$@"
```

Write `.hexstrike/scripts/hexstrike-stop.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Stopping HexStrike AI MCP server..."
pkill -f "hexstrike_server.py" 2>/dev/null && echo "Stopped." || echo "Not running."
```

Write `.hexstrike/scripts/hexstrike-setup.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
VENV="$SCRIPT_DIR/../venv"
SOURCE_DIR="$SCRIPT_DIR/../source"

echo "Setting up HexStrike AI..."

# Create virtual environment
if [ ! -d "$VENV" ]; then
  echo "Creating virtual environment..."
  python3 -m venv "$VENV"
fi

source "$VENV/bin/activate"

# Install Python dependencies
echo "Installing Python dependencies..."
pip install -r "$SOURCE_DIR/requirements.txt"

echo ""
echo "HexStrike AI setup complete."
echo "Start server: .hexstrike/scripts/hexstrike-start.sh"
echo "Health check: curl http://localhost:8888/health"
```

Make scripts executable:

```bash
chmod +x .hexstrike/scripts/*.sh
```

### Step 4 — Link the source directory

```bash
ln -sfn "$SOURCES/hexstrike-ai" .hexstrike/source
```

### Step 5 — Run setup

```bash
.hexstrike/scripts/hexstrike-setup.sh
```

### Step 6 — Update lockfile

Append to `.security-kit/installed.yaml`:

```yaml
hexstrike:
  installed: true
  installed_at: <timestamp>
  source: <SOURCES>/hexstrike-ai
  skill_path: .claude/skills/hexstrike
  deploy_dir: .hexstrike/
  venv: .hexstrike/venv/
```

### Step 7 — Report

```
HexStrike AI installed.

Setup complete. Server not yet running.

To start:
  .hexstrike/scripts/hexstrike-start.sh          # Start MCP server
  .hexstrike/scripts/hexstrike-start.sh --debug   # Start with debug logging
  .hexstrike/scripts/hexstrike-stop.sh            # Stop server

Health check:
  curl http://localhost:8888/health

MCP client config:
  See .claude/skills/hexstrike/SKILL.md for Claude Desktop, VS Code Copilot,
  and Cursor integration snippets.

To remove: /security-kit uninstall-hexstrike
```

### Step 8 — Alert on missing dependencies

Check for `python3`, `pip3`, and key security tools. If missing, print:

```
Note: python3 not found. Install Python 3.8+ before starting HexStrike.
Note: nmap not found. Install security tools for full functionality.
Note: nuclei not found. Install nuclei for vulnerability scanning.
```
