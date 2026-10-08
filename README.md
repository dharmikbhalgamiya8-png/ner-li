# NER-LI: Logistics Intelligence

Live fleet map, place search (like Google Maps pickup/drop), route risk scoring and incident reports.

## Run on your computer
Needs Node.js 18 or newer.

    node server.js

Open http://localhost:8080

No packages to install for local use.

## Put it on Google Cloud (Cloud Run)

1. Create a project at https://console.cloud.google.com and turn on billing.
2. Install the Google Cloud CLI, then:

       gcloud auth login
       gcloud config set project YOUR_PROJECT_ID
       gcloud services enable run.googleapis.com cloudbuild.googleapis.com firestore.googleapis.com
       gcloud firestore databases create --location=asia-south1

3. Deploy from this folder:

       gcloud run deploy ner-li \
         --source . \
         --region asia-south1 \
         --allow-unauthenticated \
         --max-instances 1 \
         --set-env-vars STORAGE=firestore,ADMIN_KEY=choose-a-long-secret,DEVICE_KEY=choose-another-secret

4. Google prints a public https link. Share it with anyone.

Keep `--max-instances 1` for now. Vehicle positions are held in memory, so more than one instance would show different data.

### What the keys do
- ADMIN_KEY: needed to create shipments, mark them delivered, and resolve incidents. Anyone can still view the site and report incidents. Leave it empty only for demos.
- DEVICE_KEY: needed by GPS devices or the driver app to send positions.

### Your own domain
Cloud Run > Manage custom domains > Add mapping.

## Send real GPS positions
Each vehicle sends its position, for example every 10 seconds:

    curl -X POST https://YOUR-LINK/api/vehicles/NER-102/location \
      -H "Content-Type: application/json" -H "x-device-key: YOUR_DEVICE_KEY" \
      -d '{"lat": 26.65, "lng": 92.79, "speedKmh": 38}'

While a vehicle sends positions it shows as "Live GPS". If it stops for 2 minutes it goes back to the demo simulation.

## Where the data comes from
| Feature | Source | Key needed |
|---|---|---|
| Map tiles | OpenStreetMap | No |
| Place search | Photon (OpenStreetMap data) | No |
| Road routes | OSRM demo server | No |
| Rain forecast and elevation | Open-Meteo | No |
| Incident reports | Your own users | No |

These free public services are fine for a pilot. For heavy use, switch to paid or self-hosted ones:
set PHOTON_URL and OSRM_URL to your own servers, or replace `searchPlaces` and `getRoute` in server.js with Google Maps Platform or Mapbox calls.

## Environment settings
STORAGE (file or firestore), ADMIN_KEY, DEVICE_KEY, SIM_SPEED (make demo vehicles move faster, for example 20), PHOTON_URL, OSRM_URL, PORT.

## Next steps for a full product
- User accounts and roles (Firebase Authentication) instead of one admin key
- Photo upload for incident reports (Cloud Storage)
- Driver mobile app that sends GPS
- Live traffic data (Google Routes API) and official landslide data
- Replace the rule-based risk score with a trained model once you have enough past data
