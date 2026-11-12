# kestrel

A chat assistant with a personality: a web chat UI, an API that calls a local LLM (Ollama on the GPU), and conversation memory stored in PostgreSQL.

Around it sits a full delivery platform: one image promoted by digest through Dev → QA → Stage → Prod with GitOps, supply-chain security (SBOMs, signing, admission policy), observability with an SLO, and canary releases with automatic rollback.

Status: Week 1 — foundation.

## Architecture (target, built over 8 weeks)

```mermaid
flowchart LR
  dev[Developer<br/>WSL2 Ubuntu] -->|git push| ci[GitHub Actions<br/>CI]
  ci -->|signed image| ghcr[(GHCR)]
  ci -->|digest PR| repo[(Git repo<br/>deploy/overlays)]

  subgraph nonprod ["kind cluster: nonprod"]
    argo[Argo CD]
    subgraph envs ["dev · qa · stage namespaces"]
      fe[Nginx frontend] --> be[FastAPI backend] --> db[(PostgreSQL)]
    end
  end

  subgraph prod ["kind cluster: prod"]
    fe2[Nginx frontend] --> be2[FastAPI backend] --> db2[(PostgreSQL)]
  end

  repo -->|pull| argo
  argo -->|sync| envs
  argo -->|sync| prod
  ghcr -->|pull image| envs
  ghcr -->|pull image| prod
  be -->|HTTP 11434| ollama[Ollama on Windows<br/>GPU]
  be2 -->|HTTP 11434| ollama
```
