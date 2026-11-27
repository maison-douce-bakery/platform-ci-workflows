# platform-ci-workflows

Reusable GitHub Actions workflows for the platform repos.

| Workflow | Does |
|---|---|
| `terraform-static.yml` | `terraform fmt`, `validate`, optional `terraform test`, `tflint`. No cloud credentials |
| `terraform-plan.yml` | `terraform plan` per stack with a read-only role. Fails on changes outside PRs (drift check) |
| `docker-build-push.yml` | Build a container image on a native arm64 runner and push it to ECR, tagged with the commit SHA |

## Usage

Pin the commit SHA of a release tag. The check shows up as `terraform / static checks`.

```yaml
jobs:
  terraform:
    uses: maison-douce-bakery/platform-ci-workflows/.github/workflows/terraform-static.yml@<sha> # terraform-static/v1.0.0
    with:
      directories: modules/*/
      run_tests: true
```

| Input | Default | Description |
|---|---|---|
| `directories` | `.` | Space-separated directories or globs to validate |
| `run_tests` | `false` | Run `terraform test` in each directory |
| `terraform_version` | `1.17.1` | Terraform version |

The caller needs a `.tflint.hcl` at its root.

### terraform-plan

Checks show up as `plan / plan (<stack>)`.

```yaml
jobs:
  plan:
    uses: maison-douce-bakery/platform-ci-workflows/.github/workflows/terraform-plan.yml@<sha> # terraform-plan/v1.1.0
    with:
      stacks: '["bootstrap", "audit"]'
      role_arn: ${{ vars.AWS_CI_PLAN_ROLE_ARN }}
      aws_region: ${{ vars.AWS_REGION }}
    secrets:
      aws_access_key_id: ${{ secrets.AWS_CI_PLAN_ACCESS_KEY_ID }}
      aws_secret_access_key: ${{ secrets.AWS_CI_PLAN_SECRET_ACCESS_KEY }}
```

| Input | Default | Description |
|---|---|---|
| `stacks` | | JSON array of stack directories |
| `role_arn` | | Role to assume |
| `aws_region` | | AWS region |
| `backend_config` | `""` | Partial backend config file, relative to the repo root |
| `changed_stacks` | `""` | JSON array of stacks to plan; the others finish green without planning. Empty plans all |
| `terraform_version` | `1.17.1` | Terraform version |

Optional secret `tf_vars`: `name=value` lines, exported as `TF_VAR_name`.

### docker-build-push

Checks show up as `image / push`. The caller needs the push credentials; only `main` should push.

```yaml
jobs:
  image:
    if: github.event_name == 'push'
    uses: maison-douce-bakery/platform-ci-workflows/.github/workflows/docker-build-push.yml@<sha> # docker-build-push/v1.0.0
    with:
      ecr_repository: mdb/commerce-api
      role_arn: ${{ vars.AWS_CI_PUSH_ROLE_ARN }}
      aws_region: ${{ vars.AWS_REGION }}
    secrets:
      aws_access_key_id: ${{ secrets.AWS_CI_PUSH_ACCESS_KEY_ID }}
      aws_secret_access_key: ${{ secrets.AWS_CI_PUSH_SECRET_ACCESS_KEY }}
```

| Input | Default | Description |
|---|---|---|
| `ecr_repository` | | Repository name in ECR |
| `role_arn` | | Role to assume |
| `aws_region` | | AWS region of the registry |
| `context` | `.` | Build context |
| `dockerfile` | `Dockerfile` | Dockerfile path |

Outputs `digest` and `image` (repository plus digest). The image is tagged with the commit SHA and built
natively on `ubuntu-24.04-arm`. Re-running a commit whose image is already in the registry reports that image
instead of failing on the immutable tag, so the job is safe to re-run and to dispatch by hand.
