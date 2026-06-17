<p align="center">
  <a href="https://wbso.ai"><img src="./assets/logo.svg" alt="WBSO.ai" width="160"></a>
</p>

<h1 align="center">WBSO-administratie, direct vanuit jouw coding harness.</h1>

<p align="center">
  Behoud jouw focus terwijl je moeiteloos WBSO-uren boekt. Jouw agent
  boekt, jij blijft coden.
</p>

---

Software developers boeken hun uren nog makkelijker en direct vanuit hun
coding harness. Zonder context-switch boek je direct jouw uren met
WBSO-waardigheid onderbouwing op het juiste project. Maak van jouw coding
agent een WBSO-expert die jouw aanvraag kent en helpt uren te boeken.

Zo boek je moeiteloos jouw WBSO-waardige uren en is jouw WBSO-administratie
altijd up-to-date en klaar voor een RVO-controle!

Deze skill draait in jouw werk-omgeving, leest waar je vandaag aan bezig
was (commits + lokale agent-sessies, volledig lokaal), beoordeelt het tegen
de RVO-criteria, en boekt het direct in op WBSO.ai.

Compatible met Claude Code, Codex, Cursor en [70+ andere
agents](https://skills.sh).

Op onze website staat meer informatie over hoe de
[WBSO-administratie](https://wbso.ai/wbso-administratie) werkt.

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
- 🧠 **Met geheugen** — leest je `git log` van vandaag én lokale
  Claude/Codex-prompts, zodat je niet hoeft terug te denken wat je deed
- 📚 **Transparant** — beoordeelt elke activiteit tegen de RVO-criteria
  en laat zien waarom 't wel of niet WBSO is
- ⚡ **Snel** — gemiddeld één Enter-druk om een dag te boeken
- 🔒 **Lokaal-first** — code, commit-messages en sessie-inhoud blijven op
  je laptop; alleen project + uren + datum gaan naar de API

## Installatie

Via [skills.sh](https://skills.sh) — één commando voor elke ondersteunde
agent:

```bash
npx skills add wbso-ai/skill
```

Globaal (user-level, alle projecten):

```bash
npx skills add wbso-ai/skill -g
```

Alle skills non-interactief installeren:

```bash
npx skills add wbso-ai/skill --all -y
```

Elke skill shipt een self-contained `scripts/wbso` CLI; geen aparte
plugin- of marketplace-install nodig.

Roep daarna de skill aan in gewone taal:

```text
boek mijn WBSO-uren van vandaag
```

Of expliciet — afhankelijk van je agent:

```text
/wbso              # Claude Code
$wbso              # Codex
```

De eerste keer vraagt de skill of je al een WBSO.ai-account hebt.
**Heb je 'm nog niet?** Je maakt 'm aan via de skill zelf — naam,
e-mail, bedrijf. Geen wurgcontracten, geen jaarbinding, gewoon een
account in 60 seconden.

## Skills

| Skill      | Wat het doet                              |
| ---------- | ----------------------------------------- |
| `wbso`     | Registreer uren via een gesprek           |
| `signup`   | Maak een nieuw WBSO.ai-account aan        |
| `auth`     | Log in met een bestaande API key          |
| `whoami`   | Toon het ingelogde account                |
| `logout`   | Verwijder de lokale config                |
| `feedback` | Stuur feedback over het platform naar het team |

Optioneel snelboeken: geef uren mee bij de aanroep, bv. `/wbso 4` of
`boek 4 uur WBSO`.

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
npx skills update wbso-ai/skill
```

Globaal:

```bash
npx skills update wbso-ai/skill -g
```

## Geen account? Start hier

Maak een account aan op
[wbso.ai/wbso-administratie/aanmelden](https://wbso.ai/wbso-administratie/aanmelden)
of doe het rechtstreeks vanuit de `signup` skill.

Liever full-service een aanvraag laten regelen? Meld je dan aan op
[wbso.ai/aanmelden](https://wbso.ai/aanmelden).

Vragen? Mail [product@wbso.ai](mailto:paul@wbso.ai) of bel **085 333 26 15**.

WBSO.ai helpt Nederlandse softwarebedrijven met hun WBSO-aanvragen én
WBSO-administratie. Op een aanvraag kan veel geld bespaard worden, de
transparante tarieven staan op [wbso.ai/aanmelden](https://wbso.ai/aanmelden).
Je kan echter ook alleen de administratie afnemen als los product. Je boekt
dan de eerste 100 uur kosteloos zodat je het kan uitproberen.

---

## Voor plugin-ontwikkelaars

<details>
<summary>Lokaal testen, releasen en wisselen tussen versies</summary>

### Repo-structuur

```
.claude-plugin/marketplace.json     # Claude marketplace metadata
.agents/plugins/marketplace.json    # Codex marketplace metadata
bin/
├── sync-wbso-cli                   # kopieer bin/wbso naar elke skill
└── check-wbso-cli-sync             # CI-check dat sync up-to-date is
packages/
└── wbso/
    ├── .claude-plugin/plugin.json  # Claude plugin metadata
    ├── .codex-plugin/plugin.json   # Codex plugin metadata
    ├── bin/wbso                    # Canonical CLI (bash)
    └── skills/
        ├── wbso/
        │   ├── SKILL.md            # uren registreren
        │   └── scripts/wbso        # self-contained CLI (synced from bin/wbso)
        ├── signup/
        │   ├── SKILL.md            # account aanmaken
        │   └── scripts/wbso
        ├── auth/
        │   ├── SKILL.md            # inloggen
        │   └── scripts/wbso
        ├── whoami/
        │   ├── SKILL.md            # wie ben ik
        │   └── scripts/wbso
        ├── logout/
        │   ├── SKILL.md            # uitloggen
        │   └── scripts/wbso
        └── feedback/
            ├── SKILL.md            # platform feedback
            └── scripts/wbso
```

### CLI en versie

Twee lagen — het how-plugins-work `build`/`check`-patroon:

| Rol | Pad |
|-----|-----|
| **Canonical CLI** | `packages/wbso/bin/wbso` — hier bewerk je |
| **Versie** | `packages/wbso/.claude-plugin/plugin.json` → `version` |
| **Generated targets** | `packages/wbso/skills/*/scripts/wbso` — kopie + ingebakken `WBSO_SKILL_VERSION` |

`npx skills add` installeert alleen de skill-map. De versie zit daarom
**in** elke `scripts/wbso` (regel 2: `WBSO_SKILL_VERSION="1.27"`), niet
in een apart `plugin.json` naast de skill. Sync leest de versie uit
`plugin.json` en stampt die in.

Na wijzigingen aan de canonical CLI **of** aan `version` in
`plugin.json`:

```bash
bin/sync-wbso-cli
bin/check-wbso-cli-sync
git add packages/wbso/skills/*/scripts/wbso
```

Met de repo-hook gebeurt sync automatisch bij commit:

```bash
git config core.hooksPath hooks
```

CI draait `bin/check-wbso-cli-sync` — commit nooit alleen
`packages/wbso/bin/wbso` zonder gesyncte skill-scripts.

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

Doet in één keer:

1. Berekent versie (`1.<commit-count>`) en zet die in beide
   `plugin.json`-manifests
2. Roept `bin/sync-wbso-cli` aan — stampt `WBSO_SKILL_VERSION` in alle
   skill-scripts
3. Commit, tag (`v1.N`), push, GitHub Release

Handmatig sync draaien hoef je bij release dus niet — wel bij gewone
commits die `bin/wbso` of `version` wijzigen (of via de pre-commit hook).

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
