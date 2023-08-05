Tailscale is VPN service using the open soure WireGuard protocol.
This is a description about how to automatically connect a k8s cluster,
to my "tail-net". 

This way I can reach pods and services with internal clusterIP (10.43.X.X) even from my dev laptop, without creating a NodePort or an Ingress.

# Usage

```
kubectl apply -k https://github.com/lalyos/tailscale-k8s
```

## Auto-approvers

By default it will create subnetrouter tagged machine (https://login.tailscale.com/admin/machines) but you have to approve the route settings.

You can make it automatic by adding this to your ACL:

[https://login.tailscale.com/admin/acls/file](https://login.tailscale.com/admin/acls/file)

```json
    "autoApprovers": {
      "routes": {
        "10.42.0.0/16": ["lalyos@github"],
        "10.43.0.0/16": ["lalyos@github"],
      }
    },
```

## tl;dr

follow this doc: [https://tailscale.com/kb/1185/kubernetes/](https://tailscale.com/kb/1185/kubernetes/)

```bash
svn co https://github.com/tailscale/tailscale/trunk/docs/k8s
cd k8s
```

## Setup

create secret

Get/create an authkey: [https://login.tailscale.com/admin/settings/keys](https://login.tailscale.com/admin/settings/keys)
Make sure you mark it as **Reusable**

Vanilla k8s secret
```bash
k create secret generic tailscale-auth --from-literal TS_AUTHKEY=tskey-auth-xxxxxxCNTRL-XXXXXYYYYYYZZZZZ
```

Or alternatively create a [Bitnami SealedSecret](https://github.com/bitnami-labs/sealed-secrets)
```
kubectl create secret generic tailscale-auth \
   --from-literal TS_AUTHKEY=tskey-auth-xxxxxxCNTRL-XXXXXYYYYYYZZZZZ \
   --dry-run \
   -o yaml \
   | kubeseal \
     --cert ~/.kubeseal/tls.crt \
     --format yaml \
     -n tailscale \
    > ss.yaml

kubectl apply -f ss.yaml
```

create RBAC

```bash
export SA_NAME=tailscale
export TS_KUBE_SECRET=tailscale-auth
make rbac | k apply -f -
```

create subnet router

```bash
SERVICE_CIDR=10.43.0.0/16
POD_CIDR=10.42.0.0/16
export TS_ROUTES=$SERVICE_CIDR,$POD_CIDR
make subnet-router | k apply -f -

```

