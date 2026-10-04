# Loyalty Engine Deployments

This repository contains deployment configurations for the Loyalty Engine microservices architecture. It includes Docker Compose and Kubernetes manifests for local development and production deployments.

## Architecture

The Loyalty Engine consists of the following microservices:

- **nginx** - API Gateway (port 80)
- **auth-service** - Authentication and Users Management API (port 3001)
- **properties-service** - Properties API (port 3002)
- **points-service** - Points API (port 8080)
- **coupons-service** - Coupons API (port 8081)

### Infrastructure

- **Redis** - Caching and session storage (port 6379)
- **PostgreSQL** - Database for auth, properties, and points services
- **MongoDB** - Database for coupons service

## Prerequisites

- Docker and Docker Compose
- (For Kubernetes) kubectl and a Kubernetes cluster (minikube, kind, or cloud provider)

## Quick Start with Docker Compose

1. Clone the repository:
```bash
git clone <repository-url>
cd deployments
```

2. Configure environment variables:
```bash
cd docker-compose/env
# Create and configure .env files for each service or update the example files
# .env.auth, .env.properties, .env.coupons, .env.points
```

3. Start all services:
```bash
cd docker-compose
docker compose up -d
```

4. Verify services are healthy:
```bash
docker compose ps
```

5. Access the application:
- API Gateway: http://localhost
- Auth Service: http://localhost:3001
- Properties Service: http://localhost:3002
- Points Service: http://localhost:8080
- Coupons Service: http://localhost:8081

## Docker Compose Commands

- Start all services: `docker compose up -d`
- Stop all services: `docker compose down`
- Restart a service: `docker compose restart [service-name]`

## Kubernetes Deployment

### Prerequisites

- kubectl configured with cluster access

### Deployment Steps

1. Create the namespace:
```bash
kubectl create namespace loyalty-engine
```

2. Install ingress-nginx (if not already installed):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

3. Configure environment variables in the Kubernetes manifests:
```bash
cd kubernetes
# Update ConfigMaps and Secrets for each service
```

4. Apply the manifests:
```bash
# Deploy each service using sub-folders

# Auth Service
kubectl apply -f auth-service/database
kubectl apply -f auth-service/redis
kubectl apply -f auth-service/api

# Properties Service
kubectl apply -f properties-service/database
kubectl apply -f properties-service/api

# Points Service
kubectl apply -f points-service/database
kubectl apply -f points-service/api

# Coupons Service
kubectl apply -f coupons-service/database
kubectl apply -f coupons-service/api

# Ingress
kubectl apply -f ingress/
```

5. Verify deployment:
```bash
kubectl get pods -n loyalty-engine
kubectl get pods -n ingress-nginx
```

### Accessing Services

#### Port Forwarding

To access the application via ingress:
```bash
kubectl port-forward svc/ingress-nginx-controller -n ingress-nginx <PORT>:80
```

Then access the application at: http://localhost:<PORT>

#### Viewing Logs

View logs for a specific pod:
```bash
kubectl logs <pod-name> -n loyalty-engine
```

## Container Images

The following container images are used in this deployment:

- **Auth Service**: `ghcr.io/loyaltyengine/loyalty-engine-auth:1.0.0`
- **Properties Service**: `ghcr.io/loyaltyengine/loyalty-engine-properties:1.0.0`
- **Points Service**: `ghcr.io/loyaltyengine/loyalty-engine-points:1.0.0`
- **Coupons Service**: `ghcr.io/loyaltyengine/loyalty-engine-coupons:1.0.0`

## Default Database Credentials

**PostgreSQL:**
- Username: postgres
- Password: password
- Databases: le_auth, le_properties, le_points

**MongoDB:**
- Username: user
- Password: password
- Database: le_coupons

**Redis:**
- No authentication by default

## Health Checks

All services include health checks:

- Auth Service: `GET http://localhost:3001/auth/health`
- Properties Service: `GET http://localhost:3002/properties-api/health`
- Points Service: `GET http://localhost:8080/actuator/health`
- Coupons Service: `GET http://localhost:8081/actuator/health`

## Troubleshooting

### Services not starting

Check service logs:
```bash
docker compose logs [service-name]
```

### Database connection issues

Verify databases are healthy:
```bash
docker compose ps
```

### Port conflicts

Ensure ports 80, 3001, 3002, 6379, 8080, 8081 are available on your host machine.

## Volume Management

Docker Compose uses named volumes for data persistence:
- `auth-service-db-data`
- `properties-service-db-data`
- `points-service-db-data`
- `coupons-service-db-data`
- `coupons-service-db-configdb`
- `redis-data`

To remove volumes (WARNING: deletes all data):
```bash
docker compose down -v
```

## License

See [LICENSE](LICENSE) file for details.
