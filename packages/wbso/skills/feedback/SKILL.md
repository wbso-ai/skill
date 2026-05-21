---
name: feedback
description: 'Stuur feedback over het WBSO.ai platform naar het team. Gebruik wanneer de gebruiker een bug meldt, een idee/verbeterpunt heeft, een vraag over het platform stelt, een klacht of frustratie uit, of een compliment wil geven. Werkt ook anoniem zonder ingelogde sessie.'
---

# Platform feedback

Stuur feedback over WBSO.ai naar het team in één korte API-call. Werkt
met of zonder ingelogde sessie — met API key wordt de feedback gekoppeld
aan de gebruiker, anders is 'ie anoniem.

## Wanneer gebruiken

Roep deze skill aan wanneer:

- De gebruiker iets meldt dat lijkt op een bug ("hij crasht", "fout
  bij X", "klopt niet")
- De gebruiker een idee of feature-verzoek noemt ("het zou handig
  zijn als...", "kunnen jullie...", "ik mis...")
- De gebruiker een vraag heeft die niet uit de skill-context te
  beantwoorden is en die het product zelf raakt
- De gebruiker een compliment geeft over het platform
- De gebruiker expliciet zegt "ik wil feedback geven" of soortgelijk

Roep 'm **niet** aan voor:

- Vragen over hoe je een API-call doet (los dat op met de docs)
- Twijfels over een WBSO-inschatting (dat gaat via `suggest-project`
  of een gewoon gesprek)
- Vragen die je gewoon zelf kunt beantwoorden

## Vraagstijl

Standaard plain tekst, zoals in de hoofd-`wbso` skill. Eén vraag
tegelijk, kort.

## Stap 1: Categorie kiezen

Op basis van wat de gebruiker zei, kies één van deze categorieën:

| Categorie    | Wanneer                                          |
|--------------|--------------------------------------------------|
| `bug`        | Iets werkt aantoonbaar niet zoals het hoort      |
| `idea`       | Feature-verzoek, verbeterpunt, "zou handig zijn" |
| `question`   | Vraag over het platform die je niet kunt beantwoorden |
| `compliment` | Positieve feedback                               |
| `complaint`  | Frustratie of klacht over het platform (irritant, te traag, onlogisch) |
| `other`      | Rest                                             |

Vraag bij twijfel kort terug. Anders: ga door.

## Stap 2: Bericht ophalen

Heeft de gebruiker net al iets concreets verteld (de aanleiding voor
deze skill): gebruik dat als bericht, eventueel kort opgeschoond.

Anders, vraag plain tekst:

> *"Wat wil je doorgeven? Eén à twee zinnen is genoeg."*

Wacht op vrije tekst. Neem die letterlijk over, zonder polish.

## Stap 3: Versturen

Claude Code zet plugin-root `bin/` in `$PATH`, dus daar werkt `wbso`
direct. Codex gebruikt de skill-local wrapper `scripts/wbso`.

```bash
WBSO_CLI="$(command -v wbso 2>/dev/null || find "$PWD" "$HOME/.codex/plugins/cache/wbso-ai/wbso" "$HOME/.codex/plugins/cache" -path '*/skills/*/scripts/wbso' -type f 2>/dev/null | sort -V | tail -1)"
test -n "$WBSO_CLI" || { echo "WBSO CLI niet gevonden"; exit 127; }
"$WBSO_CLI" feedback \
  --message "<wat de gebruiker zei>" \
  --category "<bug|idea|question|compliment|complaint|other>"
```

Optioneel `--context '{"step":"track-time"}'` om context mee te
sturen die het team helpt te reproduceren (bij bugs). Houd JSON klein
en zonder secrets.

## Stap 4: Bevestigen

Server retourneert `201` op succes en het team krijgt automatisch
een Slack-melding. Geef kort terug:

> *"Doorgegeven aan het WBSO.ai team. Bedankt!"*

Bij compliment: extra warm.

> *"Doorgegeven aan het team — ze zullen blij zijn met je
> compliment. Bedankt!"*

Geen JSON, HTTP-codes of IDs naar de gebruiker.

## Foutafhandeling

- `400` / `422`: validatiefout (lege boodschap, onbekende
  categorie) — vraag opnieuw
- `401` met token: token ongeldig — feedback alsnog versturen
  zonder header (anoniem), of de gebruiker laten weten dat 'ie
  niet ingelogd is
- Netwerkfout: meld kort *"Kon de feedback niet versturen, je
  internet of het platform doet iets vreemds."* en stop
