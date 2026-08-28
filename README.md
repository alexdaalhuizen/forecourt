# Forecourt

Petrol spend and fuel economy for our two cars, shared. Log a fill-up at the
pump — amount, litres, odometer — and the app works out the monthly spend,
price per litre, km/L and cost per km for each car.

Same shape as [meal-plan](https://github.com/alexdaalhuizen/meal-plan): one
static `index.html` on GitHub Pages, data in Supabase over PostgREST, config
carried in the URL hash. Offline-first — it opens and records fill-ups with no
signal, and syncs when it gets one.

## Setup

1. **Pages** — Settings → Pages → Source: *Deploy from a branch*, branch `main`,
   folder `/ (root)`.
2. **Supabase** — reuses the same project and the same `items` table as
   meal-plan, under a different `household` code, so the two apps never see each
   other's rows. Nothing to create.
3. **Connect** — open the app and tap the pill in the top right, or just open the
   share link (below), which fills in the connection and saves it to the device.

## Sharing

Config travels in the URL fragment as base64url `{u, k, h}` — project URL,
publishable key, household code:

```
https://alexdaalhuizen.github.io/forecourt/#s=<base64url>
```

Open that link once on each device and it joins the same log. The app reads it,
saves it to `localStorage`, and strips the hash from the address bar. Anyone with
the link can read and write the log, so keep it to the household.

## Data

One row per fill-up in the shared `items` table:

| column | value |
|---|---|
| `id` | `f-…` for a fill-up, `c-a` / `c-b` for a car |
| `household` | this app's household code |
| `kind` | `fill` or `car` |
| `doc` | `{date, car, amount, litres, odo}` — or `{name, order}` for a car |
| `deleted` | soft delete |
| `updated_at` | last-write-wins on conflict |

Fill-ups are independent rows, so two people logging at the same time never
collide. Deletes are soft, so a delete on one phone reaches the other.

Fuel economy uses the full-tank method: distance is the odometer difference
since that car's previous fill-up, divided by the litres in the current one.
It assumes you fill the tank each time. The first fill-up for each car has no
economy figure — there's nothing to measure from yet.

## Files

| file | what it is |
|---|---|
| `index.html` | the whole app — markup, styles, logic |
| `sw.js` | offline shell; app cached, data never |
| `manifest.webmanifest` | installable PWA |
| `icon-*.png`, `apple-touch-icon.png` | home-screen icons |

## Local

```sh
python3 -m http.server 8000
```

Then open `http://localhost:8000/`. Service workers and `localStorage` both need
an origin, so opening `index.html` as a file won't behave.
