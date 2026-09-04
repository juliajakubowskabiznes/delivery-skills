---
name: workflow
description: >
  Główny workflow pracy nad wdrożeniem: poznaj → zaplanuj pod wycenę → skrytykuj →
  umowa → PRD + taski + testy → buduj → sprawdź. Używaj gdy użytkownik mówi: "odpal
  workflow", "przejdźmy cały proces", "zbuduj X od początku do końca", "działamy
  według workflow".
---

# Workflow: od pomysłu do zbudowanego

## ŚCIĄGA — który skill kiedy

| Faza po ludzku | Skill | Efekt |
|---|---|---|
| 0. Setup (raz na klienta) | `/init-klienta` · `/setup-matt-pocock-skills` | CLAUDE.md klienta · tracker |
| 1. POZNAJ | `/analiza` → `/grill-with-docs` | raport z materiałów → przesłuchanie użytkownika (+ADR-y); pytania, na które zna odpowiedź tylko klient → `/to-questionnaire` |
| 2. ZAPLANUJ POD WYCENĘ | `/wayfinder` (duży system) · `/to-spec` (mały) | DUŻY system: spec całości = ryzyko + wykonalność + PODZIAŁ NA KROKI WDROŻENIA (każdy krok dostanie własny PRD w fazie 5) · MAŁY system: od razu PRD |
| 3. SKRYTYKUJ | `/hejt` | dyskusja + sędzia → tabela → użytkownik wybiera |
| 4. UMOWA | `/zalacznik` + umowa | podpis + płatność z góry PRZED dalszą robotą |
| 5. DOMKNIJ I POTNIJ (po umowie) | domknięcie PRD z klientem → `/to-tickets` + testy z EARS (+ hejt delty) | PRD finalny + taski z zależnościami + testy przed budową |
| 6. BUDUJ | `/implement` | na ticketach, aż testy zielone |
| 7. SPRAWDŹ | `/code-review` | standards + spec |

Silniki i narzędzia na żądanie (nie wołaj wprost bez potrzeby): `/grilling` (jądro
przesłuchania — używane przez grill-with-docs i wayfinder), `/domain-modeling`,
`/codebase-design`, `/improve-codebase-architecture`.

Kolejność stała; po każdej bramce 🚧 czekaj na użytkownika. **Wchodząc w fazę, przeczytaj
jej plik w `references/`** — tam żyją szczegółowe reguły (celowo osobno: krótki
kręgosłup przetrwa długą sesję, gruby regulamin tonie w kontekście).

**Test wagi:** drobiazg — bez nowych decyzji architektonicznych — omija pełną ścieżkę:
od razu budowa + test + code-review. Wątpliwość → zapytaj użytkownika. Wielki spec do
20-minutowej roboty to strata, nie staranność.

**Z użytkownikiem rozmawiasz wg `references/tryb-pracy.md`** (jeden wątek do decyzji,
dane eksperckie przy pytaniach, zero spotkań operacyjnych) — we WSZYSTKICH fazach.

**Bramka naniesień (każda faza):** wyniki hejtu i researchu NIE zmieniają planu/PRD
same z siebie. Najpierw krótka tabela ZA/PRZECIW z rekomendacją → użytkownik wybiera →
nanosisz TYLKO wybrane. „To tylko mechanika/techniczne" NIE omija bramki — dotyczy
każdej zmiany zatwierdzonego dokumentu.

**Zero założeń (każda faza):** luka w wiedzy = pytanie do klienta
(`/to-questionnaire`) albo do użytkownika — NIGDY robocza hipoteza wpisana po cichu.

**Sygnalizacja fazy (żeby użytkownik zawsze wiedział, gdzie jest):** pracując w
workflow, zaczynaj odpowiedzi od znacznika `[WORKFLOW faza N/7 — NAZWA]` + jedno
zdanie: co się właśnie dzieje i jaka jest najbliższa bramka. Przy wznowieniu sesji
NAJPIERW odczytaj stan (CLAUDE.md / kontekst klienta / handoff / tracker) i zamelduj
fazę, zanim cokolwiek zrobisz. Użytkownik NIE musi znać nazw skilli — mówi po ludzku,
routing jest po stronie agenta.

## 0. SETUP (raz na klienta) → `references/setup.md`

`/init-klienta` gdy brak CLAUDE.md klienta · `/setup-matt-pocock-skills` gdy brak
trackera · zastane pliki analizy technicznej ze sprzedaży → rozdziel wg setup.md.

## 1. POZNAJ 🚧

`/analiza <folder>` (fork czyta materiały, wraca z raportem) → `/grill-with-docs`:
zaczynaj od WHY (po co, pieniądze/czas klienta, miara sukcesu), potem CO/JAK —
architektura przychodzi do użytkownika jako propozycje z opcjami, nie pytania "jak zbudować".
Materiały mogą kłamać → `references/wiarygodnosc.md`. Odpowiedź zna tylko klient →
`/to-questionnaire`. Grill kończy użytkownik ("wystarczy").
**BRAMKA: użytkownik potwierdza zrozumienie.**

## 2. ZAPLANUJ POD WYCENĘ 🚧 → `references/plan.md`

Duży system: `/wayfinder` (plan całości jako mapa decyzji). Mały system: `/to-spec`.

**Spec całości (duży system) robi DWIE rzeczy:** (1) weryfikuje ryzyko
i wykonalność całości, (2) **ROZBIJA system na nazwane KROKI WDROŻENIA**.
Zasady cięcia na kroki:
- **Wielkość:** każdy krok = samodzielna wartość dla klienta — da się go osobno
  sprzedać, odebrać i zafakturować. NIE za małe (klient płaci za każdy krok;
  mikrokroki = mikrofaktury i zmęczenie klienta), nie za duże (jeden wielki
  odbiór na końcu = całe ryzyko w jednym miejscu).
- **Łączenie:** dla każdego kroku spisz zależności (czego wymaga od poprzednich)
  i punkt styku (co zostawia następnym: dane, interfejs, konwencje). Krok
  późniejszy NIE może psuć odebranego wcześniejszego — jeśli grzebie w tym samym
  mechanizmie, zaplanuj izolację (osobna gałąź, backup, test regresyjny).
- **Wykonalność per krok:** każdy krok z osobna wykonalny i weryfikowalny
  (własne kryteria odbioru); element najbardziej ryzykowny całości dowodzony
  NAJWCZEŚNIEJ (najlepiej dowodem technicznym jeszcze przed umową).
- **Kolejność:** najpierw kroki odblokowujące i zdejmujące ryzyko, potem
  rozszerzenia; MVP przed wodotryskami.

Kroki z tego podziału to jednostki faz 4-5: umowa może obejmować krok/etap,
a **każdy krok dostaje WŁASNY PRD dopiero w fazie 5** (nie piszemy PRD wszystkich
kroków z góry). Mały system = jeden krok → `/to-spec` może od razu zejść do PRD.

**Szkielet specu całości (sprawdzony w praktyce — sekcje w tej kolejności):**
1. Co budujemy i po co — liczby klienta (godziny/pieniądze) + JEDNA zasada
   spinająca architekturę całości (np. „jeden rejestr pisze, kroki tylko czytają"
   — dzięki niej kroki kupuje się i odbiera osobno, a system zostaje całością)
2. Obawy/cele klienta JEGO SŁOWAMI (cytaty ze źródeł) → odpowiedź systemu na każdą
3. Jak to działa — jeden obrazek (całość na pół strony)
4. Dane/schemat całości — co jest źródłem prawdy, kto pisze, kto czyta
5. Mapa kroków — tabela: co klient dostaje · SEDNO kryterium odbioru · cena · czas
   (pełne kryteria dopiero w PRD kroku; seedy kryteriów zbierane per krok obok planu)
6. Dlaczego ta kolejność i jak kroki się łączą (zależności + punkty styku)
7. Czego świadomie NIE budujemy (i dlaczego — z decyzją/ADR przy każdym)
8. Bezpieczeństwo/ryzyka — każde z DOWODEM (test, incydent, pomiar), nie deklaracją
9. Zależności zewnętrzne z fallbackami — żadna nie może blokować podpisu umowy
10. Hierarchia dokumentów: umowa+załącznik > PRD > plan > notatki; odbiór kroku
    WYŁĄCZNIE wg PRD kroku + załącznika (plan = mapa, nie podstawa odbioru)

**Głębokość: GŁĘBOKO w ryzyko i wykonalność, płytko
w wykonanie.** Plan przedsprzedażowy MUSI zweryfikować: **wykonalność dowodem**
(test na realnych danych klienta, nie deklaracja), **rejestr ryzyk** (co może zabić
projekt i czym to sprawdzimy), przypadki brzegowe widoczne w materiałach, podstawę
wyceny. NIE schodzi w poziom wykonawczy krok-po-kroku (kontrakty danych, kolejność
budowy, tie-breakery) — to PRD w kroku 5. W małym systemie `/to-spec` może od razu
zejść do pełnego PRD, jeśli wycena tego wymaga (praktyka: głęboki spec + dowód
wykonalności na danych klienta przed wyceną podnosi cenę, bo sprzedajesz pewność,
nie obietnicę). Research każdej decyzji wg `references/research.md`
(równoległe sub-agenty, min. 2 źródła z linkami).

**Słownik poziomów (żeby nazwy nie myliły):** poziom 1 =
„spec ogólny" (mapa całości, tylko DUŻE systemy); poziom 2 = „PRD"
(dokument wykonawczy z EARS — to robi `/to-spec`, niezależnie od tego, że plik
nazywa się „spec"); poziom 3 = taski (krok 5, `/to-tickets`). Mały system
(jeden moduł) POMIJA poziom 1 — rolę mapy pełni podział na etapy z oferty.
**BRAMKA: użytkownik zatwierdza plan.**

## 3. SKRYTYKUJ 🚧

`/hejt <dokument>` na tym, co idzie do wyceny/oferty — dyskusja GPT↔Claude
(dokument nietykalny) → świeży sędzia → krótka tabela propozycji.
**BRAMKA: użytkownik wybiera zmiany — dopiero wtedy nanosisz.**
Po naniesieniu wybranych zmian **przejdź z użytkownikiem przez finalny dokument sekcja po
sekcji** (krótko, jego językiem, jeden wątek na raz) i domknij otwarte [DO DOPYTANIA].

## 4. UMOWA 🚧

Oferta wg wyceny → po "tak" klienta: umowa + `/zalacznik <PRD>` (7 sekcji językiem
klienta; wysyłane RAZEM z umową). Płatność z góry PRZED dalszą robotą.
**BRAMKA: użytkownik akceptuje przed wysyłką.**

## 5. DOMKNIJ I POTNIJ (po podpisaniu i wpłacie) 🚧

Płatna część planowania — dopiero teraz szczegóły, których wycena nie potrzebowała.
**Duży system: fazę 5 robimy PER KROK WDROŻENIA z planu całości** (PRD kroku →
taski kroku → testy kroku → budowa kroku), nie hurtem dla wszystkich kroków z góry:

a) **PRD kroku + domknięcie z klientem** — `/to-spec` dla bieżącego kroku (mały
   system: delta istniejącego PRD, nie nowy dokument); rundy discovery przewidziane
   w umowie (np. mapa reguł z zamrożeniem po N rundach), odpowiedzi na pytania
   odłożone „na po umowie".
b) **`/to-tickets <PRD>`** — tracer-bullet tickety z blocking edges w trackerze.
   Tniemy z PRD ZATWIERDZONEGO. Ticket, którego PRD nie rozstrzyga = status
   **blocked + konkretne pytanie** (klient → `/to-questionnaire`, użytkownik → bramka),
   NIGDY robocza hipoteza.
c) **Testy z EARS** — ZANIM powstanie workflow; kryteria w PRD pisane tak, by każde
   konwertowało się 1:1 na test; sekcja „Decyzje testowe" mówi CO testować. Dane
   referencyjne klienta, sumy kontrolne, spreparowane błędy (mechanizm bezpieczeństwa
   nietestowany na złych danych = niedziałający). W n8n: wbudowane Evaluations;
   kod → `/tdd` (red-green-refactor); bug → do datasetu jako regresja.
d) **Hejt delty** — tylko jeśli PRD istotnie się zmienił po umowie (nowe decyzje
   architektoniczne); krótki, na delcie, nie od zera.
**BRAMKA: użytkownik zatwierdza tickety przed budową.**

## 6. BUDUJ → `references/budowa.md`

`/implement` na ticketach z 5b, aż testy z 5c zielone, potem uprość. Wariant
„buduje GPT" — patrz plik.

## 7. SPRAWDŹ

`/code-review` na gotowym; uwagi krytyczne poprawiane od razu.

## 8. PO ODBIORZE — spec zamrożony

Spec i załącznik → archiwum jako zapis umówionego zakresu (dowód przy sporze); NIE są
aktualizowane, gdy system u klienta się zmienia — źródło prawdy = działający system +
CLAUDE.md klienta. Zmiana = NOWY krok. Lekcje z budowy → wzorce techniczne CLAUDE.md.

---
Pozostałe skille (improve-codebase-architecture, domain-modeling, wayfinder
research…) to narzędzia na żądanie — poza łańcuchem.
