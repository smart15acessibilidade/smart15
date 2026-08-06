
---

# `ADR-003-deploy-com-docker-compose.md`

```markdown
# ADR-003 — Deploying Smart-15 with Docker Compose

## Status

Accepted.

## Date

2026-07-18.

## Context

Smart-15 must be delivered for execution in the university’s server environment.

The application has different components and dependencies:

- GeoLibre Web;
- Smart-15 plugin;
- FastAPI backend;
- Python geospatial libraries;
- Nginx or another reverse proxy;
- data directories;
- temporary files;
- results;
- reports;
- environment configuration.

The backend uses libraries that may require operating system dependencies, such as:

- GDAL;
- PROJ;
- GEOS;
- Rasterio;
- GeoPandas;
- OSMnx;
- Scikit-learn.

Manually installing these dependencies on the server would increase the risk of differences between development, staging, and production environments.

The project needs a deployment process that is:

- documented;
- reproducible;
- versioned;
- easy to execute;
- suitable for a Linux server;
- independent of the developer’s workstation.

## Decision

Smart-15 will be deployed using Docker containers coordinated by Docker Compose.

The initial deployment will include at least the following services:

```text
smart15-web
    GeoLibre Web with Smart-15 plugin

smart15-api
    FastAPI API and geospatial engine

reverse-proxy
    HTTP/HTTPS ingress and routing
```

The following services may be added later:

```text
worker
    asynchronous processing

redis
    queue, cache, or transient state

postgis
    geospatial persistence

object-storage
    storage for large artifacts
```

Deployment architecture

```text
Browser
    │
    │ HTTPS
    ▼
Nginx
    ├── /        → GeoLibre Web
    ├── /api     → Smart-15 API
    ├── /files   → results and exports
    └── /reports → reports
```

Nginx will act as the single point of access.

This will allow the browser to access frontend and backend under the same domain, reducing CORS issues.

Expected structure

```text
deploy/
├── compose/
│   ├── docker-compose.yml
│   └── docker-compose.override.yml
├── nginx/
│   └── nginx.conf
├── env/
│   └── .env.example
└── README.md
```

The real `.env` file will not be tracked in version control.

Only a `.env.example` file will be provided.

Conceptual Docker Compose example

```yaml
services:
  web:
    build:
      context: ..
      dockerfile: geolibre/Dockerfile
    restart: unless-stopped
    depends_on:
      - api

  api:
    build:
      context: ../backend
      dockerfile: Dockerfile
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - smart15-data:/app/data
      - smart15-results:/app/results

  proxy:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ../nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
      - api

volumes:
  smart15-data:
  smart15-results:
```

The final content may differ depending on the architecture actually adopted.

## Image strategy

During development, Compose may use build.

Example:

```yaml
services:
  api:
    build:
      context: ../backend
```

For staging and production, versioned images will be preferred.

Example:

```yaml
services:
  api:
    image: ghcr.io/geosmart/smart15-api:1.0.0

  web:
    image: ghcr.io/geosmart/smart15-web:1.0.0
```

Using only the `latest` tag in production will not be recommended.

Images should use explicit versions:

```text
1.0.0
1.0.1
1.1.0
```

## Image publishing

Images may be published to GitHub Container Registry.

The expected flow will be:

```text
Commit or tag
    │
    ▼
GitHub Actions
    │
    ├── run tests
    ├── build images
    └── publish to registry
    │
    ▼
University server
    │
    ├── docker compose pull
    └── docker compose up -d
```

## Configuration

Application settings will be provided through environment variables.

Examples:

```text
SMART15_ENV
SMART15_API_URL
SMART15_DATA_DIR
SMART15_RESULTS_DIR
SMART15_LOG_LEVEL
SMART15_MAX_ANALYSIS_AREA
SMART15_ALLOWED_ORIGINS
```

Credentials and tokens should not be stored in the repository.

## Persistence

Data that must survive container recreation will be stored in volumes.

Examples:

- results;
- reports;
- exported files;
- data cache;
- files uploaded by users;
- logs, if they are not sent to another service.

Temporary files may be removed after a configurable period.

## Backup

The deployment should document:

- which volumes need backup;
- where persistent files are located;
- how to restore the data;
- how to export a release;
- how to recover a previous version.

Source code will not be considered a backup of produced data.

## Alternatives considered

### Manual installation on the server

Manual installation of Python, Node.js, geospatial libraries, and Nginx would be possible.

However, it would increase:

- differences between environments;
- support difficulty;
- incompatibility risk;
- deployment time;
- dependence on the server administrator.

### Kubernetes or OpenShift

Kubernetes and OpenShift offer:

- scalability;
- high availability;
- advanced management;
- observability;
- declarative deployment.

However, for the first version, they would represent greater operational complexity and require specific infrastructure.

Docker Compose is sufficient for an initial deployment on a single server.

The container architecture will allow future evolution to Kubernetes or OpenShift.

### Running on serverless platforms

Serverless services could simplify part of the deployment.

However, Smart-15 has:

- long-running processing;
- high memory usage;
- native dependencies;
- handling of large files;
- need for persistent storage.

These characteristics may not fit well in serverless environments.

### Positive consequences

- Reproducible environment.
- Simplified installation.
- Dependency isolation.
- Easier updates.
- Possibility of rollback.
- Greater consistency between development and production.
- Less dependence on server configuration.
- Basis for future migration to OpenShift or Kubernetes.
- Easier delivery of ready-made images to the university.

### Negative consequences

- Requirement for Docker on the server.
- Need for basic container knowledge.
- Additional management of volumes and images.
- Disk space consumption.
- Need to update base images.
- Docker Compose does not provide native high availability.

### Risks

Server without registry access

The university may use a server without external access.

Mitigation:

- create an offline package;
- export images with docker save;
- import with docker load;
- provide Compose files and documentation.

### Growing file volumes

Rasters and exports may consume large storage volume.

Mitigation:

- configure a retention policy;
- remove temporary files;
- monitor available space;
- separate persistent data;
- limit simulation size and area.

### Failure during update

A new image may have an issue in production.

Mitigation:

- use immutable tags;
- keep the previous version;
- document rollback;
- test images before deployment.

### Exposed credentials

Tokens may be inadvertently included in code or images.

Mitigation:

- use environment variables;
- keep `.env` out of Git;
- review images and workflows;
- use GitHub Actions secrets.

## Validation criteria

The decision will be considered validated when:

- the application can be started with `docker compose up -d`;
- frontend and backend are accessible under the same domain;
- data persists after restart;
- a new version can be deployed by swapping tags;
- rollback can be performed;
- the installation can be reproduced on a clean Linux server;
- the procedure is documented.

## Future evolution

The architecture may evolve to:

- task queue;
- multiple workers;
- PostGIS database;
- S3-compatible storage;
- monitoring;
- centralized logs;
- Kubernetes;
- OpenShift.

This evolution will not require changing the conceptual separation between frontend, API, and processing.

## Decision review

This decision should be reviewed if:

- the university mandates another platform;
- horizontal scaling becomes necessary;
- there are high-availability requirements;
- processing volume requires distributed infrastructure;
- Docker Compose no longer meets the institutional environment.
