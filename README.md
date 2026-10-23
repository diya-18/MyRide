# Ride Sharing Platform

A full-stack, microservices-based ride-sharing application built with Go, Next.js, RabbitMQ, MongoDB, Docker, and Kubernetes.

## Technologies

- Go 1.23+
- Next.js and React
- RabbitMQ
- MongoDB
- Docker
- Kubernetes
- Minikube
- kubectl

## Prerequisites

Install the following tools:

- [Go](https://go.dev/dl/)
- [Node.js](https://nodejs.org/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)

Verify the installations:

```bash
go version
node --version
npm --version
docker --version
kubectl version --client
minikube version
```

## Run the Frontend Only

To run only the Next.js frontend:

```bash
cd web
npm install
npm run dev
```

Open the application in your browser:

```text
http://localhost:3000
```

## Run the Full Application

### 1. Start Docker and Minikube

Start Docker Desktop, then run:

```bash
minikube start --driver=docker
```

Check the Kubernetes node:

```bash
kubectl get nodes
```

The node should have a `Ready` status.

### 2. Go to the Project Root

```bash
cd ~/Desktop/my-ride-sharing
```

### 3. Create the Local Secrets File

Copy the secrets template:

```bash
cp infra/development/k8s/secrets.template.yaml \
   infra/development/k8s/secrets.yaml
```

Open the file:

```bash
notepad infra/development/k8s/secrets.yaml
```

Update the following values:

```yaml
stringData:
  uri: "mongodb://host.minikube.internal:27017"
```

For MongoDB Atlas, use a URI similar to:

```yaml
stringData:
  uri: "mongodb+srv://USERNAME:PASSWORD@cluster.example.mongodb.net/ride-sharing"
```

Add a Stripe test key:

```yaml
stripe-secret-key: "sk_test_your_key_here"
```

Do not commit `secrets.yaml` or real credentials to GitHub.

### 4. Build the Go Services

Create the build directory:

```bash
mkdir -p build
```

Build the API Gateway:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
go build -o build/api-gateway ./services/api-gateway
```

Build the Trip Service:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
go build -o build/trip-service ./services/trip-service/cmd/main.go
```

Build the Driver Service:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
go build -o build/driver-service ./services/driver-service
```

Build the Payment Service:

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
go build -o build/payment-service ./services/payment-service/cmd/main.go
```

Check the generated binaries:

```bash
ls -lh build
```

### 5. Build the Docker Images

Build the images inside Minikube:

```bash
minikube image build -t ride-sharing/api-gateway \
  -f infra/development/docker/api-gateway.Dockerfile .
```

```bash
minikube image build -t ride-sharing/trip-service \
  -f infra/development/docker/trip-service.Dockerfile .
```

```bash
minikube image build -t ride-sharing/driver-service \
  -f infra/development/docker/driver-service.Dockerfile .
```

```bash
minikube image build -t ride-sharing/payment-service \
  -f infra/development/docker/payment-service.Dockerfile .
```

```bash
minikube image build -t ride-sharing/web \
  -f infra/development/docker/web.Dockerfile .
```

Verify the images:

```bash
minikube image ls | grep ride-sharing
```

### 6. Apply the Kubernetes Configuration

Apply the application configuration:

```bash
kubectl apply -f infra/development/k8s/app-config.yaml
```

Apply the local secrets:

```bash
kubectl apply -f infra/development/k8s/secrets.yaml
```

Start RabbitMQ:

```bash
kubectl apply -f infra/development/k8s/rabbitmq-deployment.yaml
```

Start Jaeger:

```bash
kubectl apply -f infra/development/k8s/jaeger.yaml
```

Apply the API Gateway:

```bash
kubectl apply -f infra/development/k8s/api-gateway-deployment.yaml
```

Apply the Trip Service:

```bash
kubectl apply -f infra/development/k8s/trip-service-deployment.yaml
```

Apply the Driver Service:

```bash
kubectl apply -f infra/development/k8s/driver-service-deployment.yaml
```

Apply the Payment Service:

```bash
kubectl apply -f infra/development/k8s/payment-service-deployment.yaml
```

Apply the frontend:

```bash
kubectl apply -f infra/development/k8s/web-deployment.yaml
```

### 7. Check the Application Status

Check the running pods:

```bash
kubectl get pods
```

Check the services:

```bash
kubectl get services
```

All application pods should eventually show:

```text
1/1 Running
```

Check deployment status:

```bash
kubectl rollout status deployment/api-gateway
kubectl rollout status deployment/trip-service
kubectl rollout status deployment/driver-service
kubectl rollout status deployment/payment-service
kubectl rollout status deployment/web
```

If a pod fails, view its logs:

```bash
kubectl logs <pod-name>
```

View detailed pod information:

```bash
kubectl describe pod <pod-name>
```

### 8. Forward the API Gateway Port

Open a new terminal and run:

```bash
kubectl port-forward service/api-gateway 8081:8081
```

The API Gateway will be available at:

```text
http://localhost:8081
```

The WebSocket endpoints are:

```text
ws://localhost:8081/ws/riders
ws://localhost:8081/ws/drivers
```

Keep this terminal open.

### 9. Run the Frontend

You can run the frontend directly with npm:

```bash
cd web
npm install
npm run dev
```

Open the frontend:

```text
http://localhost:3000
```

The frontend connects to the API Gateway through:

```text
http://localhost:8081
```

## Monitoring Tools

### RabbitMQ Management Dashboard

Forward the RabbitMQ management port:

```bash
kubectl port-forward service/rabbitmq 15672:15672
```

Open:

```text
http://localhost:15672
```

Default local credentials:

```text
Username: guest
Password: guest
```

### Jaeger Dashboard

Forward the Jaeger port:

```bash
kubectl port-forward service/jaeger 16686:16686
```

Open:

```text
http://localhost:16686
```

## Stop the Application

Stop the frontend and port-forwarding processes with:

```text
Ctrl+C
```

Stop Minikube:

```bash
minikube stop
```

To remove the Kubernetes resources:

```bash
kubectl delete -f infra/development/k8s
```

## Security

Do not commit passwords, API keys, database credentials, or other secrets to GitHub.

Add the following entries to `.gitignore`:

```gitignore
# Local secrets and environment files
infra/development/k8s/secrets.yaml
.env
.env.local
web/.env.local

# Compiled binaries
build/
```

Check whether any sensitive files are tracked:

```bash
git ls-files | grep -E '(^|/)(secrets\.yaml|\.env|\.env\.local)$'
```

This command should not list local secret files.
.
