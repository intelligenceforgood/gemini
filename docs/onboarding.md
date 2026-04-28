# GCA Framework Developer Onboarding

Welcome to the Gemini Code Assist (GCA) Productivity Framework! This guide will walk you through setting up your local environment to leverage the I4G platform's standardized AI workflows.

## Prerequisites

- Visual Studio Code installed.
- Gemini Code Assist extension installed and authenticated.
- Access to the I4G Git repositories, specifically the new `gemini` repository.

## Step 1: Workspace Configuration

The most effective way to use GCA is with a VSCode Multi-Root Workspace. This allows GCA to see the global standards alongside your active code.

1. Clone the `gemini` repository into the same parent directory as your other I4G projects (e.g., `core`, `ui`, `infra`).
2. Create or open your `.code-workspace` file.
3. Ensure the `gemini` repo is included in the folders array:

```json
{
  "folders": [
    { "path": "core" },
    { "path": "ui" },
    { "path": "gemini" }
  ],
  "settings": {}
}
```

## Step 2: Establish the Anchor Context

For GCA to enforce our specific architectural rules and coding standards natively, you need to link the global styleguide into the repositories you actively work in.

Run the following commands in the root of **each product repository** you develop in (e.g., `core/`, `ui/`):

```bash
mkdir -p .gemini
# Create a relative symlink pointing to the global styleguide
ln -s ../gemini/.gemini/styleguide.md .gemini/styleguide.md
```

*Why a symlink?* When GCA analyzes a file in your project, it looks for a `.gemini` directory in that project's root. The symlink ensures it always reads the most up-to-date global standards without needing to duplicate files.

## Step 3: Install Snippets

To speed up prompt execution, we provide VSCode snippets.

1. In VSCode, open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`).
2. Type and select `Snippets: Configure User Snippets`.
3. Select `New Global Snippets file...` or edit an existing one.
4. Copy the contents of `gemini/snippets/gemini.code-snippets` into your VSCode snippet file.

*(Alternatively, VSCode may automatically detect workspace snippets if they are configured in a `.vscode` folder at the workspace root, depending on your setup).*

## Step 4: Verification

Let's test that everything is working.

1. Open the GCA Chat panel in VSCode.
2. Type `gca-plan` in the chat input.
3. If snippets are configured correctly, it should expand into the "Plan Work" routine template.
4. Ask GCA: `"What are the core coding standards for this repository based on the .gemini directory?"`
5. GCA should respond by referencing rules defined in `gemini/.gemini/styleguide.md`.

## Step 5: Your First Workflow

Now that you are set up, try executing a standard workflow:

1. Use `gca-prd` to draft a mini-feature based on a fake requirement.
2. Save the output to `docs/mini_feature.md`.
3. Use `gca-impl` and ask GCA to create an implementation plan based on `docs/mini_feature.md`.
4. Use `gca-work` to ask GCA to execute Task 1 of that plan.

Congratulations! You are now using the GCA Productivity Framework. Check out the [Cookbook](cookbook.md) for more advanced use cases.
