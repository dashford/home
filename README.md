# home

## Required Tooling

- [microk8s](https://microk8s.io/)
- [flux](https://fluxcd.io/docs/installation/#install-the-flux-cli)
- [kubeseal](https://github.com/bitnami-labs/sealed-secrets)
- [helm](https://github.com/helm/helm/releases)
- kubectx
- kubens

## Create k8s Cluster

home runs on kubernetes using the `microk8s` framework. To create the cluster, run

```shell
sudo snap install microk8s --classic --channel=1.21/stable
```

> At time of writing, `1.21` was the latest release.

Join the required Linux groups

```shell
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```

You can check on the status by running

```shell
microk8s status
```

### Accessing the Cluster

Copy the generated kubeconfig file into your `~/.kube` directory i.e.

```shell
microk8s config > ~/.kube/config
```

### Install Required Addons

```shell
microk8s enable dns
```

## Bootstrap Flux

```shell
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=dashford

flux bootstrap github --owner=dashford --repository=home --branch=main --path=./clusters/home --personal
```

## Sealed Secrets

`home` uses [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) to manage secrets persisted in GitHub.

### Fetching the Certificate

The `sealed-secrets-controller` runs in the `infrastructure` namespace. The public certificate is stored in this Git repo
in the `./sealed-secrets` directory.

```shell
kubeseal --fetch-cert --context microk8s --controller-name=sealed-secrets-controller --controller-namespace=infrastructure
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

## Operations

A handy list of operations that may need to be frequently run.

### Stop/Start `microk8s`

```shell
microk8s start
[...]
microk8s stop
```