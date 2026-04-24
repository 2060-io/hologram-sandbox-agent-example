# Hologram AI Agent Example

Starter template for building your own **Hologram Verifiable AI Agent**. Fork this repo and customize the agent pack to ship a verifiable, credential-authenticated AI agent that users can reach through [Hologram Messaging](https://hologram.zone).

This example is built with the [`hologram-generic-ai-agent-vs`](https://github.com/2060-io/hologram-generic-ai-agent-vs) container — a ready-to-use chatbot with DIDComm messaging, verifiable-credential authentication, RAG, and MCP tool integration, all driven by a single `agent-pack.yaml` manifest.

## What this example gives you

- An AI agent that **authenticates users with a Hologram Demo Avatar credential** (AnonCreds, issued by the deps repo).
- Integration with **[Context7](https://context7.com) MCP** (zero-auth, publicly hosted) so the agent can answer questions about any library or framework by fetching up-to-date documentation in real time.
- A working `docker-compose.yml` for local development with hot-reload-friendly volume mounts.
- A GitHub Actions workflow that deploys the agent to Kubernetes via the `hologram-generic-ai-agent-chart` Helm chart.
- Sensible defaults in every config file — override what you need, keep the rest.

## Dependencies

This agent is a **child service** of an Organization VS and authenticates users against an Avatar credential. Both are deployed from a companion repo:

- [`hologram-ai-agent-example-deps`](https://github.com/2060-io/hologram-ai-agent-example-deps) — Organization + Avatar VS (one-time setup)

The pre-deployed demo instance lives at `organization.demos.hologram.zone` / `avatar.demos.hologram.zone`. You can point this example at it directly, or stand up your own for full control.

```
┌─────────────────────────┐
│   Hologram Demo SA      │   organization.demos.hologram.zone
│   (Organization VS)     │   ← issues your agent a Service credential
└──────────┬──────────────┘
           │
┌──────────▼──────────────┐
│   Demo Avatar Issuer    │   avatar.demos.hologram.zone
│   (Avatar VS)           │   ← issues Avatar credentials to end users
└──────────┬──────────────┘
           │
┌──────────▼──────────────┐
│   Hologram Example      │   example-agent.demos.hologram.zone
│   Agent (this repo)     │   ← authenticates users with Avatar credential
└─────────────────────────┘
```

## Repository layout

```
hologram-ai-agent-example/
├── config.env              # Service metadata, URLs, ports, LLM, MCP, DB
├── agent-pack.yaml         # Prompts, languages, flows, MCP, RAG, memory
├── deployment.yaml         # Helm chart values for K8s
├── common/
│   └── common.sh           # Shared shell helpers
├── docker/
│   └── docker-compose.yml  # Local dev stack (chatbot + redis + postgres)
├── scripts/
│   ├── setup.sh            # Local setup: VS Agent + ngrok + Service credential
│   └── start.sh            # Start Docker Compose stack
├── docs/
│   └── README.md           # User-facing guide (what the agent does)
└── .github/
    └── workflows/
        └── deploy.yml      # Deploy to Kubernetes
```

## Local development

### Prerequisites

- Docker + Docker Compose
- [ngrok](https://ngrok.com) (authenticated)
- `curl`, `jq`
- An OpenAI API key (or any other supported LLM — see the [agent-pack schema](https://github.com/2060-io/hologram-generic-ai-agent-vs/blob/main/docs/agent-pack-schema.md))
- [Hologram Messaging](https://hologram.zone) on your phone
- A Hologram Demo Avatar credential (grab one from `avatar.demos.hologram.zone`)

### Get your Avatar credential

1. Open Hologram Messaging on your phone
2. Navigate to `https://avatar.demos.hologram.zone/`
3. Scan the QR code, follow the prompts, receive your **Hologram Demo Avatar** credential

### Quick start

```bash
# 1. Set up the VS Agent (deploys container, gets Service credential)
set -a
source config.env
set +a
./scripts/setup.sh

# 2. Start the chatbot stack (chatbot + redis + postgres)
export MCP_CONFIG_ENCRYPTION_KEY=$(openssl rand -hex 32)
export OPENAI_API_KEY=sk-...
./scripts/start.sh
```

> Prefer a different LLM? `agent-pack.yaml` supports Anthropic, Ollama (local), and any OpenAI-compatible endpoint (Kimi, DeepSeek, Groq, Together AI, …). See [how-to-use-ollama.md](https://github.com/2060-io/hologram-generic-ai-agent-vs/blob/main/docs/how-to-use-ollama.md).

## Kubernetes deployment

The `.github/workflows/deploy.yml` workflow deploys the agent via Helm to your cluster.

### Required GitHub secrets

| Secret | Description |
|---|---|
| `OVH_KUBECONFIG` | Kubeconfig for the target cluster |
| `K8S_NAMESPACE` | Target namespace |
| `EXAMPLE_AGENT_OPENAI_API_KEY` | OpenAI API key for the LLM |
| `EXAMPLE_AGENT_POSTGRES_PASSWORD` | PostgreSQL password |
| `EXAMPLE_AGENT_MCP_CONFIG_ENCRYPTION_KEY` | Encryption key for per-user MCP configs (`openssl rand -hex 32`) |
| `EXAMPLE_AGENT_WALLET_KEY` | VS Agent wallet encryption key (`openssl rand -base64 32`) |
| `EXAMPLE_AGENT_VSAGENT_DB_PASSWORD` | VS Agent internal DB password |

### Deploy

Run the **Deploy Example Agent** workflow from the Actions tab with step `all`. This:

1. Creates the namespace, Postgres secret, and agent-pack ConfigMap
2. Runs `helm upgrade --install` with `hologram-generic-ai-agent-chart`
3. Obtains a Service credential from `organization.demos.hologram.zone` and links it on the new agent

The agent is then reachable at the ingress host configured in `deployment.yaml` (default `example-agent.demos.hologram.zone`).

## Customizing for your own agent

The whole point of this repo is to be forked. Typical customization path:

1. **Identity.** Edit `config.env` — `SERVICE_NAME`, `SERVICE_DESCRIPTION`, `SERVICE_LOGO_URL`, `AGENT_PUBLIC_URL`, ingress host in `deployment.yaml`.
2. **Persona.** Edit `agent-pack.yaml` — `metadata`, `languages.<lang>.greetingMessage`, `languages.<lang>.systemPrompt`, `llm.agentPrompt`.
3. **LLM.** Switch provider / model in `agent-pack.yaml:llm` and `config.env`. For OpenAI-compatible endpoints, set `OPENAI_BASE_URL`.
4. **Tools.** Add or swap MCP servers under `mcp.servers` — the Context7 entry is a template. Use `accessMode: user-controlled` for per-user tokens (e.g. GitHub PAT), `admin-controlled` for shared tokens (e.g. Wise API key).
5. **Authentication.** Change `flows.authentication.credentialDefinitionId` to require a different credential, or set `enabled: false` to make the agent open.
6. **RBAC / approvals** (advanced). Use `toolAccess.roles` and `toolAccess.approval` to gate tools by role and require managerial approval for sensitive operations. See [`rbac-approval-spec.md`](https://github.com/2060-io/hologram-generic-ai-agent-vs/blob/main/docs/rbac-approval-spec.md).

Full reference: [Agent Pack schema](https://github.com/2060-io/hologram-generic-ai-agent-vs/blob/main/docs/agent-pack-schema.md).

## Related

- [`hologram-ai-agent-example-deps`](https://github.com/2060-io/hologram-ai-agent-example-deps) — organization + avatar dependencies
- [`hologram-generic-ai-agent-vs`](https://github.com/2060-io/hologram-generic-ai-agent-vs) — the underlying agent container + Helm chart
- [`vs-agent`](https://github.com/2060-io/vs-agent) — DIDComm / verifiable credential primitives
- [`hologram-verifiable-services`](https://github.com/2060-io/hologram-verifiable-services) — more agent examples (Wise, X, GitHub, …)
- [Hologram docs](https://docs.hologram.zone) — concepts, how-tos, reference

## License

Apache-2.0
