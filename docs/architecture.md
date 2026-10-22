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

Record your plan here before editing code.

- Presentation tier decision: Keep route-level React components focused on rendering and user interaction; use the shared API client for requests and expose loading, empty, and failure states accessibly.
- Logic tier decision: Keep Express routes responsible for HTTP parsing/status codes and put validation and domain behavior in the OctoFit service. Return deliberate 4xx responses for invalid input or missing records rather than leaking database errors.
- Data tier decision: Use Mongoose schemas for required fields, defaults, references, and value constraints. Keep seed data representative and idempotent without deleting or rewriting unrelated records.
- Activity tracking approach: Validate the submitted user and team IDs and confirm both records exist before creating an activity. Persist the user, team, type, and minutes with service-computed positive points and an occurrence date.
- Team management approach: Require a name, preserve the mascot, and normalize members to an array of member IDs; return the created team and populated/listable member references through the API.
- Leaderboard approach: Aggregate activity points by team, sort descending by total, and assign 1-based ranks with the team name and total points.
- Workout suggestions approach: Find the user by email, use the fitness goal and recent activity to prioritize a matching workout, and include a reason; return a deliberate 4xx response when the user does not exist.

## Test plan

Record the commands you run and what each failure means.

- Baseline setup: `npm ci --prefix octofit-tracker/backend` and `npm ci --prefix octofit-tracker/frontend` install exactly the dependencies in each tier's lockfile.
- Baseline commands run: `npm test --prefix octofit-tracker/backend` failed with 8 of 9 tests failing (all 4 API tests and 4 of 5 data tests); `npm test --prefix octofit-tracker/frontend` failed with all 15 presentation tests failing. These are expected starter failures across the data/API contracts and pending React views, not environment failures; MongoDB was running on port `27017`.
- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`. Failures indicate missing schema constraints/defaults or non-idempotent/incomplete seed behavior.
- API command: `npm run test:api --prefix octofit-tracker/backend`. Failures indicate incorrect HTTP validation/status handling or activity, team, leaderboard, and workout service behavior.
- Frontend command: `npm test --prefix octofit-tracker/frontend`. Failures indicate missing API rendering, route behavior, or accessible loading, empty, and error states.
- Backend build command: `npm run build --prefix octofit-tracker/backend`. Failures indicate TypeScript errors in the backend.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`. Failures indicate TypeScript or Vite build errors in the frontend.
