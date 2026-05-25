---
name: signup
description: 'Maak een nieuw WBSO.ai-account aan vanuit de skill. Gebruik wanneer de gebruiker zegt "maak een account aan", "ik wil registreren", "nog geen account", of zich voor het eerst aanmeldt. Voor inloggen met een bestaande key, gebruik de auth skill.'
---

# Account aanmaken op WBSO.ai

Claude Code zet plugin-root `bin/` in `$PATH`, dus daar werkt `wbso`
direct. Codex gebruikt de skill-local wrapper `scripts/wbso`.

```bash
WBSO_CLI="$(command -v wbso 2>/dev/null || find "$PWD" "$HOME/.codex/plugins/cache/wbso-ai/wbso" "$HOME/.codex/plugins/cache" -path '*/skills/*/scripts/wbso' -type f 2>/dev/null | sort -V | tail -1)"
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
  "login_url": "https://portal.wbso.ai/login_with_code?code=ABCDE&return_to=/companies/current/compliance/api_keys",
  "message": "Er bestaat al een account met dit e-mailadres. Open de login-link om in te loggen."
}
```

Open de `login_url` in de browser (dezelfde flow als de wizard hieronder,
maar zonder `utm_source` rewrite). De gebruiker logt automatisch in en
landt direct op de API keys-pagina. Vraag dan om de nieuwe key en gebruik
de gebundelde `auth` skill (of `wbso login --api-key <KEY>`) om die op te
slaan.

```bash
url=$(echo "$response" | python3 -c 'import json,sys; print(json.load(sys.stdin)["login_url"])')
xdg-open "$url" 2>/dev/null || open "$url" 2>/dev/null || echo "Open zelf: $url"
```

Bij `422` toont het response-body een `error`-veld (bijv. ongeldig
e-mailadres). Laat dat aan de gebruiker zien.

## Browser openen voor profiel

De `login_url` is een eenmalig magic-link token (10 minuten geldig).
Leg uit waarom 't nodig is, vraag dán pas toestemming:

> *"Account is aangemaakt. Voor we uren kunnen registreren moet je
> account nog gekoppeld worden aan een WBSO-aanvraag — dat doe je in
> een korte wizard in de browser: óf je upload je RVO-beschikkings-
> PDF, óf je kiest een demo-aanvraag om mee te spelen."*
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

## Wacht tot de wizard klaar is

Direct na signup heeft het account nog geen projecten. Poll `wbso
context` tot er minstens één `<project slug=...>` blok in de respons
zit (max ~2 min, 4s per poging):

```bash
WBSO_CLI="$(command -v wbso 2>/dev/null || find "$PWD" "$HOME/.codex/plugins/cache/wbso-ai/wbso" "$HOME/.codex/plugins/cache" -path '*/skills/*/scripts/wbso' -type f 2>/dev/null | sort -V | tail -1)"
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
