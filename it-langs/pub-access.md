# pub-access.md — Pollinations Community Publisher Access request

**Purpose of this file:** everything needed to fill the official request form at
<https://github.com/pollinations/pollinations/issues/new?template=community-model-allowlist.yml>
in order to obtain **publisher access** so the managed agent **Pica (Picandra Everwise)** can be
listed publicly in the Pollinations catalog.

> The form requests **account-level access**; it does not register the agent itself.
> Submit it from the **same GitHub account** used to sign in at [enter.pollinations.ai](https://enter.pollinations.ai).
> **The issue is public — never paste API keys, bearer tokens or credentials.** The read-only
> GitHub token used by the agent is intentionally kept out of this file (it stays in the private
> `sys-prompt.MD` placeholder).

---

## Suggested issue title

```
[Community Publisher Access]: Pica — the historian of programming languages (managed agent)
```

---

## 1. What do you want to publish? (dropdown, select)

```
[x] Public managed agents
```

*(No model endpoint is being published — Pica is a prompt agent built on an existing Pollinations
text model, so "Public community models" is not needed.)*

## 2. Community model types (if applicable) (dropdown)

```
Text or chat
```

*(Only relevant if model publishing is requested; Pica is text-only anyway.)*

## 3. Publishing plans (textarea, required)

```
Agent: Pica (full name Picandra Everwise), a managed prompt agent — a historian of
programming languages with an intriguing, Socratic personality.

Under the hood: an open structured corpus ("it-langs", 212 YAML fiches and growing,
one file per language from 1945 to the AI era — Fortran to Mojo, including data formats,
query languages, hardware description, proof assistants and esoteric languages) stored
in a versioned Git repository, read live by the agent through the hosted Computer MCP
(curl to raw.githubusercontent.com, read-only token, results cached in /workspace).

Main capabilities and intended use:
- quick-start documents for ~200 languages (install, hello world, first concepts, docs links);
- genealogy queries: languages related to, influenced by, or descended from a given language;
- scope queries (domains, paradigms, status: where a language is used, whether it is alive);
- underlying-stack queries (TypeScript → JavaScript → C++ engine → LLVM IR → machine code);
- non-executable languages (HTML, JSON, Markdown, BNF, UML…): the descriptive side of computing;
- history and timeline storytelling, bilingual FR/EN.

Why it is a good fit for the public catalog:
- a unique cultural angle: preserving and transmitting the whole history of programming,
  an educational resource rather than yet another general-purpose chatbot;
- grounded answers: the agent cites the corpus fiches it uses and flags knowledge that
  falls outside the corpus, instead of improvising facts;
- safe by construction: read-only repository access, no writes, no scraping, no user data
  collection; conversation state limited to a small /workspace cache;
- free to the community: managed agent with no owner-set price — callers only pay their own
  base-model and MCP usage through their Pollinations balance;
- roadmap: the corpus is designed to grow toward exhaustive coverage of every named language,
  including the languages emerging from the AI era (Mojo, Carbon, Bend, Triton…).
```

## 4. Relevant links (input, optional)

```
https://github.com/fabricetayou/syntax-seed — corpus repository (private; contains the
agent kit: sys-prompt, description, publication notes — read access granted on request)
https://gen.pollinations.ai/docs — Pollinations API docs used to design the agent
https://gen.pollinations.ai/mcp — MCP server catalog (Computer MCP used by Pica)
```

## 5. Discord username (input, optional)

```
<your-discord-username>
```

## 6. Technical setup and limits (textarea, required)

```
Type: managed prompt agent (text-only).
Base model: a current strong text model from the public catalog (GET /v1/models).
Tools: hosted Computer MCP only (bash + persistent /workspace filesystem).
Expected request pattern per conversation: 2-5 lightweight curl fetches from
raw.githubusercontent.com (YAML files of a few KB each), jq/grep parsing locally,
then answer generation. No heavy compute, no long-running processes (each bash
command is bounded to 60s by the platform), no external services, no user data
retention beyond a small per-caller cache in /workspace.
Access: read-only token on a single private repository; no write permissions anywhere.
Expected load: low to moderate (educational Q&A traffic); happy to comply with any
rate limit guidance from the team.
```

## 7. Confirmations (checkboxes — all required)

```
[x] I am submitting from the same GitHub account I use to sign in to Pollinations.
[x] If I publish community models, their upstream endpoints will be OpenAI compatible.
[x] If I publish managed agents, callers will use their own Pollinations balance and API-key permissions.
[x] I have not included API keys, bearer tokens, or other credentials in this issue.
[x] I understand that public deployments are monitored and may be deactivated for sustained failures or abuse.
```

*(Confirmation 2 concerns community models; Pica publishes none — ticking it is harmless and expected by the form.)*

---

## After access is granted — deployment recipe

1. Prepare the system prompt: take `it-langs/sys-prompt.MD` and replace `{{GITHUB_RO_TOKEN}}`
   with the read-only GitHub token (keep it secret). The prompt is under the 8,000-character limit.

2. Create `agent.json`:

```json
{
  "title": "Pica — Picandra Everwise",
  "description": "The historian of programming languages: quick-starts, genealogies, scopes, underlying stacks and the languages that never execute. Bilingual, curious, precise.",
  "visibility": "public",
  "systemPrompt": "<contents of sys-prompt.MD, with the token substituted>",
  "baseModel": "<pick a text model ID from https://gen.pollinations.ai/v1/models>",
  "mcpServers": ["computer"]
}
```

3. Create and publish the agent (the callable model id becomes `<github-username>/pica`):

```bash
npx @pollinations/cli agents create \
  --config agent.json \
  --name pica \
  --title "Pica — Picandra Everwise" \
  --visibility public
```

4. Call it like any text model:

```bash
curl https://gen.pollinations.ai/v1/chat/completions \
  -H "Authorization: Bearer $POLLINATIONS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "fabricetayou/pica",
    "messages": [{"role": "user", "content": "Give me a quick start with Lua, and tell me who influenced it."}]
  }'
```

5. Maintenance: update the corpus in the repository (add fiches, fix `year_note` values),
   update the count in `sys-prompt.MD` if needed, then `npx @pollinations/cli agents update <agent-id> --config agent.json`.

Managed-agent economics: no owner-set price; callers pay their own base-model and MCP usage.
Managed agents are always text-only.
