# ADR-001 — Using GeoLibre as the Smart-15 Web Platform

## Status

Accepted.

## Date

2026-07-18.

## Context

Smart-15 is an urban accessibility analysis platform based on the 15-Minute City concept.

The solution must allow the user to:

- select a study area;
- configure a mobility profile;
- perform a geospatial analysis;
- visualize layers, isochrones, and points of interest;
- compare scenarios;
- export results;
- generate technical reports.

The previous version of the project was closer to a desktop application and relied on local geoprocessing tools.

For the new version, it was determined that the system should be accessible through a browser without requiring the user to install geospatial software on their workstation.

The project already uses map visualization, layer handling, and WebGL rendering features that can be supported by GeoLibre.

GeoLibre provides a web-based foundation for geospatial visualization and allows the interface to be extended through plugins.

## Decision

GeoLibre will be used as the web and mapping platform for Smart-15.

The Smart-15 project will be integrated into a GeoLibre fork maintained by the GeoSmart organization.

The fork will be used to:

- control the GeoLibre version adopted by the project;
- integrate the Smart-15 plugin;
- adjust the build process;
- configure the application for deployment;
- apply interface and visual identity adaptations as needed.

Smart-15 specific business logic will not be incorporated directly into the GeoLibre core.

The Smart-15 specific interface will be implemented through the Smart-15 plugin, while geospatial processing will be performed by a separate backend.

## Resulting Architecture

The solution will be organized as follows:

```text
User
   │
   ▼
GeoLibre Web
   │
   ▼
Smart-15 Plugin
   │
   ▼
Smart-15 API
   │
   ▼
Geospatial Engine
```
GeoLibre will be primarily responsible for:

- providing the web application;
- displaying the map;
- managing layers;
- enabling interaction with geographic objects;
- running the Smart-15 plugin;
- rendering results received from the API.

## Alternatives Considered
### Developing a mapping web application from scratch

This alternative would provide greater freedom over the interface and architecture.

However, it would require reimplementing already existing features, such as:

- map management;
- layer loading;
- map controls;
- navigation;
- WebGL rendering;
- integration with geospatial formats;
- plugin management.

This option would increase the timeline, development effort, and project risk.

### Keeping the project only as a desktop application

This alternative would reduce changes to the existing architecture.

However, it would present significant limitations:

- the need for local installation;
- greater difficulty updating;
- dependence on the user’s environment;
- less ease of demonstration;
- greater complexity for institutional availability;
- difficulty accessing from different devices.

### Modifying GeoLibre core directly

This alternative would allow Smart-15 to be inserted directly into the platform’s main codebase.

However, it would increase coupling between the project and GeoLibre, making it harder to:

- update from the original project;
- maintain;
- identify Smart-15 changes;
- reuse the plugin;
- isolate failures.

## Positive Consequences
- Reduced effort to build the mapping interface.
- Availability of the system through a browser.
- Reuse of existing GeoLibre resources.
- Separation between the mapping platform and Smart-15 specific logic.
- Possibility of updating the fork from the original project.
- Better traceability of changes.
- Greater ease of deployment to a server.
- Easier demonstration and use by the university.

## Negative Consequences
- Technological dependence on GeoLibre.
- Need to follow changes in the original project.
- Possible incompatibilities between new GeoLibre versions and the plugin.
- Need to maintain a fork.
- Part of the architecture and build cycle will be determined by GeoLibre.

## Risks
### Incompatible changes in GeoLibre

New versions may change the plugin API or the build process.

Mitigation:

- lock a GeoLibre version for each Smart-15 release;
- use tags;
- update the fork in a controlled manner;
- test the integration before incorporating new versions.

### Excessive growth of fork changes

Too many core changes may make synchronization with the original project difficult.

Mitigation:

- keep Smart-15 logic in the plugin and backend;
- avoid unnecessary core changes;
- document every modification made in the fork.

### Validation criteria

The decision will be considered validated when:

- GeoLibre Web can run locally;
- the Smart-15 plugin can be loaded;
- the plugin can add panels and layers;
- the plugin can call the Smart-15 API;
- API results can be displayed on the map;
- the application can be built for server deployment.

### Decision review

This decision should be reviewed if:

- GeoLibre no longer meets the essential requirements;
- the plugin API becomes unviable for the project;
- the fork maintenance cost outweighs the benefit of reuse;
- a new mandatory institutional platform is defined.
