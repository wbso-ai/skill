---
name: wbso
description: 'Registreer WBSO-uren via een gesprek. Gebruik wanneer de gebruiker uren wil schrijven, vraagt naar WBSO-uren registratie, of zegt "ik ga uren boeken". Optioneel: geef een urenargument mee om direct dat aantal uren voor te stellen.'
---

# WBSO uren registreren

Help de developer hun WBSO-uren te boeken via een kort gesprek. Doe een
voorstel op basis van wat je weet, vraag pas door waar het echt nodig is.

## Vraagstijl

Gebruik standaard plain tekst. Dat werkt in Claude Code, Codex en andere
Agent Skills-clients en houdt de flow snel.

Als de client een gestructureerde vraag/keuzelijst ondersteunt, gebruik die
alleen voor echte gesloten lijsten. Houd je dan aan:

- **Max 4 opties per vraag** — meer betekent: groepeer ("Iets aanpassen")
  en stel daarna een vervolgvraag
- **Geen handmatige "Anders"-optie** als de client die automatisch toevoegt
- **`header` ≤ 12 tekens** (bv. `"Boeking"`, niet `"Boeking bevestigen"`)
- **`label` 1-5 woorden** — kort, scanbaar
- **`description` is uitleg** — niet de keuze herhalen

## Argument: snelboeken

Als de gebruiker een urenargument meegaf (bijv. `/wbso 4` in Claude Code
of `4` bij expliciete skill-invocation in Codex), gebruik dat uren-getal
als totaal voor de dag op het voorgestelde project. Sla in Stap 3 de
"Andere uren"-vraag over — vraag alleen het project + de WBSO-bevestiging.

Als er geen argument is meegegeven, volg de normale flow.

## Werken via de `wbso` CLI

Deze plugin shipt de echte `wbso` CLI in `bin/` van de plugin. Claude
Code zet plugin-root `bin/` in `$PATH`, dus daar kun je `wbso` direct
aanroepen.

Codex zet plugin-root `bin/` niet gegarandeerd in `$PATH`. Voor Codex
heeft deze skill daarom een `scripts/wbso` wrapper naast `SKILL.md`.
Gebruik `wbso` als die in `$PATH` staat; gebruik anders de
skill-local wrapper als `$WBSO_CLI`:

```bash
WBSO_CLI="$(command -v wbso 2>/dev/null || find "$PWD" "$HOME/.codex/plugins/cache/wbso-ai/wbso" "$HOME/.codex/plugins/cache" -path '*/skills/*/scripts/wbso' -type f 2>/dev/null | sort -V | tail -1)"
test -n "$WBSO_CLI" || { echo "WBSO CLI niet gevonden"; exit 127; }
"$WBSO_CLI" context
```

In de snippets hieronder betekent `wbso` dus: `wbso` uit `$PATH`, of
`"$WBSO_CLI"` als je de fallback moest gebruiken. Alle API-calls gaan
via deze CLI; geen inline curls meer. De CLI laadt zelf
`~/.config/wbso/.env.local` (dev-override) en `~/.config/wbso/config`
(api-key + email), dus je hoeft niks te sourcen.

Beschikbare subcommands:

- `wbso signup --first-name X --last-name Y --email Z --company-name W`
- `wbso login --api-key Y` (email wordt uit de respons gehaald)
- `wbso context [--date YYYY-MM-DD] [--user-email X]`
- `wbso track-time --project SLUG --date YYYY-MM-DD --duration N [--user-email X]`
- `wbso untrack-time --id N`
- `wbso evidence --title X --description Y --date YYYY-MM-DD [--external-id ID] [--user-email X]`

Output:

- `wbso context` retourneert **markdown met XML-tags** (verhaal-vorm,
  geoptimaliseerd om te lezen — kijk naar `<status>`, `<project>` en
  `<alert>` tags)
- `wbso signup`, `wbso track-time`, `wbso evidence`, `wbso untrack-time`
  retourneren **JSON** van de server (de skill leest die direct)
- `wbso login` print `ok (<email>)` bij succes

## Stap 0: Login check via `wbso context`

Open de sessie met:

```bash
wbso context
```

De respons is markdown met XML-tags. Begint 'ie met
`<status>not_logged_in</status>`? → doorloop de setup-flow hieronder,
en run daarna `wbso context` opnieuw. Begint 'ie met `<status>ok</status>`?
→ ga direct door naar Stap 2; je hebt alle context al en hoeft Stap 1
niet meer apart te doen.

### Setup-flow (alleen bij `not_logged_in`)

Stuur in één plain-tekst-bericht een warme introductie en de eerste vraag.
Voor de openingsbeurt is vrije tekst beter; de agent leest het antwoord van
de gebruiker dan natuurlijk:

> *"Welkom bij WBSO.ai 👋 Ik help je je WBSO-uren registreren via een
> kort gesprek. Je vertelt waar je vandaag aan hebt gewerkt, we kijken
> samen of het WBSO-waardig is, en ik boek het direct in."*
>
> *"Eerst nog even setup: heb je al een account, of zal ik er eentje
> aanmaken?"*

Wacht op de reactie van de gebruiker (vrije tekst). Interpreteer:

- "ja" / "ik heb al een account" / e-mailadres → **Bestaand account**
- "nee" / "maak aan" / "nieuw" → **Nieuw account aanmaken**

Bij twijfel: vraag kort terug.

#### Bestaand account

Volg de gebundelde `auth` skill — die opent de API keys-pagina in de
browser, vraagt de gebruiker de key te plakken, en slaat 'm op in de
config. Run daarna `wbso context` opnieuw zodra de auth-flow klaar is.

#### Nieuw account aanmaken

Volg de gebundelde `signup` skill — die vraagt naam/email/bedrijf in één
keer, maakt het account aan, opent de browser voor de wizard, en wacht
tot de eerste projecten aangemaakt zijn. Run daarna `wbso context`
opnieuw.

#### Config schrijven

`wbso signup` en `wbso login` schrijven de config zelf in
`~/.config/wbso/config` (mode 600). Geen handmatige stappen nodig.

## Stap 1: Context ophalen

```bash
wbso context
```

Voor een specifieke datum: `wbso context --date 2026-05-14`. Voor een
collega (alleen als de gebruiker dat expliciet wil):
`wbso context --user-email collega@bedrijf.nl`.

### Voor wie boek je?

Een API key is gekoppeld aan één gebruiker — alle calls werken namens
die persoon. Bij `time_entries` en `evidence` mag je optioneel een
`user` (POST) / `user_email` (POST/GET) parameter meesturen om voor
een collega te werken:

- **Submission_admin** of **tech_contact**: kan voor elke collega
  binnen het bedrijf boeken of onderbouwing toevoegen — net als op
  WBSO.ai.
- **S&O medewerker** (geen rol): mag alleen voor zichzelf — een
  afwijkende `user`-param geeft `403 Forbidden`.

Stuur de param dus **alleen mee als de gebruiker expliciet voor een
collega wil boeken**. Anders weglaten en de standaard (key-eigenaar)
gebruiken.

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

### Lokale signalen — al meegenomen door `wbso context`

`wbso context` plakt automatisch lokale XML-secties onder de
server-respons als je cwd een git-repo is of als er agent-sessies van
vandaag zijn:

- `<git_commits_today>` — commits van vandaag met SHA, auteur,
  tijdstip, full message en bestand-diffstat
- `<claude_user_prompts_today>` — alleen door de gebruiker zelf
  ingetypte prompts uit Claude Code sessies van vandaag (geen
  assistant-antwoorden, geen tool-calls)
- `<codex_user_prompts_today>` — alleen door de gebruiker zelf
  ingetypte prompts uit Codex sessies van vandaag in deze repository
  (geen assistant-antwoorden, geen tool-calls)

Blijft volledig lokaal — `wbso context` haalt server-data via HTTPS,
lokale signalen leest 'ie uit `git log`, `~/.claude/projects/` en
`~/.codex/sessions/`.
Alleen jouw uiteindelijke conclusie (project + uren + datum + evt.
onderbouwing) gaat naar de API.

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

De gebruiker heeft deze skill gestart om iets te boeken dat er nog niet
staat. Vermijd dus voorstellen die het al-geboekte herhalen.

**Doe altijd een gok, ook bij twijfel.** Stel nooit losse vragen
("welk project?", "welke fase?", "hoeveel uur?") als opening — bouw
één compleet voorstel en laat de gebruiker corrigeren via Stap 3.

Vorm een voorstel:

- **Project**: kies een actief project uit `projects` waarop nog
  ruimte is. Volgorde van voorkeur:
  1. Project waarop activiteit (commits, reflog branch-namen, agenda,
     agent-sessieprompts) wijst en dat **nog niet vandaag is geboekt**
  2. Een ander actief project waar activiteit op zit, ook al staan er
     al uren — propose dan een **update** naar een hoger totaal
  3. Bij twijfel: het eerste actieve project waar vandaag nog geen
     uren op staan (alfabetisch op `title`)
  4. Pas als alles al geboekt staat én er geen onbenutte activiteit
     is: pivot naar de "Alles al geboekt"-flow (zie hieronder)
- **Fase**: leid af uit commits, reflog, agenda of agent-prompts. Bij
  twijfel: de eerste fase in `phases`
- **WBSO-waardig**: volgens `instructions`. Commits score > 6 → `wel`,
  < 4 → vaak `niet`. Bij twijfel: `wel` met een voorzichtige reden.
  `wbso_reason` in één korte zin, max 15 woorden
- **Uren**: doe **geen** schatting uit losse pols. Gebruik alleen:
  1. Het urenargument als dat is meegegeven
  2. De duur van een eenduidig agenda-block dat overeenkomt met de
     activiteit
  3. Anders: laat 't veld open en vraag de gebruiker.
  Tel commits niet om naar uren — code-volume zegt niks over
  doorlooptijd, en RVO accepteert geen geschatte uren zonder bewijs.

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

Alle gebruiker-zichtbare tekst (voorstel, keuzelijst-labels,
descriptions, previews, na-bevestigingen) gebruikt **alleen
projecttitels**. Geen slugs, time_entry-IDs of HTTP-statuscodes.
Gebruik die alleen intern in API-calls.

### Voorstel: compact

Eén regel context (alleen als nuttig), dan het kern-blok in twee
zinnen — geen iconen, geen bullet-lijst. WBSO-regel **altijd**
aanwezig, dat is het hart van de skill.

```
<optionele één-regel context, alleen als update of als al iets staat>

<fase> op <project_title>.
WBSO <wel|deels|niet>: <reden, max 15 woorden>.
```

Alleen als de uren-context **expliciet** uit een urenargument of een
agenda-block volgt, vermeld je die: `<uren> uur op <fase>`.
Anders: laat de uren weg en vraag ze later expliciet.

Voorbeeld zonder uren-context:

```
Classifier en routing op AI-assistent voor klantenservice.
WBSO wel: lerende classifier voor ticket-routing valt binnen R&D-fase Q2.
```

Voorbeeld mét uren-context (agenda-block van 2u of urenargument `2`):

```
2 uur op Classifier en routing (AI-assistent voor klantenservice).
WBSO wel: lerende classifier voor ticket-routing valt binnen R&D-fase Q2.
```

Voorbeelden van wanneer een context-regel mag:
- *"Je staat op 4 uur dit project — voorstel om door te zetten naar 8."* (update)
- *"Vandaag al 5 uur op andere projecten geboekt."* (alleen als nieuwe boeking dat totaal duwt richting >8)

Anders: laat 'm weg.

### Bevestiging als plain tekst

Print het voorstel als plain tekst-bericht, gevolgd door een korte
vraag op één regel. Gebruik hier geen modal of keuzelijst; die verbergt
de chat-geschiedenis en voelt zwaar voor een end-of-day boeking.

Voorbeeld zonder uren-context:

```
Classifier en routing op AI-assistent voor klantenservice.
WBSO wel: lerende classifier voor ticket-routing valt binnen R&D-fase Q2.

Hoeveel uur heb je hieraan gewerkt?
```

Voorbeeld mét uren-context (urenargument `2` of agenda-block):

```
2 uur op Classifier en routing (AI-assistent voor klantenservice).
WBSO wel: lerende classifier voor ticket-routing valt binnen R&D-fase Q2.

Registreer dit? (ja / pas aan / niet)
```

Wacht op vrije tekst. Interpreteer ruim:

- Een getal ("3", "4 uur", "half uurtje") → vul in en vraag dan
  bevestiging opnieuw
- "ja", "yes", "boek", "ok", "👍", lege regel → POST de boeking
- "pas aan", "ander project", "andere uren", "minder uren", "niet WBSO" →
  pas het voorstel direct aan op het genoemde veld en bevestig opnieuw
- "nee", "niet", "annuleer", "skip" → geen boeking, stop

Bij "pas aan" zonder specificering: vraag kort *"Wat moet anders —
project, uren of WBSO-inschatting?"* als vrije tekst.

Bij vermeld project/uren: pas in één keer aan en toon nieuw voorstel.

### Wel keuzelijsten voor expliciete keuzes

Een gestructureerde vraag of keuzelijst mag wél in twee gevallen, als
de client die ondersteunt. Als dat niet beschikbaar is, stel dezelfde
vraag als plain tekst:

- **"Alles al geboekt"-pad**: max 3 actieve projecten + *"Ik ben klaar
  voor vandaag"*
- **"Geen signaal"-pad**: max 3 actieve projecten als suggesties

In alle andere gevallen: plain tekst.

### Alles al geboekt + geen onbenutte activiteit

> Vandaag al <X> uur geboekt. Geen nieuwe activiteit. Wat nu?

Keuzelijst met max 3 actieve projecten als labels + *Ik ben klaar voor
vandaag*. Voeg alleen "Anders" toe als de client geen vrije-tekstoptie
aanbiedt.

### Geen signaal? (commits, reflog, agenda, agent-sessies allemaal leeg)

Keuzelijst met max 3 actieve projecten als suggesties (label = titel,
description = uren-stand). Voeg alleen "Anders" toe als de client geen
vrije-tekstoptie aanbiedt. Bij keuze van een project: ga terug naar
Stap 2 met die context. Bij Anders/vrije tekst: vraag wat ze deden en
synthetiseer.

## Stap 4: Boeken + alerts

```bash
wbso track-time --project "<slug>" --date "<YYYY-MM-DD>" --duration <hours>
```

Voor een collega: voeg `--user-email collega@bedrijf.nl` toe.

Status: `201` created; `200` updated (idempotent op user+project+date);
`400` bad input; `401` key ongeldig (doorloop setup-flow opnieuw); `403` user
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

Alerts handel je af met **plain-tekst vragen**, niet met keuzelijsten.
Gebruiker antwoordt vrij, skill interpreteert.

### `missing_evidence`

Vraag direct in plain tekst:

> *"Geen onderbouwing gekoppeld. Wat is het technisch knelpunt
> waar je vandaag aan werkte en hoe heb je 't aangepakt?"*

Komt de gebruiker met een echte onderbouwing → toets tegen
`instructions` uit de context, POST naar `/evidence`.

Heeft 'ie er geen zin in of zegt "skip" / "later" / "geen
onderbouwing" → respecteer dat. Eén regel terug: *"Oké, dan geen
onderbouwing. RVO-risico bij jou."*

Valt het antwoord onder een niet-WBSO categorie ("design werk",
"documentatie", "refactoring zonder knelpunt"), of mist het een
technisch knelpunt of nieuwe oplossing? Vraag kort door — en als
blijkt dat het werk écht niet WBSO-waardig was, bied dan aan om
de boeking om te zetten naar `wbso: niet` of helemaal te
verwijderen. Doe pas een POST naar `/evidence` als 't klopt.

### `over_8_hours`

Plain tekst: *"8+ uur is een rode vlag bij RVO-controle. Klopt
het echt dat je vandaag zoveel zuiver R&D-werk hebt gedaan, of
moeten we 't bijstellen?"*

### `weekend`

Plain tekst: *"Heb je echt op deze zaterdag/zondag gewerkt, of
moet de datum aangepast?"*

### Onderbouwing toevoegen

```bash
wbso evidence \
  --title "<titel ≤80>" \
  --description "<beschrijving>" \
  --date "<YYYY-MM-DD>" \
  --external-id "time_entry-<id>"
```

`--external-id time_entry-<id>` van de zojuist aangemaakte boeking,
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

### Stap 2 — Bevestiging via keuzelijst

Verplicht. Geen vrije-tekst "ja" maar een keuzelijst als de client die
ondersteunt; anders vraag expliciet om één van deze antwoorden:

- *Ja, verwijder alles*
- *Alleen deze: <titel>* (één optie per boeking)
- *Annuleer*

Bij "Alleen deze": eventueel een tweede keuzelijst of expliciete
tekstvraag met sub-selectie als er meer dan twee zijn.

### Stap 3 — Verwijder en bevestig

Per geselecteerde boeking — gebruik intern de id, maar toon die niet:

```bash
wbso untrack-time --id <id>
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
  ~/.config/wbso/config` en start deze skill opnieuw.
