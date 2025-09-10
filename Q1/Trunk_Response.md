# Technical Interview – Q1 Email-Style Answer

---

**Subject**: Trunk-Based vs Git Flow Development Recommendation  

Hi `<Name>`,

Since you want to standardize on a single branching strategy, based on the information you have given the recommendation would be choosing Trunk-based Development (TBD) as your organization’s default methodology. TBD optimizes for flow, faster development cycles, and lower merge risk because the number of changes within a PR is minimized. I also recommend keeping a lightweight release branch when you need stabilization or hotfixes outside of your normal sprints. As you are already aware, Gitflow can be used for separate projects, but only if you ship infrequent, versioned releases to external customers where parallel maintenance of multiple versions is required.

---

## Why TBD Wins for Most Teams

- Smaller batches and frequent itterations reduce merge debt and rework.  
- CI runs on every change, so defects surface earlier and are cheaper to fix.  
- Simpler branch model improves contributor onboarding and cross-team collaboration.  
- Aligns with modern DevOps practices and DevOps Research and Assessment (DORA) metrics that favor fast lead time and high deployment frequency.  

---

## When Gitflow Can Be Appropriate

- You publish packaged, versioned artifacts on a fixed cadence and must maintain several supported versions in parallel.  
- Releases are batched with planned hardening windows, and regulatory sign-offs happen on a release branch.  
- You accept the operational cost of long-lived branches and more complex merges in exchange for strict release staging.  

---

## Decision Guide

- **Pick TBD** if most of these are true: you deploy often, value fast feedback, have strong automated tests, rely on feature flags for incomplete work, and want fewer merge conflicts.  
- **Consider Gitflow** if most are true: you release quarterly or less, must harden on a release branch, support many customer versions at once, and have change-control gates that cannot run on main.  

_I can provide you with a basic survey that you can utilize to determine if there are any projects or organizations within your enterprise that would be better served with a Git Flow strategy._

---

## Recommended Model (TBD with Lightweight Releases)

1. Developers branch from `main` for short-lived feature branches.  
2. Open small PRs, target < 1–3 days of work.  
3. Protect `main` with required checks: status checks green, code review, conversation resolved, linear history, and a merge queue.  
4. Use feature flags to decouple deploy from release.  
5. Cut a release branch only when you enter stabilization. Cherry-pick critical fixes to it. Tag releases from the release branch or from `main` if you do continuous delivery.  
6. Keep release branches short-lived. Do not maintain permanent `develop` or long-running feature branches.  

---

## How to Set This Up on GitHub

- **Branch protection:** require PRs, required reviewers, required status checks, and signed commits if needed.  
- **Merge queue:** enable to serialize merges and keep `main` green under load.  
- **CODEOWNERS:** route reviews automatically.  
- **Environments:** set environment-specific approvals and secrets for staging and production.  
- **Actions:** run test, security, and quality gates on every PR to `main`.  
- **Release management:** use Releases and tags. For products with SLAs on old versions, keep a short list of supported maintenance branches with a clear end-of-support policy.  

---

## Migration from Gitflow to TBD (Incremental and Low Risk)

1. Freeze creation of new long-lived branches. New work branches from `main` only.  
2. Introduce feature flags to avoid long-running work on branches.  
3. Tighten CI gates and add a merge queue on `main`.  
4. Pilot on 1–2 services, measure lead time, change failure rate, and time to restore.  
5. Adopt a simple release branch for hardening, then retire it promptly after GA.  
6. Document the policy in `CONTRIBUTING.md` and enforce with repo rules.  

---

## Common Concerns

- **“We need stabilization.”** Use a short-lived release branch with cherry-picks and keep `main` moving.  
- **“We have big features.”** Slice into vertical increments and hide incomplete paths behind flags.  
- **“We must support old customers.”** Maintain only N active release branches with clear EOL dates.  
- **“Compliance requires approvals.”** Use required reviewers, environment approvals, and immutable tags.  

---

### Additional Recommendations to consider

- **Process documentation:** Document branching expectations, CI/CD requirements, and release procedures in `CONTRIBUTING.md`. This ensures new contributors follow the model consistently.  
- **Workflow templates:** Provide reusable GitHub Actions workflows (build, test, lint, security scan) as templates for all repos.  
- **Reusable workflows:** Centralize common pipelines in a single repo so teams inherit best practices automatically.  
- **Observability & quality:** Require tests, security scans, and lint checks as part of the Actions workflow, with results surfaced in PRs.  

---

## Bottom Line

Making Trunk-based Development (TBD) the standard, with a flexible option for short-lived release branches. This gives Team B no change, and gives Team A a simpler, faster model with better quality signals.

I would be happy to meet with you to discuss these points in detail, answer any questions you may have, and help create a project plan template for this.

---

Best,  
Jacy York  
Customer Solutions Architect  
jacy.york@github.com   

<img src="../Preso/assets/GitHub_logo.png" alt="drawing" width="100"/>

