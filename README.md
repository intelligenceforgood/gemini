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

To leverage the framework, simply open the unified parent directory containing all your I4G repositories as your single workspace root (e.g., the folder containing `core`, `ui`, `gemini`, etc.).

In VS Code, go to **File > Open Folder...** and select this parent directory. You do not need to create a complex multi-root workspace; opening the parent directory is sufficient.

### 2. Establish "Anchor" Styles

Because VS Code treats the parent directory as the single workspace root, you only need to symlink the global `.gemini` configuration once at the parent root, rather than inside every single repository:

```bash
cd /path/to/your/parent/i4g/directory
rm -rf .gemini
ln -s gemini/.gemini .gemini
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
│   └── styles/                 # Unified I4G architectural & coding standards
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

- **Global Enforcement**: The `.gemini/styles/` directory contains the extracted knowledge of all former Copilot instructions. By symlinking this directory into your target repos, GCA inherently understands the overarching I4G rules without needing manual prompting.
- **Local Overrides**: Repositories can define their own local `.gemini/context.md` files alongside the symlink. GCA intelligently merges the global platform rules with the specific, local nuances of the repository you are actively working in.

---

## 📖 Framework Documentation Suite

Ready to master the GCA workflow? Dive into the official documentation:

1. **[Onboarding Guide](docs/onboarding.md)**: Step-by-step setup for new developers.
2. **[Routine Catalog](docs/routine-catalog.md)**: Explore every available prompt and its purpose.
3. **[Cookbook](docs/cookbook.md)**: See real-world examples of feature development and bug hunting using GCA.
4. **[Customization Guide](docs/customization-guide.md)**: Learn how to contribute new routines and update the platform styleguide.

---

_For support, issues, or suggestions regarding the GCA Productivity Framework, please open an issue in this repository._
