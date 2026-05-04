---
name: wbso
user-invocable: true
description: Registreer WBSO-uren via een gesprek. Gebruik wanneer de gebruiker uren wil schrijven, vraagt naar WBSO-uren registratie, of zegt "ik ga uren boeken". Optioneel: `/wbso <uren>` om direct dat aantal uren voor te stellen.
---

# WBSO uren registreren

Help de developer hun WBSO-uren te boeken via een kort gesprek. Doe een
voorstel op basis van wat je weet, vraag pas door waar het echt nodig is.

## AskUserQuestion regels (lezen voordat je 'm aanroept)

De tool weigert calls die buiten de schema-limieten vallen. Houd je aan:

- **Max 4 opties per vraag** — meer betekent: groepeer ("Iets aanpassen")
  en stel daarna een vervolgvraag
- **Geen handmatige "Anders"-optie** — Claude Code voegt die automatisch toe
- **`header` ≤ 12 tekens** (bv. `"Boeking"`, niet `"Boeking bevestigen"`)
- **`label` 1-5 woorden** — kort, scanbaar
- **`description` is uitleg** — niet de keuze herhalen

## Argument: snelboeken

Als de gebruiker `/wbso <uren>` typte (bijv. `/wbso 4`), gebruik dat
uren-getal als totaal voor de dag op het voorgestelde project. Sla in
Stap 3 de "Andere uren"-vraag over — vraag alleen het project + de
WBSO-bevestiging.

Als er geen argument is meegegeven, volg de normale flow.

## Stap 0: Config check

```bash
state="missing"
[ -f "$HOME/.config/wbso/config" ] && state="present"
echo "wbso_config=$state"
```

| state | actie |
|-------|-------|
| `present` | Source de config en ga door naar Stap 1 |
| `missing` | Walk door de setup-flow hieronder, dan Stap 1 |

### Setup-flow (alleen bij `missing`)

Eén voor één vragen, niet alles tegelijk:

1. *"Op welk e-mailadres ben je in het portal bekend?"*
2. *"Plak je API key. Maak er een aan op
   <https://portal.wbso.ai/companies/current/compliance/api_keys> — die
   link kiest automatisch je bedrijf, of laat een keuzelijst zien als
   je er meerdere hebt."*

Valideer de key:

```bash
curl -sS -o /tmp/wbso-validate.$$ -w '%{http_code}' \
  -H "Authorization: Bearer <KEY>" \
  "${WBSO_API_BASE_URL:-https://portal.wbso.ai}/api/v1/compliance/context?user=<EMAIL>&date=$(date +%F)"
```

`200` = OK; `401`/`404` = vraag opnieuw; anders = toon respons.

Schrijf de config (mode 600):

```bash
mkdir -p "$HOME/.config/wbso"
umask 077
cat > "$HOME/.config/wbso/config" <<EOF
WBSO_USER="<EMAIL>"
WBSO_API_KEY="<KEY>"
WBSO_API_BASE_URL="https://portal.wbso.ai"
EOF
chmod 600 "$HOME/.config/wbso/config"
```

## Stap 1: Context ophalen

Source de config en haal de server-side context op:

```bash
source ~/.config/wbso/config
WBSO_API_BASE_URL="${WBSO_API_BASE_URL:-https://portal.wbso.ai}"

curl -sS -G -H "Authorization: Bearer $WBSO_API_KEY" \
  --data-urlencode "user=$WBSO_USER" \
  --data-urlencode "date=$(date +%F)" \
  "$WBSO_API_BASE_URL/api/v1/compliance/context"
```

Velden in de respons:

| Veld | Inhoud |
|------|--------|
| `instructions` | WBSO-criteria. **Leidend voor je inschatting.** |
| `commit_score_prompt` | Hoe commits gescoord worden. Toon op verzoek. |
| `alert_rules` | Alle regels die alerts kunnen triggeren. Gebruik om **vooraf** te waarschuwen, niet alleen achteraf na een POST. |
| `projects` | Actieve projecten op deze datum. |
| `time_entries` | Wat er al geboekt staat vandaag. |
| `commits` | Commits van deze user vandaag met score (0-10). |
| `events` | Agenda-items vandaag. |
| `evidence` | Bestaande onderbouwing. |

Als de cwd een git-repo is, haal lokale reflog erbij (blijft lokaal —
niet naar de server):

```bash
git log --walk-reflogs --since=midnight --all --pretty=format:'%h %gd %gs' 2>/dev/null | head -50
git log --all --since=midnight --pretty=format:'%h %s (%an)' 2>/dev/null | head -50
git diff --stat 2>/dev/null
```

### Vooraf checken: alert-regels

Loop de `alert_rules` uit de context af en check ze tegen het voorstel
dat je gaat doen. Als een regel zou triggeren bij dit voorstel, neem
de waarschuwing op in Stap 3 voordat je boekt:

- `weekend` → klopt het echt dat je vandaag werkt?
- `over_8_hours` → 8+ uur is een rode vlag bij RVO-controle (zie
  `instructions`)
- `missing_evidence` / `low_commit_scores` → er is geen of te zwakke
  onderbouwing — bied bij het bevestigen al aan om er een toe te voegen

Zo voorkomt de gebruiker een verrassing achteraf.

## Stap 2: Synthese — kies wat nog NIET geboekt is

De gebruiker heeft `/wbso` ingetypt om iets te boeken dat er nog niet
staat. Vermijd dus voorstellen die het al-geboekte herhalen.

Vorm een voorstel:

- **Project**: kies een actief project uit `projects` waarop nog
  ruimte is. Volgorde van voorkeur:
  1. Project waarop activiteit (commits, reflog branch-namen, agenda)
     wijst en dat **nog niet vandaag is geboekt**
  2. Een ander actief project waar activiteit op zit, ook al staan er
     al uren — propose dan een **update** naar een hoger totaal als
     er nieuwe commits zijn sinds laatste boeking
  3. Pas als alles al geboekt staat én er geen onbenutte activiteit
     is: zeg dat eerlijk en pivot naar opties (zie hieronder)
- **Fase**: leid af uit commits, reflog branch-namen of agenda-titels,
  matchend met `phases` van dat project
- **WBSO-waardig**: volgens `instructions`. Commits met score > 6 zijn
  sterke indicator wel-WBSO; onder 4 zelden. `wbso_reason` in één
  korte zin, max 15 woorden
- **Uren**: als de gebruiker `/wbso <uren>` typte, gebruik dat als
  totaal voor de dag op het gekozen project. Anders: duur
  agenda-block, inschatting uit aantal commits + reflog-activiteit,
  of 8 uur bij volle werkdag

## Stap 3: Toon voorstel en vraag bevestiging

### Tone

Doelgroep is de developer zelf. Technische taal is prima. Wees
**zo kort mogelijk**: end-of-day wil de gebruiker geen audit lezen,
alleen een 2-regel proposal en een Enter-druk.

**De gebruiker boekt, jij stelt voor.** Schrijf niet "ik boek dit
niet" of "ik laat dit weg" alsof je zelf de actie uitvoert. Schrijf
in de derde persoon over de uren: *"Deze commits tellen niet als
WBSO en zitten niet in het voorstel"*, niet *"die boek ik niet"*.

### Geen slug, geen ID, geen HTTP-codes in UI

Alle gebruiker-zichtbare tekst (voorstel, AskUserQuestion-labels,
descriptions, previews, na-bevestigingen) gebruikt **alleen
projecttitels**. Geen slugs, time_entry-IDs of HTTP-statuscodes.
Gebruik die alleen intern in API-calls.

### Voorstel: compact

Eén regel context (alleen als nuttig), dan het kern-blok. WBSO-regel
**altijd** aanwezig, dat is het hart van de skill.

```
<optionele één-regel context, alleen als update of als al iets staat>

📁 **<project_title>**
🔖 <fase>
⏱️ <uren> uur
<wbso_emoji> WBSO <wel|deels|niet> — <reden, max 15 woorden>
```

Voorbeelden van wanneer een context-regel mag:
- *"Je staat op 4 uur dit project — voorstel om door te zetten naar 8."* (update)
- *"Vandaag al 5 uur op andere projecten geboekt."* (alleen als nieuwe boeking dat totaal duwt richting >8)

Anders: laat 'm weg.

WBSO-emoji: `wel` → ✅ · `deels` → 🟡 · `niet` → ❌

### Skip wat geen vraag is

Stel **geen** AskUserQuestion als er maar één plausibel antwoord is:

| Veld | Skip wanneer |
|------|--------------|
| Project | Slechts één actief project, óf commits/agenda eenduidig naar één project wijzen |
| Fase | Slechts één fase past bij de commits/agenda |
| Uren | `/wbso <uren>` argument meegegeven, óf agenda-block geeft eenduidige duur |
| WBSO-inschatting | Commits hebben score > 7 én passen in `instructions` (default `wel`), óf score < 3 (default `niet`) |

In die gevallen: zet de waarde gewoon in het voorstel-blok, sla door
naar de bevestigings-vraag.

### Bevestiging via preview

Eén `AskUserQuestion`. Gebruik het `preview` veld op de "Boek"-optie
om de volledige proposal in de side-panel te tonen, zodat de chat
zelf kort blijft. `header`: `Bevestigen`. Question: *"Boek dit?"*.

Drie opties (Claude Code voegt vanzelf een "Anders" toe voor vrije
tekst):

| Label | Description | Preview |
|-------|-------------|---------|
| Boek | Voorstel klopt | Volledig proposal-blok (zelfde tekst als hierboven) |
| Pas aan | Project, fase, uren of inschatting wijzigen | "Selecteer wat je wil aanpassen" |
| Niet boeken | Annuleer | "Geen boeking gemaakt" |

Eerste optie is default — Enter = boeken. Geen "(Recommended)"
suffix nodig in de label.

Bij **Pas aan**: één tweede `AskUserQuestion`, max 4 opties uit
*Ander project* / *Andere fase* / *Andere uren* / *Andere
WBSO-inschatting* (skip "Andere uren" als argument meegegeven).

Bij **Ander project**: top 3 actieve projecten als labels (Claude
Code voegt "Anders" toe voor de rest). Bij **Andere uren**:
*1 uur / 2 uur / 4 uur / 8 uur* (vervang er één door een
context-getal als agenda dat hint).

Beschrijvingen: alleen titels.

✅ *"Update naar 8 uur op Automatische git commit WBSO scoring agent"*
❌ *"Update naar 8 uur op kgwszv, fase Local changes & tunnel"*

### Alles al geboekt + geen onbenutte activiteit

> Vandaag al <X> uur geboekt. Geen nieuwe activiteit. Wat nu?

`AskUserQuestion` met max 3 actieve projecten als labels +
*Ik ben klaar voor vandaag*. Claude Code voegt zelf "Anders" toe.

### Geen signaal? (commits, reflog, agenda allemaal leeg)

`AskUserQuestion` met max 3 actieve projecten als suggesties (label =
titel, description = uren-stand). Claude Code voegt "Anders" toe voor
vrije tekst. Bij keuze van een project: ga terug naar Stap 2 met die
context. Bij Anders: vraag wat ze deden en synthetiseer.

## Stap 4: Boeken + alerts

```bash
source ~/.config/wbso/config
WBSO_API_BASE_URL="${WBSO_API_BASE_URL:-https://portal.wbso.ai}"

curl -sS -X POST "$WBSO_API_BASE_URL/api/v1/compliance/time_entries" \
  -H "Authorization: Bearer $WBSO_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"time_entry\":{\"user\":\"$WBSO_USER\",\"project\":\"<slug>\",\"date\":\"<YYYY-MM-DD>\",\"duration\":<hours>}}"
```

Status: `201` created; `200` updated (idempotent op user+project+date);
`400` bad input; `401` key ongeldig (run setup-flow opnieuw); `403` user
mag geen uren boeken op dit bedrijf; `404` user/project niet gevonden;
`422` jaar afgesloten of validatiefout (geef letterlijk terug); `429`
rate limit (60/min). HTTP-codes nooit naar gebruiker — vertaal naar
mensentaal.

### Bevestiging (kort)

Standaard: één regel + alerts indien aanwezig.

```
✓ Geboekt — <uren> uur op <project_title>.
```

Géén "Vandaag staat nu...", géén dagtotaal, géén dump van de respons.
Tenzij er alerts zijn (zie hieronder).

Lees `alerts` uit de respons en geef die kort terug. **Spreek over
"onderbouwing"**, niet "evidence" — dat is interne API-naamgeving.

**Iedere alert MOET via `AskUserQuestion` afgehandeld worden — nooit
in vrije-tekst-vragen.** Geen open vragen waarbij de gebruiker zelf
"geen onderbouwing nodig" of "ja klopt" moet typen. Altijd een
keuzelijst.

### `missing_evidence`

Toon kort: *"Geen onderbouwing gekoppeld aan deze boeking."* Stel
direct een `AskUserQuestion` met deze opties (in deze volgorde):

| Label | Beschrijving | Wanneer tonen |
|-------|--------------|---------------|
| Korte beschrijving toevoegen | Ik typ in 1-2 zinnen wat WBSO-waardig was | altijd |
| Komt goed via mijn commits | Lokale commits worden vanzelf onderbouwing zodra ze gepushed/ingested zijn | alleen als er lokale commits zijn die nog niet in `commits` zitten |
| Geen onderbouwing nodig | Accepteer en sluit af, RVO-risico bij mij | altijd |

Bij "Korte beschrijving toevoegen": dán pas vraag je in vrije tekst
*"In 1-2 zinnen — welk knelpunt, welke oplossing?"* en POST daarna
naar `/evidence`.

### `over_8_hours`

`AskUserQuestion`: *Ja, klopt* / *Nee, pas uren aan*.

### `weekend`

`AskUserQuestion`: *Ja, ik heb echt op die dag gewerkt* / *Nee, pas
datum aan*.

### Onderbouwing toevoegen

```bash
curl -sS -X POST "$WBSO_API_BASE_URL/api/v1/compliance/evidence" \
  -H "Authorization: Bearer $WBSO_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"evidence\":{\"user_email\":\"$WBSO_USER\",\"title\":\"<titel ≤80>\",\"description\":\"<beschrijving>\",\"starts_at\":\"<DATE>T09:00:00Z\",\"ends_at\":\"<DATE>T17:00:00Z\",\"all_day\":true,\"source\":\"wbso-skill\",\"external_id\":\"time_entry-<id>\"}}"
```

`external_id` = `time_entry-<id>` van de zojuist aangemaakte boeking,
zodat een tweede aanroep idempotent is.

## Verwijderen

Als de gebruiker uren wil verwijderen of een boeking wil annuleren:
**nooit direct deleten zonder bevestiging.** Verwijderen is
onomkeerbaar — geboekte uren met onderbouwing kunnen weg zijn voor
RVO-controle.

### Stap 1 — Toon eerst wat er weg gaat

Haal de relevante `time_entries` op (uit Stap 1 context, of GET op
specifieke datum). Toon ze als lijst met **alleen titel en uren**.
Geen IDs, slugs of andere interne details.

```
Wil je deze boekingen verwijderen?

- 4 uur op *Automatische git commit WBSO scoring agent*
- 1 uur op *Intake naar aanvraag conversie*

Totaal: 5 uur.
```

### Stap 2 — Bevestiging via AskUserQuestion

Verplicht. Geen vrije-tekst "ja" maar altijd een keuzelijst:

- *Ja, verwijder alles*
- *Alleen deze: <titel>* (één optie per boeking)
- *Annuleer*

Bij "Alleen deze": eventueel een tweede `AskUserQuestion` met
sub-selectie als er meer dan twee zijn.

### Stap 3 — Verwijder en bevestig

Per geselecteerde boeking — gebruik intern de id, maar toon die niet:

```bash
curl -sS -X DELETE "$WBSO_API_BASE_URL/api/v1/compliance/time_entries/<id>" \
  -H "Authorization: Bearer $WBSO_API_KEY"
```

Bevestig kort wat er is verwijderd, met titel en uren. Geen IDs en
geen HTTP-status codes naar de gebruiker:

```
Verwijderd:
- 4 uur op *Automatische git commit WBSO scoring agent*
- 1 uur op *Intake naar aanvraag conversie*

Vandaag staat nu nog 0 uur geboekt.
```

Bij API-fout (404/422 e.d.): vertaal naar mensentaal ("die boeking
bestaat niet meer", "het jaar is afgesloten — verwijderen kan niet").
Geen HTTP-codes oplezen.

## Notes

- API is idempotent op (user, project, date): tweede call op dezelfde
  dag overschrijft, geen duplicaat
- Reflog blijft **lokaal**; alleen uren + project + datum gaan naar de
  API. Geen git history of commit messages naar de server sturen.
- Config zit in `~/.config/wbso/config` (mode 600). Resetten: `rm
  ~/.config/wbso/config` en run `/wbso` opnieuw.
