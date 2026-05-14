---
name: signup
user-invocable: true
description: Maak een nieuw WBSO.ai-account aan vanuit de skill. Gebruik wanneer de gebruiker zegt "maak een account aan", "ik wil registreren", "nog geen account", of zich voor het eerst aanmeldt. Voor inloggen met een bestaande key, gebruik `/wbso:auth`.
---

# Account aanmaken op WBSO.ai

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
wbso signup \
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

Bij `422` toont het response-body een `error`-veld. Laat dat aan de
gebruiker zien. Komt het neer op "bestaat al"? Schakel over naar
`/wbso:auth` zodat de gebruiker met z'n bestaande key inlogt.

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

Voeg `utm_source=claude` toe aan het `return_to`-pad in `login_url`
zodat de portal weet dat de gebruiker uit Claude komt (dat triggert
o.a. een persoonlijke welkomsttitel). Concreet: vervang
`return_to=%2Fsignup%2Fapplication` door
`return_to=%2Fsignup%2Fapplication%3Futm_source%3Dclaude`.

```bash
url="<login_url met utm_source=claude in return_to>"
xdg-open "$url" 2>/dev/null || open "$url" 2>/dev/null || \
  echo "Open zelf: $url"
```

Direct na het openen, plain tekst:

> *"Rond de wizard af in je browser — daarna kun je hier verder met
> uren registreren."*

Bij "nee" / "later": toon dezelfde URL (mét `utm_source=claude` in
`return_to`) als platte tekst zodat de gebruiker 'm zelf kan openen,
en stuur dezelfde afmaak-melding.

## Wacht tot de wizard klaar is

Direct na signup heeft het account nog geen projecten. Poll `wbso
context` tot er minstens één `<project slug=...>` blok in de respons
zit (max ~2 min, 4s per poging):

```bash
for i in $(seq 1 30); do
  wbso context > /tmp/wbso-ctx.md
  grep -q '<project slug=' /tmp/wbso-ctx.md && break
  sleep 4
done
cat /tmp/wbso-ctx.md
```

Zie je projecten? Noem ze kort op en check of de gebruiker uren wil
boeken via `/wbso`. Blijft het leeg na 2 minuten? Vraag of er hulp
nodig is in het stappenplan.
