<p align="center">
  <a href="https://vercel.com">
    <img src="https://assets.vercel.com/image/upload/v1588805858/repositories/vercel/logo.png" height="96">
    <h3 align="center">Vercel</h3>
  </a>
</p>

# SignaSure MVP – *Fork‑and‑Prune* Blueprint

We’re bootstrapping the SignaSure monorepo by **forking Vercel’s open‑source mono‑workspace** and trimming it down to just what we need.  This preserves all the heavy‑duty plumbing (Turbo, pnpm, Husky, Vitest, CI templates) while giving us a clean canvas for the SignaSure web/app/API stack.

---

## 📁 High‑Level Folder / File Tree

```text
signasure/                     # 👉 forked from github.com/vercel/vercel
├─ apps/                       # Our runnable products
│  ├─ web/                     # Next.js 15 front‑end (from create‑next‑app)
│  │  ├─ src/
│  │  ├─ public/
│  │  ├─ Dockerfile
│  │  └─ package.json
│  └─ api/                     # FastAPI service (REST + webhooks)
│     ├─ src/
│     ├─ tests/
│     ├─ Dockerfile
│     └─ pyproject.toml
│
├─ packages/                   # KEEP the Vercel utility libs we still use
│  └─ lint‑config/             # ESLint + Prettier presets reused across apps
│
├─ infra/                      # IaC + deployment
│  ├─ terraform/               # VPC, ECS, RDS, S3, ACM PCA, etc.
│  └─ github‑actions/          # CI/CD workflows (build, test, deploy)
│
├─ docs/                       # ADRs, diagrams, OpenAPI spec
│
├─ .env.example                # Canonical list of env vars
├─ docker‑compose.yml          # Local dev stack (web + api + db)
├─ turbo.json                  # Updated Turbo pipeline
├─ pnpm‑workspace.yaml         # 🔥 edited to include only `apps/*`, `packages/*`, `infra/*`
└─ README.md                   # ← this file
```

> **Removed** from the upstream Vercel repo: `examples/`, `errors/`, most internal builder `packages/`, legacy scripts.  Upstream CI
> remains but is narrowed to only lint/test/build SignaSure packages.

---

## 🚀 Quick‑Start (Local Dev)

```bash
# 1. clone the fork
git clone git@github.com:8om8/signasure.git && cd signasure

# 2. install workspace deps
corepack enable               # ensures pnpm@10
pnpm install

# 3. spin up everything
cp .env.example .env
pnpm turbo run dev            # web:3000  |  api:8000
```

*(The **dev** task calls `next dev` for apps/web and `uvicorn` reload for apps/api in parallel.)*

---

## 🛠️ Build & Deploy Commands

| Action        | Turbo / pnpm script                          |
| ------------- | -------------------------------------------- |
| Lint all pkgs | `pnpm lint`                                  |
| Unit tests    | `pnpm test`                                  |
| Build web     | `pnpm --filter apps/web run build`           |
| Build api img | `pnpm --filter apps/api run build`           |
| Compose prod  | `docker compose -f docker‑compose.yml up -d` |

### CI Matrix (GitHub Actions)

1. **lint‑test** ➜ runs on every PR.
2. **build‑web**  ➜ pushes to Vercel via `vercel pull && vercel --prod`.
3. **build‑api**  ➜ docker‑build + push to ECR, then `terraform apply` to ECS.

---

## 🔐 Environment Variables

See `.env.example` for the full list.  Core ones:

| Var                   | Purpose                                    |
| --------------------- | ------------------------------------------ |
| `DATABASE_URL`        | Postgres DSN for FastAPI                   |
| `NEXT_PUBLIC_API_URL` | Public endpoint that web calls             |
| `CRL_BUCKET`          | S3 bucket for certificate revocation lists |
| `TWILIO_VERIFY_SID`   | Verify Service SID for OTP 2FA             |
| `AWS_REGION`          | Default AWS region (e.g. `us‑east‑2`)      |

---

## 🧩 Adopting the Fork

1. **Prune** – Delete any leftover upstream folders not referenced above.
2. **Rename** – Update `package.json#name` fields to `@signasure/*`.
3. **CI Secrets** – Add `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `VERCEL_TOKEN`, and `VERCEL_ORG_ID` to GitHub repo secrets.
4. **Vercel Project** – Point root to `apps/web`; build command `pnpm run build`; output dir `.next`.
5. **Terraform Back‑End** – use remote state in S3 + DynamoDB lock table.

---

## ✨ Roadmap After MVP

* **eIDAS signature module**
* **Audit‑trail viewer** in admin portal
* **Signer video verification** (future)

---

© 2025 8om8 · Licensed under MIT
