
---

# `ADR-002-plugin-e-backend-separados.md`

```markdown
# ADR-002 — Separation between the Smart-15 plugin and the geospatial backend

## Status

Accepted.

## Date

2026-07-18.

## Context

Smart-15 has two main categories of responsibility.

The first is related to user interaction:

- selecting the analysis area;
- configuring the profile;
- choosing parameters;
- submitting the simulation;
- monitoring processing;
- displaying layers;
- presenting indicators;
- downloading results.

The second is related to geospatial processing:

- obtaining the road network;
- processing OpenStreetMap data;
- obtaining and processing rasters;
- calculating slope;
- processing surface temperature;
- classifying vegetation and coverage;
- weighting the network;
- calculating routes;
- generating isochrones;
- H3 aggregation;
- exporting files;
- generating reports.

These two categories have different technical characteristics.

The interface must run in the browser and remain responsive.

Geospatial processing can consume large amounts of:

- CPU;
- memory;
- disk;
- runtime;
- native libraries;
- temporary data.

The browser is not the appropriate environment to perform this processing entirely.

## Decision

The Smart-15 interface will be implemented as a GeoLibre plugin.

The geospatial processing will be implemented in an independent backend using Python and FastAPI.

Communication between the plugin and backend will occur via HTTP API.

The main division will be:

```text
Smart-15 Plugin
    interface, interaction, and visualization

Smart-15 API
    validation, orchestration, and result delivery

Geospatial Engine
    calculations, environmental data, routing, and export
```

## Plugin responsibilities

### The plugin will be responsible for:

- presenting application controls;
- receiving user parameters;
- obtaining the selected area on the map;
- sending requests to the API;
- presenting the simulation status;
- consuming the results;
- adding layers to GeoLibre;
- displaying isochrones, routes, POIs, and grids;
- displaying summary indicators;
- providing download and report links;
- presenting error messages in a comprehensible way.

### The plugin will not be responsible for:

- executing heavy raster processing;
- calculating complete routes;
- processing large graphs;
- performing large-scale geospatial classification;
- storing result files;
- generating Shapefile, KML, or PDF files.

## Backend responsibilities

### The backend will be responsible for:

- validating received parameters;
- creating a simulation identifier;
- coordinating processing;
- querying external services;
- processing the road network;
- processing rasters;
- applying weighting factors;
- calculating isochrones and indicators;
- generating output files;
- storing artifacts;
- making results available;
- generating the technical report;
- logging errors and execution information.

## Communication contract

The plugin will send requests in JSON.

Conceptual example:

```json
{
  "origin": {
    "latitude": -22.90,
    "longitude": -43.20
  },
  "profile": "adult_walking",
  "maximum_time_minutes": 15,
  "factors": {
    "slope": true,
    "temperature": true,
    "vegetation": true
  }
}
```

The API will return a simulation identifier and its status.

Example:

```json
{
  "simulation_id": "sim-20260718-001",
  "status": "processing"
}
```

After processing, the API may return:

```json
{
  "simulation_id": "sim-20260718-001",
  "status": "completed",
  "layers": {
    "isochrones": "/api/simulations/sim-20260718-001/isochrones.geojson",
    "routes": "/api/simulations/sim-20260718-001/routes.geojson",
    "pois": "/api/simulations/sim-20260718-001/pois.geojson",
    "hexagons": "/api/simulations/sim-20260718-001/hexagons.geojson"
  },
  "indicators": {
    "reachable_area_m2": 850000,
    "reachable_pois": 37,
    "slope_penalty_percent": 12.4,
    "heat_penalty_percent": 8.1,
    "vegetation_benefit_percent": 3.7
  },
  "exports": {
    "zip": "/api/simulations/sim-20260718-001/export.zip",
    "report": "/api/simulations/sim-20260718-001/report.pdf"
  }
}
```

## Data formats

The following will be used as needed:

- JSON for parameters, states, and indicators;
- GeoJSON for vector layers;
- PNG or GeoTIFF for raster results;
- ZIP for grouped exported files;
- PDF for reports;
- HTTP for data transfer.

GeoJSON is a response format, not an independent architecture component.

## Alternatives considered
### Implement all processing in the plugin

This approach would reduce the number of services.

However, it would be inadequate because:

- the browser has limited resources;
- Python geospatial libraries would not be available;
- large rasters could block the interface;
- processing would be repeated on every workstation;
- the code would become tightly coupled to the frontend;
- it would be difficult to control and reproduce the environment.

### Implement the interface directly in the backend

This alternative would allow creating a conventional web application with templates.

However, it would reduce GeoLibre reuse and make it harder to:

- integrate with the map;
- create layers;
- provide interactive controls;
- reuse the plugin architecture;
- evolve the interface independently.

### Incorporate the backend inside GeoLibre’s internal service

This alternative would reduce the number of containers.

However, it would create greater coupling and mix platform and domain responsibilities.

The Smart-15 backend has specific geospatial dependencies and its own evolution cycle.

## Positive consequences

- Clear separation of responsibilities.
- Lighter and more responsive interface.
- Backend reusable by other clients.
- Possibility to test frontend and backend separately.
- Easier to scale processing.
- Isolation of Python dependencies.
- Better code organization.
- Possibility of future API integrations.
- Easier report and export generation.
- Lower coupling to the GeoLibre core.

## Negative consequences

- Increased number of deployed components.
- Need to define and maintain an API contract.
- Need to handle network failures.
- Need to configure CORS or reverse proxy.
- Need to manage files and simulation states.
- Greater observability complexity.

## Risks

### Long-running requests

Simulations may exceed the normal duration of an HTTP request.

Mitigation:

- utilizar processamento assíncrono;
- retornar um identificador de simulação;
- permitir consulta de estado;
- configurar limites e timeouts adequados;
- considerar fila de processamento em versões futuras.

### Respostas muito grandes

Camadas GeoJSON podem atingir tamanhos elevados.

Mitigação:

- simplificar geometrias;
- disponibilizar camadas por URL;
- utilizar compressão;
- avaliar formatos vetoriais mais eficientes no futuro;
- limitar a área de análise.

### Incompatibilidade entre frontend e API

Mudanças no contrato podem quebrar o plugin.

Mitigação:

- versionar a API;
- documentar os esquemas;
- criar testes de integração;
- manter compatibilidade durante as releases.

## Requisitos de implementação

A API deverá:

- possuir documentação OpenAPI;
- utilizar modelos de entrada e saída;
- validar parâmetros;
- retornar erros estruturados;
- registrar logs;
- identificar cada simulação;
- evitar nomes globais de arquivos;
- separar resultados por simulação;
- não depender de URLs fixas de localhost;
- permitir configuração por variáveis de ambiente.

O plugin deverá:

- utilizar uma URL de API configurável;
- tratar os estados de processamento;
- tratar falhas de comunicação;
- não depender de caminhos absolutos;
- validar entradas antes do envio;
- apresentar erros ao usuário.

## Critérios de validação

A decisão será considerada validada quando:

- o plugin conseguir iniciar uma simulação;
- a API receber e validar a requisição;
- a API executar ou coordenar o processamento;
- o plugin conseguir consultar o estado;
- o plugin carregar as camadas resultantes;
- os indicadores puderem ser apresentados;
- o relatório puder ser acessado pela interface.

## Revisão da decisão

Esta decisão deve ser revisada caso:

- o processamento passe a ser suficientemente leve para execução local;
- o GeoLibre ofereça uma arquitetura nativa adequada para serviços geoespaciais;
- surja a necessidade de múltiplos backends especializados;
- seja adotada uma arquitetura orientada a eventos ou filas.
