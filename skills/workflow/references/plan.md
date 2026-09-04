# Krok 2 — PLAN: szczegółowe reguły

## Poziom 1 — plan całości (raz na projekt)

Duża/mglista całość → `/wayfinder` (rozstrzygaj decyzje, aż droga jasna) → JEDEN
dokument: architektura całości + mapa wszystkich kroków (per krok: cel, zakres, efekt,
kryterium odbioru). Cała droga widoczna, nawet gdy zamówiona jest część kroków.

**Równa wysokość:** każdy krok opisany na tym samym, ŚREDNIM poziomie — żadnych
szczegółów budowy (węzły, markery, przepływy) nawet dla kroku najbliższego czy
najlepiej udokumentowanego. Powód: ilość materiału o kroku ≠ ilość miejsca w planie;
szczegóły należą do PRD kroku. Bez tej granicy model rozpisuje to, o czym ma najwięcej
kontekstu.

**Ścieżka bez wayfindera:** całość mała i jasna ALBO już rozstrzygnięta grillem
(decyzje + ADR-y, droga jasna) → plan pisze od razu `/to-spec`. Warunek
bezpieczeństwa: decyzja z grilla podjęta BEZ danych (sama argumentacja w rozmowie)
nie jest przepisywana do planu na wiarę — przy pisaniu dociągnij źródło/aktualne
praktyki; nie da się uźródłowić → jawne ZAŁOŻENIE z planem weryfikacji. Sędzia
w hejcie sprawdza to pytaniem "źródła czy pamięć?".

## Poziom 2 — PRD kroku (przed budową każdego kroku)

`/to-spec` dla JEDNEGO kroku: realizuje plan całości i odsyła do niego (nie powtarza).
Kroki 3-6 workflow wykonujesz PER KROK.

## Higiena dokumentów

- Nowy dokument = jedyna aktualna wersja; po zatwierdzeniu stare wersje → archiwum
  klienta (nic nie kasujemy) — jeden dokument prawdy, zero "miliona wersji".
- **Poziom dokumentu — dokument trzyma tylko swoją wysokość:** plan/spec = treść
  projektu; reguła procesu/metody → skill; zasada na zawsze → memory; fakt/pojęcie →
  CONTEXT.md; decyzja z uzasadnieniem → ADR; szczegół wykonawczy → seed/PRD.
  Test na każdy fragment: "obowiązuje też u innego klienta / przy innym kroku?"
  → TAK = nie tu. Hook w settings.json klienta przypomina przy edycji spec/plan/prd.
