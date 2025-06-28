# Microservices-Task

## Overview
This document provides details on testing various services after running the `docker-compose` file. These services include User, Product, Order, and Gateway Services. Each service has its own endpoints for testing purposes.

---

## Services and Endpoints

### **User Service**
- **Base URL:** `http://localhost:3000`
- **Endpoints:**
  - **List Users:**  
    ```
    curl http://localhost:3000/users
    ```
    Or open in your browser: [http://localhost:3000/users](http://localhost:3000/users)

---

### **Product Service**
- **Base URL:** `http://localhost:3001`
- **Endpoints:**
  - **List Products:**  
    ```
    curl http://localhost:3001/products
    ```
    Or open in your browser: [http://localhost:3001/products](http://localhost:3001/products)

---

### **Order Service**
- **Base URL:** `http://localhost:3002`
- **Endpoints:**
  - **List Orders:**  
    ```
    curl http://localhost:3002/orders
    ```
    Or open in your browser: [http://localhost:3002/orders](http://localhost:3002/orders)

---

### **Gateway Service**
- **Base URL:** `http://localhost:3003/api`
- **Endpoints:**
  - **Users:**  
    ```
    curl http://localhost:3003/api/users
    ```
  - **Products:**  
    ```
    curl http://localhost:3003/api/products
    ```
  - **Orders:**  
    ```
    curl http://localhost:3003/api/orders
    ```

---

## Instructions
1. Start all services using the `docker-compose` file:
   ```
   docker-compose up
   ```
2. Once the services are running, use the above endpoints to verify the functionality.

docker tag microservices-task-k8s-gateway:latest demoteam88/gateway-service:latest
docker push demoteam88/gateway-service:latest

docker tag microservices-task-k8s-user-services:latest demoteam88/user-service:latest
docker push demoteam88/user-service:latest

docker tag microservices-task-k8s-product-services:latest demoteam88/product-service:latest
docker push demoteam88/product-service:latest

docker tag microservices-task-k8s-order-services:latest demoteam88/order-services:latest
docker push demoteam88/order-services:latest

# Microservices Task - Kubernetes Deployment

This project demonstrates the deployment of a microservices architecture on Kubernetes using Minikube. The system consists of four services: User Service, Product Service, Order Service, and Gateway Service.

## Architecture Overview

- **User Service** (Port 3000): Handles user management operations
- **Product Service** (Port 3001): Manages product catalog
- **Order Service** (Port 3002): Processes order transactions
- **Gateway Service** (Port 3003): API gateway with proxy handling

## Prerequisites

- Docker Desktop installed and running
- Minikube installed
- kubectl installed
- curl or similar tool for testing

## Minikube Setup and Configuration

### 1. Start Minikube

```bash
# Start Minikube with Docker driver
minikube start --driver=docker

# Verify Minikube status
minikube status

# Enable necessary addons
minikube addons enable ingress
minikube addons enable dashboard
```

### 2. Configure kubectl Context

```bash
# Set kubectl context to minikube
kubectl config use-context minikube

# Verify cluster connection
kubectl cluster-info
```

### 3. Enable Docker Environment (Optional)

```bash
# Point Docker CLI to Minikube's Docker daemon
eval $(minikube docker-env)
```

## Deployment Process

### 1. Create Namespace

```bash
kubectl apply -f namespace.yml
```

### 2. Deploy Services

Deploy all services in the correct order:

```bash
# Deploy all services
kubectl apply -f services/user-service.yaml
kubectl apply -f services/product-service.yaml
kubectl apply -f services/order-service.yaml
kubectl apply -f services/gateway-service.yaml
```
![alt text](image.png)

### 3. Deploy Applications

```bash
# Deploy all deployments
kubectl apply -f deployments/user-service.yaml
kubectl apply -f deployments/product-service.yaml
kubectl apply -f deployments/order-service.yaml
kubectl apply -f deployments/gateway-service.yaml
```

![alt text](image-1.png)

### 4. Deploy All at Once (Alternative)

```bash
# Deploy everything in one command
kubectl apply -f .
```

## Validation and Testing

### 1. Check Pod Status

```bash
# Check all pods in the namespace
kubectl get pods -n mico-k8s

# Check pod details
kubectl describe pods -n mico-k8s

# Check pod logs
kubectl logs -n mico-k8s <pod-name>
```

Expected output:
```
NAME                              READY   STATUS    RESTARTS   AGE
gateway-service-xxxxxxxxx-xxxxx   1/1     Running   0          2m
order-service-xxxxxxxxx-xxxxx     1/1     Running   0          2m
product-service-xxxxxxxxx-xxxxx   1/1     Running   0          2m
user-service-xxxxxxxxx-xxxxx      1/1     Running   0          2m
```

### 2. Check Services

```bash
# List all services
kubectl get services -n mico-k8s

# Get service details
kubectl describe service <service-name> -n mico-k8s
```

### 3. Service Communication Testing

#### Method 1: Using kubectl port-forward

```bash
# Port-forward Gateway Service
kubectl port-forward -n mico-k8s service/gateway-service 8080:3003

# Test gateway endpoint
curl http://localhost:8080/health

# Port-forward individual services
kubectl port-forward -n mico-k8s service/user-service 8081:3000
kubectl port-forward -n mico-k8s service/product-service 8082:3001
kubectl port-forward -n mico-k8s service/order-service 8083:3002
```

#### Method 2: Inter-service Communication Test

```bash
# Create a test pod for internal testing
kubectl run test-pod --image=curlimages/curl -n mico-k8s --rm -it --restart=Never -- sh

# Inside the test pod, test service communication:
curl http://user-service:3000/health
curl http://product-service:3001/health
curl http://order-service:3002/health
curl http://gateway-service:3003/health
```

#### Method 3: Using Minikube Service

```bash
# Expose service via Minikube (creates tunnel)
minikube service gateway-service -n mico-k8s --url

# This will provide a URL like: http://192.168.49.2:30001
# Use this URL to test the service externally
```

### 4. Log Analysis

```bash
# View logs for specific services
kubectl logs -n mico-k8s deployment/user-service
kubectl logs -n mico-k8s deployment/product-service
kubectl logs -n mico-k8s deployment/order-service
kubectl logs -n mico-k8s deployment/gateway-service

# Follow logs in real-time
kubectl logs -n mico-k8s deployment/gateway-service -f

# View logs from all containers
kubectl logs -n mico-k8s --selector=app=gateway-service
```

## Health Check Endpoints

Each service provides health check endpoints:

- **Liveness Probe**: `/health` - Indicates if the service is running
- **Readiness Probe**: `/ready` - Indicates if the service is ready to accept traffic

## Troubleshooting Tips

### Common Issues and Solutions

#### 1. Pod Not Starting

```bash
# Check pod events
kubectl describe pod <pod-name> -n mico-k8s

# Check if images are available
kubectl get events -n mico-k8s --sort-by='.lastTimestamp'
```

#### 2. Service Discovery Issues

```bash
# Verify service endpoints
kubectl get endpoints -n mico-k8s

# Check DNS resolution
kubectl run dns-test --image=busybox -n mico-k8s --rm -it --restart=Never -- nslookup user-service
```

#### 3. Resource Issues

```bash
# Check resource usage
kubectl top pods -n mico-k8s
kubectl top nodes

# Check resource quotas
kubectl describe namespace mico-k8s
```

#### 4. Network Issues

```bash
# Test network connectivity between pods
kubectl exec -n mico-k8s <source-pod> -- ping <target-service>

# Check network policies
kubectl get networkpolicies -n mico-k8s
```

### Debugging Commands

```bash
# Get detailed cluster information
kubectl get all -n mico-k8s

# Check cluster events
kubectl get events -n mico-k8s --sort-by='.lastTimestamp'

# Delete specific pods if they're stuck (deployment will recreate them)
kubectl delete pod <pod-name> -n mico-k8s

# Restart deployment if needed
kubectl rollout restart deployment/<deployment-name> -n mico-k8s

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=2 -n mico-k8s
```

## Pod Readiness Issues (0/1 Status)

### Diagnosing 0/1 Ready Status

When pods show "0/1" in the READY column, it means the readiness probe is failing:

```bash
# Check specific pod details
kubectl describe pod <pod-name> -n mico-k8s

# Check pod logs for errors
kubectl logs <pod-name> -n mico-k8s

# Check if the service endpoints are ready
kubectl get endpoints -n mico-k8s

# View real-time pod events
kubectl get events -n mico-k8s --field-selector involvedObject.name=<pod-name>
```

### Common Causes and Solutions

#### 1. Health Check Endpoints Missing
If your services don't have `/health` and `/ready` endpoints, temporarily remove the probes:

```bash
# Edit deployment to remove probes temporarily
kubectl edit deployment <deployment-name> -n mico-k8s

# Or patch the deployment
kubectl patch deployment <deployment-name> -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","livenessProbe":null,"readinessProbe":null}]}}}}'
```

#### 2. Service Taking Too Long to Start
Increase probe delays:

```bash
# Check current probe configuration
kubectl get deployment <deployment-name> -n mico-k8s -o yaml | grep -A 10 Probe

# Increase initialDelaySeconds if needed
kubectl patch deployment <deployment-name> -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","readinessProbe":{"initialDelaySeconds":60}}]}}}}'
```

#### 3. Port Issues
Verify the service is listening on the expected port:

```bash
# Execute into the pod to check
kubectl exec -it <pod-name> -n mico-k8s -- sh

# Inside the pod, check if service is running
netstat -tlnp
curl localhost:3000/health  # Replace with appropriate port
```

#### 4. Image Issues
Check if the Docker image is correct:

```bash
# Verify image can be pulled
kubectl describe pod <pod-name> -n mico-k8s | grep -A 5 "Events:"

# Check image logs
kubectl logs <pod-name> -n mico-k8s --previous
```

### Quick Fix Commands

```bash
# Remove health probes temporarily for testing
kubectl patch deployment user-service -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"user-service","livenessProbe":null,"readinessProbe":null}]}}}}'

kubectl patch deployment product-service -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"product-service","livenessProbe":null,"readinessProbe":null}]}}}}'

kubectl patch deployment order-service -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"order-service","livenessProbe":null,"readinessProbe":null}]}}}}'

kubectl patch deployment gateway-service -n mico-k8s -p='{"spec":{"template":{"spec":{"containers":[{"name":"gateway-service","livenessProbe":null,"readinessProbe":null}]}}}}'

# Wait for pods to restart and check status
kubectl get pods -n mico-k8s -w
```

### Alternative Health Check Configuration

If services don't have dedicated health endpoints, use TCP socket checks:

```yaml
# Example for TCP-based health checks
livenessProbe:
  tcpSocket:
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  tcpSocket:
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5
```

# ...existing code...
