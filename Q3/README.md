Contains the research and potential clarifying questions for Email response Question 3

# Question 3
*We want to backup our cloud hosted Git repos to an internally, self-hosted git server, so that we can backup the cloud repos. How would we go about setting up a one way repo sync between the two?*

## Answer:
[Internal Read-Only Mirror of my GitHub cloud Repositories](repo_sync_plan.md)

## Clarifying Questions & Research for One-Way Git Repo Sync

## Top Clarifying Questions
1. **Internal Git target:** What internal system do you want to mirror into? (Bare Git repos, GitLab CE, Bitbucket Server, Gitea, custom git-shell?)  
2. **Scope of repos:** Are we mirroring all repos in the org, a subset, or only critical repos?  
3. **Size & features:** Do any repos exceed several GBs or use Git LFS/submodules?  
4. **Triggering:** Do you prefer sync on every push (near real-time), scheduled syncs (e.g. nightly), or both?  
5. **Network architecture:** Does your internal server allow inbound connections from cloud runners, or must we pull from inside only?  
6. **Security policies:** Are SSH keys or GitHub OIDC tokens acceptable for auth? Any restrictions on outbound traffic or IP allowlisting?  
7. **One-way enforcement:** Should internal repos be strictly read-only (no human pushes)? If so, how do we enforce?  
8. **Compliance & retention:** Do you require immutable history (audit bundles) or is a current mirror enough?  
9. **Monitoring:** Where should mirror job failures be surfaced (email, Slack, SIEM)?  
10. **Disaster recovery:** In case of cloud provider outage, what’s the expected RTO/RPO using the internal backups?  

---

## Research Necessary
To be ready with a customer-facing plan, we should investigate:

- **Git mirroring mechanics:** Difference between `git clone --mirror` vs `git clone --bare` and how `git push --mirror` works  
- **Git LFS handling:** Ensuring large file objects are mirrored (`git lfs fetch --all`, `git lfs push --all`)  
- **Automation options:**  
  - GitHub Actions as push source (security considerations for secrets)  
  - Internal schedulers (cron, Jenkins, other job runners) for pull-based mirroring  
- **Org-wide repo enumeration:** Using `gh api` or GitHub REST/GraphQL API to list all repos programmatically  
- **Network/security patterns:**  
  - SSH deploy keys with limited permissions  
  - OIDC-based federation (avoid long-lived credentials)  
  - Firewall/egress design if Actions runners must reach internal network  
- **Target server capabilities:**  
  - Does the internal Git server support Git LFS?  
  - Can it enforce read-only permissions?  
- **Audit and compliance:** How to optionally export metadata (issues, PRs) via API since Git alone won’t mirror them  
- **Monitoring integrations:** GitHub Action status checks, webhook alerts, or internal logging/alerting  
- **Restore procedures:** Verifying repos can be recreated in GitHub from the internal mirror if needed  

---
