# Lakekeeper with Keycloak Integration

This project demonstrates the integration of Lakekeeper (an Apache Iceberg catalog service) with Keycloak for authentication and authorization using OpenID Connect (OIDC).

## Overview

This setup provides a complete testing environment for Lakekeeper integrated with:
- **Keycloak** - Identity and Access Management (IAM) provider
- **OpenFGA** - Fine-grained authorization service
- **PostgreSQL** - Database backend for Lakekeeper and OpenFGA
- **MinIO** - S3-compatible object storage
- **Jupyter** - Notebook environment for examples

## Architecture

The integration uses OpenID Connect (OIDC) for authentication:
- Lakekeeper authenticates users via Keycloak
- OpenFGA handles authorization decisions and also authenticates via Keycloak
- All services communicate over a Docker network (`iceberg_net`)

## Prerequisites

- Docker
- Docker Compose

## Quick Start

### Starting the Services

docker compose upThis will start all services in the correct order with proper health checks and dependencies.

### Service Endpoints

Once all services are running, you can access:

- **Lakekeeper**: http://localhost:8181
- **Keycloak Admin Console**: http://localhost:30080
  - Username: `admin`
  - Password: `admin`
- **Jupyter Notebooks**: http://localhost:8888
- **MinIO Console**: Available on the MinIO service (port 9001 internally)
- **PostgreSQL**: localhost:5432
  - Username: `postgres`
  - Password: `postgres`

## Services

### Lakekeeper
- **Port**: 8181
- **Image**: `quay.io/lakekeeper/catalog:latest-main`
- Configured with OpenID Connect authentication via Keycloak
- Uses OpenFGA for authorization

### Keycloak
- **Port**: 30080 (mapped from internal 8080)
- **Image**: `quay.io/keycloak/keycloak:26.0.7`
- Realm configuration imported from `keycloak/realm.json`
- Features: Token exchange enabled

### OpenFGA
- **Image**: `openfga/openfga:v1.8`
- Configured with OIDC authentication via Keycloak
- Uses PostgreSQL as the datastore

### PostgreSQL
- **Port**: 5432
- **Image**: `postgres:17`
- Two instances: one for Lakekeeper, one for OpenFGA

### MinIO
- **Image**: `minio/minio:RELEASE.2025-07-23T15-54-02Z`
- S3-compatible object storage
- Pre-configured bucket: `examples`

### Jupyter
- **Port**: 8888
- **Image**: `quay.io/jupyter/pyspark-notebook:2024-10-14`
- Notebooks directory mounted at `/home/jovyan/examples/`

## Configuration

### Keycloak Realm
The Keycloak realm configuration is imported from `keycloak/realm.json`. Ensure this file contains your realm configuration before starting the services.

### Environment Variables
Key configuration is set via environment variables in `docker-compose.yaml`:
- `LAKEKEEPER__OPENID_PROVIDER_URI`: Keycloak realm endpoint
- `LAKEKEEPER__OPENID_AUDIENCE`: OIDC audience (`lakekeeper`)
- `LAKEKEEPER__UI__OPENID_CLIENT_ID`: Client ID for UI (`lakekeeper`)

## Usage

1. Start all services: `docker compose up`
2. Wait for all health checks to pass
3. Access Lakekeeper at http://localhost:8181
4. Authenticate using Keycloak credentials
5. Use Jupyter notebooks for examples and testing

## Stopping Services

docker compose downTo remove volumes as well:

docker compose down -v## Notes

- This is a **testing/development** setup and should not be used in production
- Default passwords and keys are not secure (e.g., `LAKEKEEPER__PG_ENCRYPTION_KEY`)
- Health checks ensure services start in the correct order
- The `migrate` service runs database migrations before Lakekeeper starts
- The `createbuckets` service initializes MinIO buckets before Lakekeeper starts
