---
name: logout
description: 'Log uit van WBSO.ai door de lokale config te verwijderen. Gebruik wanneer de gebruiker wil uitloggen, van account wil wisselen, of zegt "ik wil uitloggen".'
---

# Uitloggen van WBSO.ai

Verwijder de lokale config (`~/.config/wbso/config`) zodat de
volgende wbso-flow opnieuw vraagt om een API key.

## Stap 1: Bevestig

> *"Weet je zeker dat je wil uitloggen? Je API key in
> `~/.config/wbso/config` wordt verwijderd."*
>
> - Ja, uitloggen
> - Nee, laat staan

## Stap 2: Verwijder de config

Bij "Ja":

```bash
rm -f "$HOME/.config/wbso/config" && echo "Uitgelogd"
```

Bevestig kort:

> *"Uitgelogd. Gebruik de auth skill om opnieuw in te loggen, of de
> wbso skill om meteen een nieuw account aan te maken."*

## Geen config? Niets te doen

Als `~/.config/wbso/config` niet bestaat: zeg dat de gebruiker al
uitgelogd is en doe verder niets.
