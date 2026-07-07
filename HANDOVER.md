# Teetigu-kaart — HANDOVER

**Uuendatud:** 2026-07-08
**Mis see on:** kiire õpi-MVP. Veebirakendus (Lovable = React + Supabase), kus avalik saab kaarti
vaadata ja sisseloginud kasutajad (Facebook + Google) märgivad kaardile hispaania teetigu
(*Arion vulgaris*) **olemasolu/puudumise**, sh mitu tigu hävitasid. Eesmärk: ehituskogemus +
päris kasutajate tagasiside. Konkurent olemas (`teokaart.keskkonnaamet.ee`) — ehitame teadlikult
madalama hõõrdumise + hävitamis-mänguelemendi + isikliku "aia riski" nurgaga.

## Kus pooleli jäime
Disain valmis ja heaks kiidetud (4 rolli: oponent/mentor/coach/uurija sisend põimitud). Valmis:
- **`LOVABLE-PROMPT.md`** — copy-paste prompt Lovable'i.
- **`seed/keskkonnaamet-teetigu.geojson`** (1,9 MB) + **`seed/seed-observations.sql`** (1,2 MB, 16
  batch-INSERT'i) — **7702 punkti** Keskkonnaameti teokaardilt: `arion_vulgaris` **3248** +
  `teadmata` **4454** (mustpeanälkjas 741 välja jäetud). PII eemaldatud, kommentaarid regex-puhastatud
  (0 e-posti/telefoni-leket, kontrollitud). Lahendab tühja-kaardi cold-start'i.
- Täielik disain: `~/.claude/plans/fluffy-leaping-bengio.md`.

**GitHub:** https://github.com/hv-tec/teetigu-kaart (avalik, MIT). Lovable saab siia app-koodi pushida.

**Järgmine samm = Heikki teeb Supabase + OAuth setupi, siis kleebib prompti Lovable'i.**

**Supabase-projekt (soovitus):** loo UUS tasuta Supabase-org + free-projekt `teetigu-kaart` — mitte
äri-orgi (hv-tec) alla. Põhjus: avalikult kirjutatav tabel (RLS blast-radius), eraldi `auth.users`,
erinev andmekaitse-piir (mittetulunduslik vs äri). Free tier katab MVP + seemet kergelt, ja on tasuta.
Kiiruse eelistamisel saab ka olemasolevasse projekti panna — Heikki otsus.

## Otsused (lukustatud)
| Teema | Otsus |
|---|---|
| Auth | Facebook + Google (Google töötab kohe; FB pärast Meta review'd) |
| Foto | Ei v1-s |
| Nähtavus | Kaart avalik; teate lisamiseks login |
| Skoop | Eesti + eesti keel |
| Kaart | Maa-amet ortofoto + katastripiirid + lähiaadress (nagu X-GIS Maainfo), react-leaflet EPSG:3857 |
| V1 lisad | "sinu aia risk" + üks tagasiside-küsimus + "mitu hävitasid" + kogukonna kokku-loendur |
| Seeme | ~7500 Keskkonnaameti punkti, avaandmena atribuudiga "Allikas: Keskkonnaamet" |
| DB | Uus eraldi Supabase-projekt (mitte jagatud — RLS blast-radius) |

## Setup-sammud (Heikki, Lovable-väline)
1. **Uus Supabase-projekt** samas orgis (nt `teetigu-kaart`). Kopeeri URL + anon key.
2. **Google OAuth** Supabase Auth → Providers (kohe, pole review'd) — primaarne, väldib launch-blokki.
3. **Facebook OAuth**: Meta app (Consumer) → Facebook Login → Redirect
   `https://<ref>.supabase.co/auth/v1/callback`. Dev-mode töötab kohe sulle; avalikuks vajab
   app-review'd — **ära hoia launchi selle taga**, Google katab.
4. Lovable → ühenda Supabase → kleebi `LOVABLE-PROMPT.md` → jooksuta migratsioon (tabelid+RLS).
5. **Lae seeme:** Supabase SQL-editoris jooksuta `seed/seed-observations.sql`.
6. Publish + dogfood telefonis → postita FB aiandusgruppidesse (`Aiandushuvilised`, `Koduaed`).

## Verifitseerimine
Vt `~/.claude/plans/fluffy-leaping-bengio.md` "Verifitseerimine" (9 punkti): auth, teade, RLS-test,
risk-badge, hävitamine, tagasiside, privaatsus-ümardus, Maa-ameti kihid, seeme.
**Kriitiline:** testi RLS — teise user_id-ga INSERT/UPDATE peab ebaõnnestuma.

## Lahtised (mitte v1)
- FB-provider live (Meta app-review + privaatsuspoliitika URL).
- Litsents: anna Keskkonnaametile viisakuse mõttes teada andmete taaskasutusest.
- Seeme = staatiline hetktõmmis; hiljem korduv sync ArcGIS FeatureServer'ist kui idee edasi läheb.
- Skaleerudes: server-side risk-RPC (PostGIS), rate-limit, foto + async liigivalideerimine.

## Allikad (tehniline, verifitseeritud 2026-07)
- Seeme-API: `services9.arcgis.com/X7QfjKHQutd5fAAN/.../survey123_f50da40017564ae28439e93aefc44159/FeatureServer/0`
- Maa-amet tiles: `tiles.maaamet.ee/tm/tms/1.0.0/{kaart|foto|hybriid}@GMC/{z}/{x}/{y}.png&ASUTUS=..&KESKKOND=..` (`tms:true`, `{y}`)
- Kataster WMS: `gsavalik.envir.ee/geoserver/kataster/wms`, layer `ky_kehtiv_aadress`
