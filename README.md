<h1 align="center">
  <br>
  MITRE ATT&CK MCP Server
  <br>
</h1>

<h4 align="center">A Model-Context Protocol server for the MITRE ATT&CK knowledge base</h4>

<p align="center">
  <a href="#key-features">Key Features</a> •
  <a href="#installation">Installation</a> •
  <a href="#how-to-use">How To Use</a> •
  <a href="#use-cases">Use Cases</a> •
  <a href="#credits">Credits</a>
</p>

## Overview

The MITRE ATT&CK MCP Server acts as an intelligent bridge between a Large Language Model (LLM), such as Claude, and the comprehensive MITRE ATT&CK® knowledge base. Its primary purpose is to enable an LLM to programmatically query, retrieve, and work with structured information about cybersecurity threats. This includes detailed data on tactics, techniques, threat actor groups, malware, campaigns, and mitigations.

This server implements the Model-Context Protocol (MCP), which allows the LLM to invoke specific "tools" (functions) on this server. These tools provide precise data from the ATT&CK dataset, empowering the LLM with accurate and up-to-date cybersecurity information.

## Installation

To clone and run this server, you'll need [Git](https://git-scm.com) and [Python](https://www.python.org/) installed on your computer. From your command line:

```bash
# Clone this repository
$ git clone https://github.com/stoyky/mitre-attack-mcp

# Go into the repository
$ cd mitre-attack-mcp

# Install dependencies
$ pip install -r requirements.txt

# OR install the most important dependencies manually

$ pip install mcp
$ pip install mitreattack-python

```

## How To Use

### Launching the Server & Initial Data Setup

The server is launched by running the Python script from the command line:

```bash
python mitre-attack-mcp-server.py PATH/TO/YOUR/mitre-attack-data-directory
```

Replace `PATH/TO/YOUR/mitre-attack-data-directory` with the actual path to a directory where you want the server to store and manage the MITRE ATT&CK STIX data.

When launched, the script will:
1.  **Initialize for MCP Communication:** Prepare to listen for and respond to Model-Context Protocol (MCP) requests from an LLM environment (like Claude Desktop).
2.  **Ensure Data Availability:** Check the specified data directory. If the latest MITRE ATT&CK STIX data (for Enterprise, Mobile, and ICS domains) isn't present in a versioned subdirectory (e.g., `v17.0/`), it will automatically download and organize it. This ensures the server operates with the most current threat intelligence.

Details about how the server manages data versions, downloads, and loading are covered in the <a href="#data-management">Data Management</a> section.

### MCP Server Operation

As an MCP server, `mitre-attack-mcp-server.py` acts as a backend service for Large Language Models. When configured with an LLM environment (typically via stdio, as with Claude Desktop), it listens for incoming MCP requests. These requests are essentially instructions from the LLM to execute specific "tools."

The "tools" are Python functions within the `mitre-attack-mcp-server.py` script. Each tool is designed to perform a specific query or data retrieval task against the loaded MITRE ATT&CK knowledge base. When the LLM needs ATT&CK information to answer a user's query, it sends an MCP request specifying the tool name and any necessary parameters. The server executes the function and returns the structured result to the LLM.

### Claude Desktop Configuration

To use this server with Claude AI Desktop, add the following to your `claude_desktop_config.json` file. This file is typically found at:

```
C:\Users\[YourUsername]\AppData\Roaming\Claude\claude_desktop_config.json
# or
C:\Users\[YourUsername]\AppData\Local\AnthropicClaude\claude_desktop_config.json
```

```json
{
  "mcpServers": {
    "mitre-attack": {
      "command": "python",
      "args": [
        "PATH/TO/mitre-attack-mcp-server.py",
        "PATH/TO/YOUR/mitre-attack-data-directory"
      ]
    }
  }
}
```

> **Note:** The `PATH/TO/YOUR/mitre-attack-data-directory` in the `args` should be the same base directory path provided when launching the server from the command line. The server manages versioned subdirectories for STIX data within this path.

### Example LLM Workflow

Here’s a conceptual example of how an LLM, this MCP server, and a user might interact:

1.  **User Query:** The user asks the LLM a question like, "What techniques are commonly associated with the threat actor FIN7?"
2.  **LLM Tool Selection:** The LLM, understanding the user's intent, identifies that it needs to query the MITRE ATT&CK knowledge base. It consults the list of available tools provided by this MCP server (which are declared by the server on startup via MCP) and selects an appropriate tool, for example, `get_techniques_used_by_group`.
3.  **MCP Request:** The LLM constructs and sends an MCP request to `mitre-attack-mcp-server.py`. This request would specify the tool `get_techniques_used_by_group` and the parameter `group_name="FIN7"` (or its ATT&CK ID).
4.  **Server Processing & Response:** The `mitre-attack-mcp-server.py` receives the request. It executes its internal `get_techniques_used_by_group` function, which queries the local STIX data for techniques related to FIN7. The server then sends the findings (e.g., a list of technique IDs, names, and descriptions) back to the LLM as a structured MCP response.
5.  **LLM Answer Formulation:** The LLM receives the structured data from the server. It uses this information to formulate a comprehensive, human-readable answer and presents it to the user. For instance, it might list the techniques and perhaps offer to provide more details on any specific one.

This workflow allows the LLM to leverage the vast and detailed MITRE ATT&CK knowledge base to answer complex cybersecurity questions accurately.

## Data Management

This section details how the MITRE ATT&CK MCP Server handles the cybersecurity knowledge base it relies upon.

*   **Data Source:** The server uses the official MITRE ATT&CK® knowledge base, which is provided in the STIX™ 2.1 (Structured Threat Information eXpression) format. This ensures that the server operates with standardized and community-recognized threat information.

*   **Download & Versioning Process:** When the server is launched with the `<data_path>` argument (as shown in <a href="#launching-the-server-initial-data-setup">Launching the Server & Initial Data Setup</a>), it checks this path for the required STIX data.
    *   Specifically, it looks for `enterprise-attack.json`, `mobile-attack.json`, and `ics-attack.json` files within a version-specific subdirectory (e.g., `<data_path>/v17.0/`).
    *   If these files are missing, or if MITRE has released a newer version of the ATT&CK dataset, the server automatically proceeds to download the latest set.

*   **Acquiring the Latest Version:** The server uses the `release_info` module from the `mitreattack-python` library to determine the most recent ATT&CK dataset version available from MITRE. The `download_stix_data` function in `mitre-attack-mcp-server.py` then fetches this latest version.

*   **Local Storage Structure:** The downloaded STIX 2.1 JSON files are stored in the versioned subdirectory created by the server under your specified `<data_path>` (e.g., `PATH/TO/YOUR/mitre-attack-data-directory/v17.0/`). This method organizes different ATT&CK data versions and prevents redundant downloads if the latest data is already on disk.

*   **Loading Data into Memory:** Once the correct STIX data is available locally (either pre-existing or newly downloaded), the `load_stix_data` function within the server script loads it into memory. This makes the ATT&CK knowledge base accessible for the server's query tools upon startup.

This automated data management ensures the server utilizes current MITRE ATT&CK intelligence without needing manual data acquisition or version control by the user.

## Key Features

The MITRE ATT&CK MCP Server offers a comprehensive suite of over 50 tools, enabling detailed querying and analysis of the MITRE ATT&CK knowledge base. These tools can be broadly categorized as follows:

*   **Object Lookup & Retrieval:**
    *   **Functionality:** Fetch specific ATT&CK objects (techniques, tactics, threat actor groups, software/malware, mitigations, campaigns, data sources, data components, and assets) by their ID, name, or by searching keywords within their descriptions.
    *   **Example questions this category can answer:** "Get details for technique T1548.", "Find software named 'Cobalt Strike'.", "What groups mention 'phishing' in their description?", "What is data source DS0017?", "Describe asset I0001."

*   **Relationship & Association Queries:**
    *   **Functionality:** Discover and explore the connections between different ATT&CK objects. This includes identifying techniques used by threat actors, software used in campaigns, groups associated with certain malware, and mitigations for specific techniques. This category directly supports capabilities like "Threat Actor and Malware Attribution."
    *   **Example questions this category can answer:** "What techniques are used by Group G0022 (APT28)?", "What software is associated with Campaign C0001?", "Which groups use software S0154 (Cobalt Strike)?", "What mitigations address technique T1059 (Command and Scripting Interpreter)?", "What data components detect technique T1055 (Process Injection)?"

*   **Technique Analysis & Comparison:**
    *   **Functionality:** Analyze and compare sets of techniques, such as identifying overlaps between those used by different threat actors or malware families. This directly enables "Technique Overlap Analysis."
    *   **Example questions this category can answer:** "What techniques are common to both APT28 and FIN7?", "Show techniques used by malware A and malware B, highlighting the differences."

*   **Bulk Data Queries:**
    *   **Functionality:** Retrieve all objects of a specific type from the ATT&CK matrices (Enterprise, Mobile, ICS).
    *   **Example questions this category can answer:** "List all techniques in the Enterprise ATT&CK matrix.", "Get all known threat actor groups.", "Fetch all mitigations for ICS environments."

*   **Tactical & Platform-Based Queries:**
    *   **Functionality:** Query data based on ATT&CK tactics (e.g., Initial Access, Execution, Exfiltration) or specific platforms (e.g., Windows, Linux, macOS, Azure AD, Office 365).
    *   **Example questions this category can answer:** "What techniques belong to the 'Execution' tactic?", "Show me all Linux techniques for Privilege Escalation.", "List all data sources relevant to the Mobile platform."

*   **ATT&CK Navigator Layer Generation:**
    *   **Functionality:** Automatically create and format data for visualization in the MITRE ATT&CK Navigator. This includes generating layers based on techniques associated with threat actors, software, mitigations, or data components.
    *   **Details:**
        *   `generate_layer_for_group`, `generate_layer_for_software`, `generate_layer_for_mitigation`, `generate_layer_for_datasource_or_component`: These tools create the core data for a layer file, highlighting relevant techniques and providing scores or comments.
        *   `get_navigator_layer_metadata`: This tool provides the necessary boilerplate JSON structure for a valid ATT&CK Navigator layer file, into which the generated technique data can be inserted.
    *   **Example use case:** "Generate an ATT&CK Navigator layer showing all techniques used by the group APT29, color-coding them by tactic."

## Use Cases

* Query for detailed information about specific malware, tactics, or techniques
* Discover relationships between threat actors and their tools
* Generate visual ATT&CK Navigator layers for threat analysis
* Find campaign overlaps between different threat actors
* Identify common techniques used by multiple malware families

Please see the author's [blog post](https://www.remyjaspers.com/blog/mitre_attack_mcp_server/) for more information and examples. 

## Credits

This server leverages several key resources:

- **MITRE ATT&CK®:** The foundational knowledge base of adversary tactics and techniques. ([attack.mitre.org](https://attack.mitre.org/))
- **MITRE ATT&CK Python Library:** Used for interacting with the ATT&CK knowledge base. ([github.com/mitre-attack/mitreattack-python](https://github.com/mitre-attack/mitreattack-python))
- **ATT&CK Navigator:** The visualization tool for which this server can generate layers. ([github.com/mitre-attack/attack-navigator](https://github.com/mitre-attack/attack-navigator))
- **Anthropic:** Developers of the Model-Context Protocol (MCP) that this server implements. ([www.anthropic.com](https://www.anthropic.com/))

## Changelog

- v1.0.0 - Initial release
- V1.0.1 - Improved robustness of layer metadata generation and error handling in layer generation function

---

> Created by [Remy Jaspers](https://github.com/stoyky)
