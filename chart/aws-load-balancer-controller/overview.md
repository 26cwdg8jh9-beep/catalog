## About this Helm chart

This is an AWS Load Balancer Controller Docker Helm chart built from the upstream AWS Load Balancer Controller Helm
chart and using a hardened configuration with Docker Hardened Images.

The following Docker Hardened Images are used in this Helm chart:

- `dhi/aws-load-balancer-controller`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://github.com/aws/eks-charts/tree/master/stable/aws-load-balancer-controller](https://github.com/aws/eks-charts/tree/master/stable/aws-load-balancer-controller)

### About AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS Elastic Load Balancers for a Kubernetes cluster. It provisions Application
Load Balancers (ALB) for Kubernetes Ingress resources and Network Load Balancers (NLB) for Kubernetes Service resources
of type `LoadBalancer`.

For more information and documentation see https://kubernetes-sigs.github.io/aws-load-balancer-controller/.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with near-zero known CVEs, include signed provenance, and come with a complete Software Bill
of Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly
into existing Docker workflows.

## Trademarks

This listing is prepared by Docker. All third-party product names, logos, and trademarks are the property of their
respective owners and are used solely for identification. Docker claims no interest in those marks, and no affiliation,
sponsorship, or endorsement is implied.
