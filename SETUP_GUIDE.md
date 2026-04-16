# Vault setup (CloudVault and Cursor Cloud Agents)

This repository holds the knowledge vault. Inbound chat (Telegram or local CLI via CloudVault) starts a **Cursor Cloud Agent** run against this GitHub repo. The agent reads and updates Markdown under the layout in `life-admin-knowledge-spec.md` and `AGENTS.md`.

## Connect Git and Cloud Agents

In [Cursor](https://cursor.com): **Dashboard → Cloud Agents → Connect Git**, and grant access to this repository so agents can clone, commit, and push.

## Pull requests from agents

CloudVault starts agents with **`target.autoCreatePr` enabled by default** (see CloudVault `.env.cloudvault` / Terraform). That asks Cursor to open a pull request when the run completes. If that API flag is off, you may get commits on a branch without a PR, and the auto-merge workflow below does not apply.

Cloud Agents typically work on a branch whose name starts with `cursor/`, then open a pull request into the default branch. Enable **Allow auto-merge** for this repo: **Settings → General → Pull requests → Allow auto-merge**.

### Automatic enablement of auto-merge

Workflow [`.github/workflows/enable-auto-merge.yml`](.github/workflows/enable-auto-merge.yml) runs when a pull request is opened, reopened, or marked ready for review. It enables **squash** auto-merge only when the head branch name starts with `cursor/`, so other pull requests are unchanged.

After auto-merge is enabled on a PR, GitHub merges it when required checks and branch protection rules pass. If nothing runs on the PR, merge can proceed as soon as the branch is mergeable.

### Optional: stricter rules

Use **Settings → Branches** to add protection on `main` (required reviews, required status checks, etc.). Auto-merge waits until those requirements are satisfied.

### If auto-merge does not appear

Confirm **Allow auto-merge** is on, the workflow has run successfully on the PR, and the repository allows the merge strategy you use (squash must be allowed under **Settings → General → Pull requests** if the workflow uses `--squash`).
