# SSH Pods on Kubernetes

This project deploys two pods for SSH testing:

- `ssh-server`: an Alpine pod running `sshd` on port `22`
- `ssh-client`: an Alpine pod with `openssh` tools and a mounted private key

It also defines:

- a `Service` named `ssh-server` exposing TCP `22`
- a `Secret` named `ssh-rsa-keys` with `id_rsa` and `id_rsa_pub`

## Files in `src`

- `pod-ssh-server.yaml`: SSH server pod
- `pod-ssh-client.yaml`: SSH client pod
- `service-ssh-server.yaml`: service for the server pod
- `secret.yaml`: key material used by both pods
- `kustomization.yaml`: kustomize entrypoint

## Prerequisites

- Kubernetes cluster access (`kubectl` configured)
- Kustomize support (`kubectl apply -k`)

## Deploy

The current `kustomization.yaml` includes:

- `pod-ssh-client.yaml`
- `pod-ssh-server.yaml`
- `service-ssh-server.yaml`

Apply all manifests referenced by kustomization:

```bash
kubectl apply -k src
```

Apply the secret (if not already included in your kustomization):

```bash
kubectl apply -f src/secret.yaml
```

## Verify

```bash
kubectl get pods
kubectl get svc ssh-server
kubectl get secret ssh-rsa-keys
```

Wait until `ssh-server` is Ready.

## Test SSH from client pod

Open a shell in the client pod:

```bash
kubectl exec -it pod/ssh-client -- sh
```

Connect to the SSH server service:

```bash
ssh -o StrictHostKeyChecking=no root@ssh-server
```

If the key pair in `secret.yaml` matches, authentication should succeed without a password.

## Cleanup

```bash
kubectl delete -k src
kubectl delete -f src/secret.yaml
```

## Security note

`src/secret.yaml` contains a base64-encoded private key. Treat this repository as sensitive and rotate keys before using in shared or production environments.
