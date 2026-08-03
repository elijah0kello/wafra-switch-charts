# wafra-switch-charts

A plain Helm repository, served by GitHub Pages, for charts a Wafra switch
deploys that are not published upstream.

```bash
helm repo add wafra-switch https://elijah0kello.github.io/wafra-switch-charts
helm repo update
helm search repo wafra-switch
```

## Why a Helm repo and not an OCI registry

The charts were previously pushed to the switch's own Zot as OCI artifacts.
That works for `helm pull` and does not work well for Argo CD:

- Argo needs an explicit repository registration for any OCI registry that is
  not a well-known public one, or it treats a scheme-less `repoURL` as a
  classic Helm repo and runs `helm pull --repo <url> <chart>`, which fails with
  `invalid chart URL format`.
- Its `insecure` flag maps to `--insecure-skip-tls-verify`, which still speaks
  TLS — against a plain-HTTP registry that fails at the handshake, not the
  certificate.
- Serving it over the switch's own HTTPS ingress fixed both and introduced a
  worse problem: the chart's address then depended on the switch's own domain
  and certificate, so a clean rebuild could not fetch the chart until DNS and
  TLS for the new domain already existed. A deployment tool should not need
  the thing it is deploying to be up first.

A static Helm repo over GitHub Pages has none of that. It is public, needs no
credentials or registry registration, and its address does not change when a
switch changes domain.

Container images are a separate question and stay in a registry.

## Publishing a new version

```bash
helm package <chart-dir> -d .
helm repo index . --url https://elijah0kello.github.io/wafra-switch-charts
git add -A && git commit -m "orbit x.y.z" && git push
```

`index.yaml` is the repository. A chart that is not in it does not exist as far
as Helm is concerned, so regenerate it every time.
