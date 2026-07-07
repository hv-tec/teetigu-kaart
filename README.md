# 🐌 Teetigu-kaart

Kogukonna hüvanguks tehtud kodanikuteaduse rakendus **hispaania teetigu** (*Arion vulgaris*) leviku
kaardistamiseks Eestis. Inimesed logivad sisse (Google / Facebook) ja märgivad kaardile teetigu
**olemasolu või puudumise** oma asukohas — sh mitu tigu hävitasid. Eesmärk on koguda kogukondlik
ülevaade teetigu levikust ja pakkuda igaühele "sinu aia teetigu-risk".

**Mittetulunduslik projekt, ilma ärilise eesmärgita. Kood on vabalt kasutatav (MIT).**

## Tehnoloogia
- **Frontend:** React + Vite + Tailwind (ehitatud [Lovable](https://lovable.dev)'iga)
- **Kaart:** react-leaflet + [Maa-ameti](https://geoportaal.maaamet.ee) ortofoto + katastripiirid + lähiaadress
- **Backend:** [Supabase](https://supabase.com) (Postgres + Auth + RLS)

## Andmed
Kaart on seemendatud **[Keskkonnaameti teokaardi](https://teokaart.keskkonnaamet.ee)** avalike
andmetega (~7700 punkti), millest on eemaldatud isikuandmed. Allikas: Keskkonnaamet.

## Ülesehitus
| Fail | Sisu |
|---|---|
| `LOVABLE-PROMPT.md` | Täielik ehitus-prompt Lovable'i jaoks (kleebi Lovable'i) |
| `seed/keskkonnaamet-teetigu.geojson` | Seemet-andmed (GeoJSON) |
| `seed/seed-observations.sql` | Valmis SQL seemet-import Supabase'i |
| `HANDOVER.md` | Setup-sammud, otsused, verifitseerimine |

## Kuidas ise üles seada
1. Loo tasuta Supabase-projekt.
2. Lülita sisse Google + Facebook auth (Supabase → Auth → Providers).
3. Ava Lovable, ühenda Supabase, kleebi `LOVABLE-PROMPT.md` sisu.
4. Jooksuta migratsioon (prompti sees), seejärel `seed/seed-observations.sql` Supabase SQL-editoris.
5. Publish.

## Litsents
[MIT](LICENSE) © 2026 Heikki Visnapuu
