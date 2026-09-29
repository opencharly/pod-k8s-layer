# pod-k8s-layer

The `k8s-layer` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships `harness-k8s-fixture` — a fake Kubernetes API
surface that stands in for a real cluster in the `kube:` check harness.

## What it provides

Real k3s cannot run inside the check sandbox: the rootless user-namespace slice
does not delegate the `cpuset` cgroup controller, so k3s aborts with
`failed to find cpuset cgroup (v2)` before kubelet starts. This layer instead
runs a minimal Python `http.server` that replays just the GET endpoints the
`kube:` check verb (client-go dynamic, in `candy/plugin-kube`) reaches: nodes,
services, storageclasses, ingressclasses, and the three k3s addons (traefik /
local-path-provisioner / svclb-traefik). Every list returns a single Ready object
so the harness's wait-nodes / addons rollups exit 0.

| Property | Value |
|---|---|
| Service | `harness-k8s-fixture` (`python3 -u /opt/harness-k8s/fake_apiserver.py`, `restart: always`) |
| Port | `6443` (plain HTTP) |
| Requires | `layer-supervisord` |
| Packages | `python3`, `iproute`, `procps-ng` (Fedora) |
| Endpoints | `/api/v1/nodes`, `/api/v1/services`, `/apis/storage.k8s.io/v1/storageclasses`, `/apis/networking.k8s.io/v1/ingressclasses`, the three kube-system addons, `/version` |

It listens on `0.0.0.0:6443` over plain HTTP — the matching kubeconfig sets
`insecure-skip-tls-verify: true` and uses an `http://` server URL, which client-go
honors.

## How to use it

Compose the candy into a harness box; the fixture service starts automatically.
The candy's own `plan:` checks assert the script is installed and executable at
`/opt/harness-k8s/fake_apiserver.py`, the service is running, port `6443` is
listening, and the fixture endpoints answer with the expected Ready objects.

```bash
charly box validate
```

## Layout

- `charly.yml` — the `k8s-layer:` candy entity (description, `require`,
  `distro`, `port`, `service`, `plan`), including the inline `fake_apiserver.py`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-kubernetes:check-k8s` — the `kube:` check verb (nodes,
  pods, ingress, storage classes, addon health, apply/delete, raw GETs).
- `/charly-check:check` — the check/R10 framework.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
