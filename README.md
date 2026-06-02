# MCP servers for Windows performance, debugging, and lab orchestration

A curated index of [MCP](https://modelcontextprotocol.io) servers built for
Windows perf / debugging / observability workflows. Designed for AI agents
(Copilot CLI, Claude Code, Cursor, ...).

These MCPs follow a **federation pattern**: each MCP is narrowly scoped to
one domain or data source. Install the ones you need, skip the ones you
don't.

---

## Servers

| Server | Purpose | Status |
|---|---|---|
| [**etw-mcp**](https://github.com/nijosmsft/etw-mcp) | Analyze and capture Windows ETW/WPR traces (`.etl`). Wraps `xperf`, native ETW (OpenTraceW), a .NET TraceEvent sidecar, and pktmon. 60+ analysis tools, 4 capture-authoring tools, 9 curated `.wprp` profiles. | beta — v0.3.0 |
| [**perfmon-mcp**](https://github.com/nijosmsft/perfmon-mcp) | Windows performance counter (PDH) capture, `.blg` analysis, and live `Get-Counter` snapshots. 14 tools, 4 curated counter profiles including NIC RSS-queue validation. | alpha — v0.1.0 |
| [**sysinternals-mcp**](https://github.com/nijosmsft/sysinternals-mcp) | Wraps the Sysinternals tool suite (Handle, Sigcheck, PsList, AccessChk, ProcMon) for process introspection, binary triage, ACL audit, and ProcMon capture authoring. Ships zero binaries — user provides Sysinternals install. | alpha — v0.1.0 |
| [**LabLink**](https://github.com/nijosmsft/LabLink) | Give your AI assistant secure remote-hands on a fleet of Windows lab machines. MCP server + lightweight node agent (mTLS + token). Execute commands, move files, manage processes across many nodes. Pure Go, no runtime deps. | beta |

---

## Quick install — Copilot CLI

Add to `~/.copilot/mcp-config.json`:

```jsonc
{
  "mcpServers": {
    "etw-mcp": {
      "command": "uv",
      "args": [
        "run", "--no-project", "--with",
        "https://github.com/nijosmsft/etw-mcp/releases/latest/download/etw_mcp-py3-none-any.whl",
        "python", "-m", "etw_analyzer.server"
      ],
      "env": {
        "ETW_MCP_DOTNET_SIDECAR": "C:\\install\\etw-extract.exe"
      }
    },
    "lablink": {
      "command": "C:\\install\\lablink-mcp.exe"
    }
  }
}
```

For per-MCP install details, EULA acceptance (where relevant), and version
pinning, see each repo's `README.md`.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              AI client (Copilot CLI, Claude, Cursor)            │
└──────────────────────────────┬──────────────────────────────────┘
                               │ MCP / stdio
       ┌──────────┬────────────┼────────────┬──────────────┐
       ▼          ▼            ▼            ▼              ▼
   ┌────────┐ ┌──────────┐ ┌───────────────┐ ┌──────────┐
   │etw-mcp │ │perfmon-  │ │sysinternals-  │ │ LabLink  │
   │(.etl)  │ │mcp (.blg)│ │mcp (procmon)  │ │(remote)  │
   └────────┘ └──────────┘ └───────────────┘ └──────────┘
```

Each MCP runs in its own stdio process. The AI client connects to whichever
ones are configured; MCPs do not need to know about each other.

---

## Contributing / feedback

File issues on the individual repo whose surface area is affected.
Cross-cutting design discussion (new MCPs, naming, federation patterns)
goes here in `mcp-servers/issues`.

---

## Status

This index is hand-maintained. If a link is stale or a new MCP is missing,
open an issue.
