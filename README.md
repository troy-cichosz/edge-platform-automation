# edge-platform-automation

Automation control plane for the AI Legal Platform.

GitHub is the authoritative source for automation code. The `chatgpt` branch is the development branch used during implementation and vetting. Azure DevOps Server consumes the vetted automation repository to orchestrate the existing service CI/CD system.

## Current workflow

```
GitHub service/chatgpt
        |
        v
GitHub webhook
        |
        v
edge-platform-automation - CI
        |
        +--> identify repository/ref/SHA
        +--> retrieve GitHub chatgpt source
        +--> synchronize matching ADO service/chatgpt
        +--> create ADO synchronization commit
        +--> push ADO chatgpt
        |
        v
existing edge-<service> - CI/CD
        |
        +--> existing build / scan / registry / deployment
        +--> existing release behavior
        |
        v
GitHub public
```

GitHub service `chatgpt` is authoritative for source and history.

ADO service `chatgpt` is an operational build-triggering mirror. Its commit history does not need to match GitHub. Synchronization replaces the ADO working tree with the GitHub source tree so files deleted in GitHub are also absent in ADO.

The existing service pipelines are intentionally not modified by this automation increment.

## Proven webhook behavior

The GitHub-to-ADO webhook, repository identification, `chatgpt` branch validation, SSH authentication, strict GitHub host-key verification, and exact GitHub SHA checkout/verification have been proven with the `edge-gps` pilot.

The current implementation adds the ADO service-repository synchronization step.

## Synchronization responsibility

This repository orchestrates source movement only.

It does not:

- build service Docker images;
- replace the existing service CI pipelines;
- deploy service containers directly;
- modify GitHub service `public` branches.

It does:

- identify the affected service;
- retrieve its GitHub `chatgpt` source;
- synchronize the source tree into the matching ADO service repository;
- create the ADO synchronization commit;
- push the ADO `chatgpt` branch;
- record the GitHub revision and resulting ADO revision for correlation.

## Security

- Webhook authentication uses the dedicated ADO Incoming Webhook secret/HMAC configuration.
- GitHub source access uses the existing ADO secure-file SSH key mechanism.
- GitHub host verification remains enabled with strict host-key checking.
- ADO repository writes use the pipeline OAuth/System.AccessToken and Build Service repository permissions.
- Secrets and credentials must never be committed to this repository.
- OAuth tokens and other credentials must never be echoed to logs.

## Promotion

Public promotion is not an automation-repository responsibility.

The existing service CI/CD may continue its current `public` behavior while the migration is being verified. A later increment will define and test the long-term promotion mechanism.

## Next increment

**Increment B — GitHub → ADO service-repository synchronization**

Pilot: `edge-gps`.

Acceptance:

- GitHub `edge-gps/chatgpt` push reaches automation.
- Automation pulls the GitHub source.
- Matching ADO `edge-gps/chatgpt` is updated.
- Deleted GitHub files are removed from ADO.
- ADO synchronization commit is pushed.
- Existing `edge-gps - CI` triggers from the ADO branch change.
- Existing service CI/CD remains unchanged.
