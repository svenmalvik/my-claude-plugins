---
name: interview
description: This skill should be used when the user asks to be "interviewed", runs "/interview", says "interview me about...", or requests a structured discovery process for requirements gathering. Conducts a focused interview for VCE-CLI features and generates a CLI command specification.
---

# VCE-CLI Interview Command

Conduct a focused interview about a new VCE-CLI command or feature. Use the AskUserQuestion tool to gather requirements specific to this Go/Cobra CLI tool for managing Vipps Core Engine AKS infrastructure.

# Context (Assumed)

This interview assumes the following tech stack - skip discovery questions for these:
- **Language**: Go with Cobra CLI framework
- **Auth**: Azure CLI credentials (`azidentity.AzureCLICredential`)
- **Infrastructure**: Azure AKS clusters
- **APIs**: Internal vce-meta API (`https://vce-meta.vippstech.no/api/v1/clusters`)
- **CRDs**: VippsService Kubernetes custom resources
- **Dependencies**: kubectl, kubelogin, az CLI, gh CLI

# Goal

Through iterative questioning, extract:
- What the command does and why it's needed
- How it integrates with existing VCE-CLI commands
- Which clusters/environments it targets
- Azure permissions and API interactions required
- Output format and error handling
- Edge cases specific to AKS/Kubernetes operations

# Interview Process

You MUST use the AskUserQuestion tool throughout. Never assume - always ask.

## Phase 1: Command Purpose (2-3 rounds)

**Round 1 - Core Need**
- What problem does this command solve?
- What manual process does it replace or automate?
- Who will use this command and when?
- What does success look like?

**Round 2 - Scope**
- Is this a new top-level command or subcommand of an existing one?
- What's the minimum viable version vs nice-to-have features?
- Are there related commands this should be consistent with?
- What's explicitly out of scope?

**Round 3 - User Workflow**
- What commands would a user run before this one?
- What would they do with the output?
- How often would this be run (one-off, daily, CI/CD)?

## Phase 2: Command Design (2-3 rounds)

**Round 4 - Interface**
- What should the command be named? (e.g., `vce <noun> <verb>` or `vce <verb>`)
- What flags/arguments are required vs optional?
- Should it support `--output json/yaml/table` formats?
- Should it work on single cluster or multiple clusters?

**Round 5 - Cluster Targeting**
- Which environments should it work with (dev, test, prod, all)?
- Should it use the current kubectl context or accept cluster names?
- Should it operate on all clusters in parallel or sequentially?
- How should it handle cluster filtering (by name pattern, environment, region)?

**Round 6 - Data Sources**
- Does it need to call the vce-meta API?
- Does it need to query Kubernetes resources directly?
- Does it need Azure Resource Manager APIs?
- What data needs to be combined from multiple sources?

## Phase 3: Behavior & Edge Cases (2-3 rounds)

**Round 7 - Output & Feedback**
- What information should be displayed during execution?
- Should it show progress for long-running operations?
- What should the success output look like?
- Should it support `--quiet` or `--verbose` modes?

**Round 8 - Error Handling**
- What happens if Azure auth fails?
- What if some clusters are unreachable?
- What if the vce-meta API is down?
- Should it continue on partial failures or fail fast?

**Round 9 - Edge Cases**
- What if there are no matching clusters?
- What if the user lacks permissions on some clusters?
- Are there rate limits or timeouts to consider?
- What happens with very large result sets?

## Phase 4: Integration (1-2 rounds)

**Round 10 - Existing Code**
- Which existing packages should this command use?
  - `pkg/azure/` for auth?
  - `pkg/context/` for kubectl operations?
  - `pkg/meta/` for vce-meta API?
  - `pkg/vippsservice/` for VippsService CRDs?
- Does this need new functionality in those packages?
- Are there similar commands to use as a template?

**Round 11 - Testing & Docs**
- What scenarios need test coverage?
- Are there fixtures or mocks needed?
- What should the `--help` text explain?
- Any gotchas to document?

## Interview Guidelines

1. **Build on VCE-CLI patterns**: Reference existing commands like `vce context`, `vce vs`, `vce update`

2. **Ask about cluster scope early**: Most VCE commands need to decide single vs multi-cluster behavior

3. **Explore failure modes**: AKS operations have many failure points (auth, network, permissions, API limits)

4. **Consider CI/CD use**: Many VCE commands are used in pipelines - ask about non-interactive mode

5. **Use the AskUserQuestion tool properly**:
   - Ask 1-4 related questions per round
   - Provide concrete options based on existing VCE patterns
   - Use multiSelect for features/flags
   - Keep headers short (e.g., "Scope", "Output", "Errors")

## After Interview: Write Specification

Once complete (typically 8-11 rounds), write a CLI command specification.

### Specification Structure

```markdown
# vce [command] - Specification

## Summary
- One-line description
- Problem it solves
- Primary use case

## Command Interface

### Usage
\`\`\`
vce <command> [flags]
vce <command> <subcommand> [flags]
\`\`\`

### Arguments
| Argument | Required | Description |
|----------|----------|-------------|
| name     | Yes      | ... |

### Flags
| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| --output | -o | table | Output format (table/json/yaml) |
| --cluster | -c | "" | Target specific cluster |
| --all-clusters | -A | false | Run across all clusters |

### Examples
\`\`\`bash
# Basic usage
vce <command> <args>

# With flags
vce <command> --output json

# Pipeline usage
vce <command> --output json | jq '.items[]'
\`\`\`

## Behavior

### Happy Path
1. Step-by-step what happens on success
2. ...

### Error Handling
| Scenario | Behavior |
|----------|----------|
| Azure auth fails | Exit 1 with auth error message |
| Cluster unreachable | Skip cluster, warn, continue |
| No results | Exit 0 with "no results" message |

### Output Formats

**Table (default)**
\`\`\`
NAME        CLUSTER     STATUS
foo         dev-01      Running
\`\`\`

**JSON**
\`\`\`json
{"items": [...]}
\`\`\`

## Implementation

### Package Structure
- `pkg/cmd/<command>.go` - Cobra command definition
- `pkg/<domain>/` - Business logic (if needed)

### Dependencies
- [ ] `pkg/azure/` - Azure auth
- [ ] `pkg/context/` - Kubectl operations
- [ ] `pkg/meta/` - vce-meta API client
- [ ] `pkg/vippsservice/` - VippsService operations

### New Code Required
- List new functions/types needed
- Changes to existing packages

### Testing Strategy
- Unit tests for business logic
- Integration tests with fixtures
- Test cases for error scenarios

## Open Questions
- Remaining unknowns
- Decisions deferred to implementation
```

### Specification Guidelines

1. **Match existing VCE-CLI style**: Look at similar commands for patterns
2. **Be concrete**: Include actual flag names, output formats, error messages
3. **Think about scripting**: JSON output, exit codes, stderr vs stdout
4. **Consider backwards compatibility**: If modifying existing commands

## Example Interview Round

```
<AskUserQuestion>
{
  "questions": [
    {
      "question": "Which clusters should this command operate on by default?",
      "header": "Cluster scope",
      "multiSelect": false,
      "options": [
        {
          "label": "Current kubectl context only",
          "description": "Requires user to set context first, simple and explicit"
        },
        {
          "label": "All clusters from vce-meta",
          "description": "Broader scope, may be slow, needs --cluster flag to narrow"
        },
        {
          "label": "Clusters matching a pattern",
          "description": "e.g., 'dev-*' or 'prod-*', flexible but needs pattern flag"
        },
        {
          "label": "Explicit cluster list required",
          "description": "User must specify --cluster flag, no default"
        }
      ]
    },
    {
      "question": "How should the command handle clusters it can't reach?",
      "header": "Unreachable",
      "multiSelect": false,
      "options": [
        {
          "label": "Skip and warn, continue with others",
          "description": "Best for multi-cluster operations, shows partial results"
        },
        {
          "label": "Fail immediately",
          "description": "Strict mode, good for CI where partial success is failure"
        },
        {
          "label": "Configurable via --fail-fast flag",
          "description": "User chooses behavior, more complex"
        }
      ]
    }
  ]
}
</AskUserQuestion>
```

## When to Stop

Continue until you understand:
1. The complete command interface (name, args, flags)
2. Which clusters/environments it targets
3. What APIs and packages it needs
4. Output format and error handling
5. Edge cases and failure modes

Typically 8-11 rounds for a new command.

## Notes

- **Always use AskUserQuestion tool**: Required for the entire interview
- **Reference existing commands**: `vce context`, `vce vs`, `vce update` for patterns
- **Think about pipelines**: JSON output, exit codes, non-interactive mode
- **Consider permissions**: Azure RBAC, Kubernetes RBAC, cluster access

After the spec, ask if the user wants to refine it or start implementation.
