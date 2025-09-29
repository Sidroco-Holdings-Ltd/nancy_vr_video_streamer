# NANCY VR Video Streamer Helm Chart

This Helm chart deploys the NANCY VR Video Streaming Server with Analytics to a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `nancy-vr`:

```bash
helm install nancy-vr ./nancy-vr-video-streamer
```

The command deploys the NANCY VR Video Streamer on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `nancy-vr` deployment:

```bash
helm delete nancy-vr
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

### Global parameters

| Name                      | Description                                     | Value                               |
| ------------------------- | ----------------------------------------------- | ----------------------------------- |
| `global.imageRegistry`    | Global Docker image registry                    | `ghcr.io/sidroco-holdings-ltd`     |

### Video Server Configuration

| Name                                    | Description                                | Value                    |
| --------------------------------------- | ------------------------------------------ | ------------------------ |
| `videoServer.enabled`                   | Enable video server deployment             | `true`                   |
| `videoServer.replicaCount`             | Number of video server replicas            | `1`                      |
| `videoServer.image.repository`         | Video server image repository              | `nancy-video-server`     |
| `videoServer.image.tag`                | Video server image tag                     | `latest`                 |
| `videoServer.image.pullPolicy`         | Video server image pull policy             | `IfNotPresent`          |
| `videoServer.service.type`             | Video server service type                  | `ClusterIP`              |
| `videoServer.service.port`             | Video server service port                  | `80`                     |
| `videoServer.ingress.enabled`          | Enable ingress for video server            | `false`                  |

### gRPC Services Configuration

| Name                                    | Description                                | Value                    |
| --------------------------------------- | ------------------------------------------ | ------------------------ |
| `grpcClientSide.enabled`               | Enable gRPC client side deployment         | `true`                   |
| `grpcClientSide.replicaCount`          | Number of gRPC client replicas             | `1`                      |
| `grpcServerSide.enabled`               | Enable gRPC server side deployment         | `true`                   |
| `grpcServerSide.replicaCount`          | Number of gRPC server replicas             | `1`                      |

### Persistence Configuration

| Name                                    | Description                                | Value                    |
| --------------------------------------- | ------------------------------------------ | ------------------------ |
| `persistence.videos.enabled`          | Enable persistent storage for videos       | `true`                   |
| `persistence.videos.size`              | Size of video storage                      | `10Gi`                   |
| `persistence.logs.enabled`             | Enable persistent storage for logs         | `true`                   |
| `persistence.logs.size`                | Size of log storage                        | `1Gi`                    |

### Configuration

| Name                                    | Description                                | Value                    |
| --------------------------------------- | ------------------------------------------ | ------------------------ |
| `config.videoSourceUrl`               | URL for video source                       | `""`                     |
| `config.grpcHost`                      | gRPC host address                          | `10.10.10.40`            |

## Examples

### Basic Installation

```bash
helm install nancy-vr ./nancy-vr-video-streamer
```

### Installation with Custom Values

```bash
helm install nancy-vr ./nancy-vr-video-streamer \
  --set videoServer.ingress.enabled=true \
  --set videoServer.ingress.hosts[0].host=nancy-video.example.com \
  --set persistence.videos.size=50Gi
```

### Installation with Values File

Create a `values-prod.yaml` file:

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
    size: 50Gi
  logs:
    size: 5Gi

config:
  videoSourceUrl: "https://example.com/videos"
  grpcHost: "nancy-grpc.example.com"

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 200m
    memory: 256Mi
```

Then install:

```bash
helm install nancy-vr ./nancy-vr-video-streamer -f values-prod.yaml
```

### Accessing the Application

After installation, you can access the video streaming interface:

1. **Via Port Forward** (for testing):
   ```bash
   kubectl port-forward service/nancy-vr-nancy-vr-video-streamer-video-server 8080:80
   ```
   Then open http://localhost:8080

2. **Via Ingress** (if enabled):
   Access using the configured hostname

3. **Via LoadBalancer** (if enabled):
   ```bash
   kubectl get service nancy-vr-nancy-vr-video-streamer-load-balancer
   ```

### Monitoring and Logs

View application logs:

```bash
# Video server logs
kubectl logs -l app.kubernetes.io/component=video-server

# gRPC client logs
kubectl logs -l app.kubernetes.io/component=grpc-client

# gRPC server logs  
kubectl logs -l app.kubernetes.io/component=grpc-server
```

## Troubleshooting

1. **Check pod status**:
   ```bash
   kubectl get pods -l app.kubernetes.io/name=nancy-vr-video-streamer
   ```

2. **Check service endpoints**:
   ```bash
   kubectl get endpoints
   ```

3. **Check persistent volumes**:
   ```bash
   kubectl get pvc
   ```

4. **Check configuration**:
   ```bash
   kubectl describe configmap nancy-config
   ```
