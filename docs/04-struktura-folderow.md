# Struktura folderów — jak zakładam i porządkuję folder klienta

Skille zakładają konkretny sposób trzymania plików. Nie jest to estetyka: agent szuka
materiałów po ścieżkach, a człowiek po tygodniu przerwy musi w 10 sekund odpowiedzieć na
pytanie „która wersja jest aktualna". Ten dokument opisuje wzorzec i **reguły, które
utrzymują go w ryzach** — bo folder projektu nie psuje się od razu, tylko po trzech
tygodniach, gdy leży w nim pięć wersji tego samego.

---

## Trzy pytania przed utworzeniem albo przeniesieniem czegokolwiek

1. **Czy to ma numer?** Każdy podfolder ma prefiks (`00_`, `01_`, `02_`…), który wymusza
   kolejność. Nowy folder dostaje **kolejny wolny numer**. Po usunięciu czegoś **przenumeruj
   resztę**, żeby nie było dziur (00, 01, 02, 03 — nie 00, 01, 04, 07).
2. **Czy nie leży luzem?** W folderze, który ma podfoldery, **nie zostawiamy pojedynczych
   plików**. Plik trafia do istniejącego podfolderu albo do nowego, ponumerowanego.
3. **Czy to na pewno tu?** Jeśli plik nie pasuje do żadnej kategorii — pytasz, zamiast
   zostawiać „na razie tutaj". „Na razie tutaj" jest trwałe.

---

## Anatomia folderu klienta

```
<klient>/
├── CLAUDE.md              ← pamięć projektu (patrz niżej)
├── .claude/
│   └── settings.json      ← hook pilnujący bramek
├── 01_wdrozenie_1/        ← pierwszy sprzedany krok/etap
├── 02_wdrozenie_2/        ← drugi krok — osobny folder, osobna umowa
└── 99_archiwum/           ← jedno archiwum na klienta, na poziomie głównym
```

**Jedno wdrożenie = jeden numerowany folder.** Nie miesza się dwóch sprzedanych kroków
w jednym drzewie, nawet jeśli dotyczą tego samego systemu — mają osobne umowy, osobne
kryteria odbioru i osobne archiwum ustaleń.

## Anatomia folderu wdrożenia

```
02_wdrozenie_2/
├── 00_kontekst.md            ← stan i plan: gdzie jesteśmy, co dalej
├── 01_rozmowy/               ← transkrypcje, notatki, wiadomości do klienta
│   └── 02_umowa/             ← wersje umowy, uwagi klienta, odpowiedzi
├── 02_materialy/             ← wszystko, co przyszło OD klienta
│   ├── 01_<temat>/
│   └── 02_<temat>_RRRR-MM-DD/
├── 03_planowanie/            ← raport analizy, plan całości, PRD, ankiety
├── 04_techniczne/            ← skrypty, dowody wykonalności, konfiguracje
└── 99_archiwum/              ← nieaktualne pliki TEGO wdrożenia
```

**Co gdzie ląduje — i dlaczego akurat tam:**

| Folder | Zawartość | Reguła |
|---|---|---|
| `00_kontekst.md` | stan wdrożenia, faza, następny krok | pierwszy plik, który agent czyta przy wznowieniu sesji |
| `01_rozmowy/` | transkrypcje (`.txt`), notatki (`.md`), wiadomości do klienta (`.txt`) | wiadomości **zawsze `.txt`** — żeby dało się skopiować i wysłać bez czyszczenia formatowania |
| `02_materialy/` | pliki od klienta: PDF-y, arkusze, zrzuty ekranu, jego odpowiedzi | to jest **warstwa faktów** — źródło pierwotne dla `/analiza` i dla sędziego w `/hejt` |
| `03_planowanie/` | raport z `/analiza`, plan całości, PRD, ankiety z `/to-questionnaire` | tu zapisują domyślnie skille wdrożeniowe |
| `04_techniczne/` | skrypty, dowody wykonalności na danych klienta, configi | dowód wykonalności trzyma się blisko planu, nie w repo kodu |
| `99_archiwum/` | wszystko nieaktualne | jedno na wdrożenie; nic nie kasujemy |

**Daty w nazwach.** Pliki, które mają wersje albo dotyczą konkretnego zdarzenia, dostają
`_RRRR-MM-DD` na końcu nazwy: `notatka_meet_2026-07-29.md`,
`uwagi_klienta_do_umowy_2026-08-13.md`. Sortują się chronologicznie i od razu widać,
która runda uwag jest ostatnia.

**Podfoldery materiałów też mają datę**, gdy klient dosyła kolejne partie:
`02_nowe_materialy_2026-08-06/`. Bez tego po trzech dosyłkach nie wiadomo, co jest czym.

---

## Reguły higieny — co utrzymuje ten porządek

### 1. Trzymaj folder CLEAR

Dokument nieaktualny — **zastąpiony**, **niewysłany i przeterminowany**, albo **sprzeczny
z podjętymi decyzjami** — idzie do archiwum **od razu, w tej samej sesji**. Nie „przy
najbliższych porządkach". Do nazwy dopisujesz `_STARY` albo `_NIEWYSLANY`, żeby po
otwarciu archiwum nie zgadywać, dlaczego tam trafił.

Foldery robocze pokazują **wyłącznie aktualny stan**. To jest cała funkcja tej reguły:
jeśli w folderze roboczym leżą trzy wersje, każda kolejna sesja agenta (i każdy powrót
człowieka po tygodniu) zaczyna się od śledztwa.

### 2. Przed przeniesieniem — zgrepuj odwołania

Dokumenty linkują do siebie. Przed przeniesieniem pliku do archiwum przeszukujesz folder
po jego nazwie i aktualizujesz każde odwołanie. Martwe odnośniki to znana klasa błędu —
agent trafia na link, nie znajduje pliku i albo się zatrzymuje, albo (gorzej) uzupełnia
lukę domysłem.

### 3. Nowy dokument = jedyna aktualna wersja

Po zatwierdzeniu nowej wersji stara idzie do archiwum. Nic nie kasujemy, ale nie
utrzymujemy też „miliona wersji" w folderze roboczym. Jeden dokument prawdy.

### 4. Poziom dokumentu — każdy plik trzyma tylko swoją wysokość

To reguła, która najczęściej ratuje projekt przed dokumentami-workami. Test na każdy
fragment, który chcesz gdzieś zapisać: **„czy to obowiązuje też u innego klienta / przy
innym kroku?"**. Jeśli TAK — to nie należy do tego dokumentu:

| Rodzaj treści | Miejsce |
|---|---|
| treść projektu (co budujemy dla tego klienta) | plan / spec / PRD |
| reguła procesu albo metody | skill |
| zasada obowiązująca zawsze | pamięć agenta |
| fakt albo pojęcie | `CONTEXT.md` |
| decyzja z uzasadnieniem | ADR |
| szczegół wykonawczy | PRD / seed kryterium |

Pilnuje tego **hook** — patrz niżej.

### 5. Archiwum jest jedno, na poziomie głównym

Nieaktualne pliki klienta lądują w `99_archiwum/`. Nie tworzy się osobnych archiwów
w podfolderach faz (`04_techniczne/.../99_archiwum_stare/` to antywzorzec) — po miesiącu
masz cztery archiwa i szukasz w każdym.

---

## CLAUDE.md klienta — pamięć projektu

Zakładany przez [`/init-klienta`](../skills/init-klienta/SKILL.md), opisany szczegółowo
w [`docs/02-skille-wdrozeniowe.md`](02-skille-wdrozeniowe.md). Tu tylko rola w strukturze:

- **Struktura folderu jest w nim opisana** — agent nie musi jej odgadywać z `ls`
  przy każdej sesji.
- **Linki zewnętrzne żyją tylko tutaj** — jedna tabela na wszystkie arkusze, dokumenty
  umowy, instancje narzędzi, nagrania. Inne pliki *odsyłają* do tej sekcji. Powód: URL
  skopiowany do trzech dokumentów to trzy miejsca do zaktualizowania, gdy link się zmieni
  — i dwa nieaktualne, gdy zaktualizujesz jedno.
- **Dziennik błędów i wzorce techniczne rosną w trakcie** — objaw → prawdziwa przyczyna →
  odporny fix. Ta sama klasa błędu nie ma kosztować drugi raz.

**CLAUDE.md ładuje się w każdej sesji**, więc każdy zbędny akapit to stały podatek
kontekstowy. Plik ma być krótki: reguła wchodzi, gdy błąd się powtórzył albo był
kosztowny. Duplikaty i nieaktualne wpisy się usuwa.

## Hook — porządek pilnowany mechanicznie

W `.claude/settings.json` folderu klienta siedzi `PostToolUse` na `Write|Edit`, który
przy zapisie do plików spec/plan/PRD oraz do CLAUDE.md wychodzi z kodem 2 i przypomina
o dwóch rzeczach: **pilnuj poziomu dokumentu** (reguła 4 wyżej) i **nie zmieniaj
zatwierdzonego dokumentu bez zgody**.

Dlaczego hook, a nie zdanie w instrukcji: instrukcja tonie w długiej sesji dokładnie
wtedy, gdy jest najbardziej potrzebna — po godzinie pracy, przy dwudziestym zapisie.
Hook odpala się mechanicznie, niezależnie od tego, co sesja pamięta.

---

## Skrót do powieszenia nad biurkiem

- Nowy plik → **numerowany podfolder**, nigdy luzem.
- Nieaktualny plik → **archiwum od razu**, z dopiskiem `_STARY`.
- Wiadomość do klienta → **`.txt`**.
- Wersja albo zdarzenie → **data `_RRRR-MM-DD` w nazwie**.
- Link zewnętrzny → **tabela w CLAUDE.md**, nigdzie indziej.
- Fragment, który obowiązuje też u innego klienta → **nie do tego dokumentu**.
