# Smart15 — Urban Accessibility Analysis

**Smart15** is a web-based geospatial analysis platform focused on the **15-Minute City** concept.

The solution estimates pedestrian and cyclist accessibility to urban services and facilities by considering not only travel distance but also factors such as:

- terrain slope;
- surface temperature;
- vegetation cover and shading;
- road network characteristics;
- user mobility profile;
- distribution of urban services and facilities.

The platform is built on [GeoLibre](https://github.com/opengeos/GeoLibre) and combines geospatial processing, interactive visualization, result export, and automated technical report generation.

## Demonstration

<video src="https://github.com/user-attachments/assets/2dae4194-4946-4f53-a218-bbdd9836a102
" controls="controls" style="max-width: 100%;"></video>

> If the video does not display directly, access the file through the repository attachments area on GitHub.

## Key Features

- accessibility simulation for different mobility profiles;
- access analysis for urban services and facilities;
- processing of slope, temperature, and vegetation cover;
- generation of isochrones and hexagonal grids;
- upload of GeoJSON files, samples, and orthophotos;
- interactive layer visualization and inspection;
- export in geospatial formats;
- automated technical report generation.

See the [full feature description](docs/funcionalidades.md).

## Architecture

Smart15 consists of:

- a web interface based on GeoLibre;
- a plugin responsible for user interaction and result visualization;
- a FastAPI backend for geospatial processing, routing, raster handling, and exports;
- external sources and user-provided data.

See the [architecture overview](docs/arquitetura/visao-geral.md).

## Quick Installation

### Prerequisites

- Git;
- Docker;
- Docker Compose.

### Clone and Start

```bash
git clone --recurse-submodules <REPOSITORY_URL>
cd Smart15-Acessibilidade
./scripts/start.sh
```

Default addresses:

- Application: `http://localhost:8080`
- Backend: `http://localhost:8000`
- API Docs: `http://localhost:8000/docs`

For port configuration, logs, shutdown, and troubleshooting, see the [installation guide](docs/instalacao.md).

## Repository Structure

```text
Smart15-Acessibilidade/
├── backend/       # API and geospatial processing
├── deploy/        # Docker Compose, Dockerfiles, and Nginx
├── docs/          # Technical documentation
├── geolibre/      # GeoLibre submodule
├── plugin/        # Smart15 plugin source code
├── scripts/       # build and startup scripts
└── README.md
```

## Documentation

- [Features](docs/funcionalidades.md)
- [Installation](docs/instalacao.md)
- [Development](docs/desenvolvimento.md)
- [Architecture overview](docs/arquitetura/visao-geral.md)
- [Architectural decisions](docs/adr/)

## Team

- **Leandro Gregório** — design, architecture, and development;
- **Fábio Gregório** — development, integration, reports, testing, and documentation.
- **André Costa** - idealization, concept and reviewer

## Base Project

Smart15 uses [GeoLibre](https://github.com/opengeos/GeoLibre) as its foundational geospatial platform.

Licenses and credits for the projects and libraries used must be preserved according to their respective terms.

## License

See the [`LICENSE`](LICENSE) file for the project's terms of use and distribution.
