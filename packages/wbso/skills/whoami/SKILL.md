---
name: whoami
description: 'Toon onder welk account en bedrijf je nu ingelogd bent bij WBSO.ai. Gebruik wanneer de gebruiker vraagt "wie ben ik", "welk account", of "ben ik nog ingelogd".'
---

# Wie ben ik?

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
"$WBSO_CLI" whoami
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

> *"Je bent niet ingelogd. Gebruik de auth skill om in te loggen of
> de wbso skill om een nieuw account aan te maken."*

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
