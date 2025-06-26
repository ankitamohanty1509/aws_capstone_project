# aws_capstone_project
# Save Hobby to RDS - Kubernetes Application

A full-stack application that saves user hobbies to AWS RDS MySQL database, deployed on Kubernetes.

## 🏗️ Architecture

- **Frontend**: React.js with Nginx (LoadBalancer)
- **Backend**: Node.js with Express (ClusterIP)
- **Database**: AWS RDS MySQL
- **Deployment**: Kubernetes with Docker containers

## 📁 Project Structure

```
demo/
├── new-app/
│   ├── bk/                      # Backend (Node.js)
│   │   ├── index.js            # Main server with MySQL connection pooling
│   │   ├── package.json        # Node.js dependencies
│   │   └── Dockerfile          # Backend container configuration
│   └── ft/                     # Frontend (React)
│       ├── src/
│       │   ├── App.js          # Main React component
│       │   └── index.js        # React entry point
│       ├── public/
│       │   └── index.html      # HTML template
│       ├── package.json        # React dependencies
│       ├── nginx.conf          # Nginx config with API proxy
│       └── Dockerfile          # Frontend container configuration
├── backend-deployment.yaml      # Backend Kubernetes deployment
├── backend-service.yaml         # Backend service (ClusterIP)
├── frontend-deployment.yaml     # Frontend Kubernetes deployment
├── frontend-service.yaml        # Frontend service (LoadBalancer)
├── kubernetes-deployment-guide.md # Deployment instructions
└── README.md                    # This file
```

## 🚀 Features

- **Save Hobbies**: Users can input name and hobby
- **Data Persistence**: Stores data in AWS RDS MySQL
- **Connection Pooling**: Improved MySQL connection handling
- **Error Handling**: Graceful error handling with retry logic
- **Health Checks**: Backend health endpoint with DB status
- **Scalable**: 2 replicas each for frontend and backend

## 🛠️ Setup Instructions

### Prerequisites
- Kubernetes cluster (EKS recommended)
- AWS RDS MySQL instance
- Docker images pushed to registry
- kubectl configured

### 1. Database Setup
```bash
# Create database secret (replace with your credentials)
cp db-secret.template.yaml db-secret.yaml
# Edit db-secret.yaml with base64 encoded values
kubectl apply -f db-secret.yaml
```

### 2. Deploy Application
```bash
# Deploy backend
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml

# Deploy frontend
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

### 3. Access Application
```bash
# Get frontend URL
kubectl get service frontend-service
```

## 🔧 Technical Details

### Backend Features
- **Connection Pooling**: MySQL2 pool with 5 connections
- **Prepared Statements**: Secure SQL execution
- **Error Handling**: Connection timeout and retry logic
- **Health Endpoint**: `/health` with database status
- **API Endpoints**:
  - `POST /api/save` - Save hobby data  
  - `GET /api/users` - Get all users
  - `GET /api/count` - Get user count

### Frontend Features
- **React Components**: Modern functional components
- **API Integration**: Axios for HTTP requests
- **Error Handling**: User-friendly error messages
- **Responsive Design**: Clean UI for hobby input

### Database Schema
```sql
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  hobby VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 🔍 Troubleshooting

### Common Issues

1. **"Failed to save data"**
   - Check if RDS instance is running
   - Verify security groups allow EKS access
   - Check database credentials in secret

2. **Connection Timeouts**
   - Ensure RDS is in "available" status
   - Check network connectivity from pods
   - Verify security group rules

3. **Pod Crashes**
   - Check pod logs: `kubectl logs <pod-name>`
   - Verify image availability
   - Check resource limits

### Debug Commands
```bash
# Check pod status
kubectl get pods

# Check logs
kubectl logs -l app=backend
kubectl logs -l app=frontend

# Test API directly
kubectl exec -it <frontend-pod> -- curl http://backend-service:5000/health

# Check database connectivity
kubectl exec -it <backend-pod> -- node -e "console.log('DB Host:', process.env.DB_HOST)"
```

## 📊 Monitoring

- **Health Check**: `GET /health` endpoint
- **Pod Status**: Monitor via `kubectl get pods`
- **Database**: Check RDS console for connection metrics
- **LoadBalancer**: Monitor via AWS ELB console

## 🛡️ Security

- Database credentials stored in Kubernetes secrets
- Prepared statements prevent SQL injection
- CORS enabled for cross-origin requests
- Network policies can be added for additional security

## 🎯 Success Criteria

✅ **Working Application**: Data saves successfully to RDS  
✅ **Error Handling**: Graceful handling of connection issues  
✅ **Scalability**: Multiple replicas running smoothly  
✅ **Monitoring**: Health checks and logging in place  

## 📝 Recent Fixes

- **Connection Pooling**: Improved MySQL connection reliability
- **RDS Status**: Fixed issue with stopped database instance
- **Error Messages**: Better user feedback for connection issues
- **Health Checks**: Database connectivity monitoring

## ✅ Verification & Testing

### 1. Check Application Status
```bash
# Check if all pods are running
kubectl get pods -l app=backend
kubectl get pods -l app=frontend

# Check services
kubectl get services

# Get frontend external IP/URL
kubectl get service frontend-service
```

### 2. Test Backend Health & RDS Connection
```bash
# Method 1: Using kubectl port-forward
kubectl port-forward service/backend-service 8080:5000

# In another terminal, test health endpoint
curl http://localhost:8080/health

# Expected response for healthy system:
# {"status":"Backend and database are running","timestamp":"2024-01-01T00:00:00.000Z"}

# Method 2: From within a pod
kubectl exec -it deployment/frontend -- curl http://backend-service:5000/health
```

### 3. Test Backend API Endpoints
```bash
# Test user count endpoint
curl http://localhost:8080/api/count

# Test get all users endpoint
curl http://localhost:8080/api/users

# Test save data endpoint
curl -X POST http://localhost:8080/api/save \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","hobby":"Reading"}'

# Expected response:
# {"message":"Data saved successfully!","id":1,"data":{"name":"John Doe","hobby":"Reading"}}
```

### 4. Verify RDS Database Connection
```bash
# Check backend logs for DB connection status
kubectl logs -l app=backend | grep -i "mysql\|database\|connection"

# Look for these success messages:
# "Connected to MySQL successfully"
# "Users table ready"

# Check for connection errors:
kubectl logs -l app=backend | grep -i "error\|failed"
```

### 5. Test Complete Data Flow
```bash
# Step 1: Get current user count
curl http://localhost:8080/api/count

# Step 2: Add a new user
curl -X POST http://localhost:8080/api/save \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","hobby":"Testing"}'

# Step 3: Verify count increased
curl http://localhost:8080/api/count

# Step 4: Check user appears in list
curl http://localhost:8080/api/users
```

### 6. Frontend Application Testing
```bash
# Get frontend URL
FRONTEND_URL=$(kubectl get service frontend-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Frontend URL: http://$FRONTEND_URL"

# Test frontend accessibility
curl -I http://$FRONTEND_URL

# Check if API calls work through frontend
curl -X POST http://$FRONTEND_URL/api/save \
  -H "Content-Type: application/json" \
  -d '{"name":"Frontend Test","hobby":"UI Testing"}'
```

## 🔍 RDS Verification Commands

### Check RDS Instance Status
```bash
# AWS CLI - Check RDS instance status
aws rds describe-db-instances --db-instance-identifier YOUR_RDS_INSTANCE_ID

# Look for "DBInstanceStatus": "available"
```

### Direct Database Connection Test
```bash
# Connect to RDS from a pod (if mysql client is available)
kubectl exec -it deployment/backend -- /bin/bash

# Inside the pod:
mysql -h $DB_HOST -u $DB_USER -p$DB_PASSWORD $DB_NAME

# Check if users table exists and has data:
# SHOW TABLES;
# SELECT * FROM users;
# SELECT COUNT(*) FROM users;
```

### Verify Database Connectivity
```bash
# Check if backend can reach RDS
kubectl exec -it deployment/backend -- nslookup $DB_HOST

# Test port connectivity
kubectl exec -it deployment/backend -- nc -zv $DB_HOST 3306
```

## 🚨 Common Verification Issues & Solutions

### Backend Health Check Fails
```bash
# Check backend logs
kubectl logs -l app=backend --tail=50

# Common issues:
# 1. DB credentials incorrect - check secret
kubectl get secret db-secret -o yaml

# 2. RDS security group - ensure EKS has access
# 3. RDS in wrong VPC/subnet
```

### Data Not Persisting
```bash
# Test direct save to backend
curl -X POST http://localhost:8080/api/save \
  -H "Content-Type: application/json" \
  -d '{"name":"Persistence Test","hobby":"Data Testing"}'

# Check if data appears in users list
curl http://localhost:8080/api/users | jq '.users[] | select(.name=="Persistence Test")'
```

### RDS Connection Timeouts
```bash
# Check if RDS is in "available" state
aws rds describe-db-instances --db-instance-identifier YOUR_RDS_INSTANCE_ID \
  --query 'DBInstances[0].DBInstanceStatus'

# Verify security group allows traffic from EKS
# Check VPC and subnet configuration
```

## 📊 Success Indicators

✅ **Backend Health**: `/health` endpoint returns `"Backend and database are running"`  
✅ **RDS Connection**: Backend logs show `"Connected to MySQL successfully"`  
✅ **Data Persistence**: POST to `/api/save` succeeds and data appears in `/api/users`  
✅ **Frontend Access**: LoadBalancer provides external access to React app  
✅ **API Integration**: Frontend can successfully call backend endpoints  

## 🎯 Quick Verification Script

```bash
#!/bin/bash
echo "=== Hobby App Verification ==="

# 1. Check pod status
echo "1. Checking pod status..."
kubectl get pods -l app=backend -o wide
kubectl get pods -l app=frontend -o wide

# 2. Setup port forwarding
echo "2. Setting up port forwarding..."
kubectl port-forward service/backend-service 8080:5000 &
KUBECTL_PID=$!
sleep 3

# 3. Test health
echo "3. Testing backend health..."
curl -s http://localhost:8080/health | jq .

# 4. Test API endpoints
echo "4. Testing API endpoints..."
echo "Current user count:"
curl -s http://localhost:8080/api/count | jq .

echo "Adding test user..."
curl -s -X POST http://localhost:8080/api/save \
  -H "Content-Type: application/json" \
  -d '{"name":"Verification Test","hobby":"System Testing"}' | jq .

echo "New user count:"
curl -s http://localhost:8080/api/count | jq .

# 5. Cleanup
kill $KUBECTL_PID
echo "=== Verification Complete ==="
```

**Status**: ✅ **Working** - Application successfully saves hobbies to RDS! 
