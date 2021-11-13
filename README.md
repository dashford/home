# home

## Required Tooling

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [flux](https://fluxcd.io/docs/installation/#install-the-flux-cli)
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets)

## Create k8s Cluster

home runs on kubernetes using the `kind` binary. To create the cluster, run

```shell
kind create cluster --config=config.yaml
```

## Bootstrap Flux

```shell
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

flux bootstrap github --owner=dashford --repository=home --branch=main --path=./clusters/home --personal
```