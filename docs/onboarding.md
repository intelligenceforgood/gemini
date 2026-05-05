# GCA Framework Developer Onboarding

Welcome to the Gemini Code Assist (GCA) Productivity Framework! This guide will walk you through setting up your local environment to leverage the I4G platform's standardized AI workflows.

## Prerequisites

- Visual Studio Code installed.
- Gemini Code Assist extension installed and authenticated.
- Access to the I4G Git repositories, specifically the new `gemini` repository.

## Step 1: Workspace Configuration

The recommended way to use GCA is to open the unified parent directory containing all your I4G repositories. This allows GCA to see the global standards alongside your active code.

1. Ensure all your I4G projects (e.g., `core`, `ui`, `infra`, `copilot`, `ssi`, `ml`) are located in the same parent directory.
2. Clone the `gemini` repository into that same parent directory.
3. In VS Code, go to **File > Open Folder...** and select the parent directory.

## Step 2: Establish the Anchor Context

Because VS Code treats the parent directory as the single workspace root, GCA will look for the global `.gemini` configuration at this top level. You only need to symlink it once.

Run the following commands in your **unified parent directory**:

```bash
cd /path/to/your/parent/i4g/directory
rm -rf .gemini
ln -s gemini/.gemini .gemini
```

_Why a symlink?_ When GCA analyzes a file in your project, it looks for a `.gemini` directory in that project's root. The symlink ensures it always reads the most up-to-date global standards without needing to duplicate files.

## Step 3: Install Snippets

To speed up prompt execution, we provide VSCode snippets.

1. In VSCode, open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`).
2. Type and select `Snippets: Configure User Snippets`.
3. Select `New Global Snippets file...` or edit an existing one.
4. Copy the contents of `gemini/snippets/gemini.code-snippets` into your VSCode snippet file.

_(Alternatively, VSCode may automatically detect workspace snippets if they are configured in a `.vscode` folder at the workspace root, depending on your setup)._

## Step 4: Verification

Let's test that everything is working.

1. Open the GCA Chat panel in VSCode.
2. Type `gca-plan` in the chat input.
3. If snippets are configured correctly, it should expand into the "Plan Work" routine template.
4. Ask GCA: `"What are the core coding standards for this repository based on the .gemini directory?"`
5. GCA should respond by referencing rules defined in the `gemini/.gemini/styles/` directory.

## Step 5: Mastering Agent Mode (Quota Management)

Gemini Code Assist has a powerful "Agent Mode" that allows it to autonomously search your workspace and run terminal commands. However, **leaving Agent Mode ON constantly will rapidly drain your daily token quota.**

To optimize your quota, follow these strict toggling rules:

### Turn Agent Mode OFF (🟢 High Quota Efficiency)

Use this mode for 80% of your interactions:

- Asking general programming questions.
- Analyzing log files or stack traces (paste them in, or use explicitly scoped tags like `@file:error.log`).
- Drafting PRDs or initial architectural plans where you provide the context via text.
- Reviewing code where you only want it to look at the active file you have open.

### Turn Agent Mode ON (🔴 Quota Heavy - Use Strategically)

Only toggle this ON when you need autonomous execution:

- Executing a specific implementation plan across multiple files.
- Asking it to "run the tests and fix the errors" (where it needs autonomy to use the terminal).
- Refactoring tasks where you have explicitly scoped the target using `@folder` tags.

## Step 6: Your First Workflow

Now that you are set up, try executing a standard workflow:

1. Use `gca-prd` to draft a mini-feature based on a fake requirement.
2. Save the output to `docs/mini_feature.md`.
3. Use `gca-impl` and ask GCA to create an implementation plan based on `docs/mini_feature.md`.
4. Use `gca-work` to ask GCA to execute Task 1 of that plan.

Congratulations! You are now using the GCA Productivity Framework. Check out the [Cookbook](cookbook.md) for more advanced use cases.
