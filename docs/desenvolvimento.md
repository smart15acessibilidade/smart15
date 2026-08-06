# Smart15 Development

## Main components

The project has three central components:

- `backend/`: FastAPI API and geospatial processing;
- `plugin/`: Smart15 plugin source code;
- `geolibre/`: GeoLibre platform submodule.

The runtime infrastructure is in `deploy/`, and auxiliary scripts are in `scripts/`.

## Plugin development

The plugin uses TypeScript, Vite, and components integrated with the GeoLibre interface.

To install project dependencies:

```bash
npm install
```

To build and sync the plugin with GeoLibre:

```bash
./scripts/build-plugin.sh
```

The script:

1. builds the bundle;
2. removes the previously synced version;
3. creates the target structure;
4. copies the manifest;
5. copies the bundle;
6. validates the resulting files.

Plugin target location:

```text
geolibre/apps/geolibre-desktop/public/plugins/smart15-orquestrador/
```

## Integrated execution

To build the plugin, build the images, and start the entire environment:

```bash
./scripts/start.sh
```

## Backend

The backend is implemented in Python with FastAPI.

Main responsibilities:

- obtaining and processing geospatial data;
- processing digital elevation models;
- slope calculation;
- handling thermal and vegetation data;
- building and analyzing the road network;
- generating isochrones and grids;
- exporting results;
- making generated files available.

The interactive API documentation is available at:

```text
http://localhost:8000/docs
```

or on the port configured in `SMART15_BACKEND_PORT`.

## Relevant structure

```text
Smart15-Acessibilidade/
├── backend/
├── deploy/
│   ├── compose/
│   ├── docker/
│   └── nginx/
├── docs/
├── geolibre/
├── plugin/
├── scripts/
│   ├── build-plugin.sh
│   └── start.sh
├── .env.example
├── package.json
└── README.md
```

## GeoLibre submodule

To initialize or update the submodule manually:

```bash
git submodule update --init --recursive
```

To check the commit in use:

```bash
git submodule status
```

Changes in GeoLibre should be handled separately from changes in the main repository.

## Plugin cache

If GeoLibre loads an older bundle version, run in the browser console:

```javascript
localStorage.removeItem('geolibre.pluginBundlePins')
```

Then reload the page with `Ctrl+Shift+R`.

## Pre-commit validations

```bash
git status
git diff
./scripts/build-plugin.sh
./scripts/start.sh
```

After startup, verify:

- web application;
- backend `/health` endpoint;
- `/docs` documentation;
- plugin loading;
- simulation execution;
- report generation and opening;
- result export.
