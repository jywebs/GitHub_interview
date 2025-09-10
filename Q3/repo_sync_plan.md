# Technical Interview – Question 3 Email style answer

---
Subject: Internal Read-Only Mirror of my GitHub cloud Repositories 

Hi `<Name>`,

As I understand it the goal of this project is to create a read-only mirror on-prem for backup, DR, and compliance.
Below I am outlining two practical approachs to create a reliable, one-way sync from your cloud-hosted Git provider (assume GitHub.com) to an internally hosted Git server. You can adopt either, or even run both for belt-and-suspenders resiliency.

---

## Goals

- **One-way only:** cloud is the source of truth, internal is a passive mirror  
- **Complete fidelity:** branches, tags, full history, orphaned refs, and Git LFS objects  
- **Automated and monitored:** runs on commit and on a schedule, with alerting  
- **Least privilege and network-safe:** no writes from internal back to cloud

---

## Option A: Push-based mirroring with GitHub Actions

Each repo, or a centralized “backup” repo, runs a workflow that pushes a mirror to the internal server over SSH.

### Highlights

- Triggers on every push and can be setup for a nightly on a cron (as shown in below example)  
- No inbound firewall exceptions to your network. Outbound only from GitHub runners to your internal bastion or Git endpoint  
- Works with any internal Git target hosting bare repos  

### Prereqs

- Create a dedicated read-only service account on the internal Git server with SSH access and write permission only to the target bare repos  
- Generate an SSH key pair for GitHub Actions. Store the private key in the source repo or the central “backup” repo as an Actions secret  
- **Important caveat:** Precreate a matching bare repo internally for each mirrored repo:  

  ```bash
  mkdir -p /srv/git/<org>/<repo>.git && git init --bare
  ```

  Without this, the push will fail. If new repos are added in the cloud, they must also be provisioned internally before the mirror can succeed.

  **Automation workaround**  
  - Add a step in the workflow (or in a central “backup-ops” job) to check and create the bare repo dynamically:  
  
  ```bash
  ssh git-backup@internal "mkdir -p /srv/git/<org>/<repo>.git && git init --bare /srv/git/<org>/<repo>.git || true"
  ```  

  - Or maintain a central “backup-ops” repo that enumerates all repos in the org and ensures the target exists before pushing. This eliminates the manual bootstrap burden and automatically picks up new repos.
  (I can provide more on this if you are interested.)

### Example GitHub Action

```yaml
name: Mirror to internal git
on:
  push:
  schedule:
    - cron: "23 3 * * *"   # nightly catch-up
jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Harden runner
        run: |
          git config --global gc.auto 0
          git config --global advice.detachedHead false
      - name: Checkout as a bare mirror
        run: |
          git clone --mirror "${GITHUB_SERVER_URL}/${GITHUB_REPOSITORY}.git" repo.git
      - name: Setup SSH
        env:
          SSH_KEY: ${{ secrets.INTERNAL_GIT_DEPLOY_KEY }}
        run: |
          install -m 700 -d ~/.ssh
          printf "%s" "$SSH_KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          echo "$INTERNAL_GIT_KNOWN_HOSTS" >> ~/.ssh/known_hosts
      - name: Ensure target exists
        env:
          REPO_NAME: ${{ github.event.repository.name }}
        run: |
          ssh git-backup@internal "mkdir -p /srv/git/${{ github.repository_owner }}/$REPO_NAME.git && git init --bare /srv/git/${{ github.repository_owner }}/$REPO_NAME.git || true"
      - name: Mirror push
        env:
          INTERNAL_URL: "git@internal-git.example.com:<org>/<repo>.git"
        run: |
          cd repo.git
          git remote set-url --push origin "$INTERNAL_URL"
          git push --mirror
      - name: Mirror Git LFS
        if: always()
        run: |
          cd repo.git
          git lfs install --local || true
          git lfs fetch --all || true
          git lfs push --all "$INTERNAL_URL" || true
```

### Org-wide variant

- Create a single “backup-ops” repo with a workflow that enumerates all repos using `gh api` and mirrors each one in a loop.
  **example:**

```bash
gh api /orgs/my-org/repos?per_page=100 --paginate --jq '.[].ssh_url'
```

---

## Option B: Pull-based mirroring from inside your network

A scheduled job on an internal host pulls from cloud and updates the local bare repos.

## Highlights

- Centralized control within your internal environment  
- Works even if Actions are disabled in some repos  
- Network is outbound from your network to cloud  

## Prereqs

- A hardened Linux host with Git and Git LFS installed  
- A fine-grained PAT, read-only GitHub token or a deploy key to permit cloning private repos  
- A repo inventory list or an org-level API call to enumerate repos (see example)
  
  ```bash
  gh api /orgs/my-org/repos?per_page=100 --paginate --jq '.[].ssh_url'
  ```

### Bootstrap script

```bash
#!/usr/bin/env bash
set -euo pipefail

ORG="my-org"
INTERNAL_BASE="/srv/git/$ORG"
CLOUD_BASE="git@github.com:$ORG"
GITHUB_TOKEN="${GITHUB_TOKEN:?set a token or use ssh-agent}"

mirror_one() {
  local name="$1"
  local internal="$INTERNAL_BASE/$name.git"
  mkdir -p "$(dirname "$internal")"
  if [[ ! -d "$internal" ]]; then
    git init --bare "$internal"
  fi

  CLOUD_URL="https://$GITHUB_TOKEN@github.com/$ORG/$name.git"

  TMPDIR="$(mktemp -d)"
  trap 'rm -rf "$TMPDIR"' EXIT
  git clone --mirror "$CLOUD_URL" "$TMPDIR/repo.git"
  cd "$TMPDIR/repo.git"
  git lfs fetch --all || true
  git push --mirror "file://$internal"
}

for repo in service-a service-b infra-terraform; do
  mirror_one "$repo"
done
```

---

## One-way controls and safety

- Mark internal mirrors **read-only**  
- Do not configure any remotes from internal back to cloud  
- Use deploy keys or service accounts with **least privilege**  

---

## What gets mirrored

- `git push --mirror` transfers **all refs**: branches, tags, notes  
- Git LFS objects via `git lfs fetch --all` and `git lfs push --all`  
- **Not mirrored:** pull requests, issues, metadata (export separately via API if needed)  

---

## Monitoring, alerting, and runbook

- Emit success/failure logs  
- Add Slack/email alerts on workflow failure  
- Canary job: verify random mirrored repo commit hash matches cloud  
- Quarterly integrity checks with `git fsck`  
- Document restore drill procedures  

---

## Edge cases and gotchas

- **Force pushes/history rewrites:** mirrored as-is. For audit, add immutable bundles to WORM storage  
- **Submodules:** ensure URLs are reachable internally  
- **Large monorepos:** use `git repack` for efficiency  
- **Repo renames/transfers:** centralize inventory  

---

## Minimal pilot plan

1. Pick 3 representative repos (small, large, with LFS)  
2. Stand up an internal bare Git target or Gitea in non-prod  
3. Implement Option A centrally with deploy key  
4. Add Slack alerts and nightly schedule  
5. Validate with canary check + restore drill  
6. Expand org-wide  

---

## Security notes

- Rotate keys on a regular basis (recommend at least quarterly)  
- Store secrets in GitHub OIDC or Actions secrets  
- Pin host keys to prevent MITM  

---

Happy to help set up a short pilot this week, then when you are happy with the results you can roll it out to the full org.  
I would be glad to meet to discuss details, answer any questions, and help create the project plan and runbook for this migration.

Best,  
Jacy York  
Customer Solutions Architect  
jacy.york@github.com  

<img src="../Preso/assets/GitHub_logo.png" alt="drawing" width="100"/>

---

## Nice to have before starting

1. Which internal Git target will you use (bare, Gitea, GitLab, Bitbucket Server)?  
2. Do any repos exceed 5 GB or use Git LFS heavily?  
3. Central “backup-ops” repo vs. per-repo workflows?  
4. Where should alerts go when a mirror run fails?  
5. Any strict egress policies / bastion requirements?  
