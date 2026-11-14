# 01 - Installing Helm packages with Flux

## The pieces

- `HelmRepository` (source-controller): where the chart lives. Example: `infrastructure/base/metrics-server/repository.yaml`
- `HelmRelease` (helm-controller): which chart, which version, which values. Example: `infrastructure/base/metrics-server/release.yaml`
- Flux `Kustomization` (kustomize-controller): points at a repo path and applies whatever is there. Example: `clusters/docker-desktop/infra-controllers.yaml`
- Never run `helm install` by hand. Commit the two manifests and Flux does the install and the upgrades.

## Steps

1. Make `infrastructure/base/<app>/` with `repository.yaml` and `release.yaml`
2. Add `namespace.yaml` if the chart needs its own namespace (envoy-gateway does, metrics-server uses `kube-system`)
3. Add `kustomization.yaml` in that folder listing the files
4. Add the folder to `infrastructure/docker-desktop/controllers/kustomization.yaml`
5. Merge to `main`, the `GitRepository` only tracks `main`
6. `flux reconcile ks flux-system --with-source`, then `flux get hr -A`

## HTTP repo vs OCI repo

- Classic repo: `url: https://...` (metrics-server)
- OCI repo: `type: oci` and `url: oci://docker.io/envoyproxy`, the chart name goes in the `HelmRelease` (`gateway-helm`)
- Find the latest version in the chart repo index or the project's GitHub releases, then pin it in `spec.chart.spec.version`

## Values

- Put chart values under `spec.values` in the `HelmRelease`
- metrics-server example: `args: [--kubelet-insecure-tls]` (docker-desktop kubelet cert has no IP SANs)
- Check `helm show values <chart>` first to see the real key names

## Layout

- `base/<app>/` is one folder per app, nothing cluster-specific
- `docker-desktop/controllers/` picks the operators and their CRDs
- `docker-desktop/configs/` picks custom resources that need those CRDs (`GatewayClass`, `Gateway`)
- `clusters/docker-desktop/` holds the Flux `Kustomization`s that point at the two folders
- `infra-configs` uses `dependsOn: infra-controllers`, and `infra-controllers` has `wait: true`, so CRDs exist before the custom resources apply

## Gotchas I hit

- Kustomize only reads `kustomization.yaml`, singular. `kustomizations.yaml` is ignored and the render fails
- Flux blocks remote kustomize bases, so you can't point `resources:` at a GitHub URL. Vendor the file or use a `HelmRelease`
- Helm can't adopt resources made by `kubectl apply`. The first install failed with `spec.selector ... field is immutable`. Delete the manual install first
- A failed install marks the `HelmRelease` `Stalled`, and a plain `flux reconcile hr` won't retry it. Use `flux suspend hr` then `flux resume hr`
- Applying a `Gateway` in the same pass as the chart that ships its CRD fails with `no matches for kind`. That is why controllers and configs are split
- `prune: true` deletes anything removed from the path. Moving an app between Kustomizations causes a short outage while it is pruned and recreated
- Typos in CRD fields (`listener` vs `listeners`) only show up on apply, so run `kubectl apply --dry-run=server` once the CRDs exist

## Useful commands

- `flux get ks` and `flux get hr -A` for status
- `kubectl get hr <name> -n <ns> -o jsonpath='{.status.conditions}'` for the real error message
- `kubectl kustomize <dir>` to render before you push
- `flux reconcile hr <name> -n <ns> --with-source` to force a sync
