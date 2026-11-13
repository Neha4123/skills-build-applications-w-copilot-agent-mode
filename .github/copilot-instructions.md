# OctoFit Tracker agent instructions

## Project context

OctoFit Tracker is an intentionally incomplete exercise for a school fitness program. Read `docs/architecture.md`, the relevant contract tests, and the scoped instructions in `.github/instructions/` before changing application code. The failing starter tests are expected; implement their specified behavior rather than replacing or weakening the contracts.

Keep the three tiers separate:

- **Presentation:** React 19, Vite, React Router, and Bootstrap in `octofit-tracker/frontend`.
- **Logic:** Express 5 API routes and service functions in `octofit-tracker/backend/src`.
- **Data:** Mongoose models, shared database configuration, and seed script in `octofit-tracker/backend/src`.

Use MongoDB on port `27017`, the API on port `8000`, and Vite on port `5173`. Preserve the browser's relative `/api` requests through the Vite proxy; do not make the API port public. Do not add authentication; it is a follow-up challenge.

## Implementation workflow

- Before implementation, complete the implementation decisions and test plan in `docs/architecture.md`, as requested by Step 1.
- Make the smallest change that satisfies the relevant contract. Work one capability and tier at a time; avoid broad regeneration and unrelated refactors.
- Preserve existing TypeScript types and project conventions. Keep request handling and HTTP status codes in routes, business behavior in services, and persistence validation in Mongoose schemas.
- Validate user input explicitly and return deliberate 4xx responses for invalid or unknown inputs; do not leak cast errors as 500s or return success-shaped fallbacks.
- Keep demo seeding repeatable and safe: use stable keys/upserts, preserve unrelated records, and reconcile fixture relationships when needed.
- In the frontend, construct API requests through `src/api/client.ts` and `getJson`; components must not make their own `fetch` calls. Provide accessible loading, empty, success, and error states.

## Contracts and validation

Treat the contract tests and the protected configuration checked by `.github/scripts/grade-step.mjs` as maintainer-owned. Do not edit tests, `tsconfig.json`, or the frontend `vite.config.ts` to make grading pass. Do not weaken the grader or npm scripts. If a contract appears inconsistent, explain the evidence rather than changing the protected file.

Run the narrowest applicable check after each change:

- Data tier: `npm run test:data --prefix octofit-tracker/backend`
- API tier: `npm run test:api --prefix octofit-tracker/backend`
- Frontend: `npm test --prefix octofit-tracker/frontend`
- Backend type-check: `npm run build --prefix octofit-tracker/backend`
- Frontend type-check and production build: `npm run build --prefix octofit-tracker/frontend`

For final exercise verification, run all five commands above. Integration tests may require MongoDB; distinguish an unavailable database from an application contract failure. Report the checks run and any environmental limitation.
