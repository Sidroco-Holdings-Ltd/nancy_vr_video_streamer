# NANCY VR Video Streamer - Helm Deployment Guide

## Quick Start with Helm

This guide shows how to deploy the NANCY VR Video Streaming Server to a Kubernetes cluster using Helm.

### Prerequisites

1. **Kubernetes Cluster** (v1.19+)
   - Local: Minikube, Docker Desktop, or Kind
   - Cloud: EKS, GKE, AKS
   - On-premise: Any Kubernetes distribution

2. **Helm 3.2.0+** installed:
   ```bash
   # macOS
   brew install helm
   
   # Windows
   choco install kubernetes-helm
   
   # Linux
   curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
   ```

3. **kubectl** configured to access your cluster

### Installation Steps

1. **Clone and Navigate to the Chart**:
   ```bash
   cd nancy-vr-video-streamer
   ```

2. **Basic Installation**:
   ```bash
   helm install nancy-vr ./nancy-vr-video-streamer
   ```

3. **Installation with External Access** (LoadBalancer):
   ```bash
   helm install nancy-vr ./nancy-vr-video-streamer \
     --set loadBalancer.enabled=true \
     --set loadBalancer.port=8080
   ```

4. **Installation with Ingress**:
   ```bash
   helm install nancy-vr ./nancy-vr-video-streamer \
     --set videoServer.ingress.enabled=true \
     --set videoServer.ingress.hosts[0].host=nancy-video.yourdomain.com \
     --set videoServer.ingress.hosts[0].paths[0].path=/ \
     --set videoServer.ingress.hosts[0].paths[0].pathType=ImplementationSpecific
   ```

### Accessing the Application

After installation, follow the instructions shown in the Helm output:

1. **Port Forward (Development)**:
   ```bash
   kubectl port-forward service/nancy-vr-nancy-vr-video-streamer-video-server 8080:80
   # Access at: http://localhost:8080
   ```

2. **Check Status**:
   ```bash
   helm status nancy-vr
   kubectl get pods -l app.kubernetes.io/name=nancy-vr-video-streamer
   ```

### Configuration Options

Common configuration options you can override:

```bash
# Custom storage sizes
--set persistence.videos.size=50Gi \
--set persistence.logs.size=5Gi

# Custom resource limits
--set videoServer.resources.limits.cpu=1000m \
--set videoServer.resources.limits.memory=1Gi

# Environment variables
--set config.videoSourceUrl=https://your-video-source.com \
--set config.grpcHost=your-grpc-host.com

# Enable autoscaling
--set autoscaling.enabled=true \
--set autoscaling.minReplicas=2 \
--set autoscaling.maxReplicas=10
```

### Custom Values File

Create a `production-values.yaml`:

```yaml
videoServer:
  ingress:
    enabled: true
    hosts:
      - host: nancy-video.example.com
        paths:
          - path: /
            pathType: ImplementationSpecific

persistence:
  videos:
    size: 100Gi
  logs:
    size: 10Gi

config:
  videoSourceUrl: "https://cdn.example.com/videos"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10

videoServer:
  resources:
    limits:
      cpu: 2000m
      memory: 2Gi
    requests:
      cpu: 500m
      memory: 512Mi
```

Then install:
```bash
helm install nancy-vr ./nancy-vr-video-streamer -f production-values.yaml
```

### Management Commands

```bash
# Upgrade deployment
helm upgrade nancy-vr ./nancy-vr-video-streamer

# Upgrade with new values
helm upgrade nancy-vr ./nancy-vr-video-streamer --set videoServer.replicaCount=3

# Check release history
helm history nancy-vr

# Rollback to previous version
helm rollback nancy-vr 1

# Uninstall
helm uninstall nancy-vr
```

### Troubleshooting

1. **Check pod status**:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=nancy-vr-video-streamer
   kubectl describe pod <pod-name>
   ```

2. **View logs**:
   ```bash
   kubectl logs -l app.kubernetes.io/component=video-server
   kubectl logs -l app.kubernetes.io/component=grpc-client
   kubectl logs -l app.kubernetes.io/component=grpc-server
   ```

3. **Check storage**:
   ```bash
   kubectl get pvc
   kubectl describe pvc nancy-videos-pvc
   ```

4. **Test connectivity**:
   ```bash
   helm test nancy-vr
   ```

### Migration from Docker Compose

If you were previously using Docker Compose, here's how the services map:

| Docker Compose Service | Helm Component | Access Method |
|------------------------|---------------|---------------|
| `video-server` | `video-server` | Port 80 (HTTP) |
| `nancy-grpc-client-side` | `grpc-client` | Port 8000 (API) |
| `nancy-grpc-server-side` | `grpc-server` | Port 8000 (API) |

Your videos and logs will be stored in Kubernetes PersistentVolumes instead of local bind mounts.
