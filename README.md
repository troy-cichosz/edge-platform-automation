# edge-platform-automation

Automation control plane for the AI Legal Platform.

GitHub is the authoritative source for automation code. The `chatgpt` branch is the development branch used during implementation and vetting. Azure DevOps Server consumes the vetted automation repository to orchestrate source synchronization and the existing service CI/CD system.

## Current workflow

```
GitHub repository/chatgpt
        |
        v
GitHub webhook
        |
        v
edge-platform-automation - CI
        |
        +--> identify repository/ref/SHA
        +--> retrieve exact GitHub chatgpt source
        +--> synchronize matching ADO repository/chatgpt
        +--> create ADO synchronization commit
        |
        +--> build_service: existing service CI/CD
        |
        +--> sync_only: sync-only public-maintenance pipeline
        |
        v
GitHub public
```

GitHub `chatgpt` is authoritative for source and history. ADO `chatgpt` branches are operational mirrors.

## Repository classes

The automation registry covers all eight current project repositories:

| Repository | Class | Downstream behavior |
|---|---|---|
| `edge-controller` | `build_service` | Existing service CI/CD |
| `edge-time` | `build_service` | Existing service CI/CD |
| `edge-gps` | `build_service` | Existing service CI/CD |
| `edge-video` | `build_service` | Existing service CI/CD |
| `edge-audio` | `build_service` | Existing service CI/CD |
| `ai-legal-platform-development` | `sync_only` | Public-maintenance pipeline |
| `edge-platform-automation` | `sync_only` | Public-maintenance pipeline |
| `edge-ai` | `sync_only` | Public-maintenance pipeline |

Synchronization is complete-tree: files deleted from GitHub `chatgpt` are removed from the ADO mirror.

## Proven synchronization behavior

The GitHub-to-ADO webhook, repository identification, `chatgpt` branch validation, SSH authentication, strict GitHub host-key verification, exact GitHub SHA checkout/verification, ADO synchronization commit, and remote ADO SHA verification were proven with the `edge-gps` pilot.

Increment C extends that mechanism to the full repository registry.

Build-service synchronization now has a preflight check for the required existing `azure-pipelines.yaml` before complete-tree replacement. This protects the existing service CI trigger definition from accidental deletion.

Non-`chatgpt` GitHub webhook events remain harmless no-ops.

## Synchronization responsibility

This repository orchestrates source movement and repository classification.

It does not:

- build service Docker images;
- replace existing service CI pipelines;
- deploy service containers directly;
- push GitHub `public` directly.

For `build_service` repositories, the synchronized ADO `chatgpt` branch continues into the existing service CI/CD.

For `sync_only` repositories, the synchronized ADO `chatgpt` branch triggers the repository's `azure-pipelines.yaml` public-maintenance pipeline. That downstream pipeline performs the existing-style ADO-to-GitHub `public` maintenance operation.

The `edge-platform-automation` public push can generate another GitHub webhook event, but the automation pipeline accepts only `chatgpt` events. This is the explicit recursion boundary.

## Security

- Webhook authentication uses the dedicated ADO Incoming Webhook secret/HMAC configuration.
- GitHub source access uses the existing ADO secure-file SSH key mechanism.
- GitHub host verification remains enabled with strict host-key checking.
- ADO repository writes use the pipeline OAuth/System.AccessToken and Build Service repository permissions.
- Secrets and credentials must never be committed to this repository.
- OAuth tokens and other credentials must never be echoed to logs.

## Current increment

**Increment C - Expand and harden GitHub -> ADO synchronization**

The expanded implementation has been validated across the original seven covered repositories. `edge-ai` is now registered as the eighth repository and will follow the same GitHub `chatgpt` -> ADO `chatgpt` synchronization boundary.

Validated behavior:

1. All seven repositories are registered with the intended `build_service` or `sync_only` classification.
2. Exact GitHub `chatgpt` SHA synchronization was validated across the full registry.
3. Idempotent synchronization was validated; unchanged source does not create unnecessary ADO commits.
4. Complete-tree deletion propagation was validated.
5. Non-`chatgpt` GitHub webhook events were validated as harmless no-ops.
6. The `edge-platform-automation` recursion boundary was validated; `public` events are not accepted by the `chatgpt)-only synchronization pipeline.
7. GitHub SHA -> ADO synchronization SHA correlation was validated.
8. A controlled `edge-audio` `chatgpt` change verified the changed-source downstream path through ADO synchronization, existing ADO CI/CD, and GitHub `public` maintenance.

The synchronization control plane is validated for the original seven-repository registry. The `edge-ai` registration is a configuration extension and requires a controlled webhook/ADO validation after its ADO `chatgpt` branch is populated. Existing service CI/CD definitions remain unchanged.

Future work is tracked in the later development-process phases.
