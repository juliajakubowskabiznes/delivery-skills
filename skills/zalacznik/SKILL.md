---
name: zalacznik
description: >
  Generuje załącznik do umowy dla JEDNEGO kroku wdrożenia — z zatwierdzonego PRD kroku,
  językiem klienta (zero żargonu). Wysyłany RAZEM z umową (informacyjnie, nie do
  podpisu). Używaj po zatwierdzeniu PRD kroku — albo gdy użytkownik mówi: "załącznik do
  umowy", "zakres do umowy kroku X". Argument: ścieżka PRD kroku.
---

# Załącznik do umowy — zakres kroku

Źródło treści: zatwierdzony PRD kroku ($ARGUMENTS) + plan całości. Niczego nie wymyślaj
ponad PRD — załącznik to PRD przetłumaczony na język klienta, nie nowy dokument.

Zasady: język prosty, terminy ze słownika klienta (obce pojęcia z krótkim wyjaśnieniem),
**zero cen i marż** (ceny żyją w umowie/proformie), kryteria odbioru 1:1 z PRD
(konkretny test na danych klienta, nie "działa poprawnie").
**Warunki (to dokument umowny): kryteria odbioru NIGDY nie są warunkowe — podpis
i odbiór nie mogą wisieć na żadnej odpowiedzi/dostępie klienta. Warunkowe są
wyłącznie BONUSY, a każdy warunek ma zapisane OBIE gałęzie** („jeśli dostarczysz X
→ dodatkowo Y, bez zmiany ceny i terminu; brak X → nie wpływa na odbiór").
Opcje przyszłe/dokładki w ogóle nie wchodzą do załącznika.

**Miejsce docelowe = edytowalny dokument umowy u dostawcy chmury (np. Google Docs),
NIE repo.** Źródłem prawdy umowy i załącznika jest dokument umowy (link w sekcji
„🔗 Linki zewnętrzne" CLAUDE.md klienta) — załącznik twórz/aktualizuj bezpośrednio
w dedykowanej zakładce tego dokumentu. W repo NIE trzymamy kopii treści; roboczy plik
do bramek (hejt) trzymaj w scratchpadzie, nie w folderze klienta. Stara wersja plikowa
w repo → archiwum klienta.

## Szablon (7 sekcji + opcjonalna 8.)

```markdown
# Załącznik nr _ — Zakres wdrożenia: Krok <n> — <nazwa>

## 1. Cel projektu
<1-3 zdania: jaki problem rozwiązuje ten krok i po czym Zamawiający pozna efekt>

## 2. Jak działa system — krok po kroku
<numerowana lista prostym językiem: co wchodzi, co system robi, co widzi człowiek;
z PRD sekcja mechaniki — bez żargonu technicznego>

## 3. Zakres funkcjonalny — co wchodzi / kryteria odbioru
<lista funkcji + kryteria odbioru 1:1 z PRD: konkretny, sprawdzalny test na danych
Zamawiającego. Procedura odbioru: Zamawiający testuje i zgłasza uwagi PISEMNIE
w ciągu 7 dni od przekazania; brak uwag w tym terminie = krok odebrany.>  <!-- [PROPOZYCJA: okno 7 dni — usuń jeśli nie chcesz] -->

## 4. Co nie wchodzi w zakres
<jawna lista wyłączeń + zdanie: prace spoza zakresu wymagają odrębnej wyceny
i zlecenia (np. jako kolejny krok wdrożenia)>

## 5. Narzędzia i usługi zewnętrzne / wymagania po stronie Zamawiającego

| Narzędzie/usługa | Do czego służy | Kto płaci | Koszt mies. (szacunek) |
|---|---|---|---|
<każde narzędzie jedną linią; koszty z AKTUALNYCH cenników (sprawdź, nie z pamięci —
ceny się dezaktualizują), z datą sprawdzenia pod tabelą; na dole suma szacunkowa/mies.
Do tego: co Zamawiający zapewnia PRZED wdrożeniem (dostępy, konta, materiały) —
lista-checklista. KAŻDA pozycja checklisty musi być wykonalna przez klienta
SAMODZIELNIE (instrukcja/wideo per czynność) — bez spotkania; zasada
zero spotkań operacyjnych.>

## 6. Ograniczenia techniczne
<znane ograniczenia prostym językiem, w tym niedeterministyczność AI i wbudowane
kontrole (sumy kontrolne, weryfikacja człowieka); granice gwarancji jakości>

## 7. Warunki działania / procedura utrzymaniowa
<co musi być spełnione, żeby system działał (nie zmieniać struktury arkusza, aktywne
konta itd.); co robić gdy błąd: kto reaguje, jak zgłaszać. Utrzymanie rozwojowe =
odrębna umowa — tu tylko warunki poprawnego działania.>

## 8. Przetwarzanie danych  <!-- [PROPOZYCJA: cała sekcja — usuń jeśli nie chcesz] -->
<jakie dane przechodzą przez system, przez jakie usługi (w tym AI), z jaką konfiguracją
prywatności (np. dostawcy bez trenowania na danych)>
```

Po wygenerowaniu: przejrzyj z użytkownikiem propozycje oznaczone [PROPOZYCJA] — zostawia
albo tnie. Dokument NIE idzie do klienta bez akceptacji użytkownika.

**Kryteria odbioru — filtr doboru:** z PRD do załącznika wchodzą TYLKO kryteria,
które są (1) mierzalne dla użytkownika I klienta — klient umie sprawdzić sam, na swoich
danych, bez zaglądania w bebechy; (2) proste, zero-jedynkowe; (3) **100% PEWNE —
dowozimy je NA PEWNO, bo wynik kontrolujemy w całości sami**. Test pewności per
kryterium: „czy istnieje realny scenariusz, w którym to nie przechodzi mimo dobrze
wykonanej roboty?" — jeśli TAK (zależy od jakości odczytu AI, od zachowania klienta,
od zewnętrznej usługi, od progu %), kryterium NIE wchodzi do załącznika. Zamiast niego
wpisz to, co kontrolujemy: mechanizm/zachowanie systemu (np. „niezgodna suma →
dokument odłożony, nie wchodzi do faktury"), nie wynik statystyczny (np. „95%
dokumentów odczytanych poprawnie"). Metryki jakościowe (progi %, czasy) zostają
w PRD jako standard wewnętrzny — do umowy nie wpisuje się liczby, której się nie
kontroluje.
