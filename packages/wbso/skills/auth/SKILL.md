---
name: auth
user-invocable: true
description: Log in op WBSO.ai door een API key te plakken. Gebruik wanneer de gebruiker wil inloggen, van account wil wisselen, of zegt "ik wil opnieuw inloggen". Opent eerst de portal-pagina waar je een API key kan aanmaken.
---

# Inloggen op WBSO.ai

Help de gebruiker een nieuwe API key aan te maken en op te slaan in
`~/.config/wbso/config`. Gebruik de meegeleverde `wbso` CLI.

## Stap 1: Vraag toestemming om de browser te openen

Leg uit waarom je 't doet, dan vraag je toestemming:

> *"Om je in te loggen heb ik een API key nodig die jouw account aan
> dit apparaat koppelt. Die maak je aan in de browser op je WBSO.ai
> bedrijfspagina — ik open 'm voor je, jij maakt een key aan en plakt
> 'm hier."*
>
> *"Mag ik die pagina nu openen?"*

Wacht op antwoord. Bij "ja" / "ok" / lege regel:

```bash
URL="${WBSO_API_BASE_URL:-https://portal.wbso.ai}/companies/current/compliance/api_keys"
xdg-open "$URL" 2>/dev/null || open "$URL" 2>/dev/null || echo "Open zelf: $URL"
```

Bij "nee" / "later": toon de URL als platte tekst, gebruiker opent
zelf.

`~/.config/wbso/.env.local` wordt automatisch geladen door de CLI bij
elke aanroep, maar voor het openen van de browser-URL moet je 'm zelf
sourcen — zie het bash-blok hierboven.

## Stap 2: Vraag de API key

Stuur als plain-tekst:

> *"Plak je nieuwe API key hier."*

## Stap 3: Login via de CLI

```bash
wbso login --api-key "<KEY>"
```

Output `ok (<email> · <bedrijf>)` = klaar — bevestig kort:

> *"Ingelogd als <email> bij <bedrijf>."*

Output begint met `error:` = key ongeldig, vraag opnieuw.
