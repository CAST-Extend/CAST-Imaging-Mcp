# CAST Imaging MCP Server

## Overview

The CAST Imaging MCP Server bridges the gap between AI agents and real software architecture by exposing comprehensive software intelligence through the Model Context Protocol (MCP). It allows the user to get insights about their applications ,transactions, dependencies, quality patterns, and architectural structures by prompting AI agents.

This service provides AI Agents with the understanding of your applications from portfolio-level inventories to granular object relationships. Whether you're exploring database schemas, analyzing transaction flows, investigating quality issues, the MCP server delivers precise, paginated data that enables informed architectural decisions.

### Use Cases
- **Portfolio-level exploration**: List applications available in your CAST Imaging and provides detailed information about them.
- **Application architecture analysis**: Explore architectural graphs at different levels and visualize system structure.
- **Transaction & data flow mapping**: Analyze transactions, trace data entity interaction networks, explore complex objects and sequencing.
- **Quality & security insights**: Identify vulnerabilities, cloud readiness blockers and structural flaws in the applications.
- **Database schema exploration**: Browse database tables, columns, relationships, and constraints across applications.
- **Object-level investigation**: Search objects and their properties and analyze caller/callee relationships.
---

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration Reference](#configuration-reference)
- [Running (HTTP)](#running-http)
- [Windows Installation](#windows-installation)
- [VS Code + GitHub Copilot Setup](#vs-code--github-copilot-setup)
- [Verification & Test Queries](#verification--test-queries)
- [Toolsets & Tools](#toolsets--tools)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)
- [Package Contents](#package-contents)
- [Support](#support)

---

## Prerequisites

### Required
- **Linux** system with **Docker** installed and running.
- **CAST Imaging‑APIs** service reachable from this host
- **MCP‑aware client** (e.g., GitHub Copilot in VS Code, Claude Desktop)
- **CAST Imaging API Key** (generate from your Imaging user profile)

### Download
- Download the installer for CAST Imaging MCP server

> The MCP server should run on a Linux system. CAST Imaging‑APIs may run on the same or a different Linux machine.

---

## Quick Start
1. **Verify Imaging‑APIs** is live:
   ```bash
   curl -H "x-api-key: <your-imaging-api-key>"         http://{CONTROL_PANEL_HOST}:8090/imaging/apis/rest/ready
   # expected: true
   ```
2. **Extract** the installer zip.
3. **Configure** `config/app.config` and `.env`.
4. **Start** the server with `./run.sh --install`.
5. **Configure client** (VS Code Copilot or Claude) with your server URL.
6. **Test** queries (see examples below).

---

## Installation of CAST Imaging MCP server

### 1) Extract the installer
```text
com.castsoftware.imaging.mcpserver.docker.1.0.2-funcrel/
├─ config/
│  └─ app.config
├─ .env                      # hidden; use `ls -a`
├─ docker-compose.yml
├─ run.sh
├─ README.md
└─ copilot-instructions.md
```

### 2) Verify CAST Imaging‑APIs
Make sure the Control Panel/registry is reachable and Imaging‑APIs is ready (see Quick Start step #1).

### 3) Configure server
Edit `config/app.config`:
```ini
HOST_CONTROL_PANEL="your-control-panel-host"   # REQUIRED: Control Panel service registry host/IP
PORT_CONTROL_PANEL=8098                        # Default Control Panel registry port
IMAGING_PAGE_SIZE=1000                         # Internal fetch batch size
IMAGING_DISPLAY_PAGE_SIZE=20                   # Page size in responses
IMAGING_CODE=False                             # Source code access via MCP (security sensitive)
DEBUG_MODE=false                               # Debug mode to see the activity in logs
IMAGING_DOMAIN="default"                       # Imaging domain/tenant
CONTROL_PANEL_SSL_ENABLED=false                # Set true only if Control Panel config/eureka is on HTTPS
MCP_TOOL_SURFACE_PROFILE="full"                # MCP tool exposure profile: "intents" or "full"
MCP_INTENTS_HIDDEN_FUNCTIONS=""                # Comma-separated structural functions hidden from intents mode
SERVICE_HOST="your-service-host"               # REQUIRED: The IP/hostname of the machine on which the MCP Server is running
SSL_CA_BUNDLE=""                               # Optional PEM bundle for private/self-signed CAs
```

Set your exposed port in `.env` (hidden file):
```env
MCP_SERVER_PORT=8282
```
> **Tip:** On Linux, files starting with a dot (like `.env`) are hidden. Use `ls -a` to view.

### 4) Start
```bash
chmod +x run.sh
./run.sh --install
```
This launches the Docker Compose stack and the MCP server.

If you use `SSL_CA_BUNDLE`, place the PEM file in a `certificates/` folder next to `docker-compose.yml`. The compose file mounts that folder into the container at `/app/certificates`.

---

## Configuration Reference

### `HOST_CONTROL_PANEL` (Required)
**Default:** None (user needs to provide the value)  
**Description:** IP or hostname of the machine running the Control Panel Service.

This is a service registry used by Imaging to register and discover its internal services. It is automatically deployed when you install Imaging. You should enter the IP address of the Linux machine running the Imaging Control Panel service.

### `PORT_CONTROL_PANEL`
**Default:** `8098`  
**Description:** Control Panel registry port.

### `IMAGING_PAGE_SIZE`
**Default:** `1000`  
**Description:** Sets the number of records the MCP server fetches per request (internal batching). Tune for throughput.

### `IMAGING_DISPLAY_PAGE_SIZE`
**Default:** `20`  
**Description:** Sets how many records are shown per page in the user-facing response. Tune for UX.

### `IMAGING_CODE`
**Default:** `False`  
**Description:** Controls whether the agent can access source code via the MCP server.

It's `false` by default, as agents typically work on open repos. Enabling it can expose source code to anyone with MCP access, making it a potential security risk, especially relevant for clients like Claude desktop without direct repo access.

### `DEBUG_MODE`
**Default:** `false`  
**Description:** Debug mode will show teh activity in the logs. When set to true it will show the tool picked, the API called and the result for each user prompt. When set to false, it will only show the tool picked and the corresponding API called.

### `IMAGING_DOMAIN`
**Default:** `default`  
**Description:** Imaging domain/tenant used to fetch data in multi-tenant environments.

### `CONTROL_PANEL_SSL_ENABLED`
**Default:** `false`  
**Description:** Set to `true` when Control Panel config/eureka endpoints run on HTTPS.

### `MCP_TOOL_SURFACE_PROFILE`
**Default:** `full`
**Description:** Selects the MCP tool exposure profile. Use `full` to expose the complete toolset, or `intents` for the meta-tool surface.

### `MCP_INTENTS_HIDDEN_FUNCTIONS`
**Default:** empty
**Description:** Comma-separated list of structural functions to hide when `MCP_TOOL_SURFACE_PROFILE` is set to `intents`.

### `SERVICE_HOST` (Required)
**Default:** None (user needs to provide the value)  
**Description:** IP/hostname of the machine on which the MCP Server is running.

### `SSL_CA_BUNDLE`
**Default:** empty  
**Description:** Optional path to a PEM CA bundle for private/self-signed certificate chains. With the Docker installer, place the PEM under `./certificates/` and reference it as `/app/certificates/<file>.pem`.

---

## Running (HTTP)

Start the server with:
```bash
chmod +x run.sh
./run.sh --install
```

---

## Windows Installation

> This section provides detailed steps for installing and configuring the CAST Imaging MCP Server on Windows systems.

### Prerequisites for Windows
- **Windows** system
- **CAST Imaging‑APIs** service running and accessible
- **MCP‑aware client** (e.g., GitHub Copilot in VS Code, Claude Desktop)
- **CAST Imaging API Key** (generate from your Imaging user profile)

### 1) Download and Extract
1. Download the installer package for windows.
2. Extract the zip file to your preferred location
3. Navigate to the extracted directory

**Package contents:**
```
com.castsoftware.imaging.mcpserver.__MCP_VERSION__*/
├─ mcpserver/
├─ tools/
├─ configuration.conf
├─ mcp-server-installer.bat
└─ README.md
```

### 2) Verify CAST Imaging‑APIs
Make sure the Control Panel/registry is reachable and Imaging‑APIs is ready:
```bash
curl -H "x-api-key: <your-imaging-api-key>" 
     http://{HOSTNAME_CONTROL_PANEL}:8090/rest/ready
# expected: true
```

### 3) Configure Server
Edit `configuration.conf` with your environment settings:
```ini
HOSTNAME_CONTROL_PANEL="your-control-panel-host"      # Required: IP or hostname of Control Panel server
SERVICE_HOST="your-service-host"                      # Required: IP/hostname of the MCP Server machine
PORT_CONTROL_PANEL=8098                               # Default Control Panel registry port  
MCP_SERVER_PORT=8282                                  # Default MCP server port
INSTALL_DIR=C:\Program Files\Cast\Imaging-MCP-Server  # Default installation location
CONFIG_DIR=C:\ProgramData\CAST\Imaging-MCP-Server     # Default config files location
IMAGING_PAGE_SIZE=1000                                # Records per internal request
IMAGING_DISPLAY_PAGE_SIZE=20                          # Records per response page
IMAGING_CODE=False                                    # Enable source code access (security consideration)
DEBUG_MODE=false                                      # Debug mode to see the activity in logs
IMAGING_DOMAIN=default                                # Imaging domain/tenant
CONTROL_PANEL_SSL_ENABLED=false                       # Set true only if Control Panel config/eureka is on HTTPS
MCP_TOOL_SURFACE_PROFILE=full                         # MCP tool exposure profile: full or intents
MCP_INTENTS_HIDDEN_FUNCTIONS=                         # Comma-separated structural functions hidden from intents mode
SSL_CA_BUNDLE=                                        # Optional PEM bundle for private/self-signed CAs
```

### 4) Install as Windows Service
Run the installation using the batch script:
```bash
mcp-server-installer.bat --install configuration.conf
```

This installs the MCP server as a Windows service. After successful installation, you'll see a Windows service named **"CAST Imaging MCP Server"** in your services list.

### 5) Verify Installation
Check that the service is running:
- Open **Services** (services.msc)
- Look for **"CAST Imaging MCP Server"**
- Ensure status is **"Running"**

### Windows-Specific Operations

#### Update MCP Server
To update an existing installation:
```bash
mcp-server-installer.bat --update
```
> No configuration changes needed for updates.

#### Uninstall MCP Server
To completely remove the MCP server:
```bash
mcp-server-installer.bat --uninstall
```
> This removes the Windows service and cleans up all MCP-related files.

#### Get Help
For available commands:
```bash
mcp-server-installer.bat --help
```

**Available commands:**
```
mcp-server-installer.bat --install configuration.conf   : Install CAST Imaging MCP server
mcp-server-installer.bat --update                       : Update CAST Imaging MCP server
mcp-server-installer.bat --uninstall                    : Uninstall CAST Imaging MCP server
mcp-server-installer.bat --help                         : Display help message
```

### Windows Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Service won't start | Configuration error | Check `%CONFIG_DIR%/setup-config/app.config` |
| Connection refused | Service not running | Restart "CAST Imaging MCP Server" service |
| Installation fails | Insufficient permissions | Run installer as Administrator |

---

## VS Code + GitHub Copilot Setup

> This section provides detailed steps for for connecting the CAST Imaging MCP Server with Github Copilot.

### 1) Install GitHub Copilot on your VS code
- Open the Extensions view (Ctrl+Shift+X or Cmd+Shift+X).
- Search for "GitHub Copilot" and click Install.
- Sign in to GitHub when prompted

### 2) Generate CAST Imaging API Key
- Login to your CAST Imaging
- Navigate to your profile section.
- Generate a new API key (copy and save this key securely as you'll need it in the next steps)

### 3) Create MCP configuration (`.vscode/mcp.json`)
- Open any folder in VS Code and create a .vscode folder in its root. Inside that, create mcp.json file.

```json
{
  "inputs": [
    {
      "id": "imaging-key",
      "type": "promptString",
      "description": "CAST Imaging API Key"
    },
    {
      "id": "imaging-tenant",
      "type": "promptString",
      "description": "Imaging tenant"
    }
  ],
  "servers": {
    "imaging": {
      "type": "http",
      "url": "http://<your-mcp-server-host:port>/mcp/",
      "headers": {
        "x-api-key": "${input:imaging-key}",
        "x-user-tenant": "${input:imaging-tenant}"
      }
    }
  }
}
```

### 4) (Alternative) Manual registration flow
If VS Code doesn’t auto‑pick the server:
1. Open **Copilot Chat** → gear icon → **Select tools that are available to chat**
2. **+ Add More Tools…** → **+ Add MCP Server…** → choose **HTTP**
3. Enter `http(s)://<your-mcp-server-host:port>/mcp/` and confirm
4. VS Code writes `.vscode/copilot/mcp.json` → add the header block:
```json
"my-mcp-server-xxxx": {
  "url": "http://<your-mcp-server-host:port>/mcp/",
  "headers": {
    "x-api-key": "${input:imaging-key}",
    "x-user-tenant": "${input:imaging-tenant}"
  }
}
```

### 5) Project instructions for Copilot
Copy `copilot-instructions.md` from the installer into your VScode by creating a folder named `.github` and then pasting the `copilot-instructions.md` file inside it. This will help the agent in proving better responses.

---

## Verification & Test Queries

### Connectivity checks
```bash
# MCP container running
docker ps | grep mcp-server

# Imaging‑APIs health
curl -H "x-api-key: <your-key>"      http://{CONTROL_PANEL_HOST}:8090/imaging/apis/rest/ready
```

### Try in Copilot/Claude
```text
List all applications
List 5 transactions for application <YourApp>
List available applications datagraphs
List applications insights
```

---

## Toolsets & Tools

> The server organizes functionality into **Toolsets**.

### Available Toolsets

| Toolset        | Description |
|----------------|-------------|
| `Portfolio Tools`    | Portfolio‑level exploration: list applications, compare metrics, scan hotspots across systems. |
| `Applications Tools` | Application‑level analysis: transactions, quality issues, datagraphs, architecture. |
| `Objects Tools`      | Object element exploration: locate objects, references, callers/callees. |

### Tools

<details>
<summary><b>Portfolio</b></summary>

- **applications** — List the available applications in CAST Imaging
- **all_applications** — List all applications (cloud build)
- **my_applications** — List applications belonging to the current user (cloud build)
- **applications_transactions** — Get transactions from all applications with filtering support
- **applications_data_graphs** — Data entity interaction networks (data graphs) from all applications
- **applications_dependencies** — Inter-dependencies between all applications
- **applications_quality_insights** — Quality insights including CVE, cloud detection patterns, and green detection patterns
- **get_mcp_info** — Get MCP server and Imaging version/environment information

</details>

<details>
<summary><b>Applications</b></summary>

- **application_database_explorer** — Explore database tables and columns in an application
- **stats** — Get basic statistical information for an application
- **architectural_graph** — Get architectural graph data (nodes/links) at specific levels (layer, component, sub-component, technology-category, element-type)
- **architectural_graph_focus** — Get focused architectural graph data for specific areas
- **quality_insights** — Get quality-related insights (CVE, cloud patterns, green patterns, structural flaws, ISO-5055)
- **quality_insight_violations** — Get specific violations of quality insights by ID, with optional locations
- **packages** — Get package information and dependencies
- **package_interactions** — Get interactions with specific packages by component and version
- **transaction_profiles** — Get available transaction profiles
- **transactions** — Get transactions with optional filtering by name, fullname, or type
- **transaction_details** — Get focused information about a transaction by ID
- **add_view_document** — Add documentation to a transaction or data graph
- **data_graphs** — Get data entity interaction networks (data graphs) with optional filtering
- **data_graph_profiles** — Get available data graph profiles
- **data_graph_details** — Get focused information about a data graph by ID
- **inter_applications_dependencies** — Get inward and outward inter-application dependencies
- **inter_app_detailed_dependencies** — Get detailed dependencies between two specific applications
- **advisor_occurrences** — Get occurrences supporting a selected advisor
- **application_iso_5055_explorer** — Explore ISO 5055 characteristics and weaknesses
- **api_inventory** — Get API inventory for an application
- **advisors** — Get migration/modernization advisors, rules, and violations
- **dynamic_views** — List custom aggregation views
- **dynamic_view_details** — Get a custom view graph or objects inside a custom node
- **manage_dynamic_view** — Create, delete, rename, publish, or unpublish custom views
- **configure_dynamic_view** — Add or delete custom nodes in a custom view
- **views** — List saved views
- **view_details** — Get saved view details
- **manage_view** — Create, update, or delete saved views
- **tags** — List tags available in an application

</details>

<details>
<summary><b>Objects</b></summary>

- **objects** — Get objects in an application matching identification criteria (name, fullname, mangling, type, filepath)
- **object_profiles** — Get available object profiles
- **object_details** — Get comprehensive details for objects including properties, relationships, code snippets, and usage statistics
- **add_object_document** — Add documentation text to a specific object identified by ID
- **manage_object_tags** — Add or delete tags on explicitly identified objects
- **bulk_manage_object_tags** — Add or delete tags on objects selected by server-side criteria
- **objects_relationships** — Find relationships between multiple objects
- **pathfinder_hierarchy_details** — Find execution paths, call chains, and hierarchy details
- **source_files** — Find source files that define code objects matching file path criteria
- **source_file_details** — Get detailed information about source files and their code elements (inventory, intra, inward, outward, testing)
- **transactions_using_object** — Get transactions that use objects matching specified criteria
- **data_graphs_involving_object** — Get data entity interaction networks (data graphs) that involve specific objects

</details>

<details>
<summary><b>Structural Search (intents profile)</b></summary>

- **get_structural_search_function_syntax** — Get syntax and usage details for structural search functions
- **run_structural_search_function** — Execute a structural search function through the generic dispatcher

</details>

---

## Troubleshooting

| Symptom | Likely Cause | What to do |
|---|---|---|
| `ECONNREFUSED` from client | Server not running | `docker ps`, then `./run.sh --install` to start |
| Auth errors | Wrong/expired Imaging API key | Regenerate key from Imaging profile |
| Empty answers | Imaging‑APIs unreachable | Recheck `HOST_CONTROL_PANEL` / `PORT_CONTROL_PANEL`, health endpoint |
| HTTPS handshake/error | Private/self-signed Control Panel certificate | Set `CONTROL_PANEL_SSL_ENABLED=true` and configure `SSL_CA_BUNDLE` |
| VS Code not prompting for key | Missing `inputs` in `mcp.json` | Add the `inputs` block (see setup) |

### Logs & Checks
```bash
# Server logs
docker logs <mcp-container-id>

# Re‑verify Imaging‑APIs
curl -H "x-api-key: <your-key>"      http://{CONTROL_PANEL_HOST}:8090/imaging/apis/rest/ready
```

---

## Security Notes
- **`IMAGING_CODE` = False by default**: enabling allows MCP clients to access source code → restrict by policy, network, and credentials.
- **`SSL_CA_BUNDLE` for private CAs**: prefer a CA bundle over disabling TLS verification.
- **Principle of least privilege**: limit who can obtain/enter Imaging API keys in MCP hosts.

---

## Package Contents
```
com.castsoftware.imaging.mcpserver.docker.__MCP_VERSION__/
├─ config/app.config
├─ docker-compose.yml
├─ run.sh
├─ .env                      # hidden
├─ README.md                 # original docs
└─ copilot-instructions.md   # optional, copy to .github/
```

---

## Support
- Check container logs and the Imaging‑APIs health endpoint
- Validate `app.config`, `.env`, `mcp.json` paths and values
- Ensure network reachability between client ↔ server ↔ Imaging‑APIs

---

## License
This repository contains documentation only.  
The project’s source code is **proprietary** and is **not** published under an open-source license.  

- The source code is licensed separately under a **commercial license** and is not included in this repository.  
- No rights are granted to the source code through this repository.  
- You may view and share this documentation freely, but it may not be used to infer or imply rights to the proprietary software.  

For commercial licensing inquiries, please contact your local CAST representative (https://www.castsoftware.com/overview).

---
