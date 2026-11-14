# Part 3 - Rubric self-check

Tick each line yourself before submitting. Everything marked **[you]** can only be done by you.

## Correct Submission (5)
- [ ] README has: system description, the two roles, CI/CD screenshot, YouTube link, Docker run command **[you: screenshot + link]**
- [ ] `/docs` contains the ERD, endpoint plan and SQL script

## GitHub and CI/CD (10)
- [ ] 20 or more meaningful commits for Part 3 **[you: commit your own work as you go]**
- [ ] `.github/workflows/ci.yml` shows a green tick (validates structure, builds API + MVC + tests, builds both Docker images)

## Authentication and Role-Based Navigation (10)
- [ ] Register + login through the MVC site (`AccountController`) - calls the API only
- [ ] MVC session stores user id, role and the API session cookie (`SessionKeys`, `RaceDayApi`)
- [ ] Different menus per role (`Views/Shared/_Layout.cshtml`)
- [ ] Logout clears the session (POST `/Account/Logout`)
- [ ] Unauthenticated users are redirected to login (`RequireLoginAttribute`)

## Organiser Interface (15)
- [ ] Dashboard - events created, enrolments per event, upcoming dates (`Organiser/Dashboard`)
- [ ] Events - create / edit / delete with every field + banner upload (`ManageEvents`)
- [ ] Categories (`ManageCategories`)
- [ ] Enrolments with category and colour-coded status (`ManageEnrolments`)
- [ ] Results entry - finish time and position, only after the event date (`ManageResults`)

## Participant Interface (15)
- [ ] Home page with upcoming events and filters (`Home/Index`)
- [ ] Event detail with categories and **Enter event** (`Events/Details`)
- [ ] My events with status (`Participant/MyEnrolments`)
- [ ] My results - event, date, category, time, "47th of 312" (`Participant/MyResults`)
- [ ] Profile with picture upload (`Account/Profile`)

## Azure Blob Storage (10)
- [ ] MVC sends the file to the **API** (never to Azure) - `RaceDayApi.SendFileAsync`
- [ ] API stores it in Azure Blob Storage and saves the URL (`BlobStorageService`, `EventsController.Banner`)
- [ ] Image shows on the event detail page
- [ ] Tested against **real Azure** at least once (see `docs/Azure_Blob_Setup.md`) **[you]**

## Docker (10)
- [ ] `docker compose up --build` starts everything
- [ ] `docker run` command for the MVC container works (README)
- [ ] Container behaves the same as running locally

## UI Design (10)
- [ ] RaceDay brand theme, consistent fonts/colours/spacing (`wwwroot/css/site.css`)
- [ ] Green Confirmed / amber Pending / red Cancelled badges
- [ ] Friendly errors - no raw exception text

## Video (15) **[you]**
- [ ] Your own voice (no AI voice), shows the running app, explains MVC structure, Blob Storage and Docker
- [ ] Unlisted YouTube link added to README

## Academic integrity (read this)
- [ ] You can explain every file you submit - the video marker will ask you to
- [ ] AI use is disclosed in the README (footnote section at the bottom)
- [ ] Referencing: if your submission includes a written component, reference it in Harvard style
