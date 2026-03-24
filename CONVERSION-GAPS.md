# Claude Agent to Copilot Studio: Conversion Gap Analysis

This document tracks capabilities from the Claude-based Monique executive assistant
that are not directly portable to Microsoft Copilot Studio, and their resolution status.

## Fas 1 (Current) — Read-only tools implemented

### Implemented
- [x] Persona and instructions (agent.mcs.yml)
- [x] Québécois tone and expressions
- [x] Inbox summary (Graph API via Power Automate)
- [x] Email search (Graph API via Power Automate)
- [x] Calendar view (Graph API via Power Automate)
- [x] Availability check (Graph API via Power Automate)

### Not yet implemented (deferred to Fas 2+)

| # | Claude Capability | Gap | Proposed Resolution | Fas |
|---|-------------------|-----|---------------------|-----|
| 1 | **Sender-based access control** | Claude agent inspects `from` field to enforce per-sender rules (Claudio=full, family=calendar only, Helena=coordinator, others=rejected). Copilot Studio authenticates the *user talking to the agent*, not email senders. | Simplify to Entra ID group-based access (GroupMembership policy). Per-sender logic can only be partially replicated via instructions. | 2 |
| 2 | **Send/reply email** (Tier 2) | Claude uses json-response format via local gateway. Copilot Studio needs Power Automate with Mail.Send permission + approval flow. | Add Power Automate flows with Approvals connector for Tier 2 confirmation. | 2 |
| 3 | **Create/update/delete calendar events** (Tier 2) | Claude uses calendar/create, update, delete endpoints. Copilot Studio needs Calendars.ReadWrite + approval. | Add Power Automate flows with approval step before Graph write operations. | 2 |
| 4 | **Tier 2 approval workflow** | Claude shows draft inline and waits for user confirmation in chat. Copilot Studio has no native "confirm before executing" pattern. | Use Question node (BooleanPrebuiltEntity) before InvokeFlowAction for write operations. Or use Power Automate Approvals connector. | 2 |
| 5 | **Flag/move/mark-as-read mail** | Claude uses Graph proxy PATCH/POST for mail operations. These are Tier 1 (no approval) in Claude. | Add dedicated Power Automate flows for each operation. Low priority since these are convenience features. | 3 |
| 6 | **Agent delegation (Elsa, Julia)** | Claude delegates Google (Elsa) and LinkedIn (Julia) tasks to other agents. Copilot Studio has no multi-agent delegation equivalent. | Not portable. Could use connected agents (AgentDialog) if Elsa/Julia are also ported to Copilot Studio, or use Power Automate to call external APIs. | 3+ |
| 7 | **LinkedIn triage workflow** | Claude reads LinkedIn inbox via Julia agent, categorizes, updates state file, forwards leads. Complex multi-step workflow with state management. | Not portable to Copilot Studio. Requires external orchestration (Power Automate scheduled flow or Azure Functions). | N/A |
| 8 | **Monique's own mailbox** | Claude can read monique@enknapp.nu separately. Copilot Studio agent doesn't have its own mailbox concept. | Could add a separate Power Automate flow, but unclear use case in Copilot Studio context. | 3 |
| 9 | **File system access** | Claude reads /mnt/azure/edgepharos/ and personal knowledge base. Copilot Studio uses Knowledge Sources (SharePoint, websites). | Add SharePoint knowledge sources pointing to equivalent document libraries. | 2 |
| 10 | **Content review (/content-review)** | Claude uses a content review skill before sending external communications. | Not portable. Could be replicated via Power Automate + AI Builder for content moderation. | 3 |
| 11 | **Scheduler identity** | scheduler@edgepharos.local triggers pre-approved tasks without confirmation. Copilot Studio doesn't support automated/scheduled triggers natively. | Use Power Automate scheduled flows that call Graph API directly, bypassing the agent. | N/A |
| 12 | **Real-time date awareness** | Claude knows "today" via system clock. Copilot Studio topics receive dates as user input strings. The generative AI can infer "idag" but the flow needs YYYY-MM-DD. | Instructions guide the agent to convert natural language dates. Works for simple cases but may fail for complex relative dates. | 1 (partial) |
| 13 | **Conversation history** | Claude maintains full conversation context. Copilot Studio has session-scoped context but no persistent conversation history across sessions. | Use Dataverse or external storage for cross-session state if needed. | 3 |
| 14 | **Mail folder operations** | Claude lists and navigates mail folders. Not implemented in Fas 1. | Add Power Automate flow for folder listing/navigation. | 3 |

## Architecture Differences

| Aspect | Claude Agent | Copilot Studio |
|--------|-------------|----------------|
| **Runtime** | Local process with direct API access | Cloud-hosted, Power Automate for API calls |
| **Auth model** | Agent token (AGENT_TOKEN) | Entra ID Integrated + Key Vault for app secrets |
| **Tool execution** | Synchronous curl calls | Async Power Automate flows (1-5s cold start) |
| **Response control** | Full markdown formatting | Generative AI formats from raw JSON; less control |
| **Multi-agent** | Direct delegation to expert agents | Connected agents (AgentDialog) or separate flows |
| **State management** | File-based state (JSON on Azure Files) | Dataverse, or external via Power Automate |
| **Scheduling** | Cron-triggered via scheduler identity | Power Automate scheduled flows (separate from agent) |
| **Model** | Claude Sonnet 4.5 | Claude Sonnet 4.5 (via Copilot Studio AI model selection) |

## Foundry Agent Alternative

A parallel implementation as an Azure AI Foundry agent could bridge many of these gaps:
- Direct Azure OpenAI / Claude model access
- Custom tool definitions (functions) for Graph API
- Built-in conversation history and state management
- Programmatic approval workflows
- Multi-agent orchestration support
- Would live in a separate repo (foundry-agents)
