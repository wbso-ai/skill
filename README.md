<p align="center">
  <a href="https://wbso.ai"><img src="https://wbso.ai/logo.svg" alt="WBSO.ai" width="200"></a>
</p>

<h1 align="center">Boek je WBSO-uren rechtstreeks vanuit Claude Code</h1>

<p align="center">
  Geen context-switch naar een urenformulier. Geen einde-van-de-dag-paniek. Geen gokken of je werk WBSO-waardig was.
</p>

<p align="center">
  <strong>Eén plugin in Claude Code. Eén gesprek. Klaar.</strong>
</p>

---

Developers houden niet van administratie. Maar voor de WBSO moet
je wél bijhouden waar je elke dag aan werkt, beoordelen of het
binnen je projectaanvraag valt, en het inschrijven in een tool.
Te veel context-switch, dus blijft het liggen — of je betaalt
een bemiddelaar 20-30% van je voordeel om 't voor je te doen.

Deze plugin lost dat op. De skill draait in jouw werk-omgeving,
leest waar je vandaag aan bezig was (commits + Claude-sessies,
beide volledig lokaal), beoordeelt het tegen de RVO-criteria, en
boekt het direct in op WBSO.ai.

## Voorbeeld van een gesprek

```
> /wbso

Welkom bij WBSO.ai 👋 Ik help je je WBSO-uren registreren via een
kort gesprek. Je vertelt waar je vandaag aan hebt gewerkt, we kijken
samen of het WBSO-waardig is, en ik boek het direct in.

[ skill leest jouw commits + Claude-sessies van vandaag,
  haalt jouw actieve WBSO-projecten op ]

4 uur op Classifier en routing (AI-assistent voor klantenservice).
WBSO wel: lerende classifier voor ticket-routing valt binnen
R&D-fase Q2.

Registreer dit? (ja / pas aan / niet)

> ja

✓ Geboekt — 4 uur op AI-assistent voor klantenservice.
```

Geen modal-dialogen, geen drop-downs, geen verplichte velden. Gewoon
typen wat je hebt gedaan.

## Waarom een Claude Code-skill?

Agents zijn first-class burgers bij WBSO.ai. Onze compliance-API
spreekt markdown met XML-tags zodat Claude 't direct kan lezen, de
skill shipt z'n eigen CLI in `$PATH`, en alle vragen zijn vrije
tekst — geen wrapper-laag, geen UI-omleiding.

- 💬 **Conversational** — typ wat je deed, geen invulvelden
- 🧠 **Met geheugen** — leest je `git log` van vandaag én je Claude-prompts, zodat je niet hoeft terug te denken wat je deed
- 📚 **Transparant** — beoordeelt elke activiteit tegen de RVO-criteria en laat zien waarom 't wel of niet WBSO is
- ⚡ **Snel** — gemiddeld één Enter-druk om een dag te boeken
- 🔒 **Lokaal-first** — code, commit-messages en sessie-inhoud blijven op je laptop; alleen project + uren + datum gaan naar de API

## Installatie

```bash
claude plugins marketplace add wbso-ai/skill
claude plugins install wbso@wbso-ai
```

Daarna in Claude Code: `/wbso`.

De eerste keer vraagt de skill of je al een WBSO.ai-account hebt.
**Heb je 'm nog niet?** Je maakt 'm aan via de skill zelf — naam,
e-mail, bedrijf. Geen wurgcontracten, geen jaarbinding, gewoon een
account in 60 seconden.

## Slash-commando's

| Commando | Wat het doet |
|---|---|
| `/wbso` | Registreer uren via een gesprek |
| `/wbso:signup` | Maak een nieuw WBSO.ai-account aan |
| `/wbso:auth` | Log in met een bestaande API key |
| `/wbso:whoami` | Wie ben ik nu ingelogd als? |
| `/wbso:logout` | Verwijder de lokale config |

Optioneel: `/wbso 4` om direct 4 uur voor te stellen op je
voornaamste activiteit van vandaag.

## Privacy

WBSO.ai gelooft dat jouw broncode jouw eigendom is, niet ons
trainingsmateriaal.

- **Lokaal**: `git log`, commit-messages, `git diff` en
  Claude-sessieprompts blijven op je laptop
- **Naar de API**: alleen project + uren + datum + (optioneel) een
  korte handgeschreven onderbouwing
- **Geen geheime data**: API key staat in `~/.config/wbso/config`
  met mode 600

## Updaten

```bash
claude plugins update wbso@wbso-ai
```

Daarna Claude Code herstarten.

## Geen account? Start hier

Maak een account aan op [wbso.ai/aanmelden](https://wbso.ai/aanmelden)
of doe het rechtstreeks vanuit de skill met `/wbso:signup`. Vragen?
Mail [paul@wbso.ai](mailto:paul@wbso.ai) of bel **085 333 26 15**.

WBSO.ai helpt Nederlandse softwarebedrijven hun WBSO-administratie
te doen — zonder bemiddelaarstarieven van 20-30%, met een vaste
prijs per project en gewoon volledige transparantie over wat we
voor je doen.

---

## Voor plugin-ontwikkelaars

<details>
<summary>Lokaal testen, releasen en wisselen tussen versies</summary>

### Repo-structuur

```
.claude-plugin/marketplace.json     # marketplace metadata
packages/
└── wbso/
    ├── .claude-plugin/plugin.json  # plugin metadata
    ├── bin/wbso                    # CLI (bash, in $PATH bij activatie)
    └── skills/
        ├── wbso/SKILL.md           # uren registreren
        ├── signup/SKILL.md         # account aanmaken
        ├── auth/SKILL.md           # inloggen
        ├── whoami/SKILL.md         # wie ben ik
        └── logout/SKILL.md         # uitloggen
```

### Lokaal testen

```bash
git clone git@github.com:wbso-ai/skill.git
cd skill
claude plugins validate ./packages/wbso
claude --plugin-dir ./packages/wbso  # eenmalig test, geen install
```

### Tegen een lokale server testen

Standaard wijst de skill naar `https://portal.wbso.ai`. Override
met een `.env.local` in je config-map:

```bash
mkdir -p ~/.config/wbso
cp .env.local.example ~/.config/wbso/.env.local
# pas WBSO_API_BASE_URL aan naar bv. http://localhost:3000
```

De `wbso` CLI sourcet die file bij elke aanroep. De URL vloeit door
naar de config die na signup geschreven wordt.

### Releasen

```bash
bin/release
```

Berekent de nieuwe versie op basis van commit-count (`1.<commit-count>`),
bumpt `version` in `packages/wbso/.claude-plugin/plugin.json`, maakt
een release-commit + tag (`v1.N`), pusht naar `origin/main`. Zonder
expliciete `version` in `plugin.json` zou elke commit als nieuwe
versie tellen — daarom een bewuste bump.

### Wisselen tussen GitHub-versie en lokale dev-versie

```bash
# Eenmalig: marketplace registreren
claude plugins marketplace add ~/Documents/skill

# Switch naar lokaal (pikt elke wijziging direct op)
claude plugins uninstall wbso@wbso-ai
claude plugins install wbso@wbso-ai-local

# Of terug naar GitHub
claude plugins uninstall wbso@wbso-ai-local --scope local
claude plugins install wbso@wbso-ai
```

Met `claude plugins list` zie je welke actief is. Updaten van de
lokale versie = `git pull` in `~/Documents/skill`, Claude Code
herstarten.

### CLI vanuit een gewone shell gebruiken

De `wbso` CLI staat in `$PATH` zodra de plugin actief is. Wil je 'm
ook buiten Claude Code aanroepen:

```bash
ln -s ~/Documents/skill/packages/wbso/bin/wbso ~/.local/bin/wbso
```

Subcommands: `login`, `context`, `whoami`, `track-time`,
`untrack-time`, `evidence`. Run `wbso help` voor de details.

</details>
