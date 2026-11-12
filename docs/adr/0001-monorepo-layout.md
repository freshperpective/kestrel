# ADR-0001: One monorepo with a fixed layout

Status: Accepted
Date: 2026-11-12

## Context

The project has four kinds of change: app code, infrastructure (OpenTofu), deployment config (Kustomize overlays) and docs. Most features touch several at once; a new API endpoint changes app/backend, deploy/base and a runbook. Separate repos would split one change across several PRs and make "rebuild from git" depend on several repos.

## Decision

One repository with a fixed layout: app/, infra/, deploy/, .github/workflows, scripts/, docs/, plus a gitignored sandbox/.

infra/clusters/* creates the clusters themselves; infra/envs/* creates what lives inside them (namespaces, quotas). Each has its own state, so breaking or destroying one environment's state can never touch the cluster the other environments run on.

## Consequences

- Benefit: one PR carries a whole change, and one clone plus scripts/bootstrap.sh rebuilds everything.
- Cost: CI needs path filters, or every docs edit rebuilds images.
- Cost: one repo is one permission boundary: anyone who can merge to main can also change deploy/overlays/prod. Least privilege needs CODEOWNERS with required code-owner review on that path, or a separate deploy repo.
