# IPG Representative Finder

An interactive map tool that helps Illinois residents find their elected representatives across five layers of government — state, federal, and local — and contact them directly.

**Live at:** https://publicgoodpolicy.github.io/ipg-rep-finder/
**Embedded on:** https://publicgoodpolicy.org

Built and maintained by the Institute for the Public Good.

---

## What it does

Enter an address (or click on the map) and the tool returns your:

- **Illinois House Representative** (1 of 118 districts)
- **Illinois State Senator** (1 of 59 districts)
- **U.S. Congressional Representative** (1 of 17 districts)
- **Chicago Ward Alderperson** (1 of 50 wards — Chicago addresses only)
- **Chicago School Board Member** (1 of 20 subdistricts — Chicago addresses only)

Each rep card displays name, party, district, phone number, email (where published), and a link to their official page. When an email is available, a **Send Email** button opens a pre-filled message form; otherwise the card links to the rep's website.

An optional newsletter checkbox in the email form logs interested users to a private Google Sheet maintained by IPG.

---

## Repository contents

| File | Purpose |
| --- | --- |
| `index.html` | The entire application — HTML, CSS, JS, rep roster, and UI logic in a single file |
| `wards.geojson` | Chicago ward boundaries (2023 redistricting), bundled locally to avoid CORS issues with the city's data portal |
| `README.md` | This file |

District boundary files for IL House, IL Senate, U.S. Congress, and Chicago school board subdistricts are fetched from public GIS endpoints at runtime — only the ward file is bundled.

---

## Architecture

The tool is deliberately a single static HTML file with no build step, no framework, and no backend. That makes it easy to maintain, cheap to host, and fast to load.

**Stack:**

- **Google Maps JavaScript API** — map rendering, geocoding, and Places autocomplete
- **GeoJSON district boundaries** — fetched from state and federal GIS servers (wards bundled locally)
- **Plain JavaScript** — no React, no jQuery, no dependencies
- **GitHub Pages** — free static hosting
- **Squarespace iframe** — embedded into publicgoodpolicy.org

**Rep data** lives as a `REPS` object in `index.html`. Each legislator entry contains `name`, `party`, `district`, `phone`, optional `email`, and `website`.

**Newsletter signups** are sent via `fetch()` to a Google Apps Script web app endpoint, which appends a row to a private Google Sheet. A shared-secret token in the POST body prevents the endpoint from being abused by anyone who discovers the URL.

---

## Data sources

| Layer | Source | Notes |
| --- | --- | --- |
| IL House roster + emails | [ILGA House Member Report](https://ilga.gov/House/Members/rptMemberList) | 110 of 118 members have direct emails published |
| IL Senate roster | [ILGA Senate Member Report](https://www.ilga.gov/Senate/Members/rptMemberList) | ILGA does not publish direct emails for senators — cards link to their official pages |
| U.S. Congress roster | House.gov and Senate.gov member directories | |
| Chicago alderpersons | City of Chicago ward office directory | |
| Chicago school board | Chicago Public Schools elected board roster | |
| IL House/Senate boundaries | Illinois State GIS | |
| Congressional boundaries | U.S. Census Bureau | |
| Chicago ward boundaries | City of Chicago Data Portal (bundled as `wards.geojson`) | |
| Chicago school subdistricts | CPS elected board subdistrict shapefile | |

---

## Local development

To run the tool locally (e.g. to test a roster change before deploying):

1. Clone or download this repo
2. Open a terminal in the repo directory
3. Start a local web server:

   ```
   python3 -m http.server 8080
   ```

4. Open `http://localhost:8080/index.html` in your browser

A local server is required — opening `index.html` directly as a `file://` URL will cause the Google Maps API key to reject requests.

**Note on the API key:** the embedded key is restricted to `publicgoodpolicy.github.io`, `publicgoodpolicy.org`, and `*.publicgoodpolicy.org`. To test locally you need to temporarily add `http://localhost:8080/*` and `http://localhost/*` to the key's allowed referrers in the Google Cloud Console, then remove them when you're done.

---

## Deployment

The repo is wired to GitHub Pages. Any commit pushed to the `main` branch publishes automatically within ~60 seconds to:

```
https://publicgoodpolicy.github.io/ipg-rep-finder/
```

The Squarespace embed points at that URL via iframe, so no change to the Squarespace site is needed when updating the tool — just push to `main`.

---

## Maintenance

### Updating legislator data

Legislators change. New elections, retirements, appointments, deaths, party switches. When data goes out of date:

1. Open `index.html`
2. Find the `REPS` object — it's organized by layer (`house`, `senate`, `congress`, `ward`, `school`) and keyed by district number
3. Edit the relevant entry
4. Commit and push — the live site updates automatically

For a bulk refresh after an election, the ILGA member reports linked above are the authoritative source for IL House and Senate.

### The November 2026 school board change

The Chicago school board transitions from 10 elected / 10 appointed to **20 fully elected** districts in November 2026. When that happens:

1. Swap the subdistrict GeoJSON (CPS will publish the new boundaries)
2. Replace all 20 entries in `REPS.school` with the newly elected members

### Google Maps API key

The key is restricted to IPG's domains. If the tool moves to a new domain, the key's allowed referrers need to be updated in the Google Cloud Console under **APIs & Services → Credentials**.

If the key is ever compromised (e.g. accidentally leaked on a public domain), rotate it: generate a new key in Cloud Console, replace the old one in `index.html`, and delete the old key.

### Newsletter signups

Signup data lives in a private Google Sheet owned by IPG. The Apps Script endpoint and shared-secret token are configured in the `logNewsletterSignup()` function in `index.html`.

To rotate the token (recommended periodically or after any suspected leak):

1. Generate a new random token string
2. Update the `data.token` check in the Apps Script
3. Deploy a new version: **Deploy → Manage deployments → edit → New version → Deploy**
4. Update the `token` value in the `logNewsletterSignup()` function in `index.html`
5. Commit and push

The deployment URL stays the same across token rotations.

---

## Known limitations

- **`mailto:` throttling** — browsers rate-limit rapid clicks on `mailto:` links as spam protection. If the Send Email button doesn't respond on the first click, wait 3–5 seconds before trying again. This is browser behavior, not a code bug.
- **Senate email coverage** — ILGA doesn't publish direct emails for state senators. Senate cards link to each senator's official page, which includes a contact form.
- **CORS on city data portals** — the Chicago Data Portal blocks cross-origin browser requests, which is why `wards.geojson` is bundled rather than fetched live. If ward boundaries change, the bundled file needs to be manually refreshed.
- **Mobile `mailto:` behavior** — mobile browsers are more aggressive about throttling `mailto:` links than desktop browsers.

---

## Credits

Built for the Institute for the Public Good. Data sourced from the Illinois General Assembly, U.S. Census Bureau, City of Chicago, and Chicago Public Schools.
