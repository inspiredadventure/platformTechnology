# Project Infrastructure Platform

This project provides a pre-configured, robust infrastructure based on Docker Compose, designed for secure, isolated service communication and persistent data management.

## Platform Architecture

The platform utilizes a dual-network topology to ensure strict isolation between external-facing services and internal backend components.

### Network Topology
- **`public_net`**: A bridge network dedicated to external traffic. Only the Traefik edge proxy is exposed to this network by default.
- **`private_net`**: An internal, isolated network (`internal: true`) where all backend services reside. Services on this network cannot be reached from outside the Docker host and cannot initiate external connections unless explicitly proxied.

### Core Service Stack

| Service | Image | Network | Role |
| :--- | :--- | :--- | :--- |
| **Traefik** | `traefik:v2.10` | `public_net`, `private_net` | Edge proxy and load balancer. Routes traffic to internal services using Docker labels. |
| **PostgreSQL** | `postgres:15-alpine` | `private_net` | Primary relational database. Initialized with multiple databases via `init-db.sql`. |
| **NATS JetStream** | `nats:2.9-alpine` | `private_net` | High-performance message broker with JetStream enabled for persistence. |
| **Authentik** | `ghcr.io/goauthentik/server` | `private_net` | Identity Provider (Server & Worker). Dashboard exposed via Traefik. |
| **Temporal** | `temporalio/auto-setup` | `private_net` | Workflow orchestration cluster with automated schema management. |
| **GoRules BRMS** | `gorules/brms:latest` | `private_net` | Business Rules Management System. |
| **Redis** | `redis:alpine` | `private_net` | Distributed cache utilized by the Authentik stack. |

## Persistence & Data Management

### Persistent Volumes
Data persistence is handled through Docker named volumes to ensure that service state survives container restarts and image upgrades:
- `postgres_data`: Stores all PostgreSQL database files.
- `nats_data`: Stores NATS JetStream state and messages.

### Automated Database Initialization
On the initial startup of the database service, the `init-db.sql` script (mounted to `/docker-entrypoint-initdb.d/`) automatically creates the following databases required by the integrated services:
- `authentik`
- `temporal`
- `temporal_visibility`
- `gorules`

## External Access & Routing

**Traefik** serves as the single point of entry. Services are exposed through host-based routing. For example:
- **Authentik Dashboard**: Reachable via `authentik.localhost` on port 80.

Future services can be easily integrated by adding them to the `private_net` and applying the appropriate `traefik.http.routers` labels.

## Getting Started

To initialize the platform infrastructure:

1. Ensure the `init-db.sql` file is present in the root directory.
2. Run the deployment command:
   ```bash
   docker compose up -d
3. Traefik's internal API dashboard is available in insecure mode on port 8080 for debugging.

