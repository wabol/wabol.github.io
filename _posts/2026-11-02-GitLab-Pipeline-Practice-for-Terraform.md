---
title: "GitLab Pipeline Practice for Terraform: Managing AWS Network Resources"
date: 2026-11-02 11:26:00 -0800
categories: [DevOps]
tags: [gitlab-ci, terraform, aws, vault, network]
---

We use Terraform to manage AWS network resources, and GitLab CI to run Terraform. Network changes are risky. One wrong route can break many systems. So the pipeline must be safe first, and fast second.

This note records the practice we use every day: how the pipeline is split, how it gets AWS credentials from Vault, and what safety rules we follow.

*Note: names in this article, such as role names and paths, are simple examples.*

## 1. The basic rule: plan in the merge request, apply by hand

The most important rule is simple:

- A merge request (MR) pipeline only runs `plan`. It never runs `apply`.
- A person reads the plan and approves the MR.
- After the merge, a person starts the `apply` job by hand.

This way, no code can change the real network by itself. There is always a human check before a change.

The apply job also uses the saved plan file from the plan job. We do not run `plan` again at apply time. The plan we read is the plan we apply.

## 2. What each stage does

The pipeline has three stages:

1. **Validate.** Run `terraform fmt -check`, `terraform validate`, and a lint and security scan. If the code style is wrong, or a rule is unsafe, the pipeline stops here.
2. **Plan.** Run `terraform plan` and save the plan file as an artifact. The plan result is also shown in the MR, so reviewers can read the real change, not just the code.
3. **Apply.** Only on the protected default branch, and only by hand. One special GitLab setting, `resource_group`, makes sure only one apply job runs for the same state at the same time.

```yaml
stages: [validate, plan, apply]

plan:
  stage: plan
  script:
    - terraform init
    - terraform plan -out=plan.tfplan
  artifacts:
    paths: [plan.tfplan]
    expire_in: 3 days

apply:
  stage: apply
  resource_group: network-prod
  when: manual
  script:
    - terraform apply plan.tfplan
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

## 3. Credentials: Vault and AppRole on the runner

The pipeline needs AWS credentials, but we do not save long-term AWS keys in GitLab. Long-term keys are hard to rotate, and they are dangerous if they leak.

We use HashiCorp Vault:

1. The GitLab Runner logs in to Vault with **AppRole**. The `role_id` and `secret_id` live on the runner, not in the GitLab project.
2. Vault gives the job a short-lived token.
3. With this token, the job reads dynamic AWS credentials from Vault's AWS secrets engine.
4. The credentials live only for a short time. When the lease ends, they stop working.

So Vault is the one central place for credentials and audit logs. We can see who asked for credentials, and when.

AppRole also means Vault trusts the runner, not each job. So runner groups must be kept separate:

- Runners for production are a separate group. They only run jobs from protected branches.
- Each environment has its own AppRole and its own Vault policy.
- The `secret_id` is a real secret. We rotate it, protect it on the runner, and limit which IP addresses can use it.

## 4. State: small stacks, one state for each

We do not put all network resources in one big Terraform state. A big state means a big plan, and a big risk.

We split the code into small stacks, for example by account, by region, and by network layer. Each stack has its own state, with locking, encryption, and versioning turned on.

A small stack gives a small plan. A small plan is easy to read, and a mistake hurts a smaller area.

We also pin the Terraform version and the provider versions, and we save the lock file in Git. A provider upgrade is its own MR, never a surprise inside a normal change.

## 5. Safety rules for network changes

Some changes need more care than others. Our rules:

- **Dangerous changes need two reviewers.** For example, a plan that deletes resources, replaces resources, or removes routes.
- **Roll out one region at a time.** Do not apply the same network change to all regions at once. Apply to one, check the result, then go to the next.
- **Check drift every night.** A scheduled pipeline runs `plan` with `-detailed-exitcode`. If someone changed a resource by hand in the console, the plan shows a difference, and we get an alert.
- **Run a smoke test after apply.** For example, check DNS and check that key paths are reachable. Do not wait for a user to report a problem.
- **Keep shared pipeline code in one template.** All stacks include the same GitLab CI template, so the rules are the same everywhere. If we fix the template, all stacks get the fix.

## 6. Problems we watch for

- **Vault is a key system.** If Vault is down, pipelines cannot get credentials and cannot run. Vault needs high availability, and its own root credentials need care and rotation.
- **Credential lease time.** The lease must be a little longer than the longest job. Too short, and a job fails in the middle. Too long, and the credentials are not "short-lived" any more.
- **Manual state changes.** Commands like `terraform state mv` do not run in the normal pipeline. They follow a separate, careful process, with a state backup first.

## 7. Summary

The practice is not complex. It is a set of simple rules, used every day:

- MR pipelines only plan. People approve, and people start the apply.
- Apply uses the saved plan file, and only one apply runs for a state at a time.
- No long-term AWS keys in GitLab. The runner uses AppRole to get short-lived credentials from Vault.
- Small stacks, small plans, small risk.
- Check one region at a time, and check drift every night.

Network changes will always be risky. A good pipeline does not remove the risk. It makes each change small, clear, and easy to stop.

---

*Views are my own. Role names, paths, and other examples in this article are illustrative.*
