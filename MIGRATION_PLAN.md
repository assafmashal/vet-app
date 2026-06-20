# Plan: Copy vet-app to your own GitHub account (clean, no history)

## Context

The repo belongs to `vadim123-dev` on GitHub. You want a fully independent copy under `assafmashal` — no fork, no commit history, fresh start. AWS account (257014219799) is already yours.

---

## Steps (manual — copy-paste friendly)

### 1. Create a new empty public repo on GitHub

```bash
gh repo create vet-app --public --description "TeyaVet — Vet Clinic Management App"
```

Or create it manually at https://github.com/new (no README, no .gitignore, no license — completely empty).

### 2. Copy the project files without git history

```bash
# Make a clean copy without .git
cp -r /Users/assafmashal/Desktop/final_project/vet-app /Users/assafmashal/Desktop/final_project/vet-app-clean
rm -rf /Users/assafmashal/Desktop/final_project/vet-app-clean/.git
```

### 3. Update the Terraform GitHub repo reference

Edit `infra/variables.tf` (line ~87) — change:
```
default = "vadim123-dev/vet-app"
```
to:
```
default = "assafmashal/vet-app"
```

This controls the OIDC trust policy. Without this change, GitHub Actions from your repo won't be able to authenticate to your AWS account.

### 4. Initialize a fresh git repo and push

```bash
cd /Users/assafmashal/Desktop/final_project/vet-app-clean
git init
git add .
git commit -m "Initial commit"
git branch -M master
git remote add origin https://github.com/assafmashal/vet-app.git
git push -u origin master
```

### 5. Set up GitHub Actions secrets

Go to your new repo → Settings → Secrets and variables → Actions, and add:

| Secret | Value |
|--------|-------|
| `ECR_BACKEND_URL` | `257014219799.dkr.ecr.us-east-1.amazonaws.com/teyavet/backend` |
| `ECR_FRONTEND_URL` | `257014219799.dkr.ecr.us-east-1.amazonaws.com/teyavet/frontend` |

Check `.github/workflows/` for any other secrets your workflows expect.

### 6. (Optional) Apply Terraform to update OIDC policy

```bash
cd infra
terraform apply
```

This updates the IAM trust policy in AWS to accept OIDC tokens from `assafmashal/vet-app` instead of `vadim123-dev/vet-app`.

---

## Verification

1. Visit `https://github.com/assafmashal/vet-app` — should show all code, single commit, no fork badge
2. Trigger a GitHub Actions workflow to confirm OIDC auth works (after steps 5 + 6)
