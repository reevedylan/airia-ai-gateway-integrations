# Contributing to Airia AI Gateway Integrations

Thanks for helping document how developer tools connect to the Airia AI Gateway. This repo stays useful because every integration follows the same structure and template, so please read this before opening a PR.

## Where does my integration go?

Work out which category fits before creating a folder:

- **CLI agent** (a coding agent run from the terminal, e.g. Claude Code, Codex CLI) → `cli-agents/`
- **Editor or IDE** (e.g. Cursor, VS Code, Zed) → `editors/`
- **Language SDK or framework** (e.g. OpenAI SDK, LangChain) → `sdks/`
- **A specific tool + model provider combination that needs extra, non-obvious configuration** (e.g. routing a CLI agent to a specific cloud provider requires model ID mapping) → `runbooks/`

Don't create a runbook for every possible tool + provider pairing. Only write one when there's real friction beyond what the generic tool doc in `cli-agents/`, `editors/`, or `sdks/` already covers.

## Adding a CLI agent, editor, or SDK integration

1. Copy `templates/tool-template.md` to `<category>/<tool-slug>/README.md` (e.g. `cli-agents/codex-cli/README.md`).
2. Fill in every section, and don't leave template placeholders in the merged doc.
3. If you have screenshots, add them to a local `images/` folder next to the README (e.g. `cli-agents/codex-cli/images/01-setup.png`) and reference them with descriptive alt text.
4. Add a row to the relevant table in the root `README.md`. That table is the single source of truth for what's documented; there are no per-category index READMEs to keep in sync.

## Adding a runbook

1. Name the folder `<tool>-<provider>` (e.g. `claude-code-azure-foundry`).
2. Copy `templates/runbook-template.md` to `runbooks/<tool>-<provider>/README.md`.
3. Link back to the generic tool doc rather than repeating its setup steps.
4. Add screenshots the same way as above, co-located in an `images/` folder.
5. Add a row to the `Runbooks` table in the root `README.md`.

## Screenshots and assets

- Screenshots live next to the doc that uses them, in an `images/` folder, not in the top-level `assets/` folder.
- Number them in the order they appear: `01-`, `02-`, etc.
- Always write descriptive alt text, e.g. `![Airia dashboard showing API key creation](images/01-create-key.png)`.
- The top-level `assets/` folder is only for things reused across multiple docs: logos (`assets/logos/`) and architecture diagrams (`assets/diagrams/`).

## Style guidelines

- Number setup steps; don't use unordered lists for sequential instructions.
- Use fenced code blocks with a language tag.
- Keep the "Overview" section to two or three sentences; detail belongs in the sections below it.
- Link to `docs/what-is-ai-gateway.md` instead of re-explaining gateway concepts (auth, routing, observability) in every doc.

## Pull request checklist

- [ ] Followed `templates/tool-template.md` or `templates/runbook-template.md`
- [ ] Tested the setup steps end-to-end
- [ ] Screenshots (if any) are in a local `images/` folder with alt text
- [ ] Added a row to the relevant table in the root `README.md`
- [ ] All links resolve
