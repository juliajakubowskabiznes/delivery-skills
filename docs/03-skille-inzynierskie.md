# Skille inżynierskie (poboczne) — szczegółowo

Dziesięć skilli, które nie są częścią głównego łańcucha wdrożeniowego, ale są przez
niego wołane albo używane osobno. Baza pochodzi z open-source'owego zestawu
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT); oryginały są po
angielsku i takie zostały.

Podział na trzy grupy: **rozmowa i decyzje**, **kod**, **konfiguracja**.

---

# Rozmowa i decyzje

## `/grilling` — jądro przesłuchania

**Co robi:** przesłuchuje człowieka bez litości na temat planu, decyzji albo pomysłu.
Cztery reguły, które decydują o jakości:

1. **Jedno pytanie na raz**, z czekaniem na odpowiedź. Kilka pytań naraz jest
   dezorientujące i kończy się odpowiedzią na jedno z nich.
2. **Do każdego pytania dołącza własną rekomendowaną odpowiedź.** Człowiek reaguje na
   propozycję zamiast produkować odpowiedź z niczego — to szybsze i daje lepszy materiał.
3. **Fakty sprawdza sam.** Co da się ustalić z plików czy narzędzi, ustala; do człowieka
   idą wyłącznie *decyzje*.
4. **Nie działa, dopóki nie ma potwierdzenia**, że rozumienie jest wspólne.

Schodzi po drzewie decyzyjnym gałąź po gałęzi, rozwiązując zależności między decyzjami
pojedynczo.

**Używany przez:** `/grill-with-docs`, `/wayfinder`, `/improve-codebase-architecture`.
Rzadko woła się go wprost.

## `/grill-with-docs` — przesłuchanie z zapisem

Uruchamia `/grilling` razem z `/domain-modeling`: rozmowa toczy się normalnie, ale
ustalone pojęcia lądują w słowniku (`CONTEXT.md`), a decyzje wagi architektonicznej —
w ADR-ach. **W locie, nie hurtem na koniec.** To jest wersja używana w fazie 1 workflow.

## `/domain-modeling` — język projektu i decyzje

**Co robi:** aktywnie buduje i ostrzy model domenowy. To nie jest „czytanie CONTEXT.md"
(to robi każdy skill), tylko *zmienianie* modelu.

Cztery zachowania w trakcie sesji:

- **Konfrontuje ze słownikiem** — „twój słownik definiuje »anulowanie« jako X, ale mówisz
  o Y, które to jest?".
- **Ostrzy mgliste słowa** — „mówisz »konto«, masz na myśli Klienta czy Użytkownika?
  To dwie różne rzeczy".
- **Testuje scenariuszami** — wymyśla przypadki brzegowe, żeby wymusić precyzję na
  granicach pojęć.
- **Konfrontuje z kodem** — jeśli kod robi co innego, niż mówi człowiek, wykłada tę
  sprzeczność na stół.

**CONTEXT.md** to słownik i nic więcej — zero szczegółów implementacyjnych. Każde hasło:
jedno-dwa zdania, czym rzecz **jest** (nie co robi), plus lista `_Avoid_` z synonimami,
których nie używamy. Format: [`CONTEXT-FORMAT.md`](../skills/domain-modeling/CONTEXT-FORMAT.md).

**ADR** (zapis decyzji architektonicznej) proponowany jest **oszczędnie** — tylko gdy
zachodzą wszystkie trzy warunki: decyzja jest trudna do odwrócenia, byłaby zaskakująca
bez kontekstu, i wynikła z realnego kompromisu. Może mieć jeden akapit; wartość jest
w zapisaniu **że** i **dlaczego**, nie w wypełnieniu sekcji. Format:
[`ADR-FORMAT.md`](../skills/domain-modeling/ADR-FORMAT.md).

## `/wayfinder` — mapa dużego przedsięwzięcia

**Kiedy:** pomysł jest za duży na jedną sesję agenta i spowity mgłą — nie widać drogi
do celu.

**Model pracy:** mapa to jedno zgłoszenie w trackerze (etykieta `wayfinder:map`), a jej
dzieci to **tickety decyzyjne** — pytania, których rozwiązaniem jest decyzja, nie kawałek
implementacji. Mapa jest **indeksem**: linkuje decyzje, nie przechowuje ich (każda żyje
w swoim tickecie).

**Cztery typy ticketów:**

| Typ | Tryb | Do czego |
|---|---|---|
| `research` | AFK (agent sam) | ustalenie faktu z dokumentacji/API — sub-agent, można równolegle |
| `prototype` | z człowiekiem | tania, brzydka, konkretna rzecz do reakcji, gdy pytanie brzmi „jak to ma wyglądać/działać" |
| `grilling` | z człowiekiem | rozmowa jedno pytanie na raz — przypadek domyślny |
| `task` | zależnie | ręczna robota odblokowująca decyzję (założenie konta, dostęp, przeniesienie danych) |

**Mgła wojny.** Mapa jest celowo niekompletna. Sekcja „Not yet specified" trzyma to, co
widać, ale czego nie da się jeszcze precyzyjnie sformułować. Test: czy potrafisz **zadać
pytanie** teraz — nie czy potrafisz na nie odpowiedzieć. Potrafisz → ticket (nawet jeśli
zablokowany). Nie potrafisz → mgła. Rozwiązanie ticketu rozjaśnia mgłę przed nim
i „promuje" nowe tickety.

**Poza zakresem** to osobna sekcja i osobna kategoria: mgła gromadzi się tylko *w stronę*
celu, a to, co leży za celem, nie jest mgłą — jest wykluczone. Nigdy nie promuje się
do ticketów.

**Twarda reguła: jedna sesja = jeden ticket** (wyjątek: research). Sesja **przejmuje**
ticket przypisaniem go do siebie *przed* rozpoczęciem pracy — otwarty i nieprzypisany
znaczy wolny, więc równoległe sesje się nie zderzają.

**Referencje po nazwie, nie po numerze:** w tekście dla człowieka ticket nazywa się
swoim tytułem. Ściana `#42, #43, #44` jest nieczytelna.

---

# Kod

## `/codebase-design` — słownik projektowania

**Co robi:** dostarcza wspólne słownictwo do projektowania **głębokich modułów** — dużo
zachowania za małym interfejsem, przy czystym szwie, testowalne przez ten interfejs.

**Pojęcia, używane dosłownie** (bez podmieniania na „komponent", „serwis", „API",
„granica"):

- **Moduł** — cokolwiek z interfejsem i implementacją; celowo bez skali (funkcja, klasa,
  pakiet, przekrój przez warstwy).
- **Interfejs** — wszystko, co wołający musi wiedzieć, żeby użyć modułu poprawnie:
  sygnatura, ale też niezmienniki, kolejność, tryby błędów, konfiguracja, charakterystyka
  wydajnościowa.
- **Głębokość** — dźwignia na interfejsie: ile zachowania da się uruchomić na jednostkę
  interfejsu, której trzeba się nauczyć.
- **Szew** (Feathers) — miejsce, w którym można zmienić zachowanie bez edycji w tym
  miejscu.
- **Adapter** — konkret spełniający interfejs w szwie.
- **Dźwignia** (dla wołających) i **lokalność** (dla utrzymujących) — dwa zyski
  z głębokości.

**Zasady:**

- **Test usunięcia** — wyobraź sobie, że kasujesz moduł. Złożoność znika → był
  przelotką. Złożoność wraca u N wołających → zarabiał na siebie.
- **Interfejs jest powierzchnią testową.** Chcesz testować *za* interfejsem → moduł ma
  zły kształt.
- **Jeden adapter = szew hipotetyczny. Dwa adaptery = szew prawdziwy.** Nie wprowadzaj
  szwu, dopóki coś przez niego naprawdę nie waha się.

Rozszerzenia: [`DEEPENING.md`](../skills/codebase-design/DEEPENING.md) (jak pogłębiać
przy różnych kategoriach zależności) i
[`DESIGN-IT-TWICE.md`](../skills/codebase-design/DESIGN-IT-TWICE.md) (3+ równoległych
sub-agentów projektuje ten sam interfejs radykalnie różnie, potem porównanie i mocna
rekomendacja — bez wykładania menu).

## `/improve-codebase-architecture` — skan i raport HTML

**Co robi:** skanuje kod pod kątem okazji do pogłębienia modułów i pokazuje je jako
**samodzielny raport HTML** (Tailwind + Mermaid z CDN), zapisany do katalogu tymczasowego
systemu — nic nie ląduje w repo.

**Zakres najpierw, skan potem (YAGNI):** jeśli człowiek wskazał kierunek — bierz go.
Jeśli nie — historia commitów pokazuje gorące miejsca, bo pogłębianie opłaca się tam,
gdzie zmiany faktycznie zachodzą.

Każdy kandydat dostaje kartę: pliki · problem · rozwiązanie · zyski (w kategoriach
lokalności i dźwigni) · **diagram przed/po** · siła rekomendacji (`Strong` /
`Worth exploring` / `Speculative`). Na końcu jedna sekcja „top recommendation".

Po wyborze kandydata przez człowieka wchodzi `/grilling`, a ustalenia lądują w
`CONTEXT.md` i ADR-ach przez `/domain-modeling`. Odrzucenie kandydata z ważnego powodu =
kandydat na ADR, żeby kolejny przegląd nie zaproponował tego samego.

Szczegóły wizualne: [`HTML-REPORT.md`](../skills/improve-codebase-architecture/HTML-REPORT.md).

## `/tdd` — testy, które przeżyją refaktor

**Co robi:** dostarcza reguły pętli red-green-refactor i definicję testu wartego
utrzymywania.

**Dobry test** weryfikuje zachowanie przez publiczny interfejs, czyta się jak
specyfikacja i przeżywa refaktor, bo nie interesuje go struktura wewnętrzna.

**Szwy:** testy mieszkają w szwach, nigdy przy wnętrznościach. Twarda reguła — **testuje
się wyłącznie w szwach ustalonych wcześniej z człowiekiem**. Nie da się przetestować
wszystkiego; ustalenie szwów z góry decyduje, czy wysiłek pójdzie w krytyczne ścieżki,
czy rozpłynie się po przypadkach brzegowych.

**Trzy antywzorce:**

| Antywzorzec | Objaw |
|---|---|
| **Sprzężony z implementacją** | mockuje wewnętrznych współpracowników, testuje metody prywatne, weryfikuje bokiem (zapytaniem do bazy zamiast przez interfejs). Sygnał: test pęka przy refaktorze, choć zachowanie się nie zmieniło |
| **Tautologiczny** | oczekiwana wartość liczona tak samo jak w kodzie — test przechodzi z konstrukcji i nigdy nie może się z kodem nie zgodzić |
| **Cięcie poziome** | najpierw wszystkie testy, potem cała implementacja. Testujesz *wyobrażone* zachowanie. Zamiast tego: plasterki pionowe — jeden test, jedna implementacja, powtórz |

**Mockowanie tylko na granicach systemu** (zewnętrzne API, czas, losowość, czasem baza
i system plików). Nie mockuje się własnych klas ani niczego, co się kontroluje. Szczegóły:
[`tests.md`](../skills/tdd/tests.md), [`mocking.md`](../skills/tdd/mocking.md).

**Refaktor nie jest częścią pętli** — należy do etapu przeglądu.

## `/implement` — budowa

Realizuje pracę opisaną w specu lub ticketach: `/tdd` w ustalonych szwach, regularne
sprawdzanie typów i pojedynczych plików testowych, pełny zestaw testów raz na końcu.
Potem `/code-review` i commit.

## `/code-review` — dwie osie, osobno

**Co robi:** przegląda różnicę między `HEAD` a wskazanym punktem odniesienia (commit,
gałąź, tag, merge-base) **na dwóch osiach jednocześnie, w równoległych sub-agentach**:

- **Standards** — czy kod trzyma udokumentowane standardy repo, plus stała lista
  **zapachów Fowlera** (tajemnicza nazwa, duplikacja, zazdrość o cechy, zbitki danych,
  obsesja na punkcie typów prostych, powtarzane switche, chirurgia strzelbą, rozbieżna
  zmiana, spekulacyjna ogólność, łańcuchy komunikatów, człowiek-pośrednik, odrzucony
  spadek). Standard repo zawsze wygrywa z listą bazową, a zapachy są **osądem**, nie
  twardym naruszeniem.
- **Spec** — czy kod robi to, o co prosił dokument: braki, rzeczy niezamówione
  (rozrost zakresu), wymagania zaimplementowane źle. Każde ustalenie z cytatem ze specu.

**Raporty nie są scalane ani przerankowane.** Powód: kod może przejść jedną oś i polec
na drugiej — kod idealnie zgodny ze standardami, robiący nie to, o co proszono, i kod
robiący dokładnie to, o co proszono, łamiący konwencje. Scalenie pozwoliłoby jednej osi
zamaskować drugą.

---

# Konfiguracja

## `/setup-matt-pocock-skills` — konfiguracja repo

**Uruchamiane raz na repo**, przed pierwszym użyciem pozostałych skilli inżynierskich.
Ustawia trzy rzeczy i zapisuje je do `docs/agents/`:

- **Issue tracker** — GitHub (`gh`), GitLab (`glab`), lokalny markdown w `.scratch/`,
  albo dowolny inny opisany prozą. Skille czytają `docs/agents/issue-tracker.md` zamiast
  zakładać platformę — dzięki temu ta sama instrukcja działa niezależnie od tego, gdzie
  trzymasz zadania.
- **Etykiety triage** — pięć ról (`needs-triage`, `needs-info`, `ready-for-agent`,
  `ready-for-human`, `wontfix`) mapowanych na nazwy używane w twoim trackerze. Sekcja
  jest pomijana, jeśli skill `triage` nie jest zainstalowany.
- **Dokumenty domenowe** — układ jedno- lub wielokontekstowy (`CONTEXT.md` +
  `docs/adr/`, albo `CONTEXT-MAP.md` wskazujący konteksty). Domyślnie jednokontekstowy;
  wielokontekstowy proponowany tylko przy wykrytych sygnałach monorepo.

Skill najpierw eksploruje repo, potem pokazuje, co znalazł, pyta **sekcja po sekcji**
(z rekomendacją na początku, żeby dało się zaakceptować jednym słowem), pokazuje szkic
plików do zatwierdzenia i dopiero wtedy pisze. Nigdy nie tworzy `AGENTS.md`, gdy istnieje
`CLAUDE.md` (ani odwrotnie) — edytuje ten, który już jest.
