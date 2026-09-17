---
name: wbso-signup
description: 'Maak een nieuw WBSO.ai-account aan vanuit de skill en koppel de eerdere WBSO-aanvraag door de RVO-PDF direct te uploaden. Gebruik wanneer de gebruiker zegt "maak een account aan", "ik wil registreren", "nog geen account", "upload mijn aanvraag", of zich voor het eerst aanmeldt. Voor inloggen met een bestaande key, gebruik de wbso-auth skill.'
---

# Account aanmaken op WBSO.ai

Claude Code zet plugin-root `bin/` in `$PATH`, dus daar werkt `wbso`
direct. Anders resolve je de self-contained `scripts/wbso` naast deze
skill als `$WBSO_CLI`:

```bash
WBSO_CLI="$(
  command -v wbso 2>/dev/null ||
  { [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -x "${CLAUDE_PLUGIN_ROOT}/bin/wbso" ] && printf '%s\n' "${CLAUDE_PLUGIN_ROOT}/bin/wbso"; } ||
  find \
    "$PWD" \
    "$HOME/.claude/skills" \
    "$PWD/.claude/skills" \
    "$HOME/.cursor/skills" \
    "$PWD/.cursor/skills" \
    "$HOME/.agents/skills" \
    "$PWD/.agents/skills" \
    "$HOME/.codex/skills" \
    "$HOME/.codex/plugins/cache/wbso-ai/wbso" \
    "$HOME/.codex/plugins/cache" \
    -path '*/scripts/wbso' -type f 2>/dev/null |
  sort -V | tail -1
)"
test -n "$WBSO_CLI" || { echo "WBSO CLI niet gevonden"; exit 127; }
```

Stuur eerst een korte uitleg en vraag alle vier de velden in één
plain-tekst bericht (geen modal, geen losse vragen — anders voelt 't
als een verhoor):

> *"Top! Vul deze vier even in op aparte regels, dan maak ik je
> account aan:"*
>
> *"- Voornaam:"*
> *"- Achternaam:"*
> *"- E-mailadres:"*
> *"- Bedrijfsnaam:"*

Parse de regels. Als er één ontbreekt of onduidelijk is: vraag dán
pas specifiek dat ene veld opnieuw.

## Aanmaken via de CLI

```bash
"$WBSO_CLI" signup \
  --first-name "<VOORNAAM>" \
  --last-name "<ACHTERNAAM>" \
  --email "<EMAIL>" \
  --company-name "<BEDRIJFSNAAM>"
```

`wbso signup` slaat de `api_key` direct op in
`~/.config/wbso/config` en print een JSON-respons met:

```json
{
  "email": "...",
  "company_slug": "...",
  "api_key": "...",
  "login_url": "https://portal.wbso.ai/login_with_code?code=ABCDE&return_to=/signup/application"
}
```

### Exit code 2: het e-mailadres heeft al een account

Bestaat het adres al, dan komt `wbso signup` terug met exit code 2 en
een respons als:

```json
{
  "existing_account": true,
  "email": "...",
  "message": "Er bestaat al een account met dit e-mailadres. We hebben een login-link naar je e-mail gestuurd."
}
```

De portal stuurt automatisch een magic-link e-mail naar het opgegeven
adres. Vraag de gebruiker om de e-mail te openen en op de inlog-link te
klikken. Na inloggen moet de gebruiker naar
`Instellingen → Compliance → API keys` navigeren om een nieuwe key aan
te maken. Vraag dán om die key en gebruik de gebundelde `wbso-auth` skill (of
`wbso login --api-key <KEY>`) om die op te slaan.

Bij `422` toont het response-body een `error`-veld (bijv. ongeldig
e-mailadres). Laat dat aan de gebruiker zien.

## Aanvraag koppelen: PDF uploaden of wizard in de browser

Direct na signup heeft het account nog geen projecten. Die komen uit de
eerdere WBSO-aanvraag: de RVO-bevestiging "Aanvraag invoerweergave" als
PDF. Vraag in plain tekst of de gebruiker die bij de hand heeft:

> *"Account is aangemaakt. Om uren te kunnen boeken moet ik je
> WBSO-aanvraag kennen. Heb je de RVO-bevestiging van je aanvraag als
> PDF ('Aanvraag invoerweergave' van mijn.rvo.nl)? Geef het pad, dan
> lees ik 'm direct in. Geen PDF bij de hand? Dan open ik een korte
> wizard in je browser, daar kun je ook een demo-aanvraag kiezen."*

Interpreteer:

- Een bestandspad (of een gesleept bestand) → **Uploaden vanuit de agent**
- "nee" / "geen PDF" / "demo" / "browser" / "wizard" → **Wizard in de browser**

### Uploaden vanuit de agent

```bash
"$WBSO_CLI" upload "<pad/naar/bestand.pdf>"
```

De CLI uploadt de PDF, wacht tot de import klaar is (elke 4 seconden
een check, maximaal 3 minuten) en print dan de JSON-status:

```json
{
  "id": 42,
  "status": "imported",
  "import_error": null,
  "so_number": "SO12345678",
  "status_url": "https://portal.wbso.ai/api/v1/compliance/submissions/42",
  "projects": [
    {"slug": "ai-assistent-klantenservice", "title": "AI-assistent voor klantenservice", "number": "1"}
  ]
}
```

Exit codes: `0` geïmporteerd, `1` upload geweigerd of import mislukt,
`3` na 3 minuten nog niet klaar.

Handel het resultaat af in plain tekst, zonder slugs, ID's of
HTTP-codes:

- **`status: imported`** → noem de projecttitels kort op en ga door met
  de `wbso` skill om uren te boeken:

  > *"Je aanvraag is ingelezen. Ik zie deze projecten: AI-assistent voor
  > klantenservice, Slimme classifier. Zullen we meteen uren boeken?"*

- **`status: error`** → het document was geen RVO-bevestiging of niet
  leesbaar. Leg uit waar de juiste PDF staat en bied aan het opnieuw te
  proberen, of anders de wizard te openen:

  > *"Dit lijkt niet de RVO-bevestiging te zijn. Zo vind je de juiste
  > PDF: log in op mijn.rvo.nl met eHerkenning, open de meest recente
  > aanvraag, ga naar het tabblad Documenten en download 'Aanvraag
  > invoerweergave'. Geef het pad van die PDF, of zeg 'wizard' dan open
  > ik de browser."*

- **Upload geweigerd** (respons met alleen een `error`-veld): geen PDF,
  groter dan 10 MB, of het account mag geen aanvraag uploaden. Geef de
  melding uit `error` in mensentaal terug en val terug op de wizard.
- **Exit code 3** → zeg dat de import nog loopt. Check na een minuut
  opnieuw met `"$WBSO_CLI" upload-status --id <id>` en handel de JSON
  daarna op dezelfde manier af.

### Wizard in de browser

De `login_url` is een eenmalig magic-link token (10 minuten geldig).
Leg uit waarom 't nodig is, vraag dán pas toestemming:

> *"Dan open ik een korte wizard in je browser: daar upload je de
> RVO-PDF alsnog, óf je kiest een demo-aanvraag om mee te spelen."*
>
> *"Mag ik die wizard nu openen in je browser?"*

Bij "ja" / "ok" / lege regel:

Voeg `utm_source=agent_skill` toe aan het `return_to`-pad in `login_url`
zodat de portal weet dat de gebruiker uit een AI-agent komt. Concreet:
vervang
`return_to=%2Fsignup%2Fapplication` door
`return_to=%2Fsignup%2Fapplication%3Futm_source%3Dagent_skill`.

```bash
url="<login_url met utm_source=agent_skill in return_to>"
xdg-open "$url" 2>/dev/null || open "$url" 2>/dev/null || \
  echo "Open zelf: $url"
```

Direct na het openen, plain tekst:

> *"Rond de wizard af in je browser — daarna kun je hier verder met
> uren registreren."*

Bij "nee" / "later": toon dezelfde URL (mét `utm_source=agent_skill` in
`return_to`) als platte tekst zodat de gebruiker 'm zelf kan openen,
en stuur dezelfde afmaak-melding.

#### Wacht tot de wizard klaar is

Poll `wbso context` tot er minstens één `<project slug=...>` blok in de
respons zit (max ~2 min, 4s per poging):

```bash
WBSO_CLI="$(
  command -v wbso 2>/dev/null ||
  { [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -x "${CLAUDE_PLUGIN_ROOT}/bin/wbso" ] && printf '%s\n' "${CLAUDE_PLUGIN_ROOT}/bin/wbso"; } ||
  find \
    "$PWD" \
    "$HOME/.claude/skills" \
    "$PWD/.claude/skills" \
    "$HOME/.cursor/skills" \
    "$PWD/.cursor/skills" \
    "$HOME/.agents/skills" \
    "$PWD/.agents/skills" \
    "$HOME/.codex/skills" \
    "$HOME/.codex/plugins/cache/wbso-ai/wbso" \
    "$HOME/.codex/plugins/cache" \
    -path '*/scripts/wbso' -type f 2>/dev/null |
  sort -V | tail -1
)"
test -n "$WBSO_CLI" || { echo "WBSO CLI niet gevonden"; exit 127; }
for i in $(seq 1 30); do
  "$WBSO_CLI" context > /tmp/wbso-ctx.md
  grep -q '<project slug=' /tmp/wbso-ctx.md && break
  sleep 4
done
cat /tmp/wbso-ctx.md
```

Zie je projecten? Noem ze kort op en check of de gebruiker uren wil
boeken via de `wbso` skill. Blijft het leeg na 2 minuten? Vraag of er hulp
nodig is in het stappenplan.
