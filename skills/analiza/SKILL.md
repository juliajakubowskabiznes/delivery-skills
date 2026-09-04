---
name: analiza
description: >
  Analiza materiałów klienta na świeżym kontekście — przed grillem. Czyta cały folder
  klienta i wraca z raportem: co wiemy, sprzeczności (z rekomendacją), luki. Używaj gdy
  użytkownik mówi "przeanalizuj materiały [klienta]", "co mamy o X" — oraz jako pierwszy
  krok /workflow, gdy materiały klienta nie były jeszcze czytane. Argument: ścieżka
  folderu klienta.
context: fork
background: false
---

# Analiza materiałów klienta

Pracujesz w forku bez historii rozmowy — oceniasz materiały takimi, jakie są na dysku.
Folder klienta: $ARGUMENTS (pusty/niejasny → zwróć błąd z listą folderów klientów,
nie zgaduj).

Przeczytaj: profil, analizę techniczną, lokalny CLAUDE.md (pułapki techniczne!),
wyniki audytu, notatki, istniejące PRD/plany. Pomiń archiwum. Przy dużej liczbie plików
czytaj wybiórczo: najpierw README/nagłówki, głębiej tam, gdzie decyzyjne.

Raport ZAPISZ do folderu planowania bieżącego wdrożenia
(np. `<folder klienta>/<wdrożenie>/03_planowanie/analiza_materialow_<RRRR-MM-DD>.md`)
I zwróć jego treść — plik jest odporny na crash sesji, a grill/wayfinder mogą do niego
wracać. Struktura raportu (zwięzły, wszystko ze źródłem = ścieżka pliku):

1. **Co wiemy o procesie** (5-10 punktów)
2. **Co już zdecydowano** (z datami)
3. **Liczby kluczowe** (wolumeny, czasy, kwoty)
4. **Sprzeczności** — per sztuka: plik A [data] mówi X, plik B [data] mówi Y
   + Twoja rekomendacja rozstrzygnięcia (którą wersję i czemu)
5. **Pułapki techniczne** z CLAUDE.md klienta (top 5)
6. **Luki** — czego materiały NIE mówią, a grill musi dopytać

Nie nadinterpretuj transkrypcji (cytat ≠ intencja), niczego nie zmyślaj — brak danych
to luka, nie założenie.

**Hierarchia wiarygodności źródeł (twarda):** FAKTY = tylko to, co przyszło OD KLIENTA
(transkrypcje rozmów, maile, pliki ofert/wzory) oraz zweryfikowany stan techniczny
(działający workflow, configi). Dokumenty wygenerowane przez agenta w poprzednich
sesjach (analizy, oceny, PRD, drafty maili) = HIPOTEZY — ich twierdzenia o procesie
klienta NIE są faktami, dopóki nie wskażesz źródła pierwotnego. Decyzje użytkownika
z poprzednich sesji (zapisane w handoffach/dokumentach) raportuj OSOBNO jako ROBOCZE —
podjęte na ówczesnych, niezweryfikowanych założeniach, do re-potwierdzenia — nie jako
fakty. Uzgodnione z klientem jest TYLKO to, co klient napisał/powiedział (niewysłany
draft ≠ akceptacja). Wszystko bez źródła pierwotnego → luka/pytanie.

**Prototypy i vibe-code raportuj jako DOWODY, nie decyzje.** Działający prototyp
dowodzi wykonalności, poprawności logiki i realności edge case'ów — ale NIE tego, że
jego wybory narzędzi/modeli/struktury są najlepsze (mogły wejść awaryjnie, bez
świadomej decyzji użytkownika). Wybór bez śladu porównania opcji = decyzja DO PODJĘCIA
w grillu, z prototypem jako materiałem dowodowym.

**Istniejące plany i PRD to HIPOTEZY, nie prawda.** Mogły być słabe, nieaktualne albo
przeznaczone do wymiany — spójny dokument nie znaczy dobry. W raporcie oznacz STATUS
każdego dokumentu planistycznego (aktualny / sprzeczny z nowszymi decyzjami / do
wymiany wg użytkownika — sprawdź pamięć i CLAUDE.md klienta) i nie przepisuj ich decyzji
jako obowiązujących: do sekcji "Co już zdecydowano" trafia tylko to, co ma jawne
źródło i datę — reszta idzie do luk, do potwierdzenia w grillu. Fakty z materiałów
źródłowych (audyt, transkrypcje, prototyp) ≠ decyzje ze starych planów: pierwsze
raportuj normalnie, drugie zawsze ze statusem.
