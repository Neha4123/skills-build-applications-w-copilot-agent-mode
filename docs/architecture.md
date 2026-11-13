# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

- Presentation tier decision: Keep the React 19 views in `frontend/src/components`, use React Router for the four capabilities, and load API data through the shared `src/api/client.ts` client. Preserve accessible loading, empty, success, and error states. Use relative `/api` paths through the Vite proxy, with `VITE_API_BASE_URL` as the optional override.
- Logic tier decision: Keep `/api/*` endpoints in the Express router and put business behavior in explicit service functions. Validate request shape and referenced records before writing; return deliberate 4xx responses for invalid or unknown input rather than exposing cast errors as 500s.
- Data tier decision: Define required fields, relationships, and defaults in the Mongoose models. Use the shared database configuration and keep persistence and seed logic in the backend; seed stable demo records with upserts without deleting unrelated data.
- Activity tracking approach: List persisted activities and create records from `userId`, `teamId`, `type`, and `minutes` after validating both IDs and confirming both records exist. Store references as `user` and `team`, and set positive `points` and `occurredAt` when creating the activity.
- Team management approach: List and create teams through the service layer. Require a name, preserve the submitted name and mascot, and support a members array that defaults to empty.
- Leaderboard approach: Sum persisted activity points per team, sort by total points descending, and return `teamName`, `totalPoints`, and sequential 1-based ranks.
- Workout suggestions approach: Find the requested user by email and use that user's goal and recent activity to prioritize matching workout records. Include a human-readable reason for the leading suggestion and return a deliberate 4xx when the user is unknown.

## Test plan

Run focused contract tests while implementing each tier, then run both builds to catch type and production-bundle errors. A data-test failure points to model or seed behavior; an API-test failure points to route/service behavior or its persistence interactions; a frontend-test failure points to rendering, API usage, or accessible UI states. A build failure indicates a TypeScript or bundling issue. Backend integration tests use an in-memory MongoDB, so a database binary/network startup error is an environment issue rather than an application assertion failure.

- Data-tier contract test: `npm run test:data --prefix octofit-tracker/backend`
- API contract test: `npm run test:api --prefix octofit-tracker/backend`
- Frontend contract tests: `npm test --prefix octofit-tracker/frontend`
- Backend type-check: `npm run build --prefix octofit-tracker/backend`
- Frontend type-check and production build: `npm run build --prefix octofit-tracker/frontend`
