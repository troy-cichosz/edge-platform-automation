# edge-platform-automation

Automation control plane for the AI Legal Platform.

GitHub is the authoritative source for automation code. The `chatgpt` branch is the development branch used during implementation and vetting. Azure DevOps Server consumes the vetted automation repository for CI/CD execution.

## Source repositories

- troy-cichosz/edge-controller
- troy-cichosz/edge-time
- troy-cichosz/edge-gps
- troy-cichosz/edge-video
- troy-cichosz/edge-audio

## Current workflow

GitHub edge-service/chatgpt push -> GitHub webhook -> Azure DevOps Incoming Webhook -> identify repository/ref/SHA -> validate chatgpt -> SSH checkout -> exact SHA verification -> source validation -> build/scan/publish -> deployment/runtime verification -> user-controlled public promotion.

The GitHub-to-ADO webhook, repository identification, exact revision checkout, SSH authentication, strict GitHub host-key verification, and exact SHA verification have been proven with the edge-gps pilot.

## Repository layout

- `config/services.yaml` — supported edge-service repositories and development branches.
- `pipelines/github-chatgpt.yaml` — ADO automation pipeline.
- `README.md` — automation architecture and operating rules.

## Promotion

This repository does not automatically modify the `public` branches of the edge-service repositories. Public promotion remains a user-controlled action after validation, build, deployment, and runtime/integration verification.

## Security

- Webhook authentication uses the dedicated ADO Incoming Webhook secret/HMAC configuration.
- GitHub source access uses the existing ADO secure-file SSH key mechanism.
- GitHub host verification remains enabled with strict host-key checking.
- Secrets and credentials must never be committed to this repository.

## Next increment

The next implementation increment adds a deterministic source-validation contract to the verified checkout stage. Docker builds, registry publication, deployment, and public promotion remain separate later increments.
