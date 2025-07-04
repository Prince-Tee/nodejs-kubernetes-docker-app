## **Project Title:** Deploy a Node.js App on Kubernetes Using Docker

---

### **Goal:**

Deploy a basic "Hello World" Node.js app using Kubernetes on Minikube or Docker Desktop Kubernetes cluster.

---

### **Tools Required:**

* Node.js and npm
* Docker
* Kubernetes (Minikube or Docker Desktop with K8s enabled)

![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/enable%20kubernetes%20on%20docker.png)

![screenshot](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/our%20kubernetes%20cluster%20running%20on%20docker.png)

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
![screenshot](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/create%20folder%20and%20initiali.png)

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
![screenshot](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/content%20of%20index%20js.png)

Install express:

```bash
npm install express
```
![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/install%20express%20js.png)

Create a `Dockerfile`:

![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/creating%20dockerfile.png)

```Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```
![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/content%20of%20dockerfile.png)


### **Step 2: Build and Push the Docker Image**

If using Docker Hub:

```bash
docker build -t <your-dockerhub-username>/k8s-node-app .
docker push <your-dockerhub-username>/k8s-node-app
```
![screenshot](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/building%20and%20pushing%20our%20app%20to%20docker%20hub.png)


### **Step 3: Create Kubernetes Deployment YAML**

Create a file named `deployment.yaml`:

[image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/creating%20deployment%20yaml%20file.png)

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
![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/content%20of%20deployment%20yaml.png)

### **Step 4: Create Kubernetes Service YAML**

Create a file named `service.yaml`:

![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/creating%20service%20yaml%20file.png)

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
![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/content%20of%20service%20yaml%20file.png)

### **Step 5: Deploy to Kubernetes**

Start your local Kubernetes cluster (Minikube or Docker Desktop), then:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/applying%20kubectl%20on%20the%20files.png)

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

![screenshot](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/our%20app%20on%20local%20host.png)

### **Step 7: Verify Everything is Running**

```bash
kubectl get pods
kubectl get services
kubectl describe deployment node-app-deployment
```

![image](https://github.com/Prince-Tee/nodejs-kubernetes-docker-app/blob/main/screenshot%20from%20my%20local%20environment/verifying%20that%20everything%20is%20now%20working.png)

### **Concepts We have Practiced:**

* Creating a containerized Node.js app
* Writing a Dockerfile
* Creating a Kubernetes Deployment
* Creating a Kubernetes Service
* Using `kubectl` for deployments and inspections


## Issues Encountered During Kubernetes Node App Deployment

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
