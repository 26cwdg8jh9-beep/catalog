## About Azure ResourceManager Exporter

Azure ResourceManager Exporter is a Prometheus exporter that collects information from the Azure Resource Manager API
and exposes it in Prometheus format. It reports subscription quotas and limits, Azure Cost Management costs and budgets,
resource health status, IAM role assignments and definitions, and Microsoft Defender for Cloud secure score and
recommendations.

This hardened image is built from the official
[webdevops/azure-resourcemanager-exporter](https://github.com/webdevops/azure-resourcemanager-exporter) source and is
intended to run as a standalone metrics exporter scraped by Prometheus.

## About Docker Hardened Images

Docker Hardened Images are built to meet the highest security and compliance standards. They provide a trusted
foundation for containerized workloads by incorporating security best practices from the start.

### Why use Docker Hardened Images?

These images are published with near-zero known CVEs, include signed provenance, and come with a complete Software Bill
of Materials (SBOM) and VEX metadata. They're designed to secure your software supply chain while fitting seamlessly
into existing Docker workflows.

## Trademarks

Microsoft® and Azure® are trademarks of Microsoft Corporation. All rights in these marks are reserved to Microsoft
Corporation. Any use by Docker is for referential purposes only and does not indicate sponsorship, endorsement, or
affiliation.
