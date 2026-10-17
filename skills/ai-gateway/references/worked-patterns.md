<!-- generated from verified brain use cases by tools/askbench/export.mjs; edit the source, not this file -->
# Worked patterns: access-identity

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Give each audience its own set of models, and nothing else

**Users ask:** "different teams should see different models"; "most staff get the cheaper models, a small group gets everything"

**Where in the UI:** Access Control screen; for a virtual account, Manage Permissions on its row

**How:**
1. Create one virtual account per application (chat UI, coding agent, service) and keep its permissions minimal. ([docs](https://www.truefoundry.com/docs/platform/virtual-account-management))
2. Put each audience's models in its own virtual model provider group; the User role lets a subject call the group's models, Manager lets it change them. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model-advanced))
3. Grant access as a role on a resource to a subject (user, team, virtual account or agent), from Access Control or as a role-binding manifest for GitOps. ([docs](https://www.truefoundry.com/docs/change-announcements/removing-collaborators-from-resources))
4. Users and virtual accounts must be granted access to a model account before they can call its models; permissions are managed in Access Control. ([docs](https://www.truefoundry.com/docs/ai-gateway/authentication))
5. Clients pass virtual-model-group-name/virtual-model-name (or its slug) as the model value; it works like any other model in the AI Gateway. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model-advanced))

**Watch out:**
- Test each tier with that audience's virtual account token, not an admin key: tenant admins reach every model, so an admin call proves nothing about scoping. ([docs](https://www.truefoundry.com/docs/ai-gateway/gateway-access-control))
- Collaborators on resource manifests are the legacy path; in a manifest, omit the field rather than sending an empty list, which revokes. ([docs](https://www.truefoundry.com/docs/change-announcements/removing-collaborators-from-resources))
- A newly created virtual account can authenticate but cannot do anything until it is granted access in Access Control; that is the intended default. ([docs](https://www.truefoundry.com/docs/platform/virtual-account-management))


---

# Worked patterns: budgets-finops

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Daily caps per team, developer and coding-agent account, watched before they block

**Users ask:** "we want a daily cap per developer and per team, but start by just watching"; "how do I know which budget blocked a request"

**How:**
1. Point Claude Code at the gateway so every model call passes through it for cost tracking, rate and budget limits, access control, guardrails and logging. ([docs](https://www.truefoundry.com/docs/ai-gateway/claude-code))
2. Pick the partition per rule: Per user for people, Per virtual account for an app or agent account, Aggregate for a shared team pool. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
3. Create the rules in audit mode first: requests pass, usage is tracked and alerts fire. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
4. Overlapping rules all apply; a request is blocked if any enforcing rule that matches it is over budget. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
5. When enforcing, a breach returns HTTP 429, and the x-tfy-applied-rules header names the budget that blocked it. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))

**Watch out:**
- A budget scoped to a virtual model blocks the whole request when exceeded; its fallback targets are not tried. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
- Cost from a cancelled or disconnected stream counts against budgets like a completed request, except on do-not-bill providers (Google Vertex AI, Azure OpenAI, Azure AI Foundry), which carry zero cost. ([docs](https://www.truefoundry.com/docs/ai-gateway/cost-tracking))


---

# Worked patterns: clients

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Attribute and cap cost per person on a shared chat app that uses one gateway key

**Users ask:** "everyone shares one key, so we cannot see who spent what"; "can each employee get a daily allowance on the shared chat app"; "the chat UI forwards user headers but the gateway does not show them"

**How:**
1. Have the app (or a thin proxy in front of it) send the end user in the X-TFY-METADATA header on every request; keys and values are strings. ([docs](https://www.truefoundry.com/docs/ai-gateway/request-headers))
2. Group cost metrics by that metadata key to see spend per person. ([docs](https://www.truefoundry.com/docs/ai-gateway/request-headers))
3. Create a budget rule with Apply limit as Per metadata on that key, so each person gets an independent budget on the shared account. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
4. Start the rule in audit mode, then switch to enforce once the numbers look right. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
5. Add a Metadata Validation guardrail on the LLM Input hook with a 'Key must exist' rule for the user key; a missing key raises missing_required, and with an enforcing strategy the request is blocked. ([docs](https://www.truefoundry.com/docs/ai-gateway/metadata-validation))

**Watch out:**
- How a budget is partitioned is fixed at creation; to change aggregate vs per-entity, create a new rule and disable the old one. ([docs](https://www.truefoundry.com/docs/ai-gateway/budget-limiting-v2))
- Virtual account tags are also added to resolved metadata and, for metadata-based routing, override a request value with the same key; do not reuse a tag key as the per-person key. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))

## Move services on the OpenAI SDK onto the gateway with the smallest change

**Users ask:** "we have many services on the OpenAI SDK, each with its own OpenAI key"; "what is the minimum code change to use the gateway"; "which token should a production service use"

**How:**
1. Keep the standard OpenAI client; point its base URL at the AI Gateway and use your TrueFoundry API key. ([docs](https://www.truefoundry.com/docs/ai-gateway/native-sdk-support))
2. Provider keys stay in the gateway; services hold only the TrueFoundry token. ([docs](https://www.truefoundry.com/docs/ai-gateway/authentication))
3. Set model to the gateway model id, in the format provider_account/model_name (for example openai-main/gpt-4o-mini). ([docs](https://www.truefoundry.com/docs/ai-gateway/making-llm-requests-via-gateway))
4. Run each service on a Virtual Account token, not a personal token. ([docs](https://www.truefoundry.com/docs/ai-gateway/authentication))
5. Grant each service's account access to the model account it calls. ([docs](https://www.truefoundry.com/docs/ai-gateway/authentication))

**Watch out:**
- A personal access token stops working when its owner leaves, taking the service down with it. ([docs](https://www.truefoundry.com/docs/ai-gateway/authentication))


---

# Worked patterns: guardrails

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Different guardrail strictness per audience, with credentials blocked for everyone

**Users ask:** "mask client identifiers for most staff but let one group see raw text"; "block pasted API keys and AWS credentials for every user"; "our output moderation guardrail does not catch streamed answers"

**Where in the UI:** AI Gateway → Policies → Guardrails

**How:**
1. Write one guardrail policy per audience; a policy decides which guardrails run based on the model, the caller and request metadata. ([docs](https://www.truefoundry.com/docs/ai-gateway/guardrails-configuration))
2. Scope each policy to its audience with subject filters on users, teams or virtual accounts. ([docs](https://www.truefoundry.com/docs/ai-gateway/guardrails-configuration))
3. For tiers that must not send identifiers, attach a Regex Pattern Matching guardrail in mutate mode with custom patterns, so matches are replaced and the request continues. ([docs](https://www.truefoundry.com/docs/ai-gateway/regex-pattern-matching))
4. For credentials, attach Secrets Detection in validate mode on a policy with an empty when block, which matches every request. ([docs](https://www.truefoundry.com/docs/ai-gateway/secrets-detection))
5. An empty when block applies the policy to all requests. ([docs](https://www.truefoundry.com/docs/ai-gateway/guardrails-configuration))

**Watch out:**
- Guardrails from all matching policies are combined per hook, so a lenient policy adds to, not replaces, a stricter policy that also matches; exclude a group from the strict policy with a NOT IN subject filter. ([docs](https://www.truefoundry.com/docs/ai-gateway/guardrails-configuration))
- Output guardrails are skipped on streamed responses; use stream false where output checks must run, or check the assembled response client-side. Input guardrails always run. ([docs](https://www.truefoundry.com/docs/ai-gateway/guardrails-overview))


---

# Worked patterns: mcp-tools

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Expose a curated, read-only subset of tools from several MCP servers

**Users ask:** "business users need read-only GitHub and Slack tools, no delete or write"; "do we need to deploy a separate MCP server for a restricted tool set"; "why do tool names have a suffix on the virtual server"

**How:**
1. Create a Virtual MCP Server and pick only the safe tools from each source server. ([docs](https://www.truefoundry.com/docs/ai-gateway/mcp/virtual-mcp-server))
2. Clients connect to it like any remote MCP server; nothing new is deployed. ([docs](https://www.truefoundry.com/docs/ai-gateway/mcp/virtual-mcp-server))
3. Expect tool names with a short server identifier appended, which keeps names from different servers unique. ([docs](https://www.truefoundry.com/docs/ai-gateway/mcp/virtual-mcp-server))

**Watch out:**
- By default the whole virtual server fails if any source server is unreachable; turn on Best Effort Mode to keep serving tools from the healthy ones. ([docs](https://www.truefoundry.com/docs/ai-gateway/mcp/virtual-mcp-server))
- Tool names longer than 56 characters are truncated. ([docs](https://www.truefoundry.com/docs/ai-gateway/mcp/virtual-mcp-server))


---

# Worked patterns: models-providers

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Bedrock cross-region inference profiles: what traces show and what IAM needs

**Users ask:** "we use a us. inference profile but traces still show one region"; "is Bedrock cross-region inference actually working"; "should I change the account region to get cross-region capacity"

**Where in the UI:** AI Gateway → Models → AWS Bedrock account

**How:**
1. Add the model with an inference profile ID, not a plain model ID; cross-region routing needs it. ([docs](https://www.truefoundry.com/docs/ai-gateway/aws-bedrock))
2. Include bedrock:GetInferenceProfile in the IAM policy; the gateway uses it to resolve the profile. ([docs](https://www.truefoundry.com/docs/ai-gateway/aws-bedrock))
3. Traces always show the source region endpoint; that is expected, because AWS does not report the destination region. ([docs](https://www.truefoundry.com/docs/ai-gateway/aws-bedrock))

**Watch out:**
- Changing the account region does not change where a profile routes; the profile prefix decides the geography. ([docs](https://www.truefoundry.com/docs/ai-gateway/aws-bedrock))
- Grant IAM permissions for both the inference profile and the foundation model in every destination region, not only the default one; otherwise requests fail with Access Denied when Bedrock routes elsewhere. ([docs](https://www.truefoundry.com/docs/ai-gateway/aws-bedrock))


---

# Worked patterns: observability

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Find a specific gateway call, a whole chat session, or requests the client dropped

**Users ask:** "our backend logs a request id; how do I find the matching gateway call"; "show every LLM call from one chat session together"; "requests come back empty with client disconnected; is the gateway dropping them"

**How:**
1. Send your own request id in x-tfy-request-id; it is recorded on the request trace. ([docs](https://www.truefoundry.com/docs/ai-gateway/request-headers))
2. Send x-tfy-conversation-id with the chat session id to group the calls of one conversation. ([docs](https://www.truefoundry.com/docs/ai-gateway/request-headers))
3. Spans carry tfy.request.conversation_id, the unique identifier for the conversation when the request is part of a chat. ([docs](https://www.truefoundry.com/docs/ai-gateway/fetch-request-logs-span-attributes))
4. A client that hangs up mid-stream is recorded with HTTP 499 and tfy.error_type=client-disconnected, and may still carry cost; filter Model spans on it. ([docs](https://www.truefoundry.com/docs/ai-gateway/fetch-request-logs-use-cases-advanced))

**Watch out:**
- Disconnected requests can still cost money: the provider has usually generated and billed tokens before the client left. ([docs](https://www.truefoundry.com/docs/ai-gateway/cost-tracking))
- A hang-up right after the stream finishes is recorded as a normal success, not a disconnect. ([docs](https://www.truefoundry.com/docs/ai-gateway/cost-tracking))


---

# Worked patterns: routing-failover

Patterns that came up repeatedly in real customer questions. Each step links the docs page that states it; read that page before giving exact field names or values.

## Route by difficulty behind one model name, or fail a primary over to another vendor (one routing strategy per virtual model)

**Users ask:** "send simple questions to the cheap model and hard ones to the strong model"; "if the primary model is down, use another vendor"; "almost every request is answered by the backup model"

**Where in the UI:** AI Gateway → Models → Virtual Model

**How:**
1. Create a virtual model with complexity-based routing (Auto Routing); it classifies each request into a tier and routes to that tier's target. ([docs](https://www.truefoundry.com/docs/ai-gateway/auto-routing))
2. Give every tier a target, or the virtual model is rejected on save. ([docs](https://www.truefoundry.com/docs/ai-gateway/auto-routing))
3. Keep the default heuristic classifier unless prompts are ambiguous; the LLM classifier adds a model call per classified turn. ([docs](https://www.truefoundry.com/docs/ai-gateway/auto-routing))
4. For failover, use priority-based routing: priority 0 is the primary, the next priority takes over when it fails. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))
5. Read the x-tfy-resolved-model response header to see which target actually served each request. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))

**Watch out:**
- 429 is in the default fallback status codes, so a 429 from a target makes the gateway try a different target; the x-tfy-resolved-model response header shows the model actually used, which may differ due to fallbacks. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))
- A target only receives failover traffic if fallback_candidate is true for it. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))
- The LLM classifier model must be a catalog chat model, not a virtual model. ([docs](https://www.truefoundry.com/docs/ai-gateway/auto-routing))

## Move team routing rules out of the shared Routing Config into virtual models

**Users ask:** "one team's routing change keeps overriding another's"; "dev and prod should load-balance differently"; "is the Routing Config still the recommended way to do fallbacks"

**Where in the UI:** AI Gateway → Models → Virtual Model

**How:**
1. The tenant-level Routing Config is deprecated; move every rule to virtual models. ([docs](https://www.truefoundry.com/docs/change-announcements/deprecation-of-routing-config))
2. Create one virtual model per model name your apps send, with the same strategy, targets, retries and fallbacks. ([docs](https://www.truefoundry.com/docs/change-announcements/deprecation-of-routing-config))
3. Give each team its own virtual model group: the team's members get User to call, its owners get Manager to change configuration. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model-advanced))
4. Constrain a target with metadata_match so it only receives traffic when resolved request metadata matches (for example region: us); if no target matches, the gateway returns 404, so keep one target without metadata_match as a catch-all or cover all values. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))
5. The environment can come from the caller's virtual account tags, so apps do not have to send it. ([docs](https://www.truefoundry.com/docs/ai-gateway/virtual-model))

**Watch out:**
- Auto (complexity) routing exists only on virtual models; it was never available in the Routing Config. ([docs](https://www.truefoundry.com/docs/ai-gateway/auto-routing))

