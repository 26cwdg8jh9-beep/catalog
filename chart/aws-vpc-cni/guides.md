## Installing the chart

### Prerequisites

- Kubernetes 1.11+ running on AWS (Amazon EKS, or self-managed nodes on Amazon EC2)
- Helm 3
- Node IAM permissions to manage Elastic Network Interfaces, such as the `AmazonEKS_CNI_Policy` managed policy attached
  to the node role or to an IAM role for the `aws-node` service account

### Installation steps

All examples in this guide use the public chart and images. If you've mirrored the repository for your own use (for
example, to your Docker Hub namespace), update your commands to reference the mirrored chart instead of the public one.

For example:

- Public chart: `dhi.io/<repository>:<tag>`
- Mirrored chart: `<your-namespace>/dhi-<repository>:<tag>`

For more details about customizing the chart to reference other images, see the
[documentation](https://docs.docker.com/dhi/how-to/customize/).

#### Step 1: Optional. Mirror the Helm chart and/or its images to your own registry

To optionally mirror a chart to your own third-party registry, you can follow the instructions in
[How to mirror an image](https://docs.docker.com/dhi/how-to/mirror/) for either the chart, the image, or both.

The same `regctl` tool that is used for mirroring container images can also be used for mirroring Helm charts, as Helm
charts are OCI artifacts.

For example:

```console
 regctl image copy \
     "${SRC_CHART_REPO}:${TAG}" \
     "${DEST_REG}/${DEST_CHART_REPO}:${TAG}" \
     --referrers \
     --referrers-src "${SRC_ATT_REPO}" \
     --referrers-tgt "${DEST_REG}/${DEST_CHART_REPO}" \
     --force-recursive
```

#### Step 2: Create a Kubernetes secret for pulling images

The Docker Hardened Images that the chart uses require authentication. To allow your Kubernetes cluster to pull those
images, you need to create a Kubernetes secret with your Docker Hub credentials or with the credentials for your own
registry. The CNI runs in the `kube-system` namespace, so create the secret there.

Follow the [authentication instructions for DHI in Kubernetes](https://docs.docker.com/dhi/how-to/k8s/#authentication).

For example:

```console
kubectl create secret docker-registry helm-pull-secret \
  --namespace kube-system \
  --docker-server=dhi.io \
  --docker-username=<Docker username> \
  --docker-password=<Docker token> \
  --docker-email=<Docker email>
```

#### Step 3: Install the Helm chart

To install the chart, use `helm install`. Make sure you use `helm login` to log in before running `helm install`.
Optionally, you can also use the `--dry-run` flag to test the installation without actually installing anything.

```console
helm install aws-vpc-cni oci://dhi.io/aws-vpc-cni-chart --version <version> \
  --namespace kube-system \
  --set "imagePullSecrets[0].name=helm-pull-secret"
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

On an Amazon EKS cluster the `aws-node` DaemonSet, ConfigMap, ServiceAccount and RBAC resources already exist. Label and
annotate them for Helm ownership before installing, as described in the upstream section
[Adopting the existing aws-node resources in an EKS cluster](https://github.com/aws/eks-charts/tree/master/stable/aws-vpc-cni#adopting-the-existing-aws-node-resources-in-an-eks-cluster),
and set `originalMatchLabels=true` so Helm adopts the existing DaemonSet. The pods still roll, one node at a time,
because the pod template now points at the Docker Hardened Images.

The chart keeps the upstream security posture of the CNI: the DaemonSet uses the host network, the init container and
the network policy agent run privileged, and the `aws-node` container runs as root with the `NET_ADMIN` and `NET_RAW`
capabilities. These privileges are required to program ENIs, routes, iptables rules and eBPF programs on the node. The
Docker Hardened Images default to a nonroot user, so the chart sets the root user explicitly where the CNI needs it.

#### Step 4: Verify the installation

The `aws-node` DaemonSet should schedule one pod per Linux node and every pod should become ready once the CNI has
configured the node:

```bash
$ kubectl get daemonset aws-node -n kube-system
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
aws-node   3         3         3       3            3           <none>          60s

$ kubectl get pods -n kube-system -l k8s-app=aws-node
NAME             READY   STATUS    RESTARTS   AGE
aws-node-4xt7z   2/2     Running   0          60s
aws-node-7hkqp   2/2     Running   0          60s
aws-node-nwmgj   2/2     Running   0          60s
```

### What this chart provisions

- DaemonSet `aws-node` on every Linux node: init container `aws-vpc-cni-init` (`dhi/amazon-k8s-cni-init`), containers
  `aws-node` (`dhi/amazon-k8s-cni`) and `aws-eks-nodeagent` (`dhi/aws-network-policy-agent`)
- ConfigMap `amazon-vpc-cni` with the network policy and Windows IPAM settings
- ServiceAccount `aws-node`, ClusterRole and ClusterRoleBinding `aws-node`
- CustomResourceDefinition `eniconfigs.crd.k8s.amazonaws.com`
- Optional: `ENIConfig` resources when `eniConfig.create=true`, a Prometheus `PodMonitor` when `podMonitor.create=true`,
  and a custom `10-aws.conflist` ConfigMap when `cniConfig.enabled=true`

The network policy agent container is deployed by default, but policy enforcement stays off until you set
`enableNetworkPolicy=true`, matching the upstream chart defaults.
