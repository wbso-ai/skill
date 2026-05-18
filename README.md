<p align="center">
  <a href="https://wbso.ai"><img src="./assets/logo.svg" alt="WBSO.ai" width="160"></a>
</p>

<h1 align="center">Boek je WBSO-uren rechtstreeks vanuit Claude Code of Codex</h1>

<p align="center">
  Geen context-switch naar een urenformulier. Geen einde-van-de-dag-paniek. Geen gokken of je werk WBSO-waardig was.
</p>

<p align="center">
  <strong>Eén plugin in je AI-agent. Eén gesprek. Klaar.</strong>
</p>

---

Developers houden niet van administratie. Maar voor de WBSO moet
je wél bijhouden waar je elke dag aan werkt, beoordelen of het
binnen je projectaanvraag valt, en het inschrijven in een tool.
Te veel context-switch, dus blijft het liggen — of je betaalt
een bemiddelaar 20-30% van je voordeel om 't voor je te doen.

Deze plugin lost dat op. De skill draait in jouw werk-omgeving,
leest waar je vandaag aan bezig was (commits + lokale agent-sessies,
volledig lokaal), beoordeelt het tegen de RVO-criteria, en boekt het
direct in op WBSO.ai.

## Voorbeeld van een gesprek

```
> /wbso

Welkom bij WBSO.ai 👋 Ik help je je WBSO-uren registreren via een
kort gesprek. Je vertelt waar je vandaag aan hebt gewerkt, we kijken
samen of het WBSO-waardig is, en ik boek het direct in.

[ skill leest jouw commits + agent-sessies van vandaag,
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

## Waarom een agent-skill?

Agents zijn first-class burgers bij WBSO.ai. Onze compliance-API
spreekt markdown met XML-tags zodat Claude Code en Codex 't direct
kunnen lezen, de skill shipt z'n eigen CLI, en alle vragen zijn vrije
tekst — geen wrapper-laag, geen UI-omleiding.

- 💬 **Conversational** — typ wat je deed, geen invulvelden
- 🧠 **Met geheugen** — leest je `git log` van vandaag én lokale Claude/Codex-prompts, zodat je niet hoeft terug te denken wat je deed
- 📚 **Transparant** — beoordeelt elke activiteit tegen de RVO-criteria en laat zien waarom 't wel of niet WBSO is
- ⚡ **Snel** — gemiddeld één Enter-druk om een dag te boeken
- 🔒 **Lokaal-first** — code, commit-messages en sessie-inhoud blijven op je laptop; alleen project + uren + datum gaan naar de API

## Installatie

### Claude Code

```bash
claude plugins marketplace add wbso-ai/skill
claude plugins install wbso@wbso-ai
```

Daarna in Claude Code: `/wbso`.

### Codex

```bash
codex plugin marketplace add wbso-ai/skill
codex
/plugins
```

Installeer daarna `wbso` vanuit de WBSO.ai marketplace en start een
nieuwe thread. Gebruik de skill expliciet met `$`:

```text
$wbso registreer mijn WBSO-uren van vandaag
$whoami
$auth log me in bij WBSO.ai
```

Of vraag het in gewone taal:

```text
Gebruik WBSO.ai om mijn WBSO-uren van vandaag te registreren.
```

De eerste keer vraagt de skill of je al een WBSO.ai-account hebt.
**Heb je 'm nog niet?** Je maakt 'm aan via de skill zelf — naam,
e-mail, bedrijf. Geen wurgcontracten, geen jaarbinding, gewoon een
account in 60 seconden.

## Skills

| Skill | Claude Code | Wat het doet |
|---|---|---|
| `wbso` | `/wbso` | Registreer uren via een gesprek |
| `signup` | `/wbso:signup` | Maak een nieuw WBSO.ai-account aan |
| `auth` | `/wbso:auth` | Log in met een bestaande API key |
| `whoami` | `/wbso:whoami` | Wie ben ik nu ingelogd als? |
| `logout` | `/wbso:logout` | Verwijder de lokale config |

Optioneel: `/wbso 4` in Claude Code, of een urenargument bij expliciete
Codex skill-invocation, om direct 4 uur voor te stellen op je
voornaamste activiteit van vandaag.

## Privacy

WBSO.ai gelooft dat jouw broncode jouw eigendom is, niet ons
trainingsmateriaal.

- **Lokaal**: `git log`, commit-messages, `git diff` en
  Claude/Codex-sessieprompts blijven op je laptop
- **Naar de API**: alleen project + uren + datum + (optioneel) een
  korte handgeschreven onderbouwing
- **Geen geheime data**: API key staat in `~/.config/wbso/config`
  met mode 600

## Updaten

```bash
claude plugins update wbso@wbso-ai
codex plugin marketplace upgrade wbso-ai
```

Daarna Claude Code of Codex herstarten.

## Geen account? Start hier

Maak een account aan op [wbso.ai/aanmelden](https://wbso.ai/aanmelden)
of doe het rechtstreeks vanuit de `signup` skill. Vragen?
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
.claude-plugin/marketplace.json     # Claude marketplace metadata
.agents/plugins/marketplace.json    # Codex marketplace metadata
packages/
└── wbso/
    ├── .claude-plugin/plugin.json  # Claude plugin metadata
    ├── .codex-plugin/plugin.json   # Codex plugin metadata
    ├── bin/wbso                    # CLI (bash)
    └── skills/
        ├── wbso/
        │   ├── SKILL.md            # uren registreren
        │   └── scripts/wbso        # Codex wrapper naar ../../bin/wbso
        ├── signup/
        │   ├── SKILL.md            # account aanmaken
        │   └── scripts/wbso
        ├── auth/
        │   ├── SKILL.md            # inloggen
        │   └── scripts/wbso
        ├── whoami/
        │   ├── SKILL.md            # wie ben ik
        │   └── scripts/wbso
        └── logout/
            ├── SKILL.md            # uitloggen
            └── scripts/wbso
```

Claude Code voegt `bin/` van de plugin toe aan de Bash `PATH`, dus
skills kunnen daar `wbso` als bare command gebruiken. Codex documenteert
geen plugin-root `bin/` PATH; voor Codex exposeert elke skill daarom een
standaard Agent Skills `scripts/wbso` wrapper die naar dezelfde
plugin-root binary doorstart.

### Lokaal testen

```bash
git clone git@github.com:wbso-ai/skill.git
cd skill
claude plugins validate ./packages/wbso
claude --plugin-dir ./packages/wbso  # eenmalig test, geen install
codex plugin marketplace add .
codex
/plugins
```

Installeer daarna `wbso` vanuit de lokale WBSO.ai marketplace. Codex
cachet geïnstalleerde plugins. Als Codex na lokale wijzigingen een oude
versie blijft gebruiken, clear de cache en installeer opnieuw via
`/plugins`:

```bash
rm -rf ~/.codex/plugins/cache/wbso-ai/wbso
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
bumpt `version` in zowel `packages/wbso/.claude-plugin/plugin.json` als
`packages/wbso/.codex-plugin/plugin.json`, maakt een release-commit +
tag (`v1.N`), pusht naar `origin/main`. Zonder expliciete `version` in
`plugin.json` zou elke commit als nieuwe versie tellen — daarom een
bewuste bump.

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

Sommige clients zetten plugin-`bin/` automatisch in `$PATH`. Wil je de
CLI ook vanuit een gewone shell gebruiken:

```bash
ln -s ~/Documents/skill/packages/wbso/bin/wbso ~/.local/bin/wbso
```

Subcommands: `login`, `context`, `whoami`, `track-time`,
`untrack-time`, `evidence`. Run `wbso help` voor de details.

</details>
