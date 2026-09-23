## Installing the chart

### Prerequisites

- Kubernetes 1.22+
- Helm 3.0+
- An AWS account with the IAM permissions required by the controller (via IRSA or attached to the worker node role)

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
registry.

Follow the [authentication instructions for DHI in Kubernetes](https://docs.docker.com/dhi/how-to/k8s/#authentication).

For example:

```console
kubectl create secret docker-registry helm-pull-secret \
  --docker-server=dhi.io \
  --docker-username=<Docker username> \
  --docker-password=<Docker token> \
  --docker-email=<Docker email>
```

#### Step 3: Set up IAM permissions

The controller needs AWS IAM permissions to manage ALBs, NLBs, and related resources. The recommended approach is IAM
Roles for Service Accounts (IRSA):

```console
eksctl utils associate-iam-oidc-provider \
    --region <aws-region> \
    --cluster <your-cluster-name> \
    --approve

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

eksctl create iamserviceaccount \
    --cluster=<your-cluster-name> \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
    --approve
```

If you are not using IRSA, attach the same policy to your worker node IAM role instead, and set
`serviceAccount.create=false` with `serviceAccount.name=aws-load-balancer-controller` at install time.

#### Step 4: Install the Helm chart

To install the chart, use `helm install`. Make sure you use `helm login` to log in before running `helm install`.
Optionally, you can also use the `--dry-run` flag to test the installation without actually installing anything.

**Note**: The chart ships with a placeholder `clusterName`; always override it with your actual cluster name.

```console
helm install my-aws-load-balancer-controller oci://dhi.io/aws-load-balancer-controller-chart --version <version> \
  --set "imagePullSecrets[0].name=helm-pull-secret" \
  --set "clusterName=<your-cluster-name>" \
  --namespace kube-system
```

If you created the IAM role with `eksctl create iamserviceaccount` in the previous step, also set
`serviceAccount.create=false` and `serviceAccount.name=aws-load-balancer-controller` so the chart reuses that service
account instead of creating a new one without the IRSA annotation:

```console
helm install my-aws-load-balancer-controller oci://dhi.io/aws-load-balancer-controller-chart --version <version> \
  --set "imagePullSecrets[0].name=helm-pull-secret" \
  --set "clusterName=<your-cluster-name>" \
  --set "serviceAccount.create=false" \
  --set "serviceAccount.name=aws-load-balancer-controller" \
  --namespace kube-system
```

Replace `<version>` accordingly. If the chart is in your own registry or repository, replace `dhi.io` with your own
registry and namespace. Replace `helm-pull-secret` with the name of the image pull secret you created earlier.

#### Step 5: Verify the installation

The deployment's pods should show up and running almost immediately:

```bash
$ kubectl get pods -n kube-system
NAME                                                                    READY   STATUS    RESTARTS   AGE
my-aws-load-balancer-controller-aws-load-balancer-controller-chart-...  1/1     Running   0          20s
```

#### Step 6: Create an Ingress or a Service of type LoadBalancer

Once the controller is running, it provisions an ALB for Ingress resources annotated with
`kubernetes.io/ingress.class: alb` (or using the `alb` `IngressClass`), and an NLB for Service resources of type
`LoadBalancer`. See the
[upstream documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/)
for the full set of supported annotations.
