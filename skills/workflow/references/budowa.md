# Krok 6 — BUDOWA: warianty

Domyślnie `/implement` (Claude) — buduj, aż testy z fazy 5c przechodzą (green),
potem uprość (refactor).

**Wariant „buduje GPT" (na życzenie użytkownika, np. przy dużych buildach
albo oszczędzaniu limitu Claude):** budowę wykonuje GPT:
`codex exec -s workspace-write -c model_reasoning_effort=high` (długie zadania →
--background; ciężki job na pierwszym planie przekracza timeout Bash i zostawia
blokadę — patrz /hejt). Zasady: GPT NIE czyta ani nie edytuje `.env`, `CLAUDE.md`
i plików z sekretami; Claude NIE pisze wtedy implementacji — recenzuje diff
(poprawność, edge case'y, bezpieczeństwo) i odsyła uwagi; max 3 rundy poprawek,
potem STOP i raport.
