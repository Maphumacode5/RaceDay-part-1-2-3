# Part 3 video outline (unlisted YouTube, your own voice)

Aim for 8-12 minutes. Record the running app first, then talk over it. No AI-generated voice.

1. **Intro (30s)** - what RaceDay is, what Part 3 adds.
2. **Architecture (1 min)** - draw or show: Browser -> MVC site -> REST API -> SQL Server, and API -> Azure Blob Storage. Point out the MVC project has **no database code** and never talks to Azure.
3. **Code tour - MVC (2-3 min)**
   - `Program.cs`: session, typed `HttpClient`, error pages.
   - `Services/RaceDayApi.cs`: how the API's session cookie is captured at login and replayed per user.
   - `Infrastructure/RequireLoginAttribute.cs`: how unauthenticated users and wrong roles are handled.
   - One controller + view end to end (e.g. `ManageEventsController` + `_EventForm.cshtml`).
4. **Demo - Organiser (2 min)** - register/login, dashboard, create event **with banner**, add categories, see enrolments, capture a result.
5. **Demo - Participant (2 min)** - filters on the home page, event detail, enter an event, My events (status colours), My results (e.g. "47th of 312"), profile picture.
6. **Azure Blob Storage (1-2 min)** - show `BlobStorageService`, then the blob appearing in the Azure portal.
7. **Docker (1-2 min)** - show the Dockerfile stages, run the single `docker run` command (or `docker compose up`), prove it behaves like the local run.
8. **CI/CD (30s)** - show the green GitHub Actions run.
9. **Wrap-up** - what you learned and what you would improve.

Tips: have an Organiser and a Participant account ready (the compose demo data creates `organiser@raceday.co.za` and `participant@raceday.co.za`, password `RaceDay#2026`).
