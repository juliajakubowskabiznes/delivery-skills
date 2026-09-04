# Skille wdrożeniowe — szczegółowo

Osiem skilli napisanych pod ten proces (po polsku, bo produkują dokumenty dla polskich
klientów). Dla każdego: kiedy się uruchamia, co dostaje na wejściu, co robi, co zostawia
i na czym najłatwiej się wyłożyć.

Kolejność jak w [workflow](01-workflow.md).

---

## `/workflow` — kręgosłup procesu

**Uruchomienie:** „odpal workflow", „przejdźmy cały proces", „zbuduj X od początku
do końca".

**Co robi:** trzyma kolejność 7 faz, pilnuje bramek i routuje do pozostałych skilli.
Sam w sobie prawie nic nie produkuje — jest mapą, po której chodzi agent.

**Architektura pliku.** Kręgosłup (`SKILL.md`) jest krótki; szczegółowe reguły siedzą
w [`references/`](../skills/workflow/references/) i są doczytywane dopiero przy wejściu
w fazę:

| Plik | Zawiera |
|---|---|
| `setup.md` | co zrobić raz na klienta, jak rozłożyć zastane pliki ze sprzedaży |
| `plan.md` | poziomy dokumentów, reguła równej wysokości, higiena wersji |
| `research.md` | metoda researchu (równoległe sub-agenty, 2 źródła), bramka naniesień |
| `wiarygodnosc.md` | hierarchia źródeł — co jest faktem, a co hipotezą |
| `tryb-pracy.md` | jak rozmawiać: jeden wątek do decyzji, parking wątków, zero spotkań operacyjnych |
| `budowa.md` | warianty budowy (Claude / GPT), zasady bezpieczeństwa |

Ten podział to nie estetyka: krótki plik główny przeżyje długą sesję, a gruby regulamin
wypadłby z kontekstu dokładnie wtedy, kiedy jest potrzebny.

**Sygnalizacja fazy.** Agent zaczyna odpowiedzi od `[WORKFLOW faza N/7 — NAZWA]` i jednego
zdania o tym, co się dzieje i jaka jest najbliższa bramka. Po wznowieniu sesji najpierw
odczytuje stan (CLAUDE.md, handoff, tracker) i melduje fazę, zanim cokolwiek zrobi.

**Pułapka:** uruchamianie pełnej ścieżki do drobiazgu. Jest na to test wagi — zmiana bez
nowych decyzji architektonicznych idzie od razu do budowy i przeglądu.

---

## `/init-klienta` — pamięć projektu

**Uruchomienie:** faza 0, „załóż CLAUDE.md dla klienta". Argument: ścieżka folderu klienta.

**Co robi:** zakłada `CLAUDE.md` w folderze klienta według szablonu i dokłada hook
pilnujący bramek. Jeśli plik już istnieje — **nie nadpisuje**; proponuje brakujące sekcje.

**Sekcje szablonu i ich sens:**

- **Zasada nadrzędna: zapisuj każdy błąd tutaj** — techniczne do „Wzorce techniczne",
  procesowe i komunikacyjne do „Dziennik błędów". Format: co poszło nie tak → prawdziwa
  przyczyna → jak unikać. Przy nowym problemie agent najpierw sprawdza, czy to nie znany
  błąd.
- **Struktura folderu** — wykryta z `ls`, nie narzucona. Opisujesz to, co jest.
- **Architektura** — wypełniana przez workflow po decyzjach: stack (i dlaczego nie
  prościej), model danych, guardraile.
- **Linki zewnętrzne** — jedna tabela, jedno miejsce, gdzie żyje URL. Reszta plików
  odsyła. Wiersz ma być krótki: link, status, odsyłacz — bez faktów projektowych, bo
  CLAUDE.md ładuje się w każdej sesji i każdy zbędny akapit to stały podatek kontekstowy.
- **Dziennik błędów** i **Wzorce techniczne** — puste na start, rosną w trakcie.

**Hook.** Do `.claude/settings.json` w folderze klienta trafia `PostToolUse` na
`Write|Edit`, który wychodzi z kodem 2 przy zapisie do plików spec/plan/PRD oraz do
CLAUDE.md. Przypomina o dwóch rzeczach: (1) pilnuj poziomu dokumentu — reguła procesu
idzie do skilla, zasada na zawsze do pamięci, fakt do CONTEXT.md, decyzja do ADR;
(2) zatwierdzony dokument zmienia się tylko za zgodą, po tabeli ZA/PRZECIW.

**Dlaczego hook, a nie instrukcja w prompcie:** instrukcja tonie w długiej sesji, hook
odpala się mechanicznie przy każdym zapisie, niezależnie od tego, co sesja pamięta.

**Pułapka:** dopisywanie do CLAUDE.md „przy okazji". Plik ma być krótki — reguła wchodzi,
gdy błąd się powtórzył albo był kosztowny, jedna propozycja na raz, po zgodzie.

---

## `/analiza` — czytanie materiałów na świeżym kontekście

**Uruchomienie:** faza 1, „przeanalizuj materiały [klienta]", „co mamy o X".
Argument: ścieżka folderu klienta.

**Techniczny detal, który jest sednem tego skilla:** `context: fork`. Agent czyta
materiały **bez historii rozmowy** — nie dziedziczy tego, w co ta sesja zdążyła uwierzyć.
Ocenia to, co realnie leży na dysku.

**Wynik:** raport zapisany do pliku (odporny na crash sesji) i zwrócony w rozmowie:

1. Co wiemy o procesie (5-10 punktów)
2. Co już zdecydowano (z datami)
3. Liczby kluczowe (wolumeny, czasy, kwoty)
4. **Sprzeczności** — per sztuka: plik A [data] mówi X, plik B [data] mówi Y +
   rekomendacja rozstrzygnięcia
5. Pułapki techniczne z CLAUDE.md klienta (top 5)
6. Luki — czego materiały nie mówią, a grill musi dopytać

**Hierarchia wiarygodności (twarda):**

| Status | Co to jest |
|---|---|
| **Fakt** | materiał od klienta (transkrypcje, maile, jego pliki) + zweryfikowany stan techniczny |
| **Hipoteza** | wszystko, co wygenerował agent w poprzednich sesjach — analizy, stare PRD, drafty |
| **Robocze** | decyzje użytkownika z poprzednich sesji — podjęte na ówczesnych założeniach, do re-potwierdzenia |
| **Dowód, nie decyzja** | prototypy i vibe-code — dowodzą wykonalności i edge case'ów, nie tego, że wybór narzędzi był świadomy |

Do sekcji „co już zdecydowano" trafia wyłącznie to, co ma jawne źródło i datę. Reszta
idzie do luk.

**Pułapka:** przepisanie starego planu jako obowiązującego. Spójny dokument nie znaczy
dobry — mógł być słaby, nieaktualny albo od początku do wymiany.

---

## `/to-questionnaire` — jedna ankieta zamiast pięciu maili

**Uruchomienie:** gdy pojawia się pytanie, na które odpowiedź zna wyłącznie klient
(faza 1 i 5).

**Metoda — „grilluj wysyłkę, nie temat".** Agent nie wypytuje użytkownika o rzeczy,
których ten nie wie. Pyta o dwie rzeczy, które użytkownik zawsze wie:

1. **Do kogo to idzie** — rola, wiedza, relacja. To ustawia ton i ilość kontekstu
   w dokumencie.
2. **Co musisz dostać z powrotem** — konkretna lista decyzji i faktów, których nie
   rozstrzygniesz sam.

Pytania w dokumencie celują w **lukę** między tym, co wie odbiorca, a tym, czego
potrzebuje użytkownik.

**Zasady dokumentu:** najważniejsze pytania pierwsze (async oznacza, że możesz mieć tylko
jedno podejście), jedno pytanie = jedna myśl (nigdy złożone), pod każdym miejsce na
odpowiedź, „nie wiem" jest użyteczną odpowiedzią. Jeśli do klienta wisi już inna lista
pytań — **scalasz**: klient dostaje jeden dokument, bez duplikatów i bez ponawiania.

---

## `/hejt` — krytyka zanim zapłaci za nią klient

**Uruchomienie:** faza 3, na wszystkim, co idzie do wyceny lub oferty.
Argument: ścieżka dokumentu.

**Zasada nadrzędna:** podczas krytyki **dokument jest nietykalny**. Żaden model nic nie
poprawia. Dyskusja produkuje propozycje, sędzia je rozstrzyga, człowiek wybiera.

**Faza 1 — dyskusja** (max 3 rundy): GPT zgłasza konkretne zarzuty (luka, ryzyko,
przerost, nieaktualne narzędzia), Claude jako autor przyznaje rację albo broni jednym
zdaniem. Efekt: lista propozycji uzgodnionych + lista punktów spornych.

**Faza 2 — sędzia:** *nowe* wywołanie GPT, bez historii dyskusji. Powód: autor dziedziczy
własną ramę, a sędzia, który przeczytał całą dyskusję, dziedziczy ją razem z nim. Sędzia
dostaje dokument, obie listy, 2-3 pliki kontekstu i **źródła pierwotne = wyłącznie głos
klienta**. Nasze maile do klienta i notatki nie są źródłem — to zapis naszych intencji,
nie jego słów.

Zadania sędziego: rozstrzygnąć każdy punkt (WPROWADZIĆ / ODRZUCIĆ / DECYZJA UŻYTKOWNIKA),
sprawdzić dokument samodzielnie (skrajne przypadki, AI tam gdzie wystarczy reguła, decyzje
bez uzasadnienia, nazwy modeli i ceny z pamięci zamiast ze źródeł), zweryfikować zgodność
z transkrypcjami klienta **z cytatem**, zrobić własny research aktualnych praktyk
z linkami, wydać werdykt.

**Checklista „oś czasu i otoczenie"** — obowiązkowa w ostatniej rundzie hejtu PRD, bo
braku trudniej szukać niż błędu: błąd jest w tekście, brak trzeba przynieść z listy.
Dzień 1 (pierwsze uruchomienie na prawdziwych danych, backfill, puste bazy) · dzień 1000
(retencja, RODO, co rośnie bez ograniczeń) · jedna metryka sukcesu i budżet kolejki
eskalacji · strojenie vs testowanie na tych samych danych · historie użytkownika ·
kolizje numeracji i wiszące odwołania.

**Dieta kontekstowa sędziego:** wsad inline i odchudzony — dokument + krótkie podsumowanie
dyskusji (nie surowe rundy) + niezbędne wyciągi. Pełne transkrypcje klienta czyta tylko
sędzia **planu**; sędzia PRD ich nie powtarza. Za duży wsad = brak werdyktu i spalone
tokeny.

**Operacyjnie:** ciężkie wywołania zawsze w `--background`. Zadanie na kilka minut
myślenia nie mieści się w timeoucie Bash, a zabity proces zostawia blokadę „task still
running", której plugin nie sprząta.

---

## `/zalacznik` — zakres, który da się odebrać

**Uruchomienie:** faza 4, po zatwierdzeniu PRD kroku. Argument: ścieżka PRD.

**Czym jest:** PRD przetłumaczony na język klienta — 7 sekcji (+ opcjonalna 8. o
przetwarzaniu danych), zero żargonu, **zero cen** (ceny żyją w umowie i proformie).
Nic ponad PRD; to nie jest nowy dokument, tylko inna warstwa językowa tego samego.

**Sekcje:** cel projektu · jak działa krok po kroku · zakres i kryteria odbioru ·
co nie wchodzi · narzędzia zewnętrzne i wymagania po stronie klienta · ograniczenia
techniczne · warunki działania i procedura utrzymaniowa.

**Filtr doboru kryteriów odbioru** — do załącznika wchodzi kryterium, które jest:
1. mierzalne przez klienta samodzielnie, na jego danych, bez zaglądania w bebechy;
2. proste i zero-jedynkowe;
3. **w 100% pewne** — wynik kontrolujesz w całości.

Test pewności: „czy istnieje realny scenariusz, w którym to nie przechodzi mimo dobrze
wykonanej roboty?". Jeśli tak — kryterium nie wchodzi. Zamiast progu jakościowego wpisujesz
mechanizm, który kontrolujesz w pełni.

**Warunkowość:** kryteria odbioru nigdy nie są warunkowe. Warunkowe bywają wyłącznie
bonusy, każdy z zapisanymi **obiema** gałęziami. Opcje przyszłe i dokładki nie wchodzą
do załącznika w ogóle.

**Miejsce docelowe:** dokument umowy w chmurze (link w tabeli linków CLAUDE.md klienta),
nie repo. W repo nie trzymamy kopii treści — jedno źródło prawdy.

**Wymagania po stronie klienta** muszą być wykonalne **samodzielnie**: instrukcja albo
wideo per czynność, bez spotkania.

---

## `/to-spec` — PRD z kryteriami, które są testami

**Uruchomienie:** faza 2 (mały system) lub 5a (per krok). Nie przeprowadza wywiadu —
syntetyzuje to, co już wiadomo z rozmowy i repo.

**Struktura dokumentu:** Problem · Cel i metryka sukcesu · Historie użytkownika ·
Kryteria akceptacji (EARS) · Przypadki brzegowe i błędy · Decyzje implementacyjne ·
Decyzje testowe · Poza zakresem · Nierozstrzygnięte pytania · Notatki.

**EARS** — kryteria pisane wzorcami, bo każde takie zdanie konwertuje się 1:1 na test:

```
System MUSI <X>
GDY <zdarzenie>, system MUSI <X>
PODCZAS GDY <stan>, system MUSI <X>
JEŚLI <błąd>, TO system MUSI <X>
TAM GDZIE <opcja>, system MUSI <X>
```

Twarda reguła: **każdy krok czytający dane przez AI musi mieć kryterium typu „JEŚLI"** —
co się dzieje, gdy odczyt zawiedzie albo sumy się nie zgadzają. Cicha zła odpowiedź to
najgorsza klasa awarii.

**Sekcja przypadków brzegowych** to mapa pokrycia, nie druga lista kryteriów: każdy
przypadek (puste dane, zły format, usługa nie odpowiada, duplikaty, podwójne wysłanie,
brak dopasowania) zamykasz jednym z dwóch sposobów — wskazaniem kryterium EARS albo
wpisem w „poza zakresem". Żaden nie zostaje nierozstrzygnięty.

**Drabinka: cel → metryka → kryterium odbioru** (trzy różne rzeczy, nie mieszać):

| Poziom | Dla kogo | Wymagania |
|---|---|---|
| **Cel** | narracja | może być ogólny, nikt go nie mierzy |
| **Metryka sukcesu** | wewnętrznie, NIE do umowy | policzalna z danych, które system i tak zapisuje, jednym filtrem; nie buduje się funkcji pod metrykę |
| **Kryterium odbioru** | do umowy | proste, zero-jedynkowe, mierzalne przez obie strony w dniu odbioru, wynik w pełni kontrolowany |

**Trzy twarde reguły decyzji implementacyjnych:** sprawdź aktualne praktyki w sieci
(oficjalne docs, blogi inżynierskie, repo — ze źródłem przy decyzji) · zweryfikuj
świeżość każdej nazwy modelu, API i ceny (nigdy z pamięci treningowej, z datą sprawdzenia)
· uzasadnij prostotę — eskalacja tylko w kolejności: stała reguła → deterministyczny
workflow → krok AI → agent AI.

**Wariant kliencki:** domyślnie **każda** niepewność co do procesu klienta to pytanie
do klienta, nie domysł. Do użytkownika idą wyłącznie decyzje biznesowe po jego stronie
(cena, krój etapów, kolejność sprzedażowa, ryzyka). Miejsca zależne od odpowiedzi są
oznaczone w tekście `[DO POTWIERDZENIA → pyt. N]`, a przed każdym zapisem agent robi
**sweep źródeł**: zdanie po zdaniu sprawdza, czy twierdzenie o procesie klienta ma źródło
pierwotne. Brak źródła → zdanie wylatuje z sekcji i staje się pytaniem.

**Spec nie jest skończony, dopóki sekcja „Nierozstrzygnięte pytania" ma otwarte pozycje.**
Każde pytanie ma właściciela — pytanie bez właściciela nigdy nie dostanie odpowiedzi.
Jeśli pytań nie ma, wpisuje się „brak" — pusta sekcja to twierdzenie, nie przeoczenie.

---

## `/to-tickets` — cięcie na zadania, które da się skończyć

**Uruchomienie:** faza 5b, na **zatwierdzonym** PRD. Argument: ścieżka specu lub numer
zgłoszenia.

**Tracer bullet** — każdy ticket to wąski, ale **kompletny** przekrój przez wszystkie
warstwy (schemat, API, UI, testy), a nie poziomy plaster jednej warstwy. Skończony ticket
da się pokazać albo zweryfikować sam z siebie i mieści się w jednym świeżym kontekście.
Prefaktoring idzie pierwszy: „najpierw zrób zmianę łatwą, potem zrób łatwą zmianę".

**Zależności** są jawne: „blocked by" wskazuje tickety, które muszą się skończyć wcześniej.
Pracuje się na **froncie** — ticketach, których blokady są zamknięte.

**Zero założeń:** czego PRD nie rozstrzyga, tego ticket nie zgaduje. Powstaje jako
**zablokowany z konkretnym pytaniem** skierowanym do właściwej strony (klient →
`/to-questionnaire`, użytkownik → bramka). Założenie zapieczone w tickecie to ten sam
gatunek błędu co założenie w specu.

**Wyjątek — szerokie refaktory.** Zmiana mechaniczna o dużym promieniu rażenia (zmiana
nazwy kolumny, przetypowanie wspólnego symbolu) nie da się pociąć wertykalnie, bo jedna
edycja psuje tysiąc miejsc naraz. Sekwencja to **expand-contract**: najpierw dodaj nową
formę obok starej, potem migruj wywołania partiami (każda partia = osobny ticket,
CI zielone między partiami), na końcu usuń starą formę ticketem zablokowanym przez
wszystkie partie.

**Przed publikacją agent pyta** o granulację, poprawność zależności i ewentualne
scalenia/podziały — i iteruje aż do akceptacji.
