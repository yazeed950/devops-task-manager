# DevOps Task Manager

This project is a DevOps deployment for a Task Manager application.

The application has:

- Frontend
- Backend API
- PostgreSQL Database

The project uses:

- Docker
- AWS ECR
- Kubernetes
- Minikube
- AWS EKS
- Jenkins
- GitHub Webhook
- ngrok

---

## Docker Local Run

The project was tested locally using Docker Compose.

```bash
docker compose up --build
```

![Docker local](screenshots/01-docker-local.png)

---

## AWS ECR

Two Docker images were pushed to AWS ECR:

- Backend image
- Frontend image

![ECR backend](screenshots/02-ecr-backend.png)

![ECR frontend](screenshots/03-ecr-frontend.png)

---

## AWS EKS

An EKS cluster was created and the nodes were Ready.

![EKS nodes](screenshots/04-eks-nodes-ready.png)

---

## Kubernetes Deployment

The application was deployed to Kubernetes.

The deployment includes:

- 3 Backend pods
- 1 Frontend pod
- 1 PostgreSQL pod
- Services
- Ingress
- Secret
- Persistent Volume Claim

```bash
kubectl get pods,svc,ingress
```

![Kubernetes resources](screenshots/05-kubectl-pods-svc-ingress.png)

---

## Application Running

The application was exposed using Ingress and opened successfully in the browser.

![App running](screenshots/06-app-running-ingress.png)

---

## PostgreSQL Persistence

PostgreSQL uses PVC to keep the data after pod restart.

```bash
kubectl get pvc
```

![Postgres PVC](screenshots/07-postgres-pvc.png)

PostgreSQL pod restart test:

![Postgres restart](screenshots/08-postgres-restart-test.png)

---

## Jenkins Pipeline

Jenkins pipeline builds the Docker images, pushes them to ECR, and deploys the new version to Kubernetes.

![Jenkins success](screenshots/09-jenkins-pipeline-success.png)

---

## GitHub Webhook

GitHub Webhook was configured to trigger Jenkins automatically after a push.

![Jenkins webhook](screenshots/10-jenkins-webhook-success.png)

---

## Project Structure

```text
backend/
frontend/
k8s/
screenshots/
docker-compose.yml
Jenkinsfile
README.md
```

---

## Notes

- Each Jenkins build creates a new image tag.
- Kubernetes uses rolling updates.
- PostgreSQL data is saved using PVC.
- Secrets are stored in Kubernetes Secret.
- Backend readiness check uses `/health`.
