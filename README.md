# RaceDay

![CI](https://github.com/YOUR-USERNAME/YOUR-REPO/actions/workflows/ci.yml/badge.svg)

RaceDay is a full-stack event management platform for South African road running, walking and cycling events. Organisers create and manage events, categories, enrolments and results. Participants browse upcoming events, enter by choosing a category, and track their personal race history.

This repository holds all three parts of the PROG6212 Portfolio of Evidence:

| Part | What | Where |
|---|---|---|
| 1 | Planning: ERD, API endpoint plan, SQL script | [`docs/`](docs) |
| 2 | ASP.NET Core Web API, EF Core, Swagger, unit tests | [`src/RaceDay.Api`](src/RaceDay.Api), [`tests/RaceDay.Api.Tests`](tests/RaceDay.Api.Tests) |
| 3 | ASP.NET Core MVC site, Azure Blob Storage, Docker | [`src/RaceDay.Mvc`](src/RaceDay.Mvc), [`docker-compose.yml`](docker-compose.yml) |

> **Video (Part 3):** <!-- TODO: paste your unlisted YouTube link --> `ADD LINK HERE`
> **Earlier videos:** Part 1 `ADD LINK` - Part 2 `ADD LINK`

## CI/CD

<!-- TODO: take a screenshot of the green GitHub Actions run and save it as docs/ci-green-build.png -->
![Green CI build](docs/ci-green-build.png)

The workflow in [`.github/workflows/ci.yml`](.github/workflows/ci.yml) runs on every push: it validates the repository structure, builds the API, the MVC site and both test projects, runs every unit test, and builds both Docker images.

## The two roles

| Role | Can do |
|---|---|
| **Organiser** | Create, edit and delete their events (with a banner image); define categories; view every enrolment and confirm or cancel entries; capture finish times and finishing positions after an event. |
| **Participant** | Register and sign in; browse and filter upcoming events; enter an event by choosing a category; see their entries and status; view their personal results; manage their profile and picture. |

Roles are enforced twice: by the API on every endpoint, and by the MVC site (`RequireLoginAttribute`) so each role only ever sees its own pages and menu.

## Architecture

```
 Browser ──► RaceDay.Mvc (ASP.NET Core MVC) ──HTTP + session cookie──► RaceDay.Api (Web API) ──► SQL Server
                                                                            │
                                                                            └──► Azure Blob Storage (event banners, profile pictures)
```

* The **MVC site has no database code and no business logic.** Every operation goes through `Services/RaceDayApi.cs`, a typed `HttpClient` for the REST API.
* **Sessions:** the API identifies a user with a server-side session (`.RaceDay.Session` cookie). The MVC site is itself a server, so at login it captures that cookie, keeps it in the user's own MVC session next to their user id and role, and replays it on every API call. Each user's cookie is kept separately.
* **Azure Blob Storage is accessed only by the API.** The MVC site posts the image file to the API (`POST /api/events/{id}/banner`, `PUT /api/users/me/profile-picture`); the API validates it (size, type and file signature), uploads it to Blob Storage and stores the public URL.
* **Errors** never reach the user raw: failed API calls become friendly messages, and unhandled exceptions render a branded error page.

## Project structure

```
.github/workflows/ci.yml        CI: validate structure, build, test, Docker builds
docs/                           ERD, endpoint plan, SQL script, Part 3 notes
src/RaceDay.Api/                REST API (Part 2 + Azure Blob Storage)
src/RaceDay.Mvc/                MVC website (Part 3)
  Controllers/                  One controller per feature area
  Infrastructure/               RequireLogin filter, session helpers, formatting helpers
  Services/                     RaceDayApi (typed HttpClient), ApiResult, settings
  Models/Api/                   Records mirroring the API's JSON
  ViewModels/                   Form and page models with validation
  Views/                        Razor views and shared partials
  wwwroot/                      RaceDay theme (css), logo, scripts
  Dockerfile
tests/RaceDay.Api.Tests/        API unit tests
tests/RaceDay.Mvc.Tests/        MVC unit tests
docker-compose.yml              SQL Server + Azurite + API + MVC
```

## Prerequisites

* [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
* SQL Server (Express is fine) + SSMS - **or** just Docker
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) for the container options
* An Azure Storage account for real image storage (optional - see [`docs/Azure_Blob_Setup.md`](docs/Azure_Blob_Setup.md))

## Run it

### Option A - everything in Docker (one command)

```bash
docker compose up --build
```

| What | URL |
|---|---|
| MVC website | http://localhost:8080 |
| API + Swagger | http://localhost:8081/swagger |

This starts SQL Server, the Azurite blob emulator, the API (which creates the database and demo data on first start) and the MVC site. Demo accounts (password `RaceDay#2026`):

| Role | Email |
|---|---|
| Organiser | `organiser@raceday.co.za` |
| Participant | `participant@raceday.co.za` |

To store images in **real Azure** instead of Azurite, copy `.env.example` to `.env` and fill in `BLOB_CONNECTION_STRING` and `BLOB_PUBLIC_BASE_URL`. Stop everything with `docker compose down` (add `-v` to wipe the database).

### Option B - run the MVC site in a single Docker container

Start the API on your machine first (Visual Studio profile **RaceDay.Api (http only)**, or `dotnet run --project src/RaceDay.Api --launch-profile "RaceDay.Api (http only)"`), then:

```bash
docker build -t raceday-mvc ./src/RaceDay.Mvc
docker run --rm -p 8080:8080 --add-host=host.docker.internal:host-gateway -e ApiSettings__BaseUrl=http://host.docker.internal:5132/ raceday-mvc
```

Browse to http://localhost:8080. The container behaves identically to running locally; only the API address changes.

### Option C - run locally without Docker

1. **Database:** open `docs/RaceDay_Database_Script.sql` in SSMS and run it, *or* set `Database:AutoMigrate` to `true` for the API to build the schema with EF Core migrations. (The SQL script's seed users have placeholder password hashes - register fresh accounts, or enable `Database:SeedDemoData` on an empty database.)
2. **Connection string:** adjust `ConnectionStrings:RaceDayDb` in `src/RaceDay.Api/appsettings.json` if your server is not `(local)\SQLEXPRESS`.
3. **Azure Blob Storage:** `dotnet user-secrets set "BlobStorage:ConnectionString" "<your connection string>"` from `src/RaceDay.Api`. Without it everything works except image uploads (you get a friendly "image storage is not configured" message).
4. **Start both projects.** In Visual Studio: right-click the solution > *Configure Startup Projects* > *Multiple startup projects* > start `RaceDay.Api` and `RaceDay.Mvc`. Or in two terminals:
   ```bash
   dotnet run --project src/RaceDay.Api    # https://localhost:7132  (Swagger at /swagger)
   dotnet run --project src/RaceDay.Mvc    # https://localhost:7200
   ```
   The MVC site finds the API through `ApiSettings:BaseUrl` in `src/RaceDay.Mvc/appsettings.json`.

## Run the tests

```bash
dotnet test RaceDay.sln
```

* `RaceDay.Api.Tests` - authentication, role enforcement, events, enrolments, results, image upload validation, Blob Storage behaviour (with a fake in place of Azure) and demo seeding.
* `RaceDay.Mvc.Tests` - the API client (session cookie capture and replay, error handling, an unreachable API), role-based page protection, event filtering, formatting and input parsing.

## Part 3 feature map

| Requirement | Where |
|---|---|
| MVC consuming the API, no DB access | `Services/RaceDayApi.cs`, all controllers |
| Register / login / logout, session, role navigation, redirects | `AccountController`, `RequireLoginAttribute`, `_Layout.cshtml` |
| Organiser dashboard | `OrganiserController`, `Views/Organiser/Dashboard.cshtml` |
| Events CRUD + banner upload | `ManageEventsController`, `_EventForm.cshtml` |
| Categories | `ManageCategoriesController` |
| Enrolments with status | `ManageEnrolmentsController` |
| Results entry | `ManageResultsController` |
| Participant home + filters | `HomeController`, `_EventFilter.cshtml`, `EventFiltering.cs` |
| Event details + enter event | `EventsController`, `Views/Events/Details.cshtml` |
| My events / My results | `ParticipantController` |
| Profile + profile picture | `AccountController.Profile`, `UploadPicture` |
| Azure Blob Storage (through the API) | `BlobStorageService`, `ImageValidator`, `EventsController.Banner`, `UsersController.ProfilePicture` |
| Docker | `src/RaceDay.Mvc/Dockerfile`, `src/RaceDay.Api/Dockerfile`, `docker-compose.yml` |
| UI theme and colour-coded statuses | `wwwroot/css/site.css`, `_StatusBadge.cshtml` |

## Changes to the Part 1 / Part 2 design

Kept to a minimum and documented in [`docs/RaceDay_API_Endpoint_Plan.md`](docs/RaceDay_API_Endpoint_Plan.md) ("Part 3 additions"):

* New `GET /api/event-types` so the event form can list Run / Walk / Cycle.
* The banner and profile-picture endpoints now really store images in Azure Blob Storage.
* Deleting an event or category that is already referenced returns `409 Conflict` instead of a server error.
* Optional start-up settings (`Database:AutoMigrate`, `Database:SeedDemoData`) used by Docker Compose; both are off by default.
* The database schema is unchanged and still matches the ERD and SQL script.

## Security notes

* Passwords are hashed by the API; the MVC site never stores them.
* Every POST is protected by anti-forgery tokens.
* Uploaded images are checked for size, declared type **and** file signature before they reach Azure.
* Secrets (storage keys, SQL passwords) are not committed. Use user-secrets or `.env` (git-ignored). The SQL password and Azurite key in `docker-compose.yml` are throw-away local development values.

## AI usage disclosure

<!-- TODO: edit this so it accurately describes how YOU used AI. The module requires an honest disclosure. -->
AI assistance (Claude, by Anthropic) was used to help generate the initial scaffold of the Part 3 MVC application, Azure Blob Storage integration, Docker configuration, tests and documentation. `[Describe what you reviewed, changed and tested yourself, and which parts you wrote on your own.]`
