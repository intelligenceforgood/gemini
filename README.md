# Gemini Code Assist (GCA) Productivity Framework

Welcome to the **GCA Productivity Framework** ("The GCA Blueprint"). This repository serves as the definitive source of truth and global configuration for Gemini Code Assist across all I4G platform projects.

By leveraging standardized prompts, predefined routines, and consolidated architectural guidelines, this framework transforms GCA from a simple autocomplete tool into an integrated, context-aware AI pair programmer that enforces I4G engineering standards by default.

---

## 🎯 What This Framework Provides

- **Global Context (`.gemini/`)**: Centralized architectural constraints, coding conventions, and behavioral rules injected into every GCA chat context.
- **Developer Routines (`prompts/`)**: Markdown-based templates representing standard operating procedures (SOPs) for the entire software development lifecycle (SDLC).
- **VSCode Integration (`snippets/`)**: Quick-access snippets to instantly load complex routines into GCA Chat.
- **Comprehensive Documentation (`docs/`)**: Guides, catalogs, and cookbooks to accelerate developer onboarding and mastery of GCA.

---

## 🚀 Quick Start Guide

### 1. Workspace Integration
To leverage the framework, ensure the `gemini` repository is accessible alongside your target repositories.

Add the `gemini` directory to your VSCode Workspace:
```json
{
  "folders": [
    { "path": "../core" },
    { "path": "../ui" },
    { "path": "../gemini" }
  ]
}
```

### 2. Establish "Anchor" Context
In each of your product repositories (e.g., `core/`, `ui/`, `infra/`), create a `.gemini` folder and symlink the global styleguide to ensure GCA automatically reads our platform standards:

```bash
cd path/to/your/repo
mkdir -p .gemini
ln -s ../gemini/.gemini/styleguide.md .gemini/styleguide.md
```

### 3. Verify Setup
Open the **Gemini Code Assist Chat** in VSCode and try a snippet:
Type `gca-plan` and hit Enter. The prompt should expand to the content of `@prompts/plan-work.md`.

---

## 📂 Repository Structure

```text
gemini/
├── .gemini/
│   ├── config.yaml             # Core GCA behavior and inclusion rules
│   └── styleguide.md           # Unified I4G architectural & coding standards
├── docs/                       # GCA Framework Documentation Suite
│   ├── cookbook.md             # Advanced workflow examples and patterns
│   ├── customization-guide.md  # How to extend prompts and framework behavior
│   ├── onboarding.md           # Complete dev setup and getting started guide
│   └── routine-catalog.md      # Comprehensive list of all available routines
├── prompts/                    # The Routine Library
│   ├── architecture-template.md
│   ├── code-review.md
│   ├── ...                     # Other SDLC prompts
├── snippets/
│   └── gemini.code-snippets    # VSCode snippet expansions for routines
└── README.md                   # The document you are reading
```

---

## 🧠 The Role of `.gemini/` in Workspace Projects

The `.gemini/` directory is the crucial link between GCA and our codebases.

- **Global Enforcement**: The `.gemini/styleguide.md` file contains the extracted knowledge of all former Copilot instructions (Python, TS/React, Terraform, ML Workflows, etc.). By symlinking this file into your target repos, GCA inherently understands the overarching I4G rules without needing manual prompting.
- **Local Overrides**: Repositories can define their own local `.gemini/config.yaml` or `.gemini/context.md` files alongside the symlink. GCA intelligently merges the global platform rules with the specific, local nuances of the repository you are actively working in.

---

## 📖 Framework Documentation Suite

Ready to master the GCA workflow? Dive into the official documentation:

1. **[Onboarding Guide](docs/onboarding.md)**: Step-by-step setup for new developers.
2. **[Routine Catalog](docs/routine-catalog.md)**: Explore every available prompt and its purpose.
3. **[Cookbook](docs/cookbook.md)**: See real-world examples of feature development and bug hunting using GCA.
4. **[Customization Guide](docs/customization-guide.md)**: Learn how to contribute new routines and update the platform styleguide.

---

*For support, issues, or suggestions regarding the GCA Productivity Framework, please open an issue in this repository.*
