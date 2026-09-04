---
name: init-klienta
description: >
  Tworzy CLAUDE.md w folderze klienta — żywy dokument JAK: zasada zapisywania błędów,
  struktura folderu, architektura, wzorce techniczne.
  Używaj na starcie pracy z nowym klientem (krok 0 workflow) albo gdy użytkownik mówi:
  "załóż CLAUDE.md dla klienta", "zainicjuj klienta". Argument: ścieżka folderu klienta.
---

# Init klienta — CLAUDE.md

Sprawdź folder klienta ($ARGUMENTS). CLAUDE.md już istnieje → NIE nadpisuj; co najwyżej
zaproponuj brakujące sekcje z szablonu i czekaj na zgodę. Nie istnieje → stwórz według
szablonu niżej, wypełniając z profilu klienta (jeśli jest); formę zwracania się
(oficjalna/na Ty) sprawdź w profilu lub pamięci — nie zgaduj, w razie braku zapytaj
użytkownika. Strukturę folderu WYKRYJ (`ls`) — opisz istniejącą, nie narzucaj nowej.

## Szablon

```markdown
# CLAUDE.md — <Klient> (osoba kontaktowa: <imię>, <branża>, <forma: oficjalna/na Ty>)

## 🔁 Zasada nadrzędna: zapisuj KAŻDY błąd TUTAJ (self-improvement)

Każdy popełniony/napotkany błąd w tym projekcie — techniczny, procesowy, komunikacyjny —
ZAPROPONUJ jako wpis do tego pliku. Techniczne → „🧠 Wzorce techniczne", reszta →
„📓 Dziennik błędów". Format: co poszło nie tak → prawdziwa przyczyna → jak unikać.
Przy nowym problemie NAJPIERW sprawdź, czy to nie znany błąd.

**⛔ BRAMKA: ŻADEN zapis do CLAUDE.md bez zgody użytkownika.**
Dotyczy wszystkiego — błędów, wzorców, linków, zasad. Tryb: pokaż proponowany wpis
(1-3 linie) w momencie, gdy powstaje powód → czekaj na „tak" → dopiero wtedy dopisujesz.
JEDNA propozycja na raz (jeden wątek do decyzji), nie lista.
Minimalizm: plik ma być KRÓTKI — reguła, gdy błąd się powtarza lub jest kosztowny;
usuwaj duplikaty i nieaktualne.

## Wzorzec struktury folderu — GDZIE CO LĄDUJE

Żadnych luźnych plików w folderach faz. Faza aktualna: <faza>.
<wykryta struktura z ls + jedna linia opisu per podfolder>
- Wiadomości do klienta = .txt, forma <oficjalna/na Ty>.
- Nieaktualne pliki → jedno archiwum klienta (<99_archiwum>).
- Trzymaj folder CLEAR: dokument nieaktualny (zastąpiony, niewysłany-przeterminowany,
  sprzeczny z decyzjami) → do archiwum OD RAZU, w tej samej sesji; do nazwy dopisz
  `_STARY`/`_NIEWYSLANY`. Foldery robocze pokazują wyłącznie aktualny stan.
  Przed przeniesieniem zgrepuj folder po nazwie pliku i zaktualizuj każde odwołanie
  (inne dokumenty linkują między sobą) — martwe odnośniki to znana klasa błędu.

## 🏗️ Architektura
<wypełnia workflow po decyzjach architektonicznych — stack (co i czemu nie prościej),
model danych/źródła prawdy, guardraile (sumy kontrolne, HITL, error handling z
notyfikacją). Do tego czasu: (jeszcze nie projektowano)>

## 🔗 Linki zewnętrzne — JEDNO miejsce (= źródła prawdy)
<tabela: Co | Link/ID | Uwagi. Wiersz per zewnętrzny zasób: tablica map procesów,
arkusze i dokumenty klienta (umowa!), instancja automatyzacji + ID workflow, foldery
w chmurze, nagrania rozmów, portale i API klienta (z zaznaczeniem czy dostęp JEST
przekazany).
Zasady: (1) URL żyje TYLKO tutaj — inne pliki odsyłają do tej sekcji, nie kopiują;
(2) nowy link → dopisz wiersz OD RAZU, w tej samej sesji; (3) gdy istnieje kilka kopii
tego samego zasobu — oznacz, która jest do edycji, a która stara;
(4) wiersz = KRÓTKI (link, status, odsyłacz) — fakty projektowe (cenniki, limity,
ustalenia) i uzasadnienia decyzji NIE tu, tylko w CONTEXT.md / ADR / specu. CLAUDE.md
ładuje się w każdej sesji — każdy zbędny akapit to stały podatek kontekstowy.>

## 📓 Dziennik błędów — wpadki PROCESOWE i komunikacyjne
(pusty — rośnie w trakcie; przykłady wpisów: zła forma zwracania się, plik w złym
miejscu, złe ustalenie z klientem)

## 🧠 Wzorce techniczne — reguły z debugowania (objaw → przyczyna → odporny fix)
(pusty — rośnie w trakcie; przykłady wpisów: "ID modeli bierz z API, nie z pamięci",
"formularz X wysyła pola jako field-0". Od startu obowiązuje jedna reguła: każdy
strumień danych z AI ma niezależną sumę kontrolną z tego samego dokumentu — bez niej
nie wolno wystawiać wyników klientowi)
```

## Hook pilnujący bramek (twarda pamięć — nie tonie w kontekście)

Razem z CLAUDE.md utwórz `.claude/settings.json` w folderze klienta (jeśli już istnieje —
scal, nie nadpisuj). Hook odpala się po każdym Write/Edit i mechanicznie przypomina
o bramkach — działa w każdej sesji startującej w folderze klienta, niezależnie od tego,
co sesja pamięta:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "f=$(jq -r '.tool_input.file_path // \"\"'); case \"$f\" in */.scratch/*spec.md|*/00_architektura/plan_*.md|*/00_architektura/prd_*.md) echo 'PILNUJ POZIOMU DOKUMENTU: czy zapisany fragment to reguła procesu/metody albo zasada wielokrotnego użytku? Jeśli TAK — nie zostawiaj jej w tym dokumencie: reguła→skill, zasada na zawsze→memory, fakt→CONTEXT.md, decyzja→ADR, szczegół wykonawczy→seed/PRD. Spec/plan trzyma wyłącznie treść projektu. ⛔ BRAMKA NANIESIEŃ: jeśli dokument jest ZATWIERDZONY, każda zmiana wymagała tabeli ZA/PRZECIW i zgody użytkownika PRZED edycją — bez zgody COFNIJ tę edycję i pokaż propozycję.' >&2; exit 2;; *CLAUDE.md) echo '⛔ BRAMKA CLAUDE.md: każdy wpis do CLAUDE.md wymaga zgody użytkownika z TEJ rozmowy. Jeśli ten zapis nie miał zgody — COFNIJ edycję i pokaż propozycję (1-3 linie, JEDNA na raz).' >&2; exit 2;; esac; exit 0"
          }
        ]
      }
    ]
  }
}
```

Po utworzeniu: pokaż użytkownikowi i dopiero po jego "ok" uznaj krok za zamknięty.
