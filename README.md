# Delivery Skills — workflow projektowy dla Claude Code

**18 skilli**, które zmieniają Claude Code z narzędzia „napisz mi kod" w agenta
prowadzącego projekt od pierwszej rozmowy z klientem do odbioru gotowego systemu —
z bramkami decyzyjnymi, niezależną recenzją drugiego modelu i twardymi hamulcami
na zgadywanie.

**Stack:** Claude Code (skille + hooki + sub-agenty) · plugin
[Codex](https://github.com/openai/codex-plugin-cc) jako niezależny recenzent ·
issue tracker do wyboru (GitHub / GitLab / lokalny markdown).

---

## Problem, który to rozwiązuje

Agent kodujący jest szybki i chętny — i to jest problem. Postawiony przed projektem,
w którym połowa informacji jeszcze nie istnieje, będzie:

- **wypełniał luki hipotezami**, które w dokumencie wyglądają identycznie jak fakty,
- **poprawiał zatwierdzone dokumenty** „bo tak będzie lepiej", unieważniając zgodę,
  która za nimi stała,
- **budował, zanim ktokolwiek ustalił**, po czym pozna się, że zbudowane jest dobrze,
- **recenzował własną pracę** z tą samą ramą myślenia, w której ją napisał.

To repo to zestaw ograniczeń, które temu zapobiegają — i przykład tego, jak projektować
skille, żeby reguła przeżyła długą sesję zamiast utonąć w kontekście.

## Spis treści

- [Zrozum w 2 minuty](#zrozum-w-2-minuty)
- [Mechanizmy — co tu jest ciekawego inżyniersko](#mechanizmy--co-tu-jest-ciekawego-inżyniersko)
- [Workflow — 7 faz](#workflow--7-faz)
- [Katalog skilli](#katalog-skilli)
- [Rola pluginu Codex](#rola-pluginu-codex)
- [Wzorce projektowania skilli](#wzorce-projektowania-skilli)
- [Szybki start](#szybki-start)
- [Struktura folderów projektu](#struktura-folderów-projektu)
- [Co dopasować pod siebie](#co-dopasować-pod-siebie)
- [Pochodzenie i licencja](#pochodzenie-i-licencja)

Głębsze opisy: [`docs/`](docs/) — [workflow](docs/01-workflow.md) ·
[skille wdrożeniowe](docs/02-skille-wdrozeniowe.md) ·
[skille inżynierskie](docs/03-skille-inzynierskie.md) ·
[struktura folderów](docs/04-struktura-folderow.md) ·
[instalacja i konfiguracja](docs/05-instalacja-i-konfiguracja.md)

---

## Zrozum w 2 minuty

Trzy zasady przewijają się przez wszystkie 18 skilli:

1. **Bramki 🚧** — po każdej fazie agent zatrzymuje się i czeka na decyzję człowieka.
   Nie „przygotowuje już kolejnego kroku, skoro i tak trzeba będzie".
2. **Zero założeń** — luka w wiedzy jest *pytaniem* (do człowieka albo do klienta),
   nigdy cicho wpisaną hipotezą. Brak informacji ma być widoczny w dokumencie.
3. **Bramka naniesień** — krytyka i research produkują *propozycje*, nie zmiany.
   Zatwierdzony dokument zmienia się dopiero po wyborze z tabeli ZA/PRZECIW.

Plus jeden nawyk, który zmienia ekonomię projektu: **głęboko przed wyceną, płytko przed
budową**. Przed ofertą sprawdzasz wykonalność — najlepiej dowodem na prawdziwych danych.
Szczegóły wykonawcze powstają dopiero po podpisaniu umowy, bo to płatna część pracy.

Domena, na której to powstało, to projekty wdrożeniowe u klientów (automatyzacje,
integracje, systemy szyte na miarę) — ale mechanika jest domenowo obojętna. Każdy projekt,
w którym trzeba **najpierw ustalić, potem wycenić, potem zbudować**, mieści się w tym
samym szkielecie.

## Mechanizmy — co tu jest ciekawego inżyniersko

| Mechanizm | Gdzie | Po co |
|---|---|---|
| **Fork kontekstu** (`context: fork`) | [`/analiza`](skills/analiza/SKILL.md) | agent czyta materiały **bez historii rozmowy**, więc nie dziedziczy tego, w co bieżąca sesja zdążyła uwierzyć — ocenia to, co realnie jest na dysku |
| **Recenzent bez wspólnej ramy** | [`/hejt`](skills/hejt/SKILL.md) | dyskusja Claude↔Codex, a potem **świeże wywołanie sędziego bez historii dyskusji** — sędzia, który przeczytał całą debatę, dziedziczy ramę autora i przestaje być niezależny |
| **Hook jako twarda pamięć** | [`/init-klienta`](skills/init-klienta/SKILL.md) | `PostToolUse` na `Write\|Edit` wychodzi z kodem 2 przy zapisie do spec/PRD/CLAUDE.md. Instrukcja w prompcie tonie po godzinie sesji — hook odpala się mechanicznie, niezależnie od tego, co sesja pamięta |
| **Sub-agenty równoległe** | [`/code-review`](skills/code-review/SKILL.md), research, `design-it-twice` | dwie osie przeglądu w osobnych kontekstach, żeby się nie zanieczyszczały; jeden agent na pytanie badawcze; 3+ wariantów interfejsu naraz |
| **Rozdzielenie warstw dokumentu** | [`references/plan.md`](skills/workflow/references/plan.md) | test na każdy fragment: „czy obowiązuje też przy innym projekcie?" → jeśli tak, to nie należy do tego dokumentu (reguła→skill, fakt→CONTEXT.md, decyzja→ADR) |
| **Tracker jako abstrakcja** | [`/setup-matt-pocock-skills`](skills/setup-matt-pocock-skills/SKILL.md) | skille czytają `docs/agents/issue-tracker.md`, zamiast zakładać platformę — ta sama instrukcja działa na GitHubie, GitLabie i plikach markdown |
| **EARS jako kontrakt testowy** | [`/to-spec`](skills/to-spec/SKILL.md) | kryteria pisane wzorcami (`GDY <zdarzenie>, system MUSI X`) konwertują się 1:1 na testy. Twarda reguła: każdy krok czytający dane przez AI musi mieć kryterium „co, gdy odczyt zawiedzie" |
| **Progresywne ładowanie reguł** | [`skills/workflow/`](skills/workflow/) | krótki kręgosłup w `SKILL.md`, szczegóły w `references/` doczytywane przy wejściu w fazę — gruby regulamin wypadłby z kontekstu dokładnie wtedy, kiedy jest potrzebny |

## Workflow — 7 faz

Stała kolejność, bramka po każdej fazie. Agent melduje pozycję: `[WORKFLOW faza N/7 — NAZWA]`.

| # | Faza | Co się dzieje | Efekt | Bramka |
|---|---|---|---|---|
| 0 | **Setup** | raz na projekt: CLAUDE.md projektu + tracker zadań | pamięć projektu | — |
| 1 | **Poznaj** | fork czyta wszystkie materiały, potem przesłuchanie człowieka; czego nie wie nikt po naszej stronie → ankieta do klienta | raport + decyzje + pytania | 🚧 potwierdzenie zrozumienia |
| 2 | **Zaplanuj pod wycenę** | ryzyko, wykonalność **dowiedziona na realnych danych**, podział na kroki, które da się osobno sprzedać i odebrać | plan całości | 🚧 zatwierdzenie planu |
| 3 | **Skrytykuj** | dyskusja dwóch modeli → świeży sędzia rozstrzyga spory i robi własny research | tabela propozycji + werdykt | 🚧 wybór zmian |
| 4 | **Umowa** | zakres językiem klienta, kryteria odbioru = sprawdzalny test | podpis + płatność z góry | 🚧 akceptacja przed wysyłką |
| 5 | **Domknij i potnij** | *dopiero po wpłacie*: PRD, cięcie na zadania z zależnościami, **testy przed budową** | PRD + tickety + testy | 🚧 zatwierdzenie ticketów |
| 6 | **Buduj** | realizacja na ticketach, aż testy zielone, potem upraszczanie | działający krok | — |
| 7 | **Sprawdź** | przegląd na dwóch osiach równolegle: standardy i zgodność ze specem | raport | — |

Po odbiorze spec idzie do archiwum jako **zamrożony zapis ustaleń** — nie aktualizuje się
go, gdy system się zmienia. Zmiana = nowy krok.

**Test wagi:** drobiazg bez nowych decyzji architektonicznych omija całą ścieżkę — od razu
budowa, test, przegląd. Wielki spec do 20-minutowej roboty to strata, nie staranność.

Pełny opis faz, z uzasadnieniem każdej reguły: [`docs/01-workflow.md`](docs/01-workflow.md).

## Katalog skilli

**Skille procesowe** (napisane pod ten workflow, po polsku) —
szczegóły: [`docs/02-skille-wdrozeniowe.md`](docs/02-skille-wdrozeniowe.md)

| Skill | Faza | Co robi |
|---|---|---|
| [`/workflow`](skills/workflow/SKILL.md) | cały proces | kręgosłup: kolejność faz, bramki, routing do pozostałych skilli |
| [`/init-klienta`](skills/init-klienta/SKILL.md) | 0 | zakłada CLAUDE.md projektu (pamięć) + hook pilnujący bramek |
| [`/analiza`](skills/analiza/SKILL.md) | 1 | fork czyta cały folder projektu → raport: co wiemy, sprzeczności, luki |
| [`/to-questionnaire`](skills/to-questionnaire/SKILL.md) | 1, 5 | zamienia „tego nie wiemy" w jedną ankietę async, bez spotkania |
| [`/hejt`](skills/hejt/SKILL.md) | 3 | krytyka: dyskusja Claude↔Codex → świeży sędzia → tabela propozycji |
| [`/zalacznik`](skills/zalacznik/SKILL.md) | 4 | zakres do umowy, językiem klienta, z kryteriami odbioru, które da się sprawdzić |
| [`/to-spec`](skills/to-spec/SKILL.md) | 2, 5 | zamienia rozmowę w PRD z kryteriami EARS (każde = 1 test) |
| [`/to-tickets`](skills/to-tickets/SKILL.md) | 5 | tnie PRD na zadania „tracer bullet" z jawnymi zależnościami |

**Skille inżynierskie** (baza open-source, po angielsku) —
szczegóły: [`docs/03-skille-inzynierskie.md`](docs/03-skille-inzynierskie.md)

| Skill | Do czego |
|---|---|
| [`/wayfinder`](skills/wayfinder/SKILL.md) | mapa dużego przedsięwzięcia jako tickety decyzyjne — jedna sesja, jeden ticket |
| [`/grilling`](skills/grilling/SKILL.md) | przesłuchanie: jedno pytanie na raz, zawsze z rekomendowaną odpowiedzią |
| [`/grill-with-docs`](skills/grill-with-docs/SKILL.md) | to samo + zapis słownika i decyzji (ADR) w locie |
| [`/domain-modeling`](skills/domain-modeling/SKILL.md) | budowanie języka projektu (CONTEXT.md) i decyzji architektonicznych |
| [`/codebase-design`](skills/codebase-design/SKILL.md) | słownik głębokich modułów: moduł, interfejs, szew, adapter, dźwignia |
| [`/improve-codebase-architecture`](skills/improve-codebase-architecture/SKILL.md) | skan kodu pod refaktory + raport HTML z diagramami przed/po |
| [`/tdd`](skills/tdd/SKILL.md) | red-green-refactor: szwy, antywzorce, kiedy wolno mockować |
| [`/implement`](skills/implement/SKILL.md) | budowa na podstawie specu lub ticketów |
| [`/code-review`](skills/code-review/SKILL.md) | przegląd diffa na dwóch osiach równolegle, bez scalania wyników |
| [`/setup-matt-pocock-skills`](skills/setup-matt-pocock-skills/SKILL.md) | konfiguracja repo: tracker, etykiety triage, dokumenty domenowe |

## Rola pluginu Codex

Drugi model nie jest tu ozdobą — jest **jedynym miejscem, w którym ocena nie pochodzi
od autora ocenianego tekstu**. Claude pisze plan i PRD, więc jego recenzja tych
dokumentów jest recenzją własną.

[`/hejt`](skills/hejt/SKILL.md) używa Codexa dwa razy, w dwóch różnych rolach:

1. **Oponent** (faza dyskusji, max 3 rundy) — zgłasza konkretne zarzuty: luka, ryzyko,
   przerost, nieaktualne narzędzie. Claude jako autor albo przyznaje rację (→ propozycja
   zmiany), albo broni jednym zdaniem (→ punkt sporny).
2. **Sędzia** (nowe wywołanie, **bez historii dyskusji**) — dostaje wyłącznie dokument,
   listy propozycji i sporów, kontekst projektu i **źródła pierwotne = głos klienta**
   (jego transkrypcje i wiadomości; nasze drafty źródłem nie są). Rozstrzyga każdy punkt,
   sprawdza zgodność planu z tym, co klient faktycznie powiedział — z cytatem — i robi
   własny research aktualnych praktyk z linkami.

Codex czyta pliki sam, w sandboksie read-only, więc dostaje ścieżki zamiast wklejek.
Wariantowo przejmuje też budowę ([`references/budowa.md`](skills/workflow/references/budowa.md)):
`codex exec -s workspace-write`, przy czym nie dotyka `.env`, `CLAUDE.md` ani plików
z sekretami, a Claude przechodzi wtedy do recenzji diffa.

**Instalacja:**

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
```

Bez pluginu skille działają dalej — `/hejt` wykonuje obie fazy sam i **jawnie oznacza
wynik jako „ocena własna, nie niezależna"**, zamiast udawać niezależność.

**Operacyjnie:** ciężkie wywołania idą w `--background`. Zadanie na kilka minut myślenia
nie mieści się w timeoucie Bash, a zabity proces zostawia blokadę „task still running",
której plugin nie sprząta (zdejmuje ją `codex-companion.mjs cancel`).

## Wzorce projektowania skilli

Rzeczy, które okazały się ważniejsze od treści samych instrukcji:

- **Krótki kręgosłup, szczegóły w referencjach.** `SKILL.md` trzyma kolejność i bramki;
  reguły fazy leżą w `references/*.md` i są doczytywane przy wejściu w fazę. Regulamin
  wpakowany do jednego pliku wypada z kontekstu w długiej sesji.
- **Reguła, która musi zadziałać, nie może być zdaniem w prompcie.** Stąd hook: mechaniczny,
  odpalany przy każdym zapisie, obojętny na to, ile sesja pamięta.
- **Świeży kontekst jako narzędzie.** Fork do czytania materiałów, świeże wywołanie do
  sądzenia, osobne sub-agenty do dwóch osi przeglądu. Za każdym razem chodzi o to samo:
  nie dziedziczyć ramy, którą się właśnie ocenia.
- **Bramka zamiast zaufania.** Model nie zna granicy między „poprawiłem drobiazg"
  a „zmieniłem ustalenie". Granicę rysuje proces: zatwierdzony dokument zmienia się
  wyłącznie przez tabelę ZA/PRZECIW i wybór człowieka.
- **Widoczna niewiedza.** Sekcja „Nierozstrzygnięte pytania" z właścicielem przy każdym
  pytaniu; `[DO POTWIERDZENIA → pyt. N]` w treści; „brak" wpisywane jawnie, bo pusta
  sekcja to twierdzenie, nie przeoczenie.
- **Instrukcja mówi, czego NIE robić, i podaje powód.** Reguła bez uzasadnienia jest
  pierwszą, którą model zracjonalizuje — więc każda ma przy sobie zdanie „bo inaczej…".

## Szybki start

```bash
# 1. Skille do projektu (albo globalnie: ~/.claude/skills/)
cp -R skills/* /ścieżka/do/projektu/.claude/skills/

# 2. Opcjonalnie: niezależny recenzent
#    /plugin marketplace add openai/codex-plugin-cc
#    /plugin install codex@openai-codex

# 3. W Claude Code, w folderze projektu:
#    /init-klienta <ścieżka do folderu projektu>
#    /workflow
```

Dalej mówisz do agenta normalnym językiem — „przeanalizuj materiały", „zrób plan pod
wycenę", „skrytykuj to", „potnij na zadania". Nazwy skilli są po stronie agenta.
Szczegóły: [`docs/05-instalacja-i-konfiguracja.md`](docs/05-instalacja-i-konfiguracja.md).

## Struktura folderów projektu

Skille zakładają porządek pilnowany mechanicznie, nie z pamięci:

1. **Numerowane podfoldery, zero luźnych plików.** Prefiks (`00_`, `01_`…) wymusza
   kolejność; po usunięciu czegoś numeracja jest przenumerowywana, żeby nie było dziur.
2. **Jedno archiwum.** Dokument nieaktualny idzie do `99_archiwum/` **od razu, w tej samej
   sesji**, z dopiskiem `_STARY`. Foldery robocze pokazują wyłącznie aktualny stan — inaczej
   każda kolejna sesja zaczyna się od śledztwa, która wersja obowiązuje.
3. **Jedno miejsce na linki.** Wszystkie adresy zewnętrzne w jednej tabeli w CLAUDE.md
   projektu; inne pliki *odsyłają*, nigdy nie kopiują URL-a.

Pełny wzorzec, z anatomią folderu faza po fazie: [`docs/04-struktura-folderow.md`](docs/04-struktura-folderow.md).

## Co dopasować pod siebie

| Miejsce | Co zmienić |
|---|---|
| `skills/analiza`, `skills/to-spec`, `skills/to-questionnaire` | ścieżki zapisu (`<folder projektu>/<etap>/03_planowanie/`) |
| `skills/hejt`, `skills/workflow/references/budowa.md` | drugi model — domyślnie Codex; warunek merytoryczny: sędzia dostaje świeży kontekst |
| `skills/zalacznik` | miejsce docelowe umowy (domyślnie dokument w chmurze, nie repo) |
| `skills/init-klienta` | szablon CLAUDE.md i treść hooka |
| `skills/setup-matt-pocock-skills` | tracker i etykiety triage |

Skille procesowe są po polsku (produkują dokumenty dla polskich klientów), inżynierskie
po angielsku. Instrukcja dla modelu nie musi być w języku dokumentu, który powstaje —
można ujednolicić w obie strony.

## Pochodzenie i licencja

Skille inżynierskie pochodzą z open-source'owego zestawu
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — `wayfinder`, `grilling`,
`grill-with-docs`, `to-spec`, `to-tickets`, `to-questionnaire`, `tdd`, `implement`,
`code-review`, `codebase-design`, `domain-modeling`, `improve-codebase-architecture`,
`setup-matt-pocock-skills`. Zmodyfikowane zostały głównie `to-spec`, `to-tickets`
i `to-questionnaire`: doszły polskie sekcje, zasada zera założeń, routing pytań do klienta
i drabinka cel → metryka → kryterium odbioru.

Skille `workflow`, `init-klienta`, `analiza`, `hejt` i `zalacznik` powstały pod ten proces.

Licencja: MIT (patrz [`LICENSE`](LICENSE)).
