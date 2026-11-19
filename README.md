# platform-ci-workflows

Reusable GitHub Actions workflows for the platform repos.

| Workflow | Does |
|---|---|
| `terraform-static.yml` | `terraform fmt`, `validate`, optional `terraform test`, `tflint`. No cloud credentials |
| `terraform-plan.yml` | `terraform plan` per stack with a read-only role. Fails on changes outside PRs (drift check) |

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
    uses: maison-douce-bakery/platform-ci-workflows/.github/workflows/terraform-plan.yml@<sha> # terraform-plan/v1.0.0
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
| `terraform_version` | `1.17.1` | Terraform version |

Optional secret `tf_vars`: `name=value` lines, exported as `TF_VAR_name`.
