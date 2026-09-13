# Optional AWS Infrastructure

Local processing, training, tests, API, batch analysis and dashboard operation do
not require AWS. These templates are inactive until you explicitly configure and
apply them. They contain no credentials or connection to an existing deployment.

## Safety

- Use your own authorized AWS account, resource prefix, region, credentials and
  isolated Terraform state. Never reuse someone else's state or account settings.
- Review current service pricing and set billing alerts before deployment.
- Even with optional features disabled, `terraform apply` can create billable
  base resources including ALB and ECS. The example is not a no-cost sandbox.
- `scripts/aws/deploy.ps1` performs applies, image push and an ECS update. Do not
  execute it merely to validate syntax or learn command options.
- Destroy only resources tracked by your own verified state. Do not run teardown
  against shared or production infrastructure.

## Local Configuration Validation

From the repository root, with Terraform installed:

```powershell
terraform -chdir=infra/terraform fmt -check -recursive
terraform -chdir=infra/terraform init -backend=false -input=false
terraform -chdir=infra/terraform validate
```

Initialization may download providers, but these commands do not apply resources.
Successful validation does not prove IAM access, quota, service health or costs.

## Before Any Deployment

Review `staging.tfvars.example` and provide your own untracked variable file.
All optional feature flags in this distribution default to false, including
monitoring. Supply a real alert destination only when enabling alerts.
The GitHub repository URL is blank; configure your own URL before opting into
CodeBuild. No source-repository URL, token or webhook is carried over.

Prepare model artifacts and the portable registry using [local setup](../../docs/SETUP.md).
An image must contain the required model/history files. Model-training jobs
also need access to the appropriate features and registry artifacts; local
SQLite metadata is not a shared remote tracking service.

## Build and Runtime Boundaries

The buildspecs are optional templates, not active workflows. If you connect them
to CodeBuild, default-branch builds and manual builds without webhook events can
enter the deployment path. Review the conditions before enabling a trigger.
Never provide the source owner's credentials, buckets or build projects.

The API uses HTTP ALB port 80 to the application port. There is no automatic TLS
setup. CodeBuild requests ECS updates but does not include a stability waiter or
end-to-end smoke test. Verify task stability and the `/health` JSON model flags
separately; a target's HTTP health response alone is insufficient.

Monitoring schedules require their corresponding configuration and image or
pipeline identifiers. The retraining schedule sets `DeployAfterQualityGate=false`.
A successful condition step is not proof that its release branch was selected.
The templates do not connect drift alarms directly to starting retraining.

Managed AWS execution, container execution and current cloud resources must be
validated in your own environment before any production claim.