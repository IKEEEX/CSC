# My Festival API — endpoint-dokumentti / endpoint document

> FI: Täytä tämä pohja ja palauta se Moodleen (Markdown tai PDF, 1–2 sivua). Korvaa kaikki `<…>`-kohdat.
> Esimerkkivastaukset kopioit oman julkisen rajapintasi oikeista vastauksista (selain tai `curl.exe`/`curl`).
>
> EN: Fill in this template and submit it to Moodle (Markdown or PDF, 1–2 pages). Replace every `<…>`.
> Copy the example responses from your own public API's real responses (browser or `curl.exe`/`curl`).

**Tekijä / Author:** `Iikka Laakso`
**Julkinen osoite / Public URL:** `https://festival-api-laaksoii.2.rahtiapp.fi`
**GitHub:** `https://github.com/IKEEEX/CSC.git`
**Tarkistimen tulos / Checker result:** `Testing https://festival-api-laaksoii.2.rahtiapp.fi
[PASS] GET /api/artists returns 200
[PASS]   Content-Type is application/json
[PASS]   response is a non-empty JSON list
[PASS]   items have artist_id and name
[PASS] GET /api/days returns day_id + date
[PASS] GET /api/stages returns stage_id + name
[PASS] GET /api/artists/11/events returns a JSON list
[PASS] GET unknown artist returns 404
[PASS] GET /api/timetable returns a JSON list
[PASS] GET /api/timetable?day=2026-07-05 returns only that day
[PASS]   rows are in time order (starts)
[PASS] GET timetable with bad date returns 400
[PASS] POST with missing fields returns 400
[PASS] POST with bad foreign key returns 400
[PASS] POST with bad start_dt format returns 400
[PASS] POST with valid body returns 201 + event_id
[PASS]   new event is returned by GET /api/artists/{id}/events
[PASS] DELETE /api/events/16 returns 204
[PASS]   deleted event is gone
[PASS] DELETE same event again returns 404

20/20 checks passed`

---

## 1. Endpointit / Endpoints

| Metodi / Method | Polku / Path | Onnistuu / Success | Virheet / Errors |
|---|---|---|---|
| GET | `/api/artists` | 200 | – |
| GET | `/api/days` | 200 | – |
| GET | `/api/stages` | 200 | – |
| GET | `/api/artists/{id}/events` | 200 | 404 |
| GET | `/api/timetable?day=YYYY-MM-DD` | 200 | 400 |
| POST | `/api/events` | 201 | 400 |
| DELETE | `/api/events/{id}` | 204 | 404 |

## 2. Esimerkit / Examples

Kirjoita jokaisesta endpointista esimerkkikutsu ja (lyhennetty) vastaus. Malli:
For every endpoint, write an example call and a (shortened) response. Example:

### GET /api/artists

```bash
curl https://festival-api-laaksoii.2.rahtiapp.fi/api/artists
```
```json
[{"artist_id": 3, "country": "FIN", "name": "Asa"}, …]
```

### GET /api/artists/{id}/events
```bash
curl https://festival-api-laaksoii.2.rahtiapp.fi/api/artists/2/events
```
```json
  [{"day": "2026-07-31", "event_id": 12,"stage": "Pieni A", "starts": "2026-07-31 18:45:00"}]
```
Tuntematon id / unknown id → `404, {"error": "artist_id 100 not found"}`

### GET /api/timetable?day=YYYY-MM-DD
```bash
curl https://festival-api-laaksoii.2.rahtiapp.fi/api/timetable?day=2026-07-31
```
```json
[{"artist": "Angerfist", "date": "2026-07-31", "ends": "2026-07-31 19:30:00", "stage": "Pieni A", "starts": "2026-07-31 18:45:00"},
{"artist": "Are", "date": "2026-07-31", "ends": "2026-07-31 19:30:00", "stage": "Pieni B", "starts": "2026-07-31 18:45:00"},
```
Virheellinen päivämäärä / invalid date → `400, {"error": "day must be a valid date in format YYYY-MM-DD"}`

### POST /api/events
`start_dt` pitää antaa kannan muodossa `YYYY-MM-DD HH:MM:SS` (sekunnit vaaditaan).

```bash
curl -X POST https://festival-api-laaksoii.2.rahtiapp.fi \
     -H "Content-Type: application/json" \
     -d '{"artist_id": 11, "stage_id": 3, "day_id": 2, "start_dt": "2026-07-31 23:00:00"}'
```
```json
{"event_id": 15, "artist_id": 11, "stage_id": 3, "day_id": 2, "event_type": "keikka", "start_dt": "2026-07-31 23:00:00",  "end_dt": "2026-07-31 23:45:00", "duration_min": 45}
```
Puuttuva kenttä / missing field → `400, {"error": "missing fields: stage_id, day_id, start_dt"}` · Olematon artist_id / unknown artist_id → `400,"artist_id, stage_id or day_id does not exist" `

### DELETE /api/events/{id}
```bash
curl https://festival-api-laaksoii.2.rahtiapp.fi/api/events/100
```
Onnistuu / success → `204` · Tuntematon id / unknown id → `404, {"error": "event_id 100 not found"}`

## 3. Julkaisu Rahtiin / Deploying to Rahti (3–6 lausetta / sentences)

<FI: Kuvaa omin sanoin build → deployment → route: mitä Rahti teki repostasi ja mistä löysit julkisen osoitteen.
Mainitse yksi ongelma, johon törmäsit, ja miten ratkaisit sen.>
Rahti rakensi Dockerfilen mukaan Imagen ohjelmasta, käynnisti kontin, antoi minulle osoitteen ja löysin julkisen osoitteen -> Routes -> Location: "https://festival-api-laaksoii.2.rahtiapp.fi". Ongelmana tuli ohjeen Add-> Import from Git, se vaati hieman etsiskelyä mutta loppujen lopuksi löytyi kun etsin googlesta.
<EN: Describe build → deployment → route in your own words: what Rahti did with your repo and where you found
the public address. Mention one problem you ran into and how you solved it.>

## 4. Tietoturva / Security (2–4 lausetta / sentences)

<FI: Mitä tämä rajapinta tarvitsisi ennen oikeaa tuotantokäyttöä? Miten varmistit, ettei SQL-injektio onnistu?>
Harjoitus-API on avoin kaikille ilman kirjautumista. Tuotantokäytössä tietoturvaa parannettaisiin API-avaimilla tai OAuthilla, CORS-rajoituksilla, kutsurajoilla ja HTTPS-yhteydellä. Näin rajapinnan käyttöä voidaan hallita ja suojata väärinkäytöksiltä. Estin SQL-injektiot parametrisoimalla kyselyt.
<EN: What would this API need before real production use? How did you make sure SQL injection is not possible?>
