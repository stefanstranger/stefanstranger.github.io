---
layout: post
title: Simplifying Azure Verified Module Discovery with Model Context Protocol
categories: [Azure, AI, Infrastructure as Code]
tags: [Azure, AI, Bicep, MCP]
comments: true
---

- [Introduction](#introduction)
- [What is the Azure Verified Modules MCP Server?](#what-is-the-azure-verified-modules-mcp-server)
- [Why Did I Create This MCP Server?](#why-did-i-create-this-mcp-server)
- [How Does the AVM MCP Server Work?](#how-does-the-avm-mcp-server-work)
- [Key Features](#key-features)
- [Comparison with Microsoft Bicep MCP Server](#comparison-with-microsoft-bicep-mcp-server)
  - [When to Use Each Server](#when-to-use-each-server)
- [Usage Scenarios](#usage-scenarios)
  - [Discovering the Right Module](#discovering-the-right-module)
  - [Understanding Module Parameters](#understanding-module-parameters)
  - [Programmatic Integration](#programmatic-integration)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation Options](#installation-options)
  - [Configuration](#configuration)
- [Available Tools](#available-tools)
  - [1. list\_avm\_modules](#1-list_avm_modules)
  - [2. scrape\_avm\_module\_details](#2-scrape_avm_module_details)
- [Usage Examples](#usage-examples)
- [Conclusion](#conclusion)
- [References](#references)

## Introduction

After my previous blog posts about [AI LLM Agents](https://stefanstranger.github.io/2024/09/04/WhatAreAILLMAgents/) and [Agentic AI with CrewAI](https://stefanstranger.github.io/2024/12/07/HarnessingthePowerofAgenticAIwithCrewAIforInvestmentAnalysis/), I continued exploring ways AI agents can help solve real-world challenges. One challenge that consistently came up in my work with Azure infrastructure deployments was discovering and understanding Azure Verified Modules (AVM).

As someone who regularly works with Azure Bicep templates, I found myself constantly searching through documentation, trying to find the right AVM module for my use case, and then diving deep into GitHub repositories to understand the module parameters. While Microsoft provides excellent documentation, the process of discovery and exploration could be more streamlined, especially when working with AI assistants like GitHub Copilot or Claude.

That's why I created the **Azure Verified Modules MCP Server** - a Model Context Protocol (MCP) server that enables AI agents to intelligently search, discover, and retrieve detailed information about Azure Verified Modules. In this blog post, I'll introduce the AVM MCP Server, explain how it works, and show you how you can use it to simplify your Azure infrastructure development workflow.

## What is the Azure Verified Modules MCP Server?

The Azure Verified Modules (AVM) MCP Server is a specialized tool that connects AI agents to the Azure Verified Modules ecosystem through the Model Context Protocol (MCP). It enables AI assistants to:

- **Search for AVM modules** using natural language queries
- **Retrieve module versions** directly from the Microsoft Container Registry
- **Extract detailed documentation** including parameters, resource types, and usage examples
- **Provide intelligent recommendations** based on your infrastructure needs

For those unfamiliar with Azure Verified Modules (AVM), they are a collection of standardized, validated, and well-documented Infrastructure as Code (IaC) modules for deploying Azure resources using Bicep. These modules follow best practices and are maintained by Microsoft and the community, ensuring consistency and reliability across Azure deployments.

The Model Context Protocol (MCP) is an open protocol that standardizes how AI applications provide context to Large Language Models (LLMs). Think of it as a universal connector that allows AI assistants like Claude Desktop, GitHub Copilot, or custom AI agents to access external data sources and tools in a standardized way.

## Why Did I Create This MCP Server?

Working with Azure Verified Modules presents several challenges:

1. **Discovery Challenge**: With hundreds of AVM modules available in the Bicep Public Registry, finding the right module for your specific use case requires searching through extensive documentation. You might need a storage account module, but should you use `avm/res/storage/storage-account` or are there other alternatives?

2. **Parameter Complexity**: Each AVM module has numerous parameters with specific requirements, defaults, and constraints. Understanding which parameters are required, what types they accept, and how they interact with each other often requires reading through lengthy README files on GitHub.

3. **Version Management**: Keeping track of module versions and understanding what's available across the registry can be time-consuming, especially when you want to ensure you're using the latest stable version.

4. **Documentation Access**: While AVM documentation is comprehensive, it's scattered across GitHub repositories. Finding usage examples, understanding resource types, and exploring parameter details requires navigating multiple pages and repositories.

The AVM MCP Server solves these challenges by:

- Providing **fast, intelligent search** that understands variations like "key vault", "key-vault", and "keyvault"
- **Retrieving module versions** directly from the Microsoft Container Registry in real-time
- **Extracting and formatting documentation** so AI agents can understand and explain module capabilities
- **Enabling natural language interaction** - simply ask "What modules are available for storage?" and get instant results

## How Does the AVM MCP Server Work?

The AVM MCP Server is built using Python and leverages the Model Context Protocol SDK. It acts as a bridge between AI assistants and two primary data sources:

1. **Microsoft Container Registry (MCR)**: The server queries MCR's catalog and tags endpoints to discover available AVM modules and their versions. All AVM modules are published to MCR with the prefix `bicep/avm/`, making them easy to identify and filter.

2. **GitHub Repository**: The server fetches README.md files from the [bicep-registry-modules repository](https://github.com/Azure/bicep-registry-modules) to extract detailed documentation, including parameter tables, resource types, and usage examples.

Here's the workflow:

```
User Query → AI Assistant → MCP Server → MCR/GitHub → Structured Response → AI Assistant → User
```

For example, when you ask "Find storage account modules", the server:

1. Queries MCR's catalog endpoint (`mcr.microsoft.com/v2/_catalog`)
2. Filters for repositories containing "storage" and "account"
3. Retrieves available versions for matching modules
4. Returns structured JSON with module names, descriptions, versions, and documentation links

When you request module details, the server:

1. Converts GitHub tree URLs to raw content URLs
2. Fetches the README.md content
3. Extracts relevant sections using regex patterns (resource types, parameters, examples)
4. Returns formatted markdown that AI assistants can easily parse and explain

## Key Features

The AVM MCP Server provides several powerful features that make working with Azure Verified Modules more efficient:

- **Intelligent Search**: Supports multiple query formats and variations. Whether you search for "key vault", "key-vault", or "keyvault", the server understands your intent and returns relevant results.

- **Direct Registry Access**: Connects directly to Microsoft Container Registry for real-time module information, ensuring you always get the latest available versions.

- **Deep Documentation Extraction**: Goes beyond basic metadata to extract complete parameter references, resource types, and usage examples with large parameter sets.

- **Fast Filtering**: Optimized search that quickly narrows down results from thousands of repositories, providing instant feedback.

- **Structured Responses**: Returns data in well-structured JSON format that AI assistants can easily parse and present to users in a helpful way.

## Comparison with Microsoft Bicep MCP Server

Microsoft provides an [official Bicep MCP Server](https://github.com/Azure/bicep) that includes a `ListAvmMetadata` tool. You might wonder why I created a separate server when Microsoft already provides one. Here's a detailed comparison:

| Feature | Microsoft Bicep MCP Server | AVM MCP Server |
|---------|---------------------------|----------------|
| **Primary Focus** | Bicep language tools & Azure resource schemas | AVM module discovery & documentation |
| **AVM Module Search** | Lists all modules (no filtering) | Intelligent search with multiple query formats |
| **Module Details** | Basic metadata (name, description, versions) | Deep documentation extraction (parameters, resource types, examples) |
| **Installation** | Requires .NET runtime & Bicep CLI | Lightweight Python with minimal dependencies |
| **Response Format** | Newline-separated text summary | Structured JSON with rich metadata |
| **Documentation Access** | External links only | Extracted and formatted markdown from README files |

### When to Use Each Server

**Use the Microsoft Bicep MCP Server when:**
- Writing Bicep code and need authoring best practices
- Checking Azure resource schemas and API versions
- Need comprehensive Bicep ecosystem tools

**Use the AVM MCP Server when:**
- Finding AVM modules for specific Azure services
- Understanding module parameters before using them
- Extracting usage examples and documentation
- Need quick filtered search across the AVM catalog

**Use both together** for a complete Bicep + AVM development experience! They complement each other perfectly - use the Bicep MCP Server for template authoring and the AVM MCP Server for module discovery and documentation.

## Usage Scenarios

Let me walk you through some practical scenarios where the AVM MCP Server can significantly improve your workflow.

### Discovering the Right Module

Imagine you're starting a new Azure project that requires a Key Vault. Instead of manually browsing through the Bicep Public Registry or searching GitHub, you can simply ask your AI assistant:

> "Find all AVM modules for Key Vault"

The AVM MCP Server searches the registry and returns all relevant modules with their versions and documentation links. Your AI assistant can then present these options and help you choose the right one based on your requirements.

### Understanding Module Parameters

Once you've found the right module, you need to understand its parameters. Instead of navigating to GitHub and scrolling through lengthy README files, you can ask:

> "Show me the parameters for the storage account AVM module"

The AVM MCP Server extracts the parameter documentation, including:
- Required vs. optional parameters
- Parameter types and descriptions
- Default values
- Usage examples with large parameter sets

Your AI assistant can then explain these parameters in plain language, highlight the required ones, and even help you construct a proper Bicep module reference.

### Programmatic Integration

For DevOps teams working with CI/CD pipelines, the AVM MCP Server can be integrated into automated workflows. AI agents can:

- **Validate module usage** by checking if the specified module exists and is at the latest version
- **Generate Bicep code** with proper parameter references based on module documentation
- **Provide recommendations** for module upgrades or alternative modules that might better fit your use case

This is similar to the [Azure Subnet Copilot](https://stefanstranger.github.io/2024/03/23/AzureNetworkSubnettingMadeEasy/) approach I shared in a previous blog post, where automation simplifies complex infrastructure decisions.

## Getting Started

Let me show you how to set up and start using the AVM MCP Server.

### Prerequisites

Before you begin, make sure you have the following installed:

- **Python 3.11 or higher** - [Download from python.org](https://www.python.org/downloads/)
- **UV package manager** - [Installation guide](https://docs.astral.sh/uv/getting-started/installation/)
- **Internet connectivity** - Required to access Microsoft Container Registry and GitHub
- **Claude Desktop or similar MCP client** (optional) - [Download Claude Desktop](https://claude.ai/download)

For Windows users, install UV using PowerShell:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

For macOS/Linux users:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Installation Options

The AVM MCP Server offers two installation approaches:

**Option 1: Direct from GitHub (Recommended)**

This approach requires no local installation. Simply run:

```powershell
uvx --from git+https://github.com/stefanstranger/avm-mcp-server avm-mcp-server
```

This method:
- ✅ Requires no local installation or cloning
- ✅ Always uses the latest version from the main branch
- ✅ Works across different machines with the same command
- ✅ No need to manage virtual environments

**Option 2: Local Installation**

If you prefer to run from a local clone:

```powershell
# Clone the repository
git clone https://github.com/stefanstranger/avm-mcp-server
cd avm-mcp-server

# Create and activate virtual environment
uv venv
.venv\Scripts\Activate.ps1

# Install dependencies
uv pip install -e .
```

Then run the server:

```powershell
uv run .\server.py
```

### Configuration

To use the AVM MCP Server with Claude Desktop, add the following configuration to your `claude_desktop_config.json` file:

**Using uvx (recommended):**

```json
{
  "mcpServers": {
    "avm-mcp-server": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/stefanstranger/avm-mcp-server",
        "avm-mcp-server"
      ]
    }
  }
}
```

**Using local installation:**

```json
{
  "mcpServers": {
    "avm-mcp-server": {
      "command": "uv",
      "args": [
        "--directory",
        "c:/github/avm-mcp-server",
        "run",
        "server.py"
      ]
    }
  }
}
```

After adding the configuration, restart Claude Desktop and you'll see the AVM MCP Server tools available in your conversations.

## Available Tools

The AVM MCP Server provides two powerful tools that AI assistants can use:

### 1. list_avm_modules

This tool searches and lists Azure Verified Modules from the Bicep Public Registry.

**Parameters:**
- `modulename` (optional): Module name to filter by

**Supported query formats:**
- Exact match: `"storage-account"`
- With spaces: `"storage account"`
- Partial match: `"storage"`
- Compact: `"keyvault"`

**Returns:**
A JSON array with module information including:
- Module name (registry path)
- Available versions
- Description
- Documentation link

**Example queries:**
- "List all AVM modules for storage accounts"
- "Find Azure Verified Modules for key vault"
- "Show me AVM modules related to networking"

### 2. scrape_avm_module_details

This tool fetches detailed information from an AVM module's README documentation.

**Parameters:**
- `url` (required): GitHub URL of the AVM module repository
  - Example: `https://github.com/Azure/bicep-registry-modules/tree/main/avm/res/storage/storage-account`

**Returns:**
Formatted markdown containing:
- **Resource Types**: Azure resources deployed by the module
- **Parameters**: Complete parameter reference with types, defaults, and descriptions
- **Usage Examples**: Large parameter set examples showing real-world usage

**Example queries:**
- "Get the details for the storage account AVM module"
- "Show me the parameters for the key vault module"
- "What resources does the virtual network module deploy?"

## Usage Examples

Here are some practical examples of how you can interact with the AVM MCP Server through an AI assistant:

**Search for modules:**
> "Find all AVM modules for storage"
> 
> "List Azure Verified Modules for Key Vault"
> 
> "Show me networking modules"

**Get module versions:**
> "What versions are available for the storage account module?"
> 
> "List all versions of the AVM key vault module"

**Explore module details:**
> "Show me the parameters for bicep/avm/res/storage/storage-account"
> 
> "What resources does the virtual network module deploy?"
> 
> "Get usage examples for the key vault module"

**Combined workflows:**
> "Find the storage account AVM module and show me its parameters"
> 
> "I need to deploy a key vault - find the module and explain its parameters"
> 
> "Search for virtual network modules and show me usage examples"

The AI assistant will use the AVM MCP Server tools to fetch the information and present it in a clear, conversational format.

## Conclusion

The Azure Verified Modules MCP Server represents a significant step forward in simplifying Azure infrastructure development. By enabling AI assistants to intelligently discover and understand AVM modules, we can reduce the time spent searching documentation and increase the time spent building robust Azure solutions.

Whether you're a DevOps engineer looking to streamline your Bicep development workflow, a platform engineer building infrastructure standards, or a developer new to Azure infrastructure, the AVM MCP Server can help you find and understand the right modules faster.

The server is lightweight, easy to install, and integrates seamlessly with AI assistants like Claude Desktop. It complements Microsoft's official Bicep MCP Server by focusing specifically on module discovery and documentation extraction, providing a complete toolkit for Bicep and AVM development.

I encourage you to give it a try and see how it can enhance your Azure development experience! All the code is open source and available on GitHub. If you encounter any issues, have suggestions for improvements, or want to contribute, please visit the [GitHub repository](https://github.com/stefanstranger/avm-mcp-server) and submit an issue or pull request. Your feedback is greatly appreciated!

## References

1. [AVM MCP Server GitHub Repository](https://github.com/stefanstranger/avm-mcp-server)
2. [Azure Verified Modules](https://azure.github.io/Azure-Verified-Modules/)
3. [Bicep Registry Modules](https://github.com/Azure/bicep-registry-modules)
4. [Model Context Protocol](https://modelcontextprotocol.io/)
5. [Microsoft Bicep MCP Server](https://github.com/Azure/bicep)
6. [What are AI LLM Agents?](https://stefanstranger.github.io/2024/09/04/WhatAreAILLMAgents/)
7. [Harnessing the Power of Agentic AI with CrewAI](https://stefanstranger.github.io/2024/12/07/HarnessingthePowerofAgenticAIwithCrewAIforInvestmentAnalysis/)
8. [Azure Subnet Copilot](https://stefanstranger.github.io/2024/03/23/AzureNetworkSubnettingMadeEasy/)
9. [Claude Desktop](https://claude.ai/download)
10. [UV Package Manager](https://docs.astral.sh/uv/)
