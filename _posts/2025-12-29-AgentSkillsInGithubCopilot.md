---
layout: post
title: How to use Agent Skills in Github Copilot
categories: [AI, Coding, Github Copilot]
tags: [AI, Coding, Github Copilot]
comments: true
---

- [Introduction](#introduction)
- [Why use Skills?](#why-use-skills)
- [Github Copilot and Agent Skills](#github-copilot-and-agent-skills)
- [Getting Started with Agent Skills](#getting-started-with-agent-skills)
  - [1. Create a Skills Directory](#1-create-a-skills-directory)
  - [2. Create a Subdirectory for Each Skill](#2-create-a-subdirectory-for-each-skill)
  - [3. Create a SKILL.md File](#3-create-a-skillmd-file)
  - [4. Add Resources (Optional)](#4-add-resources-optional)
- [Example SKILL.md File](#example-skillmd-file)
- [Helper Scripts Reference](#helper-scripts-reference)
  - [scripts/check\_uv.py](#scriptscheck_uvpy)
  - [scripts/setup\_project.ps1](#scriptssetup_projectps1)
  - [scripts/create\_pyproject.py](#scriptscreate_pyprojectpy)
- [Why This Matters](#why-this-matters)
- [How Copilot Uses Skills](#how-copilot-uses-skills)
- [Using This Skill in Your Repository](#using-this-skill-in-your-repository)
  - [1. Add the Skill to Your Repository](#1-add-the-skill-to-your-repository)
  - [2. Example Prompts That Will Trigger This Skill](#2-example-prompts-that-will-trigger-this-skill)
  - [3. How Copilot Decides to Use This Skill](#3-how-copilot-decides-to-use-this-skill)
  - [4. What Happens When the Skill is Triggered](#4-what-happens-when-the-skill-is-triggered)
- [Availability](#availability)
- [Skills vs. Custom Instructions](#skills-vs-custom-instructions)
- [Finding Skills](#finding-skills)
- [Reference](#reference)

## Introduction

In October 2025, Anthropic announced Agent Skills. As Agents become more powerfull, the wanted a more composable, scalable, and portable way to equip these agent with domain specific expertise.

This led them to create [Agent Skills](https://www.anthropic.com/news/skills): organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks.

Building a skill for an agent is like putting together an onboarding guide for a new hire. Instead of building fragmented, custom-designed agents for each use case, anyone can now specialize their agents with composable capabilities by capturing and sharing their procedural knowledge.

## Why use Skills?

Skills are folders of instructions, scripts, and resources that agents can discover and use to perform better at specific tasks. Write once, use everywhere.

## Github Copilot and Agent Skills

They work across Copilot coding agent, Copilot CLI, and agent mode in [Visual Studio Code Insiders](https://code.visualstudio.com/insiders/). Skills support is coming to the stable version of VS Code in early January 2026.

When Copilot determines a skill is relevant to your task, it loads the instructions and follows them—including any resources you’ve included in the skill directory.

You can write your own skills, or use skills shared by others, such as those in the [anthropics/skills](https://github.com/anthropics/skills) repository or GitHub’s community created [github/awesome-copilot](https://github.com/github/awesome-copilot) collection.

## Getting Started with Agent Skills

To add skills to your repository and start using them with GitHub Copilot, follow these steps:

### 1. Create a Skills Directory

First, create a `.github/skills` directory in your repository to store your skills.

**Note:** Skills stored in the `.claude/skills` directory are also supported.

### 2. Create a Subdirectory for Each Skill

Each skill should have its own directory within `.github/skills`. For example:
- `.github/skills/webapp-testing`
- `.github/skills/github-actions-failure-debugging`

Skill directory names should be:
- Lowercase
- Use hyphens for spaces
- Typically match the `name` in the `SKILL.md` frontmatter

### 3. Create a SKILL.md File

Each skill directory must contain a `SKILL.md` file (the name must be exactly `SKILL.md`).

The `SKILL.md` file is a Markdown file with YAML frontmatter that includes:

**Required fields:**
- `name`: A unique identifier for the skill (lowercase, using hyphens for spaces)
- `description`: A description of what the skill does and when Copilot should use it

**Optional field:**
- `license`: A description of the license that applies to this skill

The Markdown body should contain the instructions, examples, and guidelines for Copilot to follow.

### 4. Add Resources (Optional)

You can optionally add scripts, examples, or other resources to your skill's directory. For example, if you're creating a skill for image conversion, you might include a script for converting SVG images to PNG.

## Example SKILL.md File

Here's a practical example of a skill for setting up Python projects on Windows 11 using `uv`. This skill includes helper scripts and ensures every Python project follows the same best practices:

**Skill Directory Structure:**
```
.github/skills/python-project-setup/
├── SKILL.md
└── scripts/
    ├── setup_project.ps1
    ├── check_uv.py
    └── create_pyproject.py
```

**Complete example available at:** [stefanstranger/agentinstructions/python-project-setup](https://github.com/stefanstranger/agentinstructions/tree/main/skills/python-project-setup)

**Location:** `.github/skills/python-project-setup/SKILL.md`

````markdown
---
name: python-project-setup
description: Sets up a new Python project with uv, virtual environment, and best practices for Windows 11. Use this when creating a new Python project or helping users set up their Python development environment with modern tooling.
---

## Python Project Setup with uv for Windows 11

When setting up a new Python project, use `uv` for fast, reliable package management. This skill includes helper scripts in the `scripts/` directory.

### 1. Check and Install uv
Use the provided script to check if uv is installed:
```powershell
# Check if uv is installed, install if missing
python scripts/check_uv.py
```

Or manually:
```powershell
# Install uv using pip
pip install uv
```

### 2. Quick Setup with Script
For a complete automated setup:
```powershell
# Run the setup script with project name
.\scripts\setup_project.ps1 -ProjectName "my-new-project"
```

### 3. Manual Setup Steps

#### Create Virtual Environment with uv
```powershell
# uv creates and manages virtual environments automatically
uv venv
```

#### Activate Virtual Environment
```powershell
# PowerShell activation
.\.venv\Scripts\Activate.ps1
```

If you get an execution policy error, run:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### Create pyproject.toml
Use the template script or create manually:
```powershell
# Generate pyproject.toml from template
python scripts/create_pyproject.py --name "your-project-name" --python-version "3.10"
```

Or manually create with this structure:
```toml
[project]
name = "your-project-name"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
    "ruff>=0.1.0",
]
```

#### Install Dependencies with uv
```powershell
# Install project dependencies (much faster than pip)
uv pip install -e ".[dev]"
```

### 4. Project Structure
The scripts will create this structure:
```
project-name/
├── .venv/          # Virtual environment (don't commit)
├── src/            # Source code
│   └── __init__.py
├── tests/          # Test files
│   └── __init__.py
├── .gitignore      # Includes .venv, __pycache__, *.pyc
├── README.md       # Project documentation
└── pyproject.toml  # Modern Python project file
```

### 5. .gitignore Configuration
Always exclude:
```
.venv/
__pycache__/
*.pyc
*.pyo
.pytest_cache/
.ruff_cache/
dist/
build/
*.egg-info/
`````

## Helper Scripts Reference

The complete example skill is available in the [python-project-setup](https://github.com/stefanstranger/agentinstructions/tree/main/skills/python-project-setup) repository.

### scripts/check_uv.py
Checks if uv is installed and installs it if missing.

[View check_uv.py](https://github.com/stefanstranger/agentinstructions/blob/main/skills/python-project-setup/scripts/check_uv.py)

### scripts/setup_project.ps1
Complete project setup automation:
- Creates project directory structure
- Initializes virtual environment with uv
- Generates pyproject.toml
- Creates .gitignore
- Sets up basic README.md

[View setup_project.ps1](https://github.com/stefanstranger/agentinstructions/blob/main/skills/python-project-setup/scripts/setup_project.ps1)

### scripts/create_pyproject.py
Generates a pyproject.toml file with common configurations.

[View create_pyproject.py](https://github.com/stefanstranger/agentinstructions/blob/main/skills/python-project-setup/scripts/create_pyproject.py)

## Why This Matters
- `uv` is 10-100x faster than pip for package installation
- Uses modern `pyproject.toml` instead of legacy `requirements.txt`
- Helper scripts automate repetitive setup tasks
- Ensures consistent setup across all team members
- Prevents common Windows-specific Python issues
- Follows best practices automatically


**Why This Skill is Valuable:**

Instead of manually explaining these steps every time someone asks "How do I set up a Python project?", Copilot will automatically follow this skill and can even use the helper scripts. This means:
- **Speed**: `uv` installs packages 10-100x faster than pip
- **Automation**: Helper scripts automate the entire project setup process
- **Modern Tooling**: Uses `pyproject.toml` (the new Python standard) instead of `requirements.txt`
- **Consistency**: Every Python project gets set up the same way using the same scripts
- **Time Saving**: No need to remember or look up the PowerShell activation syntax
- **Best Practices**: Automatically includes testing, linting, and proper .gitignore
- **Windows-Specific**: Handles PowerShell execution policies and path separators correctly
- **Reusable Scripts**: The helper scripts can be referenced and executed by Copilot when relevant

## How Copilot Uses Skills

When performing tasks, Copilot will automatically decide when to use your skills based on:
- Your prompt
- The skill's description

When Copilot chooses to use a skill:
1. The `SKILL.md` file is injected into the agent's context
2. The agent gains access to your instructions
3. It can follow those instructions and use any scripts or examples included in the skill's directory

## Using This Skill in Your Repository

To use the Python project setup skill in your repository:

### 1. Add the Skill to Your Repository

Create the directory structure in your repository:
```
your-repo/
└── .github/
    └── skills/
        └── python-project-setup/
            ├── SKILL.md
            └── scripts/
                ├── check_uv.py
                ├── setup_project.ps1
                └── create_pyproject.py
```

Copy the files from the [example repository](https://github.com/stefanstranger/agentinstructions/tree/main/skills/python-project-setup) into your `.github/skills/python-project-setup/` directory.

### 2. Example Prompts That Will Trigger This Skill

Once the skill is added to your repository, Copilot will automatically use it when you ask questions like:

**Direct Setup Requests:**

- "Set up a new Python project"
- "Create a Python project with uv"
- "Initialize a Python development environment"
- "Set up Python project structure"

**Configuration Questions:**

- "How do I set up a Python project in this repo?"
- "What's the best way to structure a Python project?"
- "Help me configure a new Python application"

**Tool-Specific Requests:**

- "Set up uv for Python development"
- "Create a pyproject.toml file"
- "Configure Python virtual environment"

**Troubleshooting:**

- "I need to set up Python dependencies"
- "How do I activate a virtual environment in PowerShell?"

### 3. How Copilot Decides to Use This Skill

Copilot evaluates the skill's description:
> "Sets up a new Python project with uv, virtual environment, and best practices for Windows 11. Use this when creating a new Python project or helping users set up their Python development environment with modern tooling."

When your prompt matches these keywords or concepts:

- Python project
- Setup/create/initialize
- Virtual environment
- Development environment
- uv (the package manager)
- Windows 11

Copilot will load the skill and follow the instructions in `SKILL.md`, including referencing the helper scripts.

### 4. What Happens When the Skill is Triggered

1. **Copilot reads the SKILL.md file** and understands the step-by-step process
2. **Accesses the helper scripts** in the scripts/ directory
3. **Provides contextual guidance** based on your specific situation
4. **Can execute or suggest running the scripts** like `setup_project.ps1` or `check_uv.py`
5. **Follows the exact patterns and best practices** defined in the skill

This means you get **consistent, repeatable results** every time someone in your team asks about Python project setup.

## Availability

Agent Skills work with:
- **Copilot coding agent** - Available in VS Code, Visual Studio, and GitHub.com
- **GitHub Copilot CLI** - Available with Copilot Pro, Pro+, Business, and Enterprise plans
- **Agent mode in Visual Studio Code Insiders** - Currently available (stable VS Code support coming soon)

**Note:** Currently, skills can only be created at the repository level. Support for organization-level and enterprise-level skills is coming soon.

## Skills vs. Custom Instructions

You can use both skills and custom instructions to teach Copilot how to work in your repository:

- **Custom Instructions**: Best for simple instructions relevant to almost every task (e.g., coding standards)
- **Skills**: Best for more detailed, specialized instructions that Copilot should access when relevant to specific tasks

## Finding Skills

You can write your own skills or use skills shared by others:
- [anthropics/skills](https://github.com/anthropics/skills) - Anthropic's official skills repository
- [github/awesome-copilot](https://github.com/github/awesome-copilot) - GitHub's community-created collection

## Reference

- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Claude Docs - Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Anthropic - Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)
- [Github - Agent Skills](https://github.com/agentskills/agentskills)
- [Github Copilot now supports Agent Skills - Blog](https://github.blog/changelog/2025-12-18-github-copilot-now-supports-agent-skills/)