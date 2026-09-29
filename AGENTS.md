# AGENTS.md — pod-k8s-layer

Standalone candy repo for the `k8s-layer` candy — `harness-k8s-fixture`, a fake
Kubernetes API surface (a stdlib Python `http.server`) that stands in for a real
cluster in the `kube:` check harness. The entire candy lives in `charly.yml` at
the repo root.

Canonical files:

- `charly.yml` — the `k8s-layer:` candy entity (description, `require`,
  `distro`, `port`, `service`, `plan`), including the inline
  `fake_apiserver.py` written by a `write:` step.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-kubernetes:check-k8s` — the owning skill: the `kube:` check verb
  (nodes, pods, ingress, storage classes, addon health, apply/delete, raw GETs)
  this fixture serves. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `kube:` verb dispatch, and `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, package sections, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

The candy carries no `skill:` entity of its own; the family skill
`/charly-kubernetes:check-k8s` covers the `kube:` verb surface. The gap is routed
to the named skill-authoring batch
[opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291);
when that lands, add the owning skill here.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The candy's own `check:` steps are the R10 witness: they assert the fixture
  script is present and executable at `/opt/harness-k8s/fake_apiserver.py`, the
  `harness-k8s-fixture` service is running, port `6443` is listening, and the
  fixture endpoints answer (`/api/v1/nodes` carries `charly-fixture-k8s`,
  `/apis/storage.k8s.io/v1/storageclasses` carries `local-path`, `/version`
  carries `v1.31.0-fake-fixture`).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `k8s-layer:` candy entity in `charly.yml`. The `fake_apiserver.py`
  body lives inline in the `write:` step's `content:`; the routing table there is
  the contract the `kube:` check verb depends on.
- The `port:` field (`6443`), the `harness-k8s-fixture` service exec, and the
  fixture's bind address must stay in step.
- Keep the fixture aligned with the endpoints the `kube:` verb actually probes;
  a fixture that drifts from the verb hides a real integration break.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
