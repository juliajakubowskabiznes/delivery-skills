# Delivery Skills — workflow wdrożeniowy dla agenta AI

Zestaw **18 skilli** dla Claude Code, które prowadzą projekt wdrożeniowy od pierwszej
rozmowy z klientem aż po odbiór gotowego systemu. Skill to instrukcja, którą agent
wczytuje i wykonuje krok po kroku — zamiast improwizować, robi za każdym razem to samo,
w tej samej kolejności, z tymi samymi hamulcami.

**Do czego to służy:** żeby wdrożenie nie rozjechało się na żadnym z pięciu klasycznych
sposobów — źle zrozumiany proces klienta, plan pisany z głowy zamiast z danych, zakres
którego nie da się odebrać, budowa bez testów, i cicha zmiana ustaleń w trakcie.

---

## Spis treści

- [Dla kogo to jest](#dla-kogo-to-jest)
- [Zrozum w 2 minuty (bez technikaliów)](#zrozum-w-2-minuty-bez-technikaliów)
- [Szybki start](#szybki-start)
- [Workflow — 7 faz](#workflow--7-faz)
- [Katalog skilli](#katalog-skilli)
- [Struktura folderu klienta](#struktura-folderu-klienta)
- [Dla technicznych](#dla-technicznych)
- [Co dopasować pod siebie](#co-dopasować-pod-siebie)
- [Pochodzenie i licencja](#pochodzenie-i-licencja)

Głębsze opisy: [`docs/`](docs/) — [workflow](docs/01-workflow.md) ·
[skille wdrożeniowe](docs/02-skille-wdrozeniowe.md) ·
[skille inżynierskie](docs/03-skille-inzynierskie.md) ·
[struktura folderów](docs/04-struktura-folderow.md) ·
[instalacja i konfiguracja](docs/05-instalacja-i-konfiguracja.md)

---

## Dla kogo to jest

Dla jednej osoby albo małego zespołu, który **sprzedaje i wdraża automatyzacje / systemy
u klientów** — i pracuje przy tym z agentem AI (Claude Code). Sprawdza się tam, gdzie:

- projekt jest sprzedawany etapami (klient płaci za krok, nie za całość z góry),
- pomiędzy „klient chce" a „system działa" jest długa droga przez ustalenia,
- ta sama osoba jest sprzedawcą, architektem i wykonawcą — i nie ma czasu pilnować
  wszystkiego z głowy.

Nazewnictwo w skillach: **„użytkownik"** = osoba prowadząca wdrożenie (Ty).
**„klient"** = firma, dla której budujesz.

## Zrozum w 2 minuty (bez technikaliów)

Wyobraź sobie, że zatrudniasz bardzo szybkiego, ale nadgorliwego juniora. Zrobi wszystko,
o co poprosisz — i jeszcze dziesięć rzeczy, o które nie prosiłeś. Wymyśli brakujące
informacje, żeby nie przerywać pracy. Zmieni ustalony dokument, „bo tak będzie lepiej".

Te skille to jego regulamin. Trzy zasady, które przewijają się przez wszystkie:

1. **Bramki 🚧** — po każdej fazie agent zatrzymuje się i czeka na Twoją zgodę.
   Nie idzie dalej sam.
2. **Zero założeń** — jeśli czegoś nie wie, to jest to *pytanie* (do Ciebie albo do
   klienta), nigdy cicho wpisana hipoteza. Brak informacji ma być widoczny.
3. **Bramka naniesień** — krytyka i research produkują *propozycje*, nie zmiany.
   Zatwierdzony dokument zmienia się dopiero po Twoim wyborze z tabeli ZA/PRZECIW.

Do tego jeden nawyk, który zmienia ekonomię projektu: **głęboko planujesz przed wyceną,
płytko przed budową**. Przed ofertą sprawdzasz, czy to w ogóle wykonalne — najlepiej
dowodem na prawdziwych danych klienta. Reszta szczegółów powstaje dopiero po podpisaniu
umowy i wpłacie, bo to płatna część pracy.

**Co realnie z tego masz:** klient dostaje zakres, który da się odebrać (kryteria odbioru
są sprawdzalnym testem, nie „działa poprawnie"); Ty masz dokument, który broni Cię przy
sporze; a agent nie dopisuje po cichu rzeczy, których nie ustaliliście.

## Szybki start

```bash
# 1. Skopiuj skille do projektu (albo globalnie do ~/.claude/skills/)
cp -R skills/* /ścieżka/do/projektu/.claude/skills/

# 2. Odpal Claude Code w folderze projektu i zainicjuj klienta
/init-klienta <ścieżka do folderu klienta>

# 3. Uruchom cały proces
/workflow
```

Od tego momentu mówisz do agenta **normalnym językiem** — „przeanalizuj materiały",
„zrób plan pod wycenę", „skrytykuj to", „potnij na zadania". Nazwy skilli są po stronie
agenta; on sam routuje. Szczegóły instalacji: [`docs/05-instalacja-i-konfiguracja.md`](docs/05-instalacja-i-konfiguracja.md).

## Workflow — 7 faz

Jedna ścieżka, stała kolejność, bramka po każdej fazie. Agent na każdym kroku melduje,
gdzie jesteście: `[WORKFLOW faza N/7 — NAZWA]`.

| # | Faza | Co się dzieje | Efekt | Bramka |
|---|---|---|---|---|
| 0 | **Setup** | raz na klienta: CLAUDE.md klienta + tracker zadań | pamięć projektu | — |
| 1 | **Poznaj** | agent czyta wszystkie materiały, potem przesłuchuje Ciebie; czego nie wiecie oboje → ankieta do klienta | raport + decyzje + pytania do klienta | 🚧 potwierdzasz zrozumienie |
| 2 | **Zaplanuj pod wycenę** | plan całości: ryzyko, wykonalność (dowodem), podział na **kroki wdrożenia**, które da się osobno sprzedać i odebrać | plan / spec całości | 🚧 zatwierdzasz plan |
| 3 | **Skrytykuj** | dokument idzie na dyskusję dwóch modeli, potem świeży „sędzia" rozstrzyga spory i robi własny research | tabela propozycji + werdykt | 🚧 wybierasz, co wprowadzić |
| 4 | **Umowa** | oferta → po „tak": umowa + załącznik z zakresem językiem klienta | podpis + płatność z góry | 🚧 akceptujesz przed wysyłką |
| 5 | **Domknij i potnij** | *dopiero po wpłacie*: PRD kroku, cięcie na zadania z zależnościami, testy pisane **przed** budową | PRD + tickety + testy | 🚧 zatwierdzasz tickety |
| 6 | **Buduj** | realizacja na ticketach, aż testy świecą na zielono, potem upraszczanie | działający krok | — |
| 7 | **Sprawdź** | przegląd na dwóch osiach: standardy kodu i zgodność ze specyfikacją | raport | — |

Po odbiorze spec i załącznik idą do archiwum jako **zamrożony zapis ustaleń** — nie
aktualizujesz ich, gdy system się zmienia. Zmiana = nowy krok = nowa umowa.

**Test wagi:** drobiazg bez nowych decyzji architektonicznych omija całą ścieżkę — od
razu budowa, test i przegląd. Wielki spec do 20-minutowej roboty to strata, nie staranność.

Pełny opis każdej fazy — co dokładnie powstaje, jakie są pułapki i dlaczego kolejność
jest właśnie taka: [`docs/01-workflow.md`](docs/01-workflow.md).

## Katalog skilli

**Skille wdrożeniowe** (napisane pod ten proces, po polsku) —
szczegóły: [`docs/02-skille-wdrozeniowe.md`](docs/02-skille-wdrozeniowe.md)

| Skill | Faza | Co robi |
|---|---|---|
| [`/workflow`](skills/workflow/SKILL.md) | cały proces | kręgosłup: kolejność faz, bramki, routing do pozostałych skilli |
| [`/init-klienta`](skills/init-klienta/SKILL.md) | 0 | zakłada CLAUDE.md klienta (pamięć projektu) + hook pilnujący bramek |
| [`/analiza`](skills/analiza/SKILL.md) | 1 | czyta cały folder klienta na świeżym kontekście → raport: co wiemy, sprzeczności, luki |
| [`/to-questionnaire`](skills/to-questionnaire/SKILL.md) | 1, 5 | zamienia „tego nie wiemy" w jedną ankietę do klienta (async, bez spotkania) |
| [`/hejt`](skills/hejt/SKILL.md) | 3 | krytyka dokumentu: dyskusja GPT↔Claude → świeży sędzia → tabela propozycji |
| [`/zalacznik`](skills/zalacznik/SKILL.md) | 4 | zakres do umowy w 7 sekcjach, językiem klienta, z kryteriami odbioru |
| [`/to-spec`](skills/to-spec/SKILL.md) | 2, 5 | zamienia rozmowę w PRD z kryteriami w formacie EARS (każde = 1 test) |
| [`/to-tickets`](skills/to-tickets/SKILL.md) | 5 | tnie PRD na zadania „tracer bullet" z jawnymi zależnościami |

**Skille inżynierskie** (baza open-source, po angielsku) —
szczegóły: [`docs/03-skille-inzynierskie.md`](docs/03-skille-inzynierskie.md)

| Skill | Do czego |
|---|---|
| [`/wayfinder`](skills/wayfinder/SKILL.md) | mapa dużego przedsięwzięcia jako tickety decyzyjne — jeden na sesję |
| [`/grilling`](skills/grilling/SKILL.md) | przesłuchanie: jedno pytanie na raz, z rekomendowaną odpowiedzią |
| [`/grill-with-docs`](skills/grill-with-docs/SKILL.md) | to samo + zapis słownika pojęć i decyzji (ADR) w locie |
| [`/domain-modeling`](skills/domain-modeling/SKILL.md) | budowanie języka projektu (CONTEXT.md) i decyzji architektonicznych (ADR) |
| [`/codebase-design`](skills/codebase-design/SKILL.md) | słownik projektowania „głębokich modułów" — moduł, interfejs, szew, dźwignia |
| [`/improve-codebase-architecture`](skills/improve-codebase-architecture/SKILL.md) | skan kodu pod kątem refaktorów + raport HTML z diagramami |
| [`/tdd`](skills/tdd/SKILL.md) | red-green-refactor: co jest dobrym testem, gdzie testować, czego nie mockować |
| [`/implement`](skills/implement/SKILL.md) | budowa na podstawie specu/ticketów |
| [`/code-review`](skills/code-review/SKILL.md) | przegląd diffa na dwóch osiach równolegle: standardy i zgodność ze specem |
| [`/setup-matt-pocock-skills`](skills/setup-matt-pocock-skills/SKILL.md) | konfiguracja repo pod te skille: tracker, etykiety, dokumenty domenowe |

## Struktura folderu klienta

Skille zakładają, że każdy klient ma własny folder, a w nim **porządek pilnowany
mechanicznie**, nie z pamięci. Trzy zasady w skrócie:

1. **Numerowane podfoldery, zero luźnych plików.** Każdy podfolder ma prefiks
   (`00_`, `01_`, `02_`…) wymuszający kolejność. Plik nie ląduje „na razie tutaj" —
   albo pasuje do istniejącego podfolderu, albo powstaje nowy, ponumerowany.
2. **Jedno archiwum na klienta.** Dokument nieaktualny (zastąpiony, przeterminowany,
   sprzeczny z decyzjami) idzie do `99_archiwum/` **od razu, w tej samej sesji** —
   z dopiskiem `_STARY` w nazwie. Nic nie kasujemy; foldery robocze pokazują
   wyłącznie aktualny stan.
3. **Jedno miejsce na linki.** Wszystkie adresy zewnętrzne (arkusze, dokumenty umowy,
   instancje narzędzi, nagrania) żyją w jednej tabeli w CLAUDE.md klienta. Inne pliki
   *odsyłają* do niej, nigdy nie kopiują URL-a.

Pełny wzorzec — jak wygląda folder faza po fazie, gdzie ląduje który artefakt, jak
działa hook pilnujący poziomu dokumentu i dlaczego wiadomości do klienta zapisujemy
jako `.txt`: [`docs/04-struktura-folderow.md`](docs/04-struktura-folderow.md).

## Dla technicznych

**Czym jest skill.** Folder z plikiem `SKILL.md`: frontmatter YAML (`name`,
`description`, opcjonalnie `context`, `background`, `disable-model-invocation`) plus
treść w Markdownie — instrukcja wykonawcza dla modelu. Pliki obok (`references/*.md`,
`CONTEXT-FORMAT.md`, `HTML-REPORT.md`) są doczytywane dopiero wtedy, gdy skill ich
potrzebuje. To celowe: krótki kręgosłup przeżyje długą sesję, gruby regulamin utonąłby
w kontekście.

**Mechanizmy, na których to stoi:**

- **Fork kontekstu** — `/analiza` ma `context: fork`: czyta materiały bez historii
  rozmowy, więc nie dziedziczy naszych wcześniejszych założeń. Ocenia to, co jest na
  dysku, a nie to, w co już uwierzyliśmy.
- **Drugi model jako recenzent** — `/hejt` woła GPT przez `codex:rescue` (fallback:
  `codex exec`). Faza sędziego to **świeże wywołanie bez historii dyskusji**, żeby
  recenzent nie odziedziczył ramy autora. Ciężkie wywołania zawsze w `--background` —
  zadanie na kilka minut nie mieści się w timeoucie Bash, a zabity proces zostawia
  blokadę.
- **Sub-agenty równoległe** — research (jeden agent na pytanie badawcze, min. 2 źródła
  z linkami), `/code-review` (dwie osie w osobnych kontekstach, żeby się nie zanieczyszczały),
  `design-it-twice` (3+ warianty interfejsu naraz).
- **Hook jako twarda pamięć** — `.claude/settings.json` w folderze klienta, `PostToolUse`
  na `Write|Edit`: wychodzi z kodem 2 przy zapisie do spec/plan/PRD i CLAUDE.md,
  przypominając o bramce. Działa niezależnie od tego, co sesja pamięta.
- **Issue tracker jest wymienny** — GitHub, GitLab albo lokalny markdown w `.scratch/`.
  Konfiguruje go `/setup-matt-pocock-skills`, zapisując do `docs/agents/issue-tracker.md`;
  pozostałe skille czytają ten plik zamiast zakładać platformę.
- **EARS** — kryteria akceptacji pisane wzorcami (`GDY <zdarzenie>, system MUSI X`;
  `JEŚLI <błąd>, TO system MUSI X`), bo każde takie zdanie konwertuje się 1:1 na test.
  Twarda reguła: każdy krok czytający dane przez AI musi mieć kryterium typu „co, gdy
  odczyt zawiedzie" — cicha błędna odpowiedź to najgorsza klasa awarii.

**Zasada eskalacji złożoności** (obowiązuje w każdej decyzji technicznej):
stała reguła → deterministyczny workflow → krok z AI → agent AI. Sięgnięcie po wyższy
poziom wymaga jednego zdania uzasadnienia, dlaczego niższy nie wystarcza.

## Co dopasować pod siebie

Skille są napisane uniwersalnie, ale kilka rzeczy zależy od Twojego setupu:

| Miejsce | Co zmienić |
|---|---|
| `skills/analiza`, `skills/to-spec`, `skills/to-questionnaire` | ścieżki folderu planowania (`<folder klienta>/<wdrożenie>/03_planowanie/`) na własne |
| `skills/hejt`, `skills/workflow/references/budowa.md` | drugi model — domyślnie Codex/GPT; bez niego skille działają w trybie „ocena własna" |
| `skills/zalacznik` | miejsce docelowe umowy (domyślnie dokument w chmurze, nie repo) |
| `skills/init-klienta` | szablon CLAUDE.md klienta i sekcje, które chcesz wymuszać |
| `skills/setup-matt-pocock-skills` | wybór trackera i etykiet triage |

Język: skille wdrożeniowe są po polsku (dokumenty idą do polskich klientów), inżynierskie
po angielsku. Można to ujednolicić — treść instrukcji nie zależy od języka dokumentów,
które produkują.

## Pochodzenie i licencja

Część skilli inżynierskich pochodzi z open-source'owego zestawu
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — `wayfinder`,
`grilling`, `grill-with-docs`, `to-spec`, `to-tickets`, `to-questionnaire`, `tdd`,
`implement`, `code-review`, `codebase-design`, `domain-modeling`,
`improve-codebase-architecture`, `setup-matt-pocock-skills`. Zostały tu zmodyfikowane —
głównie `to-spec`, `to-tickets` i `to-questionnaire`, które dostały polskie sekcje,
zasadę zera założeń i routing pytań do klienta.

Skille `workflow`, `init-klienta`, `analiza`, `hejt` i `zalacznik` powstały na potrzeby
tego procesu.

Licencja: MIT (patrz [`LICENSE`](LICENSE)) — uzupełnij pole właściciela praw przed
ewentualną publikacją.
