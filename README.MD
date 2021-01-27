# EKS demo setup

## What does it do

## Things to fix before going to production

- The sealed secrets private key should not be stored in this repository. This should be stored separately but is included for demo purposes.
- `git-poll-interval` is set to 1 minute. This should be removed (the default is 5 minutes). 1m is good for demoing.
- Change Grafana password "tempPassword" to a sealed secret.
- Add persistent volume to Loki
- Volume encryption on Grafana, Prometheus and Loki.
- It became too difficult to maintain configurations before installing perform a find and replace for the following fields:
  - `eks-demo` to your cluster name
  - `us-east-2` to your region
  - `9840170+lukesterg@users.noreply.github.com` with your email address.
  - `podinfo.questdev.xyz` to the domain you want to host podinfo.

## To setup the cluster run

```bash
eksctl create cluster -f - <<EOF
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: eks-demo
  region: us-east-2

vpc:
  nat:
    gateway: HighlyAvailable

managedNodeGroups:
  - name: eks-demo
    instanceType: t3a.medium
    instancePrefix: eks-demo-eks
    desiredCapacity: 3
    minSize: 2
    maxSize: 3
    privateNetworking: true
    volumeEncrypted: true
    volumeSize: 80
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        ebs: true
        efs: true
        certManager: true

git:
  repo:
    url: git@github.com:lukesterg/eksctl-demo.git
    email: 9840170+lukesterg@users.noreply.github.com
EOF
```

After running the above command the last line will include an SSH key. Add this to your SSH repository then run `fluxctl sync --k8s-fwd-ns flux`.

Then add the target domain to the network load balancer.

## Argo deployment

To connect to the argo server run `kubectl port-forward svc/argocd-server -n argocd 8080:443` and go to `https://localhost:8080`.

To set a password first get the initial password `kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2` then run `argocd login 127.0.0.1:8080` with the username `admin` and the password from the previous command. Then run `argocd account update-password` to update the password.
