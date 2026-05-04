# wbso-ai marketplace

Claude Code plugins van WBSO.ai. Op dit moment één plugin: `wbso` —
registreer WBSO-uren via een gesprek.

## Installatie

```bash
claude plugins marketplace add wbso-ai/skill
claude plugins install wbso@wbso-ai
```

Daarna in Claude Code: `/wbso` of zeg "ik ga uren boeken".

De eerste keer dat je `/wbso` runt vraagt de skill je portal-URL,
e-mailadres en API key, valideert die tegen de portal en slaat ze op in
`~/.config/wbso/config` (mode 600). Een API key maak je aan op
`https://portal.wbso.ai/companies/<bedrijf>/compliance/api_keys`.

## Updaten

```bash
claude plugins update wbso@wbso-ai
```

Daarna Claude Code herstarten om de nieuwe versie op te pikken.

## Wat zit erin

### `wbso`

Conversational uren-registratie:

- Pakt actieve WBSO-projecten van je bedrijf via de compliance API
- Stelt 2–3 gerichte vragen (project, technisch knelpunt, nieuwe
  oplossing of bestaande tech)
- Beoordeelt WBSO-waardigheid op basis van RVO-criteria — transparant,
  met redenering erbij
- Boekt direct via `POST /api/v1/compliance/time_entries`
- Pakt `git reflog` op als geheugensteun (vangt ook WIP-commits en
  abandoned experimenten) — blijft volledig lokaal: alleen project +
  uren + datum gaan naar de API
- Bij `missing_evidence` alert: vraagt of je in 1–2 zinnen wil
  vastleggen wat je gedaan hebt en koppelt dat als onderbouwing

## Privacy

- Reflog en commit messages blijven **lokaal** — alleen project, uren
  en datum gaan naar de API
- API key staat in `~/.config/wbso/config` met mode 600
- Geen git history of broncode naar de server

## Repo structuur

```
.claude-plugin/marketplace.json     # marketplace metadata
packages/
└── wbso/
    ├── .claude-plugin/plugin.json  # plugin metadata
    └── skills/
        └── wbso/SKILL.md           # de prompt
```

## Lokaal testen

```bash
git clone git@github.com:wbso-ai/skill.git
cd skill
claude plugins validate ./packages/wbso
claude --plugin-dir ./packages/wbso  # eenmalig test, geen install
```
