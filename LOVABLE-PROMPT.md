# Lovable-prompt — Teetigu-kaart

> Kopeeri allolev (koodiploki sisu) tervikuna Lovable'i. Kirjutatud inglise keeles (Lovable annab
> nii parema tulemuse), aga kogu kasutajaliides tuleb eesti keeles. Enne kleepimist: loo Supabase
> projekt ja ühenda see Lovable'iga (vt `HANDOVER.md` → Setup-sammud).

```
Build "Teetigu-kaart" — a citizen-science web app for reporting the Spanish slug (Arion vulgaris, Estonian "hispaania teetigu") on a map.

STACK: React + Vite + Tailwind + Supabase (already connected). Use react-leaflet with Estonian Maa-amet map tiles (free, no API key) and leaflet.markercluster for clustering. ALL user-facing text in Estonian. Keep it a lean MVP — no extra features beyond what's specified.

SUPABASE — run this migration (tables + Row Level Security). RLS must be ON:

create table public.observations (
  id uuid primary key default gen_random_uuid(),
  user_id uuid default auth.uid() references auth.users(id),  -- nullable: imported seed rows have no user
  source text not null default 'user',           -- 'user' | 'keskkonnaamet'
  species text not null default 'arion_vulgaris', -- 'arion_vulgaris' | 'teadmata'
  lat double precision not null,
  lng double precision not null,
  present boolean not null,
  killed_count integer not null default 0,
  note text,
  observed_at timestamptz not null default now()
);
alter table public.observations enable row level security;
create policy "select_all" on public.observations for select using (true);
create policy "insert_own" on public.observations for insert with check (auth.uid() = user_id);
create policy "update_own" on public.observations for update using (auth.uid() = user_id);

create table public.feedback (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null default auth.uid() references auth.users(id),
  message text not null,
  created_at timestamptz not null default now()
);
alter table public.feedback enable row level security;
create policy "fb_insert_own" on public.feedback for insert with check (auth.uid() = user_id);
create policy "fb_select_own" on public.feedback for select using (auth.uid() = user_id);

AUTH: Supabase social login with Google and Facebook buttons. The map is publicly viewable without login. Only adding a report requires login — gate the "Lisa teade" flow behind auth and prompt sign-in there. On the login screen show a short Estonian privacy note with a link to the privacy page: "Sisse logides nõustud, et sinu teate ligikaudne asukoht on avalikul kaardil nähtav. Loe lähemalt: Privaatsus." (link → /privaatsus).

PRIVACY PAGE: add a route /privaatsus rendering the static Estonian privacy policy below (headings + paragraphs, readable, max-width text column). Link to it from the login screen and from a small footer/menu link reachable from the map. The footer should read: "Kogukonnaprojekt · avatud lähtekood · Privaatsus" (with "avatud lähtekood" linking to https://github.com/hv-tec/teetigu-kaart and "Privaatsus" linking to /privaatsus). This page must be publicly accessible without login (Meta app review requires a public privacy policy URL). Content verbatim:

  # Privaatsuspoliitika — Teetigu-kaart

  Viimati uuendatud: 2026.

  ## Kes me oleme
  Teetigu-kaart on kodanikuteaduse rakendus hispaania teetigu (Arion vulgaris) leviku kaardistamiseks
  Eestis. Vastutav töötleja: [SINU NIMI/ETTEVÕTE]. Kontakt: [SINU E-POST].

  ## Mittetulunduslik kogukonnaprojekt
  See on kogukonna hüvanguks tehtud projekt, millel puudub äriline eesmärk. Me ei teeni rakendusega
  tulu ega müü andmeid. Lähtekood on avalikult saadaval Gitis (https://github.com/hv-tec/teetigu-kaart) ja vabalt kasutatav.

  ## Milliseid andmeid kogume
  - **Konto**: sisse logides Google või Facebook kaudu saame su nime ja e-posti aadressi (Supabase Auth kaudu).
  - **Teated**: sinu märgitud asukoht (koordinaadid), kas nägid teetigu või mitte, mitu tigu hävitasid,
    valikuline märkus ja aeg.
  - **Tagasiside**: kui saadad tagasiside-vormi, salvestame su vaba teksti.

  ## Kuidas andmeid kasutame ja kuvame
  Teadete andmeid kuvatakse avalikul kaardil, et koguda ülevaadet teetigu levikust. **Avalikul kaardil
  näidatakse asukohta ligikaudsena (ümardatud ~100 m täpsuseni)**, et mitte paljastada täpset kodu.
  Sinu nime, e-posti ega tagasisidet avalikult ei kuvata.

  ## Õiguslik alus
  Töötleme andmeid sinu nõusoleku alusel (GDPR art 6 lg 1 p a), mille annad kontot luues ja teadet lisades.

  ## Seemendatud andmed
  Kaart sisaldab ka Keskkonnaameti avalikult jagatud teokaardi andmeid (allikas: Keskkonnaamet),
  millest on eemaldatud isikuandmed (nimed, kontaktid).

  ## Andmete jagamine
  Me ei müü ega jaga su isikuandmeid kolmandatele isikutele. Andmeid hoitakse Supabase (PostgreSQL) teenuses.

  ## Sinu õigused
  Sul on õigus oma andmetega tutvuda, neid parandada või kustutada. Selleks kirjuta: [SINU E-POST].

  ## Küpsised
  Kasutame ainult sisselogimiseks vajalikke tehnilisi küpsiseid (Supabase seanss). Jälgimisküpsiseid ei kasuta.

  ## Muudatused
  Uuendame seda poliitikat vajadusel; muudatused avaldatakse siin lehel.

MAIN SCREEN — full-page map. Use Maa-amet tiles so it looks like the X-GIS Maainfo view: orthophoto + yellow cadastral parcel boundaries + address labels. Standard EPSG:3857 Leaflet map — do NOT add proj4 / EPSG:3301 / L-EST; the @GMC tiles are Web Mercator.
- react-leaflet MapContainer centered on Estonia ([58.6, 25.0], zoom 8).
- Base layers via LayersControl (default = Ortofoto). Build the tile URL like this:
    const ASUTUS = 'teetigu'; const ENV = 'LIVE';
    const gmc = (l) => `https://tiles.maaamet.ee/tm/tms/1.0.0/${l}@GMC/{z}/{x}/{y}.png&ASUTUS=${ASUTUS}&KESKKOND=${ENV}`;
    // <TileLayer url={gmc('foto')}    tms maxNativeZoom={18} maxZoom={20} attribution="Aluskaart: Maa-amet" />  (default checked, name "Ortofoto")
    // <TileLayer url={gmc('kaart')}   tms maxNativeZoom={18} maxZoom={20} attribution="Aluskaart: Maa-amet" />  (name "Kaart")
    // <TileLayer url={gmc('hybriid')} tms maxNativeZoom={18} maxZoom={20} attribution="Aluskaart: Maa-amet" />  (name "Hübriid")
  IMPORTANT: keep the URL verbatim — it really contains ".png&ASUTUS=" with an & and no ?. Use {y} with tms (NOT {-y}).
- Cadastral overlay (LayersControl.Overlay, default ON) via WMSTileLayer, showing parcel boundaries + nearest address:
    <WMSTileLayer url="https://gsavalik.envir.ee/geoserver/kataster/wms"
      layers="ky_kehtiv_aadress" format="image/png" transparent version="1.1.1"
      attribution="Katastripiirid: Maa-amet" />
- Load all rows from observations and render as clustered markers via leaflet.markercluster. Marker color: red = present (tigu nähtud), grey = absent (ei näinud). Rows with source = 'keskkonnaamet' are imported seed data — render them slightly muted (smaller / more transparent red circle) to distinguish them from fresh user reports; their popup shows "Allikas: Keskkonnaamet", the date, and the abundance from note. User reports (source = 'user') show date, status, and if killed_count > 0 "Hävitatud: N". Use colored circle/pin icons with a light outline (readable on the orthophoto), not the default blue pin. If you use default Leaflet icons, fix the marker-icon bundler bug (import marker-icon / marker-icon-2x / marker-shadow explicitly or set L.Icon.Default.imagePath).
- Attribution control must include: "Aluskaart: Maa-amet", "Katastripiirid: Maa-amet", "Andmed osaliselt: Keskkonnaamet".
- Privacy: when displaying markers publicly, round lat/lng to 3 decimals (~100 m) so exact home gardens aren't exposed.
- Floating primary button bottom-right: "+ Lisa teade".
- Community counter — a small compact badge pinned at the top of the map: "🐌 Kokku hävitatud: N" where N = sum(killed_count) across all observations. Next to it show total report count ("M teadet").

ADD-REPORT FLOW (logged-in only):
1. Tap "+ Lisa teade" → if not logged in, show login. If logged in, enter placement mode.
2. Two ways to set location: a "📍 Kasuta minu asukohta" button (browser geolocation) AND tap-on-map to drop a draggable pin. Store the chosen lat/lng at full precision.
3. Ask presence with two clear buttons: "🐌 Nägin teetigu" (present=true) and "✓ Vaatasin, ei näinud" (present=false). Make it explicit that "ei näinud" means the user actually looked.
4. If present=true, show a numeric field "Mitu tigu hävitasid?" (killed_count, default 0, min 0, optional). Hide this field when present=false.
5. Optional single-line note field ("Märkus (valikuline)").
6. Insert into observations (user_id defaults to auth.uid(), source defaults to 'user'). Success toast — if killed_count > 0: "Teade lisatud — hävitasid N tigu, tubli!"; else "Teade lisatud, aitäh!".

AFTER A SUCCESSFUL REPORT — two things:
- "Sinu aia teetigu-risk": compute client-side from already-loaded observations — count present=true reports within ~5 km of the new pin in the last 30 days (haversine helper). Show a colored risk badge: 0 = roheline "Madal", 1–3 = kollane "Keskmine", 4+ = punane "Kõrge", with the count ("N teadet 5 km raadiuses viimase 30 päeva jooksul").
- First report only: show a one-question feedback modal — "Mis teeks selle rakenduse sinu jaoks kasulikuks?" with a free-text box and "Saada" button → insert into feedback. Track "has given feedback" so it only appears once per user.

ANALYTICS (lightweight): console.log funnel events at each step so drop-off is visible while dogfooding: landed, clicked_login, authed, add_clicked, location_set, report_submitted.

OUT OF SCOPE for v1 (do NOT build): photo upload, species selection, comments, user profiles, admin/moderation dashboard, mobile native app, abundance counts on the form. Keep it minimal.
```
