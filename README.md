# Planner Resume Vue

This repository is the frontend of a small interactive resume. It is also a safe test repository for the Planner agent runner.

The page reads profile and project data from the companion Go API. It also calls the API for a visit counter and a demo contact form.

## Run it

Start the backend first. Then run:

```bash
pnpm install
cp .env.example .env
pnpm dev
```

Open `http://localhost:5173`.

Set `VITE_API_BASE_URL` if the Go API does not run at `http://localhost:8080`.

## Build it

```bash
pnpm build
pnpm preview
```

## Planner test ideas

- Add a dark theme switch.
- Add project search.
- Add form character counts.
- Add loading skeletons.
