# MongoDB with Mongo Express on Kubernetes

A simple Kubernetes deployment of MongoDB with Mongo Express web-based admin interface.

## Quick Start

1. **Deploy the resources:**
   ```bash
   kubectl apply -f secret.yml
   kubectl apply -f mongo-configmap.yml
   kubectl apply -f mongo.yml
   kubectl apply -f mongo-express.yml
   ```

2. **Access the Mongo Express Dashboard** (see detailed instructions below)

## 🌐 How to Access Mongo Express Dashboard

After deploying the resources, you can access the Mongo Express web interface using one of the following methods:

### Method 1: Using Port Forwarding (✅ Recommended - Works everywhere)

```bash
# Forward local port 8081 to the service
kubectl port-forward service/mongo-express-service 8081:8081

# Then open: http://localhost:8081
```

This is the **most reliable method** and works on all platforms including Docker Desktop, Minikube, kind, and cloud providers.

### Method 2: Using LoadBalancer (Docker Desktop)

The service is configured as type `LoadBalancer`, which Docker Desktop automatically exposes to localhost:

```bash
# On Docker Desktop, the LoadBalancer is automatically accessible at:
# http://localhost:8081
```

**Note:** This works automatically on **Docker Desktop only**. For other platforms, see Method 3 or 4.

### Method 3: Using NodePort (For Minikube and some local clusters)

The service is configured with NodePort `30000`. Access it using:

```bash
# For Minikube
minikube ip
# Then open: http://<minikube-ip>:30000
```

**⚠️ Important NodePort Limitations:**
- **Docker Desktop:** NodePort does **NOT** work on `localhost:30000` because the port opens inside the Kubernetes node container, not on your host. Use Method 1 (port-forward) or Method 2 (LoadBalancer) instead.
- **kind/k3s:** NodePort may require additional configuration to expose to localhost
- **Minikube:** NodePort works at the Minikube VM IP address (use `minikube ip` to find it)

### Method 4: Using LoadBalancer External IP (Cloud providers and Minikube)

If running on a cloud provider (AWS, GCP, Azure):

```bash
# Get the external IP
kubectl get service mongo-express-service

# Wait for EXTERNAL-IP to be assigned (may take a few minutes)
# Then access: http://<EXTERNAL-IP>:8081
```

**For Minikube LoadBalancer:**
```bash
# Run in a separate terminal
minikube tunnel

# Then get the external IP
kubectl get service mongo-express-service
# Access: http://<EXTERNAL-IP>:8081
```

## Credentials

### Mongo Express Web UI Login
- **Username:** `admin`
- **Password:** `pass`

### MongoDB Database
- **Username:** `master`
- **Password:** `halwa`

> ⚠️ **Note:** These are default credentials for development/testing only. Change them in `secret.yml` for production use.

## Verifying Deployment

Check that all resources are running:

```bash
# Check all resources
kubectl get all

# Check if Mongo Express is ready
kubectl get pods -l app=mongo-express

# Check service details
kubectl get service mongo-express-service

# View Mongo Express logs
kubectl logs deployment/mongo-express
```

## Troubleshooting

### Dashboard not loading?

1. **Check if pods are running:**
   ```bash
   kubectl get pods
   ```

2. **Check Mongo Express logs:**
   ```bash
   kubectl logs deployment/mongo-express
   ```

3. **Verify service is created:**
   ```bash
   kubectl get service mongo-express-service
   ```

### Connection refused?

- Ensure MongoDB is running: `kubectl get pods -l app=mongodb`
- Check MongoDB logs: `kubectl logs deployment/mongodb-deployment`
- Verify the services: `kubectl get services`

### LoadBalancer stuck in Pending?

- **Docker Desktop:** LoadBalancer should work automatically at `http://localhost:8081`
- **Minikube:** Run `minikube tunnel` in a separate terminal
- **Local clusters (kind/k3s):** Use port-forward method instead
- **Cloud providers:** Check your cloud-specific load balancer configuration

### Cannot access localhost:30000?

**This is expected behavior on Docker Desktop!** NodePort opens ports on the Kubernetes node (which runs inside a Docker container), not on your host machine. 

**Solution:** Use one of these methods instead:
- **Port forwarding (recommended):** `kubectl port-forward service/mongo-express-service 8081:8081` then access `http://localhost:8081`
- **LoadBalancer (Docker Desktop):** Access `http://localhost:8081` directly
- **Minikube:** Use `minikube ip` to get the IP, then access `http://<minikube-ip>:30000`

## Architecture

- **MongoDB**: Database running on port 27017 (ClusterIP service)
- **Mongo Express**: Web UI running on port 8081 (LoadBalancer service with NodePort 30000)
- **ConfigMap**: Stores MongoDB service URL
- **Secret**: Stores MongoDB credentials (base64 encoded)

### 🔍 Understanding Service Exposure

The `mongo-express-service` is configured as type `LoadBalancer` with NodePort 30000:
- **LoadBalancer on Docker Desktop:** Automatically exposes to `http://localhost:8081` ✅
- **LoadBalancer on Cloud:** Creates external load balancer with public IP
- **LoadBalancer on Minikube:** Requires `minikube tunnel` to get external IP
- **NodePort (30000):** Opens port on Kubernetes nodes, but **NOT** on Docker Desktop host
  - Works on Minikube at `http://<minikube-ip>:30000`
  - Does **NOT** work on Docker Desktop at `localhost:30000` (see troubleshooting)

**Recommendation:** Use `kubectl port-forward` for the most consistent experience across all platforms.

For detailed configuration information, see [MONGODB_CONFIG_SUMMARY.md](MONGODB_CONFIG_SUMMARY.md).

## Clean Up

To remove all resources:

```bash
kubectl delete -f mongo-express.yml
kubectl delete -f mongo.yml
kubectl delete -f mongo-configmap.yml
kubectl delete -f secret.yml
```

## Production Notes

This setup is for **development/testing only**. For production:
- Use strong passwords
- Enable persistent storage (PersistentVolumeClaims)
- Set up MongoDB replica sets
- Configure TLS/SSL
- Implement proper backup strategies
- Use external secret management

See [MONGODB_CONFIG_SUMMARY.md](MONGODB_CONFIG_SUMMARY.md) for detailed production considerations.
