Contains the research and potential clarifying questions for Email response Question 1

# Question 1:
We’re trying to standardize our branching strategies across teams. Team A is currently doing Git flow while Team B is doing Trunk-based Development. Which one is better? Which one should we pick?

## Answer:
[Trunk-Based vs Git Flow Development Recommendation](Trunk_Response.md)

# Clarifying Questions:

## Release cadence & delivery

- How often do your teams aim to release to production (daily, weekly, monthly, quarterly)?

- Do you need to support multiple versions of the product in production at the same time? For how long?

- Are there contractual or regulatory requirements that force you to harden/stage code before release?

## Team workflow & practices

- How comfortable are teams with feature flags or toggles to manage incomplete work?

- How mature is your automated test coverage — can you reliably trust green builds to mean “safe to deploy”?

- Do teams practice CI (every commit to main is tested and integrated) or rely on longer QA cycles?

## Risk & compliance

- Are there strict compliance checks or audits that require staged sign-offs before code reaches production?

- How do you currently manage hotfixes — do you need an isolated hotfix branch, or can urgent fixes land directly in main?

## Organizational scale & coordination

- How many developers typically contribute to a repo at the same time?

- Do multiple teams work in parallel on the same product, or are teams mostly siloed to their own services?

- Is onboarding to the current branching model a barrier for new contributors?

## Tooling & enforcement

- Do you already have GitHub branch protections, merge queues, and CI/CD pipelines in place?

- Do you have a standardized approach to tagging releases and managing release notes?

---

Optional Survey:

# Branching Strategy Input Survey

## Github Issues Form

### Branching Strategy Survey

We’re collecting input on which branching model fits our teams best.  
👉 [Fill out the Branching Strategy Survey](https://github.com/jywebs/GitHub_interview/issues/new?template=branching-survey.yml)

--- 
## MarkDown Version
We are evaluating whether **Trunk-Based Development (TBD)** or **Gitflow** is the best standard branching model for our teams.  
Please take a few minutes to answer these questions based on your team’s current practices and needs.

---

## Release Cadence & Delivery
1. How often does your team typically release to production?
   - [ ] Daily
   - [ ] Weekly
   - [ ] Monthly
   - [ ] Quarterly or less
2. Do you need to support multiple versions of the product in production at the same time?
   - [ ] Yes  
   - [ ] No  
   - If yes, for how long do you support each version? (e.g., 3 months, 1 year, etc.)
3. Do you require a separate hardening/stabilization period before release?  
   - [ ] Yes  
   - [ ] No  

## Team Workflow & Practices
4. Does your team use feature flags/toggles for incomplete work?  
   - [ ] Yes, regularly  
   - [ ] Occasionally  
   - [ ] Not at all  
5. How confident are you in your automated test coverage?  
   - [ ] Very confident – green build = releasable  
   - [ ] Somewhat confident – still need manual checks  
   - [ ] Low confidence – rely heavily on manual QA  

## Risk & Compliance
6. Are there compliance or audit requirements that mandate staged approvals before release?  
   - [ ] Yes  
   - [ ] No  
7. How do you currently handle hotfixes?
   - [ ] Hotfix branch
   - [ ] Directly to main
   - [ ] Other (please describe)

## Organizational Scale & Coordination
8. How many developers typically commit to the same repo at once?  
   - [ ] 1–5  
   - [ ] 6–15  
   - [ ] 16+  
9. Do multiple teams contribute to the same codebase simultaneously?  
   - [ ] Yes  
   - [ ] No  

## Tooling & Enforcement
10. Does your repo already enforce branch protections (e.g., required PRs, checks, merge queue)?  
    - [ ] Yes  
    - [ ] No  
11. Do you have a standardized process for tagging releases and writing release notes?  
    - [ ] Yes  
    - [ ] No  

---

### Final Thoughts
12. What do you see as the biggest challenge with your current branching model?  
13. What would you want to preserve if we switch models?  
14. Any other feedback we should consider?

---

Thank you for your input! Your responses will help us decide on a standard branching approach that balances speed, quality, and control.
