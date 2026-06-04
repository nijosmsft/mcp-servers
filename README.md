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
| [**etw-mcp**](https://github.com/nijosmsft/etw-mcp) | Analyze and capture Windows ETW/WPR traces (`.etl`). Wraps `xperf`, native ETW (OpenTraceW), a .NET TraceEvent sidecar, and pktmon. 60+ analysis tools, 4 capture-authoring tools, 9 curated `.wprp` profiles. Sidecar auto-bootstraps on first use. Honest symbol resolution (SYMFLAG_EXPORT tracking, `extra_symbol_paths`, cache-pollution diagnostics). | beta — v0.6.0 |
| [**perfmon-mcp**](https://github.com/nijosmsft/perfmon-mcp) | Windows performance counter (PDH) capture, `.blg` analysis, and live `Get-Counter` snapshots. 33 tools, 4 curated counter profiles including NIC RSS-queue validation with hot/idle detection + curated Mellanox RSS lens. | beta — v0.3.0 |
| [**sysinternals-mcp**](https://github.com/nijosmsft/sysinternals-mcp) | Wraps the Sysinternals tool suite (Handle, Sigcheck, PsList, AccessChk, ProcMon, Autoruns, Tcpvcon, Coreinfo, PsInfo, ListDLLs, ProcDump, Strings) for process introspection, binary triage, ACL audit, ProcMon capture authoring, and bootstrap install. 32 tools. Ships zero binaries — user provides Sysinternals install or invokes the `bootstrap_sysinternals` tool to download. | beta — v0.2.0 |
| [**LabLink**](https://github.com/nijosmsft/LabLink) | Give your AI assistant secure remote-hands on a fleet of Windows lab machines. MCP server + lightweight node agent (mTLS + token). Execute commands, move files, manage processes across many nodes. Pure Go, no runtime deps. Bulk `reboot_nodes` for parallel fleet reboots. Update an existing install with `scripts/Update-LabLink.ps1` (LabLink's equivalent of `pip install --upgrade`). | beta — v0.3.0 |

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
        "https://github.com/nijosmsft/etw-mcp/releases/download/v0.6.0/etw_mcp-0.6.0-py3-none-any.whl",
        "python", "-m", "etw_analyzer.server"
      ]
    },
    "perfmon-mcp": {
      "command": "uv",
      "args": [
        "run", "--no-project", "--with",
        "https://github.com/nijosmsft/perfmon-mcp/releases/download/v0.3.0/perfmon_mcp-0.3.0-py3-none-any.whl",
        "python", "-m", "perfmon_mcp.server"
      ]
    },
    "sysinternals-mcp": {
      "command": "uv",
      "args": [
        "run", "--no-project", "--with",
        "https://github.com/nijosmsft/sysinternals-mcp/releases/download/v0.2.0/sysinternals_mcp-0.2.0-py3-none-any.whl",
        "python", "-m", "sysinternals_mcp.server"
      ],
      "env": {
        "SYSINTERNALS_MCP_DIR": "C:\\Sysinternals"
      }
    },
    "lablink": {
      "command": "C:\\install\\lablink-server.exe"
    }
  }
}
```

**Note on the `etw-mcp` .NET sidecar:** v0.5+ auto-downloads `etw-extract.exe`
to `%LOCALAPPDATA%\etw-mcp\sidecar\v<version>\` on first `load_trace`. No manual
step needed. To pin a specific binary (e.g., for air-gapped use), set
`ETW_MCP_DOTNET_SIDECAR=<path>` in the server's `env`. To disable auto-fetch
entirely, set `ETW_MCP_NO_AUTO_DOWNLOAD=1` (the server then falls back to the
native ETW path that ships with the wheel).

**Note on the `sysinternals-mcp` binaries:** Sysinternals binaries are EULA-bound
and NOT redistributable. Either install the suite separately (download from
https://learn.microsoft.com/en-us/sysinternals/ and unzip to `C:\Sysinternals\`)
or let the MCP help via `bootstrap_sysinternals(target="local")` — it emits
paste-ready PowerShell to download + install + accept the EULA on your behalf.

For per-MCP install details and version pinning, see each repo's `README.md`.

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
