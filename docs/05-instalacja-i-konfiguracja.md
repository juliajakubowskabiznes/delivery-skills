# Instalacja i konfiguracja

## Wymagania

| Rzecz | Po co | Konieczne? |
|---|---|---|
| **Claude Code** (CLI albo IDE) | uruchamia skille | tak |
| **`jq`** | hook w folderze klienta parsuje nim ścieżkę pliku | tak, jeśli chcesz hooka |
| **Codex / GPT** (`codex exec`, plugin `codex:rescue`) | druga opinia w `/hejt` i wariant „buduje GPT" | nie — bez niego `/hejt` działa w trybie „ocena własna, nie niezależna" i jawnie to zaznacza |
| **`gh` albo `glab`** | tracker zadań na GitHubie/GitLabie | nie — alternatywą jest lokalny markdown w `.scratch/` |

## Instalacja

**Wariant A — dla jednego projektu** (skille widoczne tylko w tym repo):

```bash
mkdir -p /ścieżka/do/projektu/.claude/skills
cp -R skills/* /ścieżka/do/projektu/.claude/skills/
```

**Wariant B — globalnie** (skille widoczne we wszystkich projektach):

```bash
cp -R skills/* ~/.claude/skills/
```

Rekomendacja: **wariant A**. Skille wdrożeniowe (`workflow`, `analiza`, `hejt`,
`zalacznik`, `init-klienta`) mają sens tam, gdzie leżą foldery klientów; instalacja
globalna dokłada je do każdego projektu, także kodowego, gdzie tylko zaśmiecają listę.

Sprawdzenie, czy widać:

```bash
claude   # w folderze projektu, potem wpisz /workflow
```

## Pierwsze uruchomienie na nowym kliencie

```
/init-klienta <ścieżka do folderu klienta>
```

Powstaje `CLAUDE.md` klienta i `.claude/settings.json` z hookiem. Agent pokaże jedno
i drugie do akceptacji — bez niej krok nie jest zamknięty.

```
/setup-matt-pocock-skills
```

Konfiguruje tracker (dla pracy solo: **lokalny markdown**), etykiety triage i układ
dokumentów domenowych. Zapisuje do `docs/agents/` — pozostałe skille czytają stamtąd,
zamiast zakładać platformę.

```
/workflow
```

Od tego momentu agent prowadzi przez 7 faz i melduje, w której jesteście.

## Co dopasować pod siebie

### Ścieżki zapisu

Trzy skille zapisują do folderu planowania wdrożenia. Domyślnie jest to
`<folder klienta>/<wdrożenie>/03_planowanie/` — zmień, jeśli trzymasz pliki inaczej:

| Plik | Fragment do zmiany |
|---|---|
| [`skills/analiza/SKILL.md`](../skills/analiza/SKILL.md) | ścieżka raportu analizy |
| [`skills/to-spec/SKILL.md`](../skills/to-spec/SKILL.md) | miejsce docelowe zatwierdzonego specu |
| [`skills/to-questionnaire/SKILL.md`](../skills/to-questionnaire/SKILL.md) | miejsce finalnej ankiety |

Wzorzec folderu klienta opisuje [`04-struktura-folderow.md`](04-struktura-folderow.md) —
jeśli używasz innego, wystarczy podmienić ścieżki w tych trzech miejscach.

### Drugi model (krytyka i budowa)

[`skills/hejt/SKILL.md`](../skills/hejt/SKILL.md) i
[`skills/workflow/references/budowa.md`](../skills/workflow/references/budowa.md)
zakładają Codex/GPT jako niezależnego recenzenta. Można podmienić na dowolny inny model
— warunek jest jeden i jest merytoryczny: **sędzia musi dostać świeży kontekst**, bez
historii dyskusji, inaczej dziedziczy ramę autora i przestaje być niezależny.

Operacyjnie: ciężkie wywołania zawsze w tle (`--background`). Zadanie na kilka minut
myślenia nie mieści się w timeoucie Bash, a zabity proces zostawia blokadę „task still
running" (odblokowanie: `codex-companion.mjs cancel`).

### Umowa i załącznik

[`skills/zalacznik/SKILL.md`](../skills/zalacznik/SKILL.md) zakłada, że źródłem prawdy
umowy jest dokument w chmurze, a repo nie trzyma kopii treści. Jeśli wolisz repo — zmień
tę sekcję, ale zostaw zasadę **jednego źródła prawdy**: kopia umowy w dwóch miejscach
rozjeżdża się przy pierwszej rundzie uwag klienta.

### Szablon CLAUDE.md klienta

[`skills/init-klienta/SKILL.md`](../skills/init-klienta/SKILL.md) — sekcje szablonu
i treść hooka. Jeśli dokładasz sekcje, pamiętaj o koszcie: ten plik ładuje się w każdej
sesji.

### Język

Skille wdrożeniowe są po polsku (produkują dokumenty dla polskich klientów), inżynierskie
po angielsku (tak wyglądają oryginały). Można ujednolicić w obie strony — instrukcja dla
modelu nie musi być w tym samym języku, co dokument, który powstaje. Jedyne miejsce, gdzie
język jest wymuszony jawnie, to `/to-spec` (sekcja „JĘZYK") i `/to-questionnaire`
(ankieta idzie do klienta).

## Typowe problemy

| Objaw | Przyczyna | Co zrobić |
|---|---|---|
| Hook nic nie robi | brak `jq` albo `settings.json` nie w tym folderze, w którym startuje sesja | zainstaluj `jq`; sprawdź, że uruchamiasz Claude Code z folderu klienta |
| `/hejt` nie woła GPT | brak Codexa albo zablokowany job | skill sam przejdzie w tryb oceny własnej i to zaznaczy; blokadę zdejmij przez `codex-companion.mjs cancel` |
| Skill nie widoczny | plik nie nazywa się `SKILL.md` albo brak frontmattera `name` | sprawdź nazwę pliku i nagłówek YAML |
| Agent pomija bramki | pracuje poza `/workflow` | powiedz „działamy według workflow" — kręgosłup wymusza kolejność i meldowanie fazy |
| Agent dopisuje do CLAUDE.md bez pytania | brak hooka w tym folderze | uruchom `/init-klienta` ponownie (nie nadpisze istniejącego CLAUDE.md, doda brakujące) |
