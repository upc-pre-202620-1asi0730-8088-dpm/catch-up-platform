# CatchUp Platform REST API Technical Stories

## Overview
This document contains API-focused technical stories intended for frontend or mobile developers integrating with the catch-up-platform REST API.

Base resource route: `/api/v1/favorite-sources` (kebab-case route naming convention).

## Requirement Traceability Matrix

| Technical Story | Scenario                        | Implementation                                                                 |
|------------------|----------------------------------|---------------------------------------------------------------------------------|
| TS001            | Successful create                | `FavoriteSourceCommandService.Handle(CreateFavoriteSourceCommand)`             |
| TS001            | Validation error                 | `NewsApiKey`/`SourceId` constructors, `[Required]` on `CreateFavoriteSourceResource` |
| TS001            | Duplicate favorite                | unique index + `CreateFavoriteSourceError.DuplicateFavoriteSource`             |
| TS001            | Unexpected server error          | `CreateFavoriteSourceError.UnexpectedError`                                     |
| TS002            | Found                            | `FavoriteSourcesController.GetFavoriteSourceById`                               |
| TS002            | Not found                        | `IBaseRepository.FindByIdAsync` → `null` → `404`                               |
| TS003            | Returns one or more favorites    | `FavoriteSourcesController.GetFavoriteSourceFromQuery`, `IFavoriteSourceQueryService` |
| TS003            | Returns no favorites             | `FavoriteSourceRepository.FindByNewsApiKeyAsync` (empty list, not `404`)        |
| TS004            | Favorite exists                  | `FavoriteSourceRepository.FindByNewsApiKeyAndSourceIdAsync`                     |
| TS004            | Favorite does not exist          | same method → `null` → `404`                                                   |
| TS005            | Supported language requested     | `LocalizedOpenApiAssembler`, `UseRequestLocalization`                          |
| TS005            | Unsupported or missing language  | default culture fallback (`en`)                                                |
| TS006            | Successful delete                | `FavoriteSourceCommandService.Handle(DeleteFavoriteSourceCommand)`             |
| TS006            | Not found                        | `DeleteFavoriteSourceError.NotFound`                                            |
| TS006            | Invalid id                       | `DeleteFavoriteSourceCommand` constructor                                      |

## Technical Stories

### TS001 — Create a favorite source
As a frontend developer, I want to add a favorite source through the API so that I can implement the creation feature in my application.
#### Acceptance criteria:
- Scenario: Successful create
    - Given a request `POST /api/v1/favorite-sources` that includes required fields (`newsApiKey`, `sourceId`) is received
    - When the API validates and persists the new resource successfully
    - Then the API responds with `201 Created`, and returns the created resource representation (`id`, `newsApiKey`, `sourceId`).
- Scenario: Validation error
    - Given a request `POST /api/v1/favorite-sources` with missing required fields or invalid values is received
    - When the API validation fails
    - Then the API responds with `400 Bad Request` and a localized error message describing the validation failure.
- Scenario: Duplicate favorite
    - Given a request `POST /api/v1/favorite-sources` for a `newsApiKey` + `sourceId` combination that already exists is received
    - When the API detects the duplicate pair before persistence or during persistence confirmation
    - Then the API responds with `409 Conflict` and an explanatory error payload.
- Scenario: Unexpected server error
    - Given a request `POST /api/v1/favorite-sources` is received
    - When an unexpected error occurs during persistence
    - Then the API responds with `500 Internal Server Error` and a `ProblemDetails` payload.
---
### TS002 — Retrieve a favorite source by id
As a frontend developer, I want to request a favorite source by its `{id}` so that I can implement the details view in my application.
#### Acceptance criteria:
- Scenario: Found
    - Given a request `GET /api/v1/favorite-sources/{id}` is received
    - When the API finds the resource
    - Then the API responds with `200 OK` and returns the favorite source resource (`id`, `newsApiKey`, `sourceId`).
- Scenario: Not found
    - Given a request `GET /api/v1/favorite-sources/{id}` for a non-existent `{id}` is received
    - When the API does not find the resource
    - Then the API responds with `404 Not Found`.
---
### TS003 — List favorite sources by NewsAPI key
As a frontend developer, I want to request the API to list favorite sources for a given `{newsApiKey}` so that I can implement the listing feature in my application.
#### Acceptance criteria:
- Scenario: Returns one or more favorites
    - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}` is received
    - When the API finds one or more matching favorite resources
    - Then the API responds with `200 OK` and returns a non-empty array of favorite source resources, with each resource including `id`, `newsApiKey`, and `sourceId`.
- Scenario: Returns no favorites
    - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}` is received
    - When the API finds no matching resources
    - Then the API responds with `200 OK` and returns an empty array.
---
### TS004 — Check whether a source is favorite for a NewsAPI key
As a frontend developer, I want to query the API with `{newsApiKey}` and `{sourceId}` so that I can implement the UI state that indicates whether a source is favorite.
#### Acceptance criteria:
- Scenario: Favorite exists
    - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}&sourceId={sourceId}` is received
    - When the API finds a matching favorite resource
    - Then the API responds with `200 OK` and returns the matching favorite resource.
- Scenario: Favorite does not exist
    - Given a request `GET /api/v1/favorite-sources?newsApiKey={newsApiKey}&sourceId={sourceId}` with no match is received
    - When the API does not find a matching resource
    - Then the API responds with `404 Not Found`.
---
### TS005 — Read the API description in my language
As a frontend developer, I want the API description document to be served in my preferred language so that I can understand the endpoints and their responses without translating them.
#### Acceptance criteria:
- Scenario: Supported language requested
    - Given a request `GET /openapi/v1.json` with an `Accept-Language` header for a supported culture (`en`, `en-US`, `es`, `es-PE`) is received
    - When the API builds the description document
    - Then the API responds with `200 OK` and the operation summaries, descriptions, response descriptions and resource property descriptions are in that language.
- Scenario: Unsupported or missing language
    - Given a request `GET /openapi/v1.json` without an `Accept-Language` header, or with an unsupported one, is received
    - When the API builds the description document
    - Then the API responds with `200 OK` and the texts are in the default language.
---
### TS006 — Remove a favorite source
As a frontend developer, I want to remove a favorite source through the API so that I can implement the "unfavorite" feature in my application.
#### Acceptance criteria:
- Scenario: Successful delete
    - Given a request `DELETE /api/v1/favorite-sources/{id}` for an existing `{id}` is received
    - When the API removes the resource
    - Then the API responds with `204 No Content`.
- Scenario: Not found
    - Given a request `DELETE /api/v1/favorite-sources/{id}` for a non-existent `{id}` is received
    - When the API does not find the resource
    - Then the API responds with `404 Not Found` and a localized error message.
- Scenario: Invalid id
    - Given a request `DELETE /api/v1/favorite-sources/{id}` with `id` zero or negative is received
    - When the API validates the id
    - Then the API responds with `400 Bad Request`.
- solo get con querys
- comand delete and create
- event storming se modela el comand-expresan el lenguaje ubicuo-modelo del dominio 
- patron asembler para convertir de recurso a comando
- asembler, request response, c4noseque, handle para manejar el comnado o query
- query y coman va en domain model, la ejecucion el handle va en aplicacion la logica de ejecucion.
- separated interface, en al capa de dominio esta la interface y el contrato esta en otra capa
- result pattern, para manejar el resultado de la ejecucion del comando o query, si es exitoso o no, y si es exitoso devuelve el recurso creado o encontrado, si no es exitoso devuelve un error.
- notification pattern, para notificar a los interesados de un cambio en el estado del sistema, por ejemplo, cuando se crea o elimina un favorito.
- 