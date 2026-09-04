---
name: hejt
description: >
  Ocena planu/specu przed budową: dyskusja GPT↔Claude o dokumencie
  (BEZ nanoszenia zmian), świeży sędzia rozstrzyga sporne punkty, użytkownik dostaje
  krótkie zestawienie propozycji + werdykt i wybiera, co wprowadzić. Używaj po /to-spec
  i na planie całości — albo gdy użytkownik mówi: "hejtuj", "puść przez sędziego",
  "oceń ten plan", "czy to nie przekombinowane". Argument: ścieżka dokumentu.
---

# Hejt — dyskusja + sędzia + decyzja użytkownika

Zasada nadrzędna: **podczas hejtu dokument jest NIETYKALNY.** Żaden model nie nanosi
zmian. Dyskusja produkuje PROPOZYCJE, sędzia je rozstrzyga, użytkownik wybiera — dopiero
wtedy nanosisz wybrane.

GPT wołaj przez skill `codex:rescue`; fallback: Bash `codex exec`; gdy Codex w ogóle
nie działa — zrób obie fazy sam i JAWNIE zaznacz "ocena własna, nie niezależna".
**Ciężkie wywołania (werdykt sędziego, wysokie reasoning, kilkanaście tys. tokenów)
ZAWSZE w --background — nigdy pierwszy plan.** Zadanie na kilka minut myślenia nie
mieści się w timeoucie Bash, a zabity proces zostawia blokadę "task still running",
której plugin nie sprząta (odblokowanie: `codex-companion.mjs cancel`; uwaga:
`/codex:status` filtruje po sesji i potrafi pokazać "no jobs" mimo blokady na dysku).

**Priorytet hejtu zależy od poziomu dokumentu:** hejt PLANU = kierunek
(architektura, zgodność z klientem, przerost, ceny) — zarzuty techniczne kieruj
jako WYMAGANE sekcje do PRD. Hejt PRD = przede wszystkim TECHNICZNY: mechanika,
przypadki brzegowe, kontrakty danych, wykonalność w narzędziu — biznes już
rozstrzygnął hejt planu, nie otwieraj go ponownie. **ALE „nie otwieraj biznesu"
≠ klapki totalne** (znana wpadka: zakaz wyciął metrykę sukcesu i wątek zgodności
z RODO — recenzent zewnętrzny bez zakazu je znalazł): ostatnia runda hejtu PRD MUSI
objąć też **OŚ CZASU I OTOCZENIE** — obowiązkowa checklista:
- **Dzień 1 (go-live):** co się dzieje przy PIERWSZYM uruchomieniu na realnych
  danych klienta (backfill, lawina historii, puste bazy)?
- **Dzień 1000:** retencja, prawo (RODO, wymogi archiwizacji), co rośnie bez
  ograniczeń?
- **Metryka sukcesu:** czy jest JEDNA liczba, po której klient pozna, że działa —
  i budżet na kolejkę eskalacji (system eskalujący wszystko = poprawny i bezużyteczny)?
- **Strojenie vs testowanie:** czy parametry strojone są na tych samych danych,
  na których testujemy (przeuczenie) — wymagaj zbioru wstrzymanego?
- **Historie użytkownika:** przeszukaj też JE — obietnica bez kryterium i historia
  sprzeczna z decyzją to ta sama klasa błędu co sprzeczność między kryteriami.
- **Numeracja/odwołania:** kolizje identyfikatorów między sekcjami, wiszące
  odwołania (ADR-y, parametry-sieroty).
Braku trudniej szukać niż błędu — błąd JEST w tekście, brak trzeba przynieść
z checklisty. Autor dziedziczy własną ramę: gdy dokument jest ważny, ostatni
przegląd zrób świeżym wywołaniem, które dostaje TYLKO dokument (zero naszej ramy,
zero podsumowań dyskusji).

## Faza 1 — DYSKUSJA (max 3 rundy, zero edycji)

1. Wyślij dokument do GPT z prośbą o KONKRETNE zarzuty (luka, ryzyko, przerost,
   nieaktualne narzędzia).
2. Jako autor odpowiedz na każdy zarzut: przyznajesz rację (→ propozycja zmiany)
   albo bronisz (→ punkt sporny z argumentem w 1 zdaniu). Odeślij do GPT.
3. Koniec: brak nowych zarzutów ALBO 3 rundy. Efekt fazy: lista **propozycji
   uzgodnionych** + lista **punktów spornych** (zarzut GPT vs obrona Claude, po 1 zdaniu).

## Faza 2 — SĘDZIA rozstrzyga (świeże wywołanie, bez historii dyskusji)

NOWE wywołanie GPT. Dostaje TYLKO: dokument (oryginał, nieruszony) + listę propozycji
i punktów spornych + 2-3 pliki kontekstu (profil klienta, CLAUDE.md klienta) +
**ŹRÓDŁA PIERWOTNE = wyłącznie głos KLIENTA:** transkrypcje rozmów + wiadomości
OD klienta zapisane w repo (jego maile, SMS-y, wklejki z komunikatorów — pliki typu
`wiadomosc_od_*`). NIE są źródłem pierwotnym: wiadomości i szkice użytkownika
(`wiadomosc_do_*`, drafty w skrzynce) ani nasze notatki/podsumowania — to zapis
NASZYCH intencji, nie jego słów. Ścieżki plików podać Codexowi — czyta je sam
w sandboxie read-only. Zadania:

1. Rozstrzygnij KAŻDY punkt: WPROWADZIĆ / ODRZUCIĆ / DECYZJA UŻYTKOWNIKA — z uzasadnieniem
   w 1 zdaniu.
2. Dodatkowo sprawdź sam dokument: skrajne przypadki i brudne dane klienta;
   AI/agent tam, gdzie wystarczy reguła lub prosty workflow; decyzje bez uzasadnień
   ("zgoda, żeby skończyć dyskusję"); braki blokujące budowę/odbiór; nazwy
   modeli/API/narzędzi i ceny z pamięci zamiast z aktualnych źródeł (z datą).
2a. **Zgodność z rzeczywistością klienta (na podstawie transkrypcji):** czy plan
   odpowiada temu, co klient FAKTYCZNIE powiedział — wskaż z cytatem: (i) obietnice
   planu bez pokrycia w rozmowach, (ii) potrzeby/obawy klienta wypowiedziane w
   rozmowie a pominięte w planie, (iii) miejsca, gdzie plan zakłada zachowanie
   klienta, którego on nie zadeklarował.
2b. **Zgodność z aktualnymi najlepszymi praktykami — zrób WŁASNY research w sieci**
   (web search: oficjalne docs, cenniki, renomowane źródła inżynierskie) dla
   kluczowych decyzji architektonicznych i założeń narzędziowych planu; każdy
   rozjazd wskaż ze źródłem (link) i datą sprawdzenia. Nie oceniaj z pamięci.
3. Werdykt ogólny: `ZATWIERDZAM (ze zmianami: …) | DO POPRAWKI`.

**Dieta kontekstowa sędziego (znana wpadka: sędzia PRD padł na braku kontekstu):**
sędzia dostaje wsad INLINE i ODCHUDZONY — dokument + KRÓTKIE podsumowanie dyskusji
(nie surowe pliki rund) + tylko niezbędne wyciągi wzorców (nie cały CLAUDE.md).
Czytanie pełnych transkrypcji klienta = TYLKO w sędzim PLANU (zadanie zgodności);
sędzia PRD ich nie powtarza — zgodność z klientem osądzono poziom wyżej. Research
ograniczaj do wskazanych tematów. Za duży wsad = brak werdyktu i spalone tokeny.

## Raport dla użytkownika — krótko, i CZEKAJ

Pokaż zwięzłą tabelę (bez ścian tekstu):

| # | Propozycja zmiany | Skąd (dyskusja/sędzia) | Werdykt sędziego (1 zdanie) |

+ werdykt ogólny w 1-2 zdaniach. Zapytaj: **"które zmiany wprowadzam?"** (numery /
"wszystkie za wprowadzeniem" / "żadne"). Dopiero po wyborze nanieś WYBRANE zmiany
na dokument — nic poza nimi. Koniec hejtu; bez kolejnych rund, chyba że użytkownik poprosi.
