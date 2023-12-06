---
tags: 📥/📓/🟧
aliases:
  - 
cssclass:
type: learning-journal
status: 🟧
---

## Metadata
- `Title:` [[Terraform]]
- `Type:` [[!]]
- `Tags:`
- `Formation Date:` [[2023-06-14]]

---

## Workflow

```bash
# Initialize Terraform - Downloads necessary provider plugins and sets up backend configuration
terraform init

# Plan the changes
# Analyzes configuration and displays proposed changes w/out actually making modifications
terraform plan

# Apply the changes (USE WITH CAUTION - As this can have real-world effects)
terraform apply
```


## CI (Github Actions) Commit Hash Issues

- If you have a problem with Docker commit hashes not matching when a CI Docker build job is run:
	- **A:** Change the `publish` flag to `true` in the terraform file for the lambda. Then, make a no-op change to a file that is being used in that lambda. ==Performing a no-op change to something like the README.md will not work!== After this, the CI docker test build step might still show that the commit hashes do not match. This is fine. Merge the PR and watch the main "Actions" tab and click on the Docker build job. You should see the commit hashes as equal and will begin building. 🎉


🔗 Links to this page:

