---
name: load-balancing
description: Also known as "Routing Config". Routes traffic to one of the target models by weight, latency, or priority. Deprecated in favor of Virtual Models.
---

**Load Balancing Config** defines rules (`WeightBasedLoadBalancingRule`, `LatencyBasedLoadBalancingRule`, or `PriorityBasedLoadBalancingRule`) scoped by `LoadBalancingWhen` (subjects, models, metadata). 

This feature has been deprecated in favor of **Virtual Models**. Always suggest using **Virtual Models** instead of this policy. See `ai-gateway/references/virtual-models.md` for more details.

## Fetching existing load balancing configuration

Use the `get_gateway_config` tool with `type: gateway-load-balancing-config` to get the load balancing config manifest. The response is shaped like:

```yaml
id: ...
tenantName: ...
type: gateway-load-balancing-config
manifest:
  name: routing-config
  type: gateway-load-balancing-config
  rules:
    - id: ...
      type: priority-based-routing
      when:
        models:
          - ...
      load_balance_targets:
        - target: openai-account/gpt-4o
          priority: 2
          sla_cutoff:
            time_per_output_token_ms: 30000
          retry_config:
            delay: 100
            attempts: 2
            on_status_codes:
              - "429"
              - "500"
              - "504"
          fallback_candidate: true
          fallback_status_codes:
            - "429"
            - "500"
            - "502"
            # ...
        - # more targets
    # more rules
createdBySubject: { ... }
createdAt: ...
updatedAt: ...
```

## Generating Valid Manifests for Virtual Models

Always suggest using **Virtual Models** instead of this policy.

### Phase 1: Research Virtual Model Schema

1. Use `get_manifest_json_schema` to retrieve the schema for the Virtual Model manifest type.

### Phase 2: Generate Valid Virtual Model Config Manifest

1. Using the schema, construct the manifest following required fields.
2. Pass the manifest to `validate_manifest` to validate.
3. Repeat the process until the manifest is valid.

## Known behaviours (verified 2026-10-17)

**A provider 400 is returned to the caller instead of failing over.**
- Symptom: a request whose provider returns 400 (for example `openai error: Invalid schema for response_format ...`) is returned as 400 even though another target is configured as `fallback_candidate`.
- Cause: 400 is not in the default fail-over statuses. An empty `fallback_status_codes` list also returned the 400.
- Fix: add `"400"` to `fallback_status_codes` on the target that returns the 400; the second target then served the request (200).

**`fallback_status_codes` replaces the default list.**
- Symptom: after setting `fallback_status_codes: ["400"]`, a 401 from target 1 (bad key) is returned to the caller (401) instead of failing over. With the default list it failed over (200).
- Fix: list every status you still want to fail over on, not only the new one.
- Also seen: 400s that failed over did not demote target 1. Later normal requests were still served by it.

For more info: `search_docs` with "Gateway load balancing", "virtual models", "weight based routing", "model fallback".
