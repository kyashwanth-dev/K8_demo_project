# MongoDB Cluster Configuration Summary

## Overview
This repository contains Kubernetes configuration files for deploying MongoDB and Mongo Express (web-based MongoDB admin interface).

## Architecture
- **MongoDB**: Single-instance deployment (not a replica set cluster)
- **Mongo Express**: Web UI for MongoDB administration
- **Configuration**: Using ConfigMap for database URL
- **Secrets**: Using Kubernetes Secrets for credentials

## Configuration Files

### 1. secret.yml
**Purpose**: Stores MongoDB credentials as base64-encoded secrets

**Resources**:
- Secret: `mongodb-secret`
  - `mongo-root-username`: master (base64: bWFzdGVy)
  - `mongo-root-password`: halwa (base64: aGFsd2E=)
  - `mongo-express-username`: admin (base64: YWRtaW4=)
  - `mongo-express-password`: pass (base64: cGFzcw==)

**Security Note**: In production, use stronger passwords and consider using external secret management solutions like HashiCorp Vault or AWS Secrets Manager.

### 2. mongo-configmap.yml
**Purpose**: Stores non-sensitive configuration data

**Resources**:
- ConfigMap: `mongodb-configmap`
  - `database_url`: mongodb-service

**Note**: This stores the service name for MongoDB, which Mongo Express uses to connect.

### 3. mongo.yml
**Purpose**: MongoDB database deployment and service

**Resources**:
- **Deployment**: `mongodb-deployment`
  - Replicas: 1
  - Image: mongo (latest)
  - Container Port: 27017
  - Resource Limits:
    - Memory: 256Mi (request) / 512Mi (limit)
    - CPU: 250m (request) / 500m (limit)
  - Environment Variables:
    - `MONGO_INITDB_ROOT_USERNAME` (from secret)
    - `MONGO_INITDB_ROOT_PASSWORD` (from secret)
  - Health Checks:
    - Liveness Probe: TCP socket check on port 27017 every 10s (starts after 30s)
    - Readiness Probe: TCP socket check on port 27017 every 10s (starts after 5s)

- **Service**: `mongodb-service`
  - Type: ClusterIP (default)
  - Port: 27017
  - Selector: app=mongodb

### 4. mongo-express.yml
**Purpose**: Mongo Express web UI deployment and service

**Resources**:
- **Deployment**: `mongo-express`
  - Replicas: 1
  - Image: mongo-express (latest)
  - Container Port: 8081
  - Resource Limits:
    - Memory: 256Mi (request) / 512Mi (limit)
    - CPU: 250m (request) / 500m (limit)
  - Environment Variables:
    - `ME_CONFIG_MONGODB_ADMINUSERNAME` (from secret)
    - `ME_CONFIG_MONGODB_ADMINPASSWORD` (from secret)
    - `ME_CONFIG_MONGODB_SERVER` (from configmap)
    - `ME_CONFIG_BASICAUTH_USERNAME` (from secret)
    - `ME_CONFIG_BASICAUTH_PASSWORD` (from secret)
  - Health Checks:
    - Liveness Probe: HTTP GET on / port 8081 every 10s (starts after 30s)
    - Readiness Probe: HTTP GET on / port 8081 every 10s (starts after 5s)

- **Service**: `mongo-express-service`
  - Type: LoadBalancer
  - Port: 8081
  - NodePort: 30000
  - Selector: app=mongo-express

## Deployment Order

To deploy this MongoDB setup, apply the resources in the following order:

```bash
# 1. Create the secret first (required by deployments)
kubectl apply -f secret.yml

# 2. Create the configmap (required by mongo-express)
kubectl apply -f mongo-configmap.yml

# 3. Deploy MongoDB
kubectl apply -f mongo.yml

# 4. Deploy Mongo Express
kubectl apply -f mongo-express.yml
```

Or apply all at once:
```bash
kubectl apply -f secret.yml,mongo-configmap.yml,mongo.yml,mongo-express.yml
```

## Validation Checklist

### ✅ Configuration Correctness
- [x] All YAML files are properly formatted (yamllint compliant)
- [x] Kubernetes resource definitions are valid
- [x] Secret references are correct
- [x] ConfigMap references are correct
- [x] Service names match between resources
- [x] Health checks are configured for both deployments
- [x] Resource limits are defined
- [x] Port configurations are correct

### ✅ Reference Validation
- [x] MongoDB deployment references `mongodb-secret` for credentials
- [x] Mongo Express deployment references `mongodb-secret` for credentials
- [x] Mongo Express deployment references `mongodb-configmap` for database URL
- [x] ConfigMap `database_url` matches MongoDB service name (`mongodb-service`)

### ✅ Best Practices Applied
- [x] Document start markers (`---`) added
- [x] Consistent indentation (2 spaces)
- [x] No trailing spaces
- [x] Newlines at end of files
- [x] Health probes configured (liveness and readiness)
- [x] Resource requests and limits defined
- [x] Proper label selectors

## Important Notes

### Current Setup: Single Instance
This configuration deploys a **single MongoDB instance**, which is suitable for:
- Development environments
- Testing
- Learning Kubernetes
- Non-critical applications

### Production Considerations

For production use, consider the following improvements:

1. **MongoDB Replica Set**
   - Use StatefulSet instead of Deployment
   - Configure 3+ replicas for high availability
   - Set up replica set initialization

2. **Persistent Storage**
   - Add PersistentVolumeClaim for data persistence
   - Data will be lost if pod is deleted without persistent storage

3. **Security Enhancements**
   - Use strong, randomly generated passwords
   - Consider external secret management (HashiCorp Vault, AWS Secrets Manager)
   - Enable TLS/SSL for MongoDB connections
   - Use network policies to restrict access

4. **Monitoring**
   - Add Prometheus exporters for metrics
   - Configure logging and alerting
   - Set up backup strategies

5. **Resource Optimization**
   - Adjust resource limits based on actual workload
   - Use horizontal pod autoscaling if needed

## Verification Commands

After deployment, verify the setup:

```bash
# Check all resources are created
kubectl get all

# Check secrets
kubectl get secret mongodb-secret

# Check configmap
kubectl get configmap mongodb-configmap

# Check MongoDB pod logs
kubectl logs deployment/mongodb-deployment

# Check Mongo Express pod logs
kubectl logs deployment/mongo-express

# Access Mongo Express (if using LoadBalancer)
# Find the external IP
kubectl get service mongo-express-service

# If using NodePort, access via:
# http://<node-ip>:30000
```

## Troubleshooting

### MongoDB Pod Not Starting
- Check logs: `kubectl logs deployment/mongodb-deployment`
- Verify secret exists: `kubectl get secret mongodb-secret`
- Check resource availability: `kubectl describe pod <mongodb-pod>`

### Mongo Express Cannot Connect
- Verify MongoDB service is running: `kubectl get service mongodb-service`
- Check configmap has correct URL: `kubectl get configmap mongodb-configmap -o yaml`
- Verify environment variables: `kubectl describe pod <mongo-express-pod>`
- Check logs: `kubectl logs deployment/mongo-express`

### LoadBalancer Pending
- If using Minikube: Use `minikube tunnel` in a separate terminal
- If cloud provider: Check cloud-specific load balancer setup
- Alternative: Use NodePort (already configured at 30000)

## Conclusion

All MongoDB configuration files are correctly configured and follow Kubernetes best practices. The setup is ready for deployment in a development/testing environment. For production use, implement the additional considerations mentioned above.
