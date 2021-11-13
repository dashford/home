# home

## Required Tooling

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [flux](https://fluxcd.io/docs/installation/#install-the-flux-cli)
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets)
- [helm](https://github.com/helm/helm/releases)

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

## Sealed Secrets

`home` uses [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) to manage secrets persisted in GitHub.

### Fetching the Certificate

The `sealed-secrets-controller` runs in the `infrastructure` namespace. The public certificate is stored in this Git repo
in the `./sealed-secrets` directory.

```shell
kubeseal --fetch-cert --controller-name=sealed-secrets-controller --controller-namespace=infrastructure
```

### Adding a new Secret

Generate a kubernetes secret.

```shell
kubectl -n default create secret generic basic-auth \
--from-literal=user=admin \
--from-literal=password=change-me \
--dry-run=client \
-o yaml > basic-auth.yaml
```

Encrypt the secret with `kubeseal`.

```shell
kubeseal --format=yaml --cert=pub-sealed-secrets.pem < basic-auth.yaml > basic-auth-sealed.yaml
```

Delete the unencrypted secret.

```shell
rm basic-auth.yaml
```

Move the encrypted secret into the relevant directory within this Git repo.

When the encrypted secret is merged, verify the secret is created on the cluster.

```shell
$ kubectl -n default get secrets basic-auth

NAME         TYPE     DATA   AGE
basic-auth   Opaque   2      1m43s
```