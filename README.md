# LAVA Helm Chart

This repository contains the Helm chart for deploying [LAVA](https://lavasoftware.org/) (Linaro Automated Validation Architecture) on Kubernetes.

## About LAVA

LAVA is an automated validation architecture primarily aimed at testing deployments of systems software on physical and virtual hardware. It was originally designed for testing Linux kernel deployments on ARM development boards and now supports a wide range of devices and architectures.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- **ReadWriteMany storage support** (see Storage Requirements below)

## Storage Requirements

⚠️ **Important**: LAVA requires **shared storage** across multiple pods (server, publisher, worker).

- **Single-node clusters** (minikube, dev): `ReadWriteOnce` works fine (all pods on same node)
- **Multi-node clusters** (production): Requires `ReadWriteMany` (pods across nodes)

**Supported Storage Solutions:**
- **AWS**: EFS CSI driver (`storageClass: "efs-sc"`)
- **Azure**: Azure Files (`storageClass: "azurefile"`)
- **GCP**: Filestore CSI driver
- **On-premises**: NFS, Ceph, or similar shared storage

**For testing without persistence**, use `--set persistence.enabled=false` (default).

## Repository Structure

```
lava_helm/
├── charts/
│   └── lava/           # Main LAVA Helm chart
├── docs/               # Additional documentation
├── examples/           # Example configurations
└── README.md          # This file
```

## Quick Start

The simplest way to get LAVA running:

```bash
# Install with minimal configuration
helm install my-lava ./charts/lava --set lava.fqdn=my-lava.example.com

# Or with ingress enabled for external access
helm install my-lava ./charts/lava \
  --set lava.fqdn=my-lava.example.com \
  --set ingress.enabled=true \
  --set ingress.className=nginx
```

- SQLite database (no external DB required)
- Local storage (10Gi PVC)
- Standard HTTP health checks
- No external dependencies
- Standard Helm chart structure

## Installation Options

### From Repository
```bash
helm repo add lava-charts https://your-charts-repo.example.com/
helm repo update
helm install my-lava lava-charts/lava -f values.yaml
```

### From Local Chart
```bash
helm install my-lava ./charts/lava -f examples/minimal.yaml
```

## Local Testing with Minikube

You can easily test the LAVA chart on your local machine using minikube.

### Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/) installed
- [kubectl](https://kubernetes.io/docs/tasks/tools/) configured
- [Helm](https://helm.sh/docs/intro/install/) 3.2.0+

### Quick Test

```bash
# 1. Start minikube cluster
minikube start

# 2. Deploy LAVA
helm install lava ./charts/lava \
  --set lava.fqdn=lava.local \
  --set ingress.enabled=false

# 3. Wait for pod to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=lava --timeout=300s

# 4. Access LAVA via port-forward
kubectl port-forward service/lava-server 8080:80
```

Open http://localhost:8080 in your browser to access LAVA.

### With Ingress (Optional)

If you want to test with ingress:

```bash
# 1. Start minikube with ingress
minikube start
minikube addons enable ingress

# 2. Deploy with ingress enabled
helm install lava ./charts/lava -f examples/minimal.yaml \
  --set lava.fqdn=lava.local

# 3. Add to hosts file
echo "$(minikube ip) lava.local" | sudo tee -a /etc/hosts

# 4. Access via ingress
open http://lava.local
```

### Monitoring and Debugging

```bash
# Check deployment status
kubectl get pods,svc,pvc

# View logs if there are issues
kubectl logs -l app.kubernetes.io/name=lava

# Check LAVA health
curl http://localhost:8080/api/v0.2/
```

### Cleanup

```bash
# Stop port-forward
pkill -f "kubectl port-forward"

# Remove LAVA deployment
helm uninstall lava

# Optional: Delete persistent data
kubectl delete pvc lava-data

# Optional: Stop minikube
minikube stop

# Optional: Delete cluster entirely
minikube delete
```

### Troubleshooting

**Pod not starting?**
```bash
kubectl describe pod -l app.kubernetes.io/name=lava
kubectl logs -l app.kubernetes.io/name=lava
```

**Service not accessible?**
```bash
kubectl get endpoints lava-server
kubectl port-forward pod/$(kubectl get pod -l app.kubernetes.io/name=lava -o name) 8080:80
```

**Storage issues?**
```bash
kubectl get pv,pvc
kubectl describe pvc lava-data
```

## Configuration

The following table lists the configurable parameters of the LAVA chart and their default values.

### Basic Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | LAVA server image repository | `lavasoftware/lava-server` |
| `image.tag` | LAVA server image tag | `""` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `fqdn` | Fully qualified domain name for the LAVA instance | `""` |
| `environment` | Environment name (staging, production, etc.) | `""` |

### Resource Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `size` | Deployment size (default or large) | `default` |
| `resources.server.default.requests.cpu` | Default CPU request for server | `0.5` |
| `resources.server.large.requests.cpu` | Large CPU request for server | `1` |

### Storage Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `storage.size.device-types` | Device types storage size | `1Gi` |
| `storage.size.devices` | Devices storage size | `1Gi` |
| `storage.size.health-checks` | Health checks storage size | `1Gi` |
| `storage.size.dispatcher` | Dispatcher storage size | `1Gi` |
| `storage.size.job-output` | Job output storage size | `1Gi` |

### Autoscaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable autoscaling | `false` |
| `autoscaling.services.server.minReplicas` | Minimum replicas for server | `1` |
| `autoscaling.services.server.maxReplicas` | Maximum replicas for server | `4` |
| `autoscaling.services.server.targetCPU` | Target CPU percentage for scaling | `60` |

### Maintenance and Cleanup

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cleanup.enabled` | Enable cleanup cronjob | `true` |
| `cleanup.logsOnly` | Only clean logs, not job data | `true` |
| `maintenance.enabled` | Enable maintenance mode | `false` |

## Examples

See the `examples/` directory for sample configurations:

- `examples/basic.yaml` - Basic LAVA deployment
- `examples/production.yaml` - Production-ready configuration
- `examples/multi-instance.yaml` - Multi-instance setup

## Architecture

The LAVA Helm chart deploys the following components:

1. **LAVA Server**: The main LAVA web interface and API
2. **LAVA Publisher**: Handles result publishing and notifications
3. **LAVA Scheduler**: Manages job scheduling and device allocation
4. **LAVA Worker**: Executes jobs on connected devices
5. **Support Services**: Including log compression, backups, and cleanup

## Upgrading

To upgrade an existing LAVA deployment:

```bash
helm upgrade my-lava lava-charts/lava -f values.yaml
```

## Uninstalling

To uninstall/delete the `my-lava` deployment:

```bash
helm delete my-lava
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add/update tests if necessary
5. Update documentation
6. Submit a pull request

## Support

- Documentation: https://docs.lavasoftware.org/
- Mailing List: lava-devel@lists.lavasoftware.org
- Issues: Please report issues in the appropriate LAVA repository

## License

This Helm chart is released under the same license as LAVA itself. See the LAVA project for license details.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and changes.
