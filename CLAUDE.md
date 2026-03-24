# CopilotStudioAgents

This repo contains Microsoft Copilot Studio agents managed as YAML files.

## Agents

- **Monique/** — VD-assistent för Claudio Torres (Edge Pharos AB). Hanterar e-post, kalender, kunskapsbas. Sonnet 4.5, svenska, integrerad auth.
- **M365 Admin Agent/** — M365-administrationsagent med Graph API-integration.

## Structure

Each agent folder follows the Copilot Studio YAML schema:
- `agent.mcs.yml` — Agent metadata, instructions, capabilities
- `settings.mcs.yml` — Display name, auth, AI settings
- `topics/` — Conversation topics (system + custom)
- `actions/` — Custom actions (Power Automate connectors)
- `knowledge/` — Knowledge sources
- `workflows/` — Power Automate flow definitions
- `.mcs/` — Internal sync state (gitignored)

## Tooling

Uses [skills-for-copilot-studio](https://github.com/microsoft/skills-for-copilot-studio) plugin for Claude Code.
Install: `claude plugin install /c/GitProjects/skills-for-copilot-studio --scope project`

## Environment

- Tenant: Edge Pharos AB (enknapp.nu)
- Power Platform: https://edgepharos.crm4.dynamics.com/
- Language: Swedish (1053)
