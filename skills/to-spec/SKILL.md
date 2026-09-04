---
name: to-spec
description: Turn the current conversation into a spec and publish it to the project issue tracker — no interview, just synthesis of what you've already discussed.
disable-model-invocation: true
---

This skill takes the current conversation context and codebase understanding and produces a spec (you may know this document as a PRD). Do NOT interview the user — just synthesize what you already know.

**JĘZYK: PRD piszemy PO POLSKU** (dokument czyta klient i użytkownik). Nagłówki sekcji po polsku
(Problem / Cel i metryka sukcesu / Historie użytkownika / Kryteria akceptacji (EARS) /
Przypadki brzegowe i błędy / Decyzje implementacyjne / Decyzje testowe / Poza zakresem /
Nierozstrzygnięte pytania (DO DOPYTANIA) / Notatki). Wzorce
EARS w polskich odpowiednikach — struktura zostaje, język polski:
„System MUSI X" · „**GDY** <zdarzenie>, system MUSI X" · „**JEŚLI** <błąd>,
**TO** system MUSI X" · „**PODCZAS GDY** <stan>, system MUSI X" ·
„**TAM GDZIE** <opcja>, system MUSI X".

**POZIOM DOKUMENTU (najczęściej łamana zasada):** spec zawiera WYŁĄCZNIE
treść projektu na swojej wysokości. Test przed zapisaniem każdego fragmentu:
„czy to obowiązuje też u innego klienta / przy innym kroku?" → TAK = wynieś
NA BIEŻĄCO: reguła procesu/metody → skill, zasada na zawsze → memory, fakt/pojęcie
→ CONTEXT.md, decyzja z uzasadnieniem → ADR, szczegół wykonawczy → seed/PRD niżej.
Zero wykładów o procesie w dokumencie — proces żyje w /workflow.

Issue tracker: local markdown wg `docs/agents/issue-tracker.md` w folderze klienta —
wersja robocza specu w `.scratch/<slug>/spec.md`, po zatwierdzeniu przenosisz do
folderu planowania bieżącego wdrożenia (np. `<folder klienta>/<wdrożenie>/03_planowanie/`).

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template below, then publish it per `docs/agents/issue-tracker.md` (draft → `.scratch/<slug>/spec.md`).

4. **The spec is NOT done while "Nierozstrzygnięte pytania" has open items.** A gap you could not resolve from sources is ALWAYS a question — never a silent assumption baked into the sections above.

**WARIANT KLIENCKI: spec budujemy Z KLIENTEM, nie z głowy.** Routing pytań:

- **DOMYŚLNIE każda niepewność i każde założenie o procesie klienta → pytanie do
  KLIENTA** (`[KLIENT]`). Wszystkie idą przez /to-questionnaire jako JEDEN scalony
  dokument (scal z pytaniami już wiszącymi w draftach do klienta — bez duplikatów).
  Nie zgaduj „pewnie tak robią" — jak nie wiesz, jak klient pracuje, PYTASZ klienta.
- Do użytkownika (`[UŻYTKOWNIK]`, jedno pytanie na raz, opcje + rekomendacja pierwsza)
  idą TYLKO decyzje biznesowe po jego stronie: cena, krój etapów, kolejność sprzedażowa,
  ryzyka, które bierzemy na siebie.
- W treści specu każde miejsce zależne od odpowiedzi oznacz `[DO POTWIERDZENIA → pyt. N]`
  — żadna sekcja nie udaje gotowej.
- Po odpowiedziach klienta: wpleć odpowiedzi, usuń znaczniki, zaktualizuj listę pytań.
  Rund może być kilka — spec jest gotowy dopiero, gdy zniknął OSTATNI znacznik.

5. **Sweep źródeł przed każdym zapisem specu:** przejdź zdanie po zdaniu przez każde
twierdzenie o procesie klienta i sprawdź, czy ma źródło pierwotne (transkrypcja /
materiał od klienta / odpowiedź klienta / decyzja użytkownika z TEJ rozmowy). Brak źródła →
zdanie wylatuje z sekcji i staje się pytaniem `[KLIENT]`. Zero wyjątków.

**Zasada: NIE domyślaj się za klienta.** Każde twierdzenie
o procesie klienta w spec musi mieć źródło pierwotne (transkrypcja / materiał od
klienta / decyzja użytkownika z datą). Dokumenty wygenerowane przez agenta w poprzednich
sesjach (analizy, stare PRD) NIE są źródłem pierwotnym. Brak źródła = pytanie
w „Nierozstrzygnięte pytania", nigdy cicha treść sekcji wyżej.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Cel i metryka sukcesu

The business goal and ONE measurable number that tells you it worked — taken from the
WHY established during the grill (client's money/time). "Skrócenie czasu wyceny z 2 dni
do 2 h", not "lepszy UX". If no number can be named yet, that is a [DO DOPYTANIA] item —
never a vague placeholder. (The solution itself is described by User Stories and
Implementation Decisions — no separate solution essay.)

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Acceptance Criteria (EARS)

Write acceptance criteria as single-sentence EARS patterns — unambiguous, each converts 1:1 into a test:

- Ubiquitous: "The <system> shall <X>"
- Event: "**When** <trigger>, the <system> shall <X>"
- State: "**While** <state>, the <system> shall <X>"
- Unwanted: "**If** <failure>, **then** the <system> shall <X>"
- Optional: "**Where** <feature present>, the <system> shall <X>"

Every step that reads data with AI MUST have at least one Unwanted criterion (what happens when the read fails or sums don't reconcile — silent wrong output is the worst failure class). EARS handles 0-3 preconditions; when a table or diagram is clearer, use it instead of forcing the pattern.

## Przypadki brzegowe i błędy

Walk the failure sweep explicitly — this is where 80% of unspoken assumptions live: empty/missing data, wrong format, service/API down, duplicates, partial input, the user undoing or submitting twice, an item that matches nothing. List each case and close it ONE of two ways: a pointer to its EARS criterion above (usually Unwanted), or an explicit entry in Out of Scope. No case may be left undecided. Do NOT restate the criterion here — this section is the coverage map, EARS is the contract (one source of truth).

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Three hard rules for every decision in this section:

- **Look up current best practices:** for each significant architectural choice, search the web and check how it is done TODAY — trusted sources only: official docs of the tool/API, vendor guides, engineering blogs of reputable companies, high-star GitHub repos. Cite the source next to the decision. Training memory ages in weeks; random SEO blogs don't count as sources.

- **Verify freshness:** every named model, API, tool, or price must be verified against current sources (official docs, the provider's live model list) before it enters the spec — never from training memory. Note the verification date next to it. Names, availability, and pricing go stale within weeks.
- **Justify simplicity:** state why this is the SIMPLEST solution that passes the acceptance criteria — escalate only in this order: fixed rule → deterministic workflow → AI step → AI agent. Reaching for AI or an agent requires one sentence on why the simpler tier is not enough.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Nierozstrzygnięte pytania [DO DOPYTANIA]

Every gap that could not be resolved from sources or the conversation, phrased as concrete, answerable questions (Pocock: unresolved questions at the end of each plan). Each question names WHO answers it — the user (`[UŻYTKOWNIK]`), the client (`[KLIENT]`), or a named third party (lawyer, partner). A question without an owner never gets answered. If there are none, state "brak" explicitly — an empty section is a claim, not an omission.

## Further Notes

Any further notes about the feature.

</spec-template>

## Drabinka: cel → metryka → kryterium odbioru

Każdy PRD ma sekcję „Cel i metryka sukcesu" z TRZEMA poziomami — nie mieszać:
- **Cel** = narracja, może być ogólny; nikt go nie mierzy.
- **Metryka sukcesu** = DLA NAS (wewnętrzna, NIE trafia do umowy). Musi być
  ŁATWO mierzalna dla użytkownika — policzalna z danych, które system I TAK zapisuje
  (rejestr/statusy), jednym filtrem, na żądanie. NIE buduje się funkcji pod
  metrykę (licznik, raport) — to overengineering. Liczba, której nikt nie
  odczyta (np. „czas pracy klienta w h"), to ozdoba, nie metryka.
- **Kryterium odbioru** = DLA UŻYTKOWNIKA I KLIENTA (umowne). Proste, zero-jedynkowe,
  mierzalne przez OBOJE w dniu odbioru — i tylko takie, których wyniku jesteśmy
  PEWNI. Do umowy NIE wchodzi liczba, której nie kontrolujemy (progi jakościowe
  typu „≥85% automatycznie" zostają metryką wewnętrzną).
