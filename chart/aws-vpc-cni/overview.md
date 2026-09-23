## About this Helm chart

This is an AWS VPC CNI Docker Hardened Helm chart built from the upstream AWS VPC CNI Helm chart and using a hardened
configuration with Docker Hardened Images.

The following Docker Hardened Images are used in this Helm chart:

- `dhi/amazon-k8s-cni`
- `dhi/amazon-k8s-cni-init`
- `dhi/aws-network-policy-agent`

To learn more about how to use this Helm chart you can visit the upstream documentation:
[https://github.com/aws/eks-charts/tree/master/stable/aws-vpc-cni](https://github.com/aws/eks-charts/tree/master/stable/aws-vpc-cni)

### About AWS VPC CNI

The Amazon VPC CNI plugin for Kubernetes provides pod networking using Elastic Network Interfaces on AWS. It gives every
pod a real VPC IP address by managing ENI attachment and IP allocation on each node, so pods communicate directly within
the VPC without an overlay network. The chart deploys the `aws-node` DaemonSet together with the AWS Network Policy
Agent, which enforces Kubernetes network policies with eBPF.

Official documentation: https://github.com/aws/amazon-vpc-cni-k8s

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with near-zero known CVEs, include signed provenance, and come with a complete Software Bill
of Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly
into existing Docker workflows.

## Trademarks

Kubernetes® is a registered trademark of The Linux Foundation. Amazon Web Services, AWS, Amazon EKS, and the AWS logo
are trademarks of Amazon.com, Inc. or its affiliates. All rights in these marks are reserved to their respective owners.
Any use by Docker is for referential purposes only and does not indicate sponsorship, endorsement, or affiliation.
