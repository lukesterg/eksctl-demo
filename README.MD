# EKS demo setup

## What does it do

## Things to fix before going to production

- The sealed secrets private key should not be stored in this repository. This should be stored separately but is included for demo purposes.
- `git-poll-interval` is set to 1 minute. This should be removed (the default is 5 minutes). 1m is good for demoing.
- Change Grafana password "tempPassword" to a sealed secret.
- Add persistent volume to Loki
- Delete MS teams integration. Look for `msteams.url`.
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
    desiredCapacity: 2
    minSize: 2
    maxSize: 2
    privateNetworking: true
    volumeEncrypted: true
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        albIngress: true
        appMesh: true
        cloudWatch: true
        externalDNS: true
        ebs: true
        efs: true
        certManager: true
        xRay: true
EOF

EKSCTL_EXPERIMENTAL=true \
 eksctl enable repo \
 --git-url git@github.com:lukesterg/eksctl-demo.git \
 --git-email 9840170+lukesterg@users.noreply.github.com \
 --cluster eks-demo \
 --region us-east-2

kubectl apply -f docs/master.key
```

Add the SSH deployment key to your git repository and run `fluxctl sync --k8s-fwd-ns flux`.

Assign the domain to the cluster. If using Route53 add an ALIAS and select the load balancer otherwise run `kubectl get -n projectcontour service envoy -o wide` and add it as a CNAME.
