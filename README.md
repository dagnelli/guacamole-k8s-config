# Guacamole Kubernetes Configuration

Apache Guacamole deployment for Kubernetes cluster.

## Architecture

Apache Guacamole on Kubernetes consists of three main components:

### Core Components
- **PostgreSQL** - Database backend for storing connection configurations and user data
- **guacd** - The Guacamole proxy daemon that handles VNC, RDP, SSH, and Telnet connections
- **Guacamole Web App** - Tomcat-based web interface with WebSocket support

### Additional Components
- **OAuth2 Proxy** - GitHub-based authentication
- **Ingress Controller** - NGINX ingress with SSL termination via cert-manager
- **Cert-Manager** - Automatic SSL certificate management with Let's Encrypt

## Deployment

### Quick Deploy
```bash
# Apply all manifests in order
kubectl apply -f manifests/00-namespace.yaml
kubectl apply -f manifests/01-secrets.yaml
kubectl apply -f manifests/02-configmap-initdb.yaml
kubectl apply -f manifests/03-postgres/
kubectl apply -f manifests/04-guacd/
kubectl apply -f manifests/05-guacamole/
kubectl apply -f manifests/06-oauth2-proxy/
kubectl apply -f manifests/07-ingress/
```

### Verify Deployment
```bash
# Check pod status
kubectl get pods -n guacamole

# Check services
kubectl get svc -n guacamole

# Check ingress and certificate
kubectl get ingress,certificate -n guacamole
```

## Access

- **URL:** https://guacamole.coder.dagnelli.net
- **Authentication:** GitHub OAuth2

## Configuration

### Database Schema
The PostgreSQL database is initialized using the official Guacamole init script extracted from the Docker image.

### OAuth2 Setup
GitHub OAuth App configuration:
- Homepage URL: `https://guacamole.coder.dagnelli.net`
- Authorization callback URL: `https://guacamole.coder.dagnelli.net/oauth2/callback`

## Maintenance

### Database Backup
```bash
kubectl exec -n guacamole postgres-0 -- \
  pg_dump -U guacamole_user guacamole_db > backup-$(date +%Y%m%d).sql
```

### View Logs
```bash
# Guacamole app logs
kubectl logs -n guacamole deployment/guacamole --tail=50

# guacd logs
kubectl logs -n guacamole deployment/guacd --tail=50

# PostgreSQL logs
kubectl logs -n guacamole statefulset/postgres --tail=50
```

## Troubleshooting

### Check Service Connectivity
```bash
# Run a debug pod to test internal connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -n guacamole -- sh
# Inside the pod:
nc -zv postgres-service 5432
nc -zv guacd-service 4822
nc -zv guacamole-service 8080
```

### Force Restart
```bash
kubectl rollout restart deployment/guacamole -n guacamole
kubectl rollout restart deployment/guacd -n guacamole
```

## Resource Requirements

### Minimum
- PostgreSQL: 256Mi RAM, 250m CPU, 10Gi storage
- guacd: 128Mi RAM, 100m CPU
- Guacamole: 256Mi RAM, 250m CPU
- OAuth2 Proxy: 64Mi RAM, 50m CPU

### Production Recommended
- PostgreSQL: 1Gi RAM, 1 CPU, 20Gi SSD storage
- guacd: 512Mi RAM, 500m CPU (horizontal scaling for more connections)
- Guacamole: 1Gi RAM, 1 CPU (horizontal scaling for more users)
- OAuth2 Proxy: 256Mi RAM, 200m CPU

## Supported Protocols

Guacamole supports connections via:
- VNC
- RDP
- SSH
- Telnet
- Kubernetes (kubectl exec)

## License

Apache License 2.0

---

**Deployed:** 2025-10-24
**Maintained By:** Daniele Agnelli
