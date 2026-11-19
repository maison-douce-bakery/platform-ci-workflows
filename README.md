# platform-ci-workflows

Reusable GitHub Actions workflows for the platform repos.

| Workflow | Does |
|---|---|
| `terraform-static.yml` | `terraform fmt`, `validate`, optional `terraform test`, `tflint`. No cloud credentials |

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
