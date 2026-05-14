---
name: whoami
user-invocable: true
description: Toon onder welk account en bedrijf je nu ingelogd bent bij WBSO.ai. Gebruik wanneer de gebruiker vraagt "wie ben ik", "welk account", of "ben ik nog ingelogd".
---

# Wie ben ik?

```bash
wbso whoami
```

Output bestaat uit één header-regel en (optioneel) een lijst actieve
projecten:

```
<Naam> <<email>> · <Bedrijf>
Actieve projecten:
  - <project title> (<quota> uur) [<slug>]
  - ...
```

Bij `not_logged_in` (één enkele regel) → meld terug:

> *"Je bent niet ingelogd. Run `/wbso:auth` om in te loggen of
> `/wbso` om een nieuw account aan te maken."*

Anders meld de gebruiker als verhaal — toon naam met email tussen
`< >`, bedrijfsnaam, en de projecten zonder slugs:

> *"Je bent ingelogd als Jankees \<jankees@wbso.ai\> bij WBSO.ai."*
>
> *"Je hebt deze N projecten:"*
>
> *"- Automatische git commit WBSO scoring agent (250 uur)"*
> *"- ..."*

Slugs **niet** aan de gebruiker tonen — die zijn alleen voor interne
API-calls.
