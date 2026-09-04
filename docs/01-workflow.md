# Workflow wdrożeniowy — faza po fazie

Ten dokument tłumaczy **dlaczego** proces wygląda tak, jak wygląda, i co konkretnie
powstaje na każdym etapie. Sama instrukcja wykonawcza dla agenta żyje w
[`skills/workflow/SKILL.md`](../skills/workflow/SKILL.md) i w plikach
[`skills/workflow/references/`](../skills/workflow/references/) — tu jest komentarz do
niej, nie jej kopia.

**Słownik.** *Zamawiający* to strona, która zamawia i odbiera — klient zewnętrzny, dział
biznesowy albo product owner. *Użytkownik* to osoba prowadząca projekt po naszej stronie.
*Etap* to porcja pracy, którą da się odebrać osobno. Domena, w której to powstało, to
projekty wdrożeniowe u klientów zewnętrznych, więc faza 4 mówi o umowie i płatności —
w wariancie wewnętrznym jest to po prostu **zamrożenie zakresu przed budową**, a bramka
działa tak samo.

## Trzy reguły, które obowiązują we wszystkich fazach

**1. Bramka 🚧 = koniec fazy.** Agent zatrzymuje się i czeka. Nie „przygotowuje już
kolejnego kroku, skoro i tak trzeba będzie". Powód jest prozaiczny: model, który idzie
dalej sam, buduje kolejne piętra na niepotwierdzonym fundamencie, a koszt zawrócenia
rośnie z każdą fazą.

**2. Zero założeń.** Luka w wiedzy jest *pytaniem*: do klienta (przez
[`/to-questionnaire`](../skills/to-questionnaire/SKILL.md)) albo do użytkownika.
Nigdy roboczą hipotezą wpisaną po cichu w dokument. Powód: hipoteza wygląda w tekście
identycznie jak fakt, więc po tygodniu nikt już nie odróżni jednego od drugiego —
a odróżnić trzeba, bo na fakcie można podpisać umowę, na hipotezie nie.

**3. Bramka naniesień.** Krytyka i research produkują *propozycje*, nie zmiany.
Zatwierdzony dokument zmienia się tak: krótka tabela ZA/PRZECIW z rekomendacją →
wybór użytkownika → naniesienie **tylko wybranych** pozycji. Wymówka „to tylko
techniczne / to tylko mechanika" nie omija bramki. Powód: dokument zatwierdzony to
dokument, na który ktoś się zgodził — cicha zmiana unieważnia tę zgodę.

Do tego jeden nawyk komunikacyjny, opisany w
[`references/tryb-pracy.md`](../skills/workflow/references/tryb-pracy.md): **jeden wątek
aż do decyzji**. Jedno pytanie na raz, z 2-3 opcjami i rekomendacją. Fakty agent sprawdza
sam; do człowieka trafiają wyłącznie decyzje.

---

## Faza 0 — Setup (raz na klienta)

**Po co:** żeby wiedza o kliencie miała gdzie mieszkać między sesjami.

Powstają dwie rzeczy:

- **CLAUDE.md klienta** ([`/init-klienta`](../skills/init-klienta/SKILL.md)) — żywy
  dokument „JAK": wykryta struktura folderu, architektura, tabela linków zewnętrznych,
  dziennik błędów procesowych i wzorce techniczne. Plus hook, który pilnuje, żeby nikt
  (łącznie z agentem) nie dopisywał tam nic bez zgody.
- **Tracker zadań** ([`/setup-matt-pocock-skills`](../skills/setup-matt-pocock-skills/SKILL.md))
  — GitHub, GitLab albo lokalny markdown. Rekomendacja dla pracy solo: lokalny markdown.

Zastane pliki „analizy technicznej" z etapu sprzedaży nie są przepisywane w całości —
rozdziela się je: linki → tabela linków, trwałe reguły → wzorce techniczne, opis stanu →
raport analizy, oryginał → archiwum.

## Faza 1 — Poznaj 🚧

**Po co:** żeby plan powstał z materiałów klienta, a nie z wyobrażenia o nich.

Kolejność ma znaczenie:

1. [`/analiza <folder>`](../skills/analiza/SKILL.md) — agent w **osobnym kontekście**
   czyta wszystko, co jest na dysku, i wraca z raportem: co wiemy, co już zdecydowano
   (z datami), liczby, **sprzeczności z rekomendacją rozstrzygnięcia**, luki. Raport ląduje
   w pliku, nie tylko w rozmowie — przeżyje crash sesji.
2. [`/grill-with-docs`](../skills/grill-with-docs/SKILL.md) — przesłuchanie użytkownika,
   jedno pytanie na raz. Zaczyna od **WHY** (po co to klientowi, jakie pieniądze i czas
   są w grze, po czym pozna sukces), dopiero potem CO i JAK. Architektura wraca do
   człowieka jako propozycje z opcjami, nie jako pytanie „jak to zbudować".
3. Czego nie wie ani agent, ani użytkownik → [`/to-questionnaire`](../skills/to-questionnaire/SKILL.md):
   jedna ankieta do klienta, async, bez spotkania. Jeśli do klienta wisi już lista pytań —
   scalamy, klient dostaje **jeden** dokument.

**Kluczowa reguła tej fazy — hierarchia źródeł.** Faktem jest tylko to, co przyszło od
klienta (transkrypcje, maile, jego pliki) i zweryfikowany stan techniczny. Dokumenty
wygenerowane przez agenta w poprzednich sesjach — łącznie ze starymi PRD i planami — to
**hipotezy**. Spójny dokument nie znaczy prawdziwy. Prototypy dowodzą wykonalności, ale
nie tego, że użyte narzędzia były świadomym wyborem. Szczegóły:
[`references/wiarygodnosc.md`](../skills/workflow/references/wiarygodnosc.md).

**Bramka:** użytkownik potwierdza, że agent zrozumiał proces.

## Faza 2 — Zaplanuj pod wycenę 🚧

**Po co:** żeby wycena stała na dowodzie wykonalności, a nie na optymizmie.

Duży system → [`/wayfinder`](../skills/wayfinder/SKILL.md) (mapa decyzji, rozstrzygana
ticket po tickecie). Mały → od razu [`/to-spec`](../skills/to-spec/SKILL.md).

Plan całości robi **dwie** rzeczy naraz:

1. **Weryfikuje ryzyko i wykonalność** — najbardziej ryzykowny element całości ma być
   dowiedziony najwcześniej, najlepiej testem na prawdziwych danych klienta jeszcze
   przed umową. To jest ta część, która realnie podnosi cenę: sprzedajesz pewność,
   nie obietnicę.
2. **Tnie system na kroki wdrożenia.** Każdy krok = samodzielna wartość: da się go
   osobno sprzedać, odebrać i zafakturować. Za małe kroki = mikrofaktury i zmęczony
   klient; za duże = całe ryzyko skumulowane w jednym odbiorze na końcu. Dla każdego
   kroku spisuje się zależności i punkt styku z następnymi — krok późniejszy nie może
   psuć odebranego wcześniej.

**Głęboko w ryzyko, płytko w wykonanie.** Plan przedsprzedażowy nie schodzi do kontraktów
danych i kolejności budowy — to należy do PRD, który powstaje dopiero po umowie.

**Równa wysokość opisu:** wszystkie kroki opisane na tym samym, średnim poziomie —
także ten najlepiej udokumentowany. Bez tej reguły model rozpisuje najszerzej to, o czym
akurat ma najwięcej materiału, i plan traci proporcje.

Szkielet dokumentu (10 sekcji w ustalonej kolejności) jest w
[`skills/workflow/SKILL.md`](../skills/workflow/SKILL.md), reguły planowania w
[`references/plan.md`](../skills/workflow/references/plan.md), a metoda researchu —
równoległe sub-agenty, minimum dwa źródła z linkami na decyzję — w
[`references/research.md`](../skills/workflow/references/research.md).

**Trzy poziomy dokumentów, żeby nazwy nie myliły:**

| Poziom | Nazwa | Kiedy powstaje | Czym jest |
|---|---|---|---|
| 1 | spec ogólny / plan całości | przed wyceną, tylko duże systemy | mapa: ryzyko, wykonalność, podział na kroki |
| 2 | PRD (`/to-spec`) | po umowie, per krok | dokument wykonawczy z kryteriami EARS |
| 3 | tickety (`/to-tickets`) | po zatwierdzeniu PRD | zadania z zależnościami |

**Bramka:** użytkownik zatwierdza plan.

## Faza 3 — Skrytykuj 🚧

**Po co:** bo autor dziedziczy własną ramę myślenia i nie zobaczy tego, czego w dokumencie
brakuje.

[`/hejt <dokument>`](../skills/hejt/SKILL.md) ma dwie fazy i jedną żelazną zasadę:
**podczas krytyki dokument jest nietykalny**.

1. **Dyskusja** (max 3 rundy): Codex zgłasza konkretne zarzuty, Claude jako autor albo
   przyznaje rację (→ propozycja), albo broni (→ punkt sporny). Efekt: dwie listy.
2. **Sędzia** — *nowe* wywołanie, bez historii dyskusji. Dostaje dokument, listy, kontekst
   i **źródła pierwotne = wyłącznie głos klienta** (jego transkrypcje i wiadomości; nasze
   drafty i notatki nie są źródłem). Rozstrzyga każdy punkt, sprawdza zgodność planu
   z tym, co klient faktycznie powiedział, i robi własny research aktualnych praktyk.

Priorytet krytyki zależy od poziomu dokumentu: hejt **planu** patrzy na kierunek
(architektura, ceny, przerost), hejt **PRD** jest techniczny (mechanika, przypadki
brzegowe, kontrakty danych). Ale zakaz otwierania biznesu w hejcie PRD nie oznacza klapek
— ostatnia runda zawsze przechodzi checklistę „oś czasu i otoczenie": co się dzieje
w dniu 1 (pierwszy przebieg na prawdziwych danych, backfill, lawina historii), co
w dniu 1000 (retencja, RODO, co rośnie bez ograniczeń), czy istnieje jedna liczba
mówiąca, że działa, i czy parametry nie są strojone na tych samych danych, na których
testujemy.

**Wynik to tabela propozycji, nie poprawiony dokument.** Zmiany wchodzą dopiero po
wyborze użytkownika.

**Bramka:** użytkownik wybiera zmiany. Potem agent przechodzi z nim przez finalny
dokument sekcja po sekcji i domyka otwarte `[DO DOPYTANIA]`.

## Faza 4 — Zamroź zakres 🚧 (w wariancie komercyjnym: umowa)

**Po co:** żeby to, co uzgodnione, dało się odebrać bez sporu.

[`/zalacznik <PRD>`](../skills/zalacznik/SKILL.md) tłumaczy zatwierdzony PRD na język
klienta — 7 sekcji, zero żargonu, zero cen (ceny żyją w umowie). Wysyłany razem z umową,
informacyjnie.

Dwie reguły, które robią całą różnicę przy odbiorze:

- **Kryteria odbioru nigdy nie są warunkowe.** Odbiór nie może wisieć na tym, czy klient
  odpowie albo da dostęp. Warunkowe mogą być wyłącznie bonusy — i każdy warunek ma
  zapisane **obie** gałęzie („dasz X → dodatkowo Y, bez zmiany ceny i terminu; nie dasz →
  nie wpływa na odbiór").
- **Do umowy wchodzi tylko to, czego wynik kontrolujesz w 100%.** Test: „czy istnieje
  realny scenariusz, w którym to nie przechodzi mimo dobrze wykonanej roboty?". Jeśli tak
  (zależy od jakości odczytu AI, od zachowania klienta, od zewnętrznej usługi) — kryterium
  nie wchodzi. Zamiast progu jakościowego („95% dokumentów odczytanych poprawnie")
  wpisujesz mechanizm, który kontrolujesz („niezgodna suma → dokument odłożony, nie
  wchodzi do faktury"). Progi zostają w PRD jako standard wewnętrzny.

**Bramka:** użytkownik akceptuje przed wysyłką. Płatność z góry **przed** dalszą robotą.

## Faza 5 — Domknij i potnij 🚧 (po zamrożeniu zakresu)

**Po co:** bo szczegółowe planowanie ma sens dopiero wtedy, gdy wiadomo, że budujemy —
w wariancie komercyjnym jest to wprost płatna część pracy, nie prezent do oferty.

W dużym systemie faza 5 dzieje się **per krok wdrożenia**, nie hurtem dla wszystkich
kroków naraz:

- **a) PRD kroku** — [`/to-spec`](../skills/to-spec/SKILL.md) buduje dokument
  *z klientem*: każda niepewność co do jego procesu to pytanie do niego, nie domysł.
  Miejsca zależne od odpowiedzi są w tekście oznaczone `[DO POTWIERDZENIA → pyt. N]` —
  żadna sekcja nie udaje gotowej. PRD jest skończony dopiero, gdy zniknął ostatni znacznik.
- **b) Tickety** — [`/to-tickets`](../skills/to-tickets/SKILL.md) tnie **zatwierdzony**
  PRD na wertykalne plasterki („tracer bullet": wąska, ale kompletna ścieżka przez
  wszystkie warstwy), każdy z jawnymi zależnościami. Ticket, którego PRD nie rozstrzyga,
  powstaje jako **zablokowany z konkretnym pytaniem** — nigdy z hipotezą.
- **c) Testy** — pisane **zanim** powstanie implementacja. Kryteria PRD w formacie EARS
  konwertują się 1:1 na testy. Do tego dane referencyjne klienta, sumy kontrolne
  i spreparowane błędy: mechanizm bezpieczeństwa nieprzetestowany na złych danych
  to mechanizm niedziałający.
- **d) Hejt delty** — tylko jeśli PRD istotnie się zmienił po umowie. Krótki, na różnicy,
  nie od zera.

**Bramka:** użytkownik zatwierdza tickety przed budową.

## Faza 6 — Buduj

[`/implement`](../skills/implement/SKILL.md) pracuje na ticketach z 5b, aż testy z 5c
świecą na zielono, potem upraszcza. Wariant „buduje Codex" (przy dużych buildach albo
oszczędzaniu limitu) jest opisany w
[`references/budowa.md`](../skills/workflow/references/budowa.md): Codex nie dotyka `.env`,
`CLAUDE.md` ani plików z sekretami, Claude nie pisze wtedy implementacji tylko recenzuje
diff, maksymalnie 3 rundy poprawek i STOP.

## Faza 7 — Sprawdź

[`/code-review`](../skills/code-review/SKILL.md) uruchamia dwa sub-agenty równolegle
i **nie łączy ich wyników**: oś **Standards** (czy kod trzyma standardy repo + baseline
zapachów Fowlera) i oś **Spec** (czy robi to, o co prosił dokument — braki, nadmiar,
źle zaimplementowane wymagania). Rozdzielenie jest celowe: kod może przejść jedną oś
i polec na drugiej, a scalanie raportów pozwoliłoby jednej zamaskować drugą.

## Faza 8 — Po odbiorze: zamrożenie

Spec i załącznik trafiają do archiwum jako **zapis umówionego zakresu** — dowód przy
ewentualnym sporze. Nie aktualizujesz ich, gdy system u klienta się zmienia: źródłem
prawdy jest wtedy działający system i CLAUDE.md klienta. Zmiana zakresu = nowy krok
= nowa umowa. Lekcje z budowy lądują we wzorcach technicznych CLAUDE.md — żeby ta sama
klasa błędu nie kosztowała drugi raz.
