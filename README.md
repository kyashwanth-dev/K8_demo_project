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

**Note:** This works automatically on **Docker Desktop only**. For other platforms, see Method 3.

### Method 3: Using LoadBalancer External IP (Cloud providers and Minikube)

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

## Architecture

- **MongoDB**: Database running on port 27017 (ClusterIP service)
- **Mongo Express**: Web UI running on port 8081 (LoadBalancer service)
- **ConfigMap**: Stores MongoDB service URL
- **Secret**: Stores MongoDB credentials (base64 encoded)

### 🔍 Understanding Service Exposure

The `mongo-express-service` is configured as type `LoadBalancer`:
- **LoadBalancer on Docker Desktop:** Automatically exposes to `http://localhost:8081` ✅
- **LoadBalancer on Cloud:** Creates external load balancer with public IP
- **LoadBalancer on Minikube:** Requires `minikube tunnel` to get external IP
- **kind/k3s clusters:** LoadBalancer may remain pending; use port-forward instead

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
