# wbso-ai marketplace

Claude Code plugins van WBSO.ai. Op dit moment één plugin: `wbso` —
registreer WBSO-uren via een gesprek.

## Installatie

```bash
claude plugins marketplace add wbso-ai/skill
claude plugins install wbso@wbso-ai
```

Daarna in Claude Code: `/wbso` of zeg "ik ga uren boeken".

De eerste keer dat je `/wbso` runt vraagt de skill of je al een account
hebt. Twee paden:

- **Nieuw account**: naam, e-mail en bedrijfsnaam invullen → de skill
  maakt een account aan, opent direct het stappenplan in je browser
- **Bestaand account**: e-mail + API key plakken (aan te maken op
  `https://portal.wbso.ai/companies/<bedrijf>/compliance/api_keys`)

Beide paden slaan je gegevens op in `~/.config/wbso/config` (mode 600).

## In- en uitloggen

Vijf skill-commando's:

- `/wbso` — registreert uren (vraagt zelf om setup als je nog niet
  ingelogd bent)
- `/wbso:signup` — maakt een nieuw WBSO.ai-account aan en opent de
  onboarding-wizard in je browser
- `/wbso:auth` — opent de API keys-pagina in je browser, plak je
  nieuwe key terug in de skill om in te loggen. Ook handig om van
  account te wisselen
- `/wbso:whoami` — laat zien onder welk account en bedrijf je nu
  ingelogd bent
- `/wbso:logout` — verwijdert `~/.config/wbso/config`

Handmatig uitloggen kan ook met:

```bash
rm ~/.config/wbso/config
```

## Updaten

```bash
claude plugins update wbso@wbso-ai
```

Daarna Claude Code herstarten om de nieuwe versie op te pikken.

## Wat zit erin

### `wbso` skill

Conversational uren-registratie:

- Pakt actieve WBSO-projecten van je bedrijf via de compliance API
- Stelt 2–3 gerichte vragen (project, technisch knelpunt, nieuwe
  oplossing of bestaande tech)
- Beoordeelt WBSO-waardigheid op basis van RVO-criteria — transparant,
  met redenering erbij
- Pakt `git reflog` én Claude Code sessies van vandaag op als
  geheugensteun (vangt ook WIP-commits, abandoned experimenten en
  conversaties zonder commits) — blijft volledig lokaal: alleen
  project + uren + datum gaan naar de API
- Bij `missing_evidence` alert: vraagt of je in 1–2 zinnen wil
  vastleggen wat je gedaan hebt en koppelt dat als onderbouwing

### `wbso` CLI

De skill praat met de WBSO.ai compliance API via een meegeleverde CLI
in `bin/wbso`. Die staat automatisch in `$PATH` zodra de plugin actief
is — je kunt 'm ook handmatig vanuit een shell aanroepen:

```
wbso signup --first-name X --last-name Y --email Z --company-name W
wbso login --api-key Y
wbso context [--date YYYY-MM-DD] [--user-email X]
wbso track-time --project SLUG --date YYYY-MM-DD --duration N
wbso untrack-time --id N
wbso evidence --title X --description Y --date YYYY-MM-DD [--external-id ID]
```

De CLI is een dunne bash-wrapper rond de HTTP-endpoints. Hij laadt
zelf je config + eventuele dev-override en geeft de server-respons
ongewijzigd door — `wbso context` retourneert markdown met XML-tags
(geoptimaliseerd voor LLM-input), de andere commands retourneren JSON.

## Privacy

- Reflog, commit messages en Claude-sessieprompts blijven **lokaal** —
  alleen project, uren en datum gaan naar de API
- API key staat in `~/.config/wbso/config` met mode 600
- Geen git history, broncode of sessie-inhoud naar de server

## Repo structuur

```
.claude-plugin/marketplace.json     # marketplace metadata
packages/
└── wbso/
    ├── .claude-plugin/plugin.json  # plugin metadata
    ├── bin/wbso                    # CLI (bash, in $PATH bij activatie)
    └── skills/
        ├── wbso/SKILL.md           # uren registreren
        ├── signup/SKILL.md         # /wbso:signup — account aanmaken
        ├── auth/SKILL.md           # /wbso:auth — inloggen
        ├── whoami/SKILL.md         # /wbso:whoami — wie ben ik
        └── logout/SKILL.md         # /wbso:logout — uitloggen
```

## Lokaal testen

```bash
git clone git@github.com:wbso-ai/skill.git
cd skill
claude plugins validate ./packages/wbso
claude --plugin-dir ./packages/wbso  # eenmalig test, geen install
```

### Tegen een lokale server testen

Standaard wijst de skill naar `https://portal.wbso.ai`. Voor lokale
ontwikkeling kun je dit overrulen met een `.env.local` in je config-
map:

```bash
mkdir -p ~/.config/wbso
cp .env.local.example ~/.config/wbso/.env.local
# pas WBSO_API_BASE_URL aan naar bv. http://localhost:3000
```

De `wbso` CLI sourcet die file automatisch bij elke aanroep. De URL
vloeit door naar de config die na signup geschreven wordt, dus je
hoeft 'm niet steeds opnieuw te zetten. Andere gebruikers die de
plugin via de marketplace installeren krijgen de file niet, dus blijft
dit een dev-only override.

## Releasen

Een nieuwe versie publiceren doe je met:

```bash
bin/release
```

Dat script:

1. Berekent de nieuwe versie op basis van het aantal commits
   (`1.<commit-count>`)
2. Bumpt `version` in `packages/wbso/.claude-plugin/plugin.json`
3. Maakt een release-commit + git-tag (`v1.N`)
4. Pusht alles naar `origin/main`

Claude Code ziet die versie-bump als nieuwe release. Gebruikers
pakken hem op met:

```bash
claude plugins update wbso@wbso-ai
```

### Waarom een expliciete `version`?

Als `version` in `plugin.json` staat krijgen gebruikers pas een
update zodra jij dat veld bumpt. Zonder `version` zou elke commit op
deze repo als nieuwe versie tellen (gebruikt de commit-SHA), wat
ruis oplevert voor README-tweaks of CI-fixes.

## Wisselen tussen GitHub-versie en lokale versie

Je kunt de plugin op twee manieren installeren:

- **GitHub-versie** (`wbso@wbso-ai`) — stabiel, update via
  `claude plugins update`
- **Lokale versie** (`wbso@wbso-ai-local`) — pakt elke wijziging in
  `~/Documents/skill` direct op, handig voor development

Met `claude plugins list` zie je welke geïnstalleerd staan en in welke
scope (`user` = overal, `local` = alleen in huidige map). Je herkent
ze aan de suffix achter `@`.

### Overstappen naar de GitHub-versie

```bash
claude plugins uninstall wbso@wbso-ai-local --scope local
claude plugins install wbso@wbso-ai
```

### Overstappen naar de lokale versie

Eerst de marketplace registreren (eenmalig):

```bash
claude plugins marketplace add ~/Documents/skill
```

Daarna installeren:

```bash
claude plugins uninstall wbso@wbso-ai
claude plugins install wbso@wbso-ai-local
```

Updaten = `git pull` in `~/Documents/skill`. Claude Code herstarten
om de wijzigingen op te pikken.

### Allebei tegelijk geïnstalleerd?

Verwarrend, want dan kan onduidelijk zijn welke versie actief is.
Disable de versie die je niet wilt:

```bash
claude plugins disable wbso@wbso-ai-local
# of
claude plugins disable wbso@wbso-ai
```
