## **Project Title:** Deploy a Node.js App on Kubernetes Using Docker

---

### **Goal:**

Deploy a basic "Hello World" Node.js app using Kubernetes on Minikube or Docker Desktop Kubernetes cluster.

---

### **Tools Required:**

* Node.js and npm
* Docker
* Kubernetes (Minikube or Docker Desktop with K8s enabled)
* kubectl (Kubernetes CLI)
* VS Code or any code editor

---

### **Step-by-Step Guide:**

---

### **Step 1: Create a Simple Node.js App**

```bash
mkdir k8s-node-app && cd k8s-node-app
npm init -y
```

Create a file named `index.js`:

```js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World from Kubernetes!');
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`App listening on port ${port}`);
});
```

Install express:

```bash
npm install express
```

Create a `Dockerfile`:

```Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

---

### **Step 2: Build and Push the Docker Image**

If using Docker Hub:

```bash
docker build -t <your-dockerhub-username>/k8s-node-app .
docker push <your-dockerhub-username>/k8s-node-app
```

---

### **Step 3: Create Kubernetes Deployment YAML**

Create a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-app
        image: <your-dockerhub-username>/k8s-node-app
        ports:
        - containerPort: 3000
```

---

### **Step 4: Create Kubernetes Service YAML**

Create a file named `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  type: NodePort
  selector:
    app: node-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30036
```

---

### **Step 5: Deploy to Kubernetes**

Start your local Kubernetes cluster (Minikube or Docker Desktop), then:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

### **Step 6: Access Your App**

If you're using **Minikube**, run:

```bash
minikube service node-app-service
```

If using **Docker Desktop**, visit:

```
http://localhost:30036
```

You should see:
**"Hello World from Kubernetes!"**

---

### **Step 7: Verify Everything is Running**

```bash
kubectl get pods
kubectl get services
kubectl describe deployment node-app-deployment
```

---

### **Concepts You’ve Practiced:**

* Creating a containerized Node.js app
* Writing a Dockerfile
* Creating a Kubernetes Deployment
* Creating a Kubernetes Service
* Using `kubectl` for deployments and inspections


# Issues Encountered During Kubernetes Node App Deployment

## 1️⃣ Docker Build and Push Errors

### **Error:**

```
ERROR: docker: 'docker buildx build' requires 1 argument
```

### **Cause:**

* Initially ran `docker build -t princetee/k8s-node-app` without specifying a context (`.` at the end).

### **Fix:**

* Re-ran with context:

  ```bash
  docker build -t princetee/k8s-node-app .
  ```

### **Error:**

```
The push refers to repository [docker.io/princetee/k8s-node-app]
tag does not exist: princetee/k8s-node-app:latest
```

### **Cause:**

* Tried to push without successfully building the image first.

### **Fix:**

* Ensured image was built and tagged before pushing.

  ```bash
  docker push princetee/k8s-node-app
  ```

---

## 2️⃣ YAML File Parsing Errors

### **Error:**

```
error: error parsing deployment.yaml: error converting YAML to JSON: yaml: line 7: did not find expected key
```

### **Cause:**

* Improper indentation in `deployment.yaml` and `service.yaml`.
* Extra or missing spaces in YAML caused the parser to fail.

### **Fix:**

* Corrected YAML indentation.
* Used spaces instead of tabs.

---

### **Error:**

```
error: error parsing deployment.yaml: error converting YAML to JSON: yaml: line 7: mapping values are not allowed in this context
```

### **Cause:**

* YAML formatting issue due to inconsistent indentation.

### **Fix:**

* Fixed indentation using `nano` and verified with `cat` + `expand` to replace tabs with spaces.
```bash
 cat deployment.yaml | expand -t 2 > deployment.yaml
cat service.yaml | expand -t 2 > service.yaml
```
---

## 3️⃣ Kubernetes Cluster Connectivity Error

### **Error:**

```
error validating "deployment.yaml": error validating data: failed to download openapi: Get "https://kubernetes.docker.internal:6443/openapi/v2?timeout=32s": EOF
```

### **Cause:**

* Docker Desktop Kubernetes cluster was not running or reachable.

### **Fix:**

* Ensured Docker Desktop was running.
* Restarted Docker Desktop Kubernetes.

---

## ✅ Successful Deployment

After fixing YAML formatting and ensuring the Kubernetes cluster was accessible, the deployment succeeded:

```
deployment.apps/node-app-deployment created
service/node-app-service created
```
