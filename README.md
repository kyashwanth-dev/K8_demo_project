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

### Method 1: Using NodePort (Recommended for local clusters)

The service is configured with NodePort `30000`. Access it using:

```bash
# For Minikube
minikube ip
# Then open: http://<minikube-ip>:30000

# For other local Kubernetes (Docker Desktop, kind, k3s)
# Open: http://localhost:30000
```

**Direct URLs:**
- Minikube: `http://<minikube-ip>:30000`
- Docker Desktop: `http://localhost:30000`
- kind: `http://localhost:30000`

### Method 2: Using LoadBalancer (For cloud providers)

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

### Method 3: Using Port Forwarding (Works everywhere)

```bash
# Forward local port 8081 to the service
kubectl port-forward service/mongo-express-service 8081:8081

# Then open: http://localhost:8081
```

## Credentials

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

- **Minikube:** Run `minikube tunnel` in a separate terminal
- **Local clusters:** Use NodePort method instead (port 30000)
- **Cloud providers:** Check your cloud-specific load balancer configuration

## Architecture

- **MongoDB**: Database running on port 27017 (ClusterIP service)
- **Mongo Express**: Web UI running on port 8081 (LoadBalancer service with NodePort 30000)
- **ConfigMap**: Stores MongoDB service URL
- **Secret**: Stores MongoDB credentials (base64 encoded)

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
