# Developer Handoff - Epic 2327 City Weather Lookup Application

Repository: `Mandeep-Kaur-Johar/sdlc-city-weather-lookup-application` (default branch `main`).
This is the single shared repository for architecture, implementation and QA. Do not create another repository.

## Authoritative design inputs

| Artifact | Path |
|---|---|
| System Design Document (Markdown) | `docs/architecture/epic-2327/System-Design-Document.md` |
| System Design Document (DOCX) | `docs/architecture/epic-2327/System-Design-Document.docx` |
| Diagram sources (`.mmd`) | `docs/architecture/epic-2327/` |

## Technology stack (baseline, confirmed)

- Frontend: React 18 + TypeScript (Vite), Vitest for unit tests.
- Backend: ASP.NET Core .NET 8 Web API (C#), xUnit for unit tests.
- Contracts: JSON DTOs / C# entities.
- AuthN: JWT bearer token attached by the React API client.
- AuthZ: backend permission/role checks on all weather and cities endpoints.
- Transport: HTTPS only, TLS 1.2+.
- Secrets: Azure Key Vault (provider API key never in client code or config committed to git).
- Observability: Application Insights + Log Analytics, `/health` endpoint.

## Target solution structure

```text
/src
  /frontend                 React + TypeScript SPA
    /components             SearchForm, WeatherCard, UnitToggle, RecentSearches
    /services               apiClient.ts (bearer interceptor, 401 handling)
    /hooks                  useWeather, useRecentSearches, useUnitPreference
  /backend
    /Api                    Controllers, middleware, Program.cs
    /Application            IWeatherService, validators, caching
    /Infrastructure         Typed HttpClient provider client, city catalog, Key Vault config
    /Domain                 City, WeatherReading entities and DTOs
/tests
  /frontend.tests
  /backend.tests
```

## API contract

### GET /api/weather?city={name}

- 200: `{ "city": string, "temperatureCelsius": number, "condition": string, "observedAtUtc": datetime }`
- 400: `ProblemDetails` when `city` is missing, empty or longer than 100 characters.
- 401: missing or invalid bearer token.
- 403: valid token without the required permission.
- 404: city unknown to the provider.
- 429: client rate limit exceeded, includes `Retry-After` header.
- 503: provider timeout or 5xx after the retry policy is exhausted.

### GET /api/cities

- 200: JSON array of supported cities `{ id, name, country }`, used for suggestions after 2 typed characters.
- Same auth, rate-limit and error semantics as `/api/weather`.

## Data models

```text
City            : id (string, PK), name (string, required), country (string ISO-2),
                  latitude (double), longitude (double)
WeatherReading  : cityId (string, FK -> City.id), temperatureCelsius (double),
                  condition (string), humidity (int, optional), windKph (double, optional),
                  observedAtUtc (datetime, UTC)
RecentSearch    : client-side only (localStorage), cityName (string), searchedAtUtc (datetime), max 5 entries
```

## Mandatory behaviours derived from acceptance criteria

| Behaviour | Source story |
|---|---|
| Client-side validation blocks empty input or fewer than 2 characters; no API call is made | 2329 |
| Result card shows city, temperature in Celsius, condition text, last-updated time; keyboard and screen-reader accessible | 2330 |
| Loading indicator with disabled submit; `City not found` on 404; error plus Retry button on 5xx | 2331 |
| Celsius/Fahrenheit toggle; selected unit persisted in localStorage across reloads | 2332 |
| Up to 5 recent cities shown as clickable chips; clicking a chip refetches that city | 2341 |
| Bearer token attached to every backend call; on 401 clear token and route to sign-in | 2343 |
| Backend trims and normalises casing/whitespace before provider lookup | 2335 |
| In-memory cache keyed by normalised city name with 10-minute TTL; expired entries trigger a fresh provider call | 2337 |
| Provider call uses timeout and retry policy; exhausted retries return 503 | 2336 |
| Provider API key is server-side only and absent from the client bundle and all network responses | 2345 |
| Rate limit returns 429 with `Retry-After` | 2345 |

## Traceability

| Feature | User Stories |
|---|---|
| 2328 City Search & Weather Display UI | 2329, 2330, 2331, 2332 |
| 2333 Weather API (ASP.NET Core Web API) | 2334, 2335, 2336, 2337 |
| 2338 City & Weather Data Models | 2339, 2340, 2341 |
| 2342 Auth & API Security | 2343, 2344, 2345 |

Architecture work item: Task 2846 - `HLD - City Weather Lookup Application` (child of Epic 2327).

## Assumptions

- Provider is OpenWeatherMap-compatible; provider contract mapped in the infrastructure layer.
- Temperature is served in Celsius; Fahrenheit conversion is a client-side concern.
- No user accounts beyond the API token; identity provider issues the bearer token.
- Recent searches are client-side only and never sent to the backend.
- `[TBD]` Concrete identity provider tenant/issuer and provider quota limits are not yet confirmed.
