# Architektura UX/UI — aplikacja „Testy KPP”

Wersja robocza 2, 3 listopada 2026 r. Dotyczy bazy `data/kpp.json` (278 pytań, 267 w rotacji, egzamin 30 pytań / min. 27 poprawnych). Wersja 2 uwzględnia przegląd projektu: rezygnację z prognozy zdania, osobną prezentację rozbieżności klucza, doprecyzowane zasady powtórek, przerwanie i wznowienie sesji, ścisłe reguły mieszania opcji, zmienione priorytety i scenariusze (§11).

---

## 1. Punkt wyjścia: co wynika z danych

| Fakt w danych | Konsekwencja dla UX |
|---|---|
| Próg zdania **27/30 = 90%** | Trzeba opanować praktycznie całą bazę. Przy 90% skuteczności szansa zdania to tylko ok. 65%, przy 95% — ok. 94% (rozkład dwumianowy). Aplikacja prowadzi więc do *utrwalenia wszystkich pytań*, a nie do pojedynczego dobrego wyniku. Liczby te służą do uzasadnienia projektu, nie są pokazywane użytkownikowi jako prognoza (§7.3). |
| 267 pytań w rotacji; średnio ~100 znaków samej treści, ale najdłuższa karta (nr 101) ma **926 znaków** treści, kroków i opcji, a kilka kolejnych ok. 760–780 | Karta musi działać od 1 do kilku ekranów przewijania. Trzeba to sprawdzić na prototypie przy najdłuższych pytaniach, na małym telefonie i z powiększonym tekstem (§9). |
| `steps` (20 pytań) — pytania o kolejność | Osobny wariant karty: lista kroków numerowanych 1–5 nad opcjami. Kroków **nie wolno** mieszać (opcje odwołują się do numerów). |
| `image` (2 pytania) | Karta z ilustracją, powiększanie (pinch/tap). |
| `shuffleOptions`: `all` (178) / `partial` (31) / `none` (69) + `pinnedOptions` | Aplikacja **ściśle respektuje** te ustawienia (§4.6). `none` dotyczy m.in. opcji typu „prawdziwe są odpowiedzi C i D” — przestawienie liter zmieniłoby ich sens. `partial` przypina opcje typu „żadne z wymienionych” na ich miejscu. |
| `confidence` (209 high / 59 medium / 10 low), `note` (39) | Klucz jest opracowaniem własnym. Część notatek opisuje **świadomie zachowaną rozbieżność**: litera zgodna z kluczami zewnętrznymi, a aktualne wytyczne ERC 2025 mówią co innego (np. 226, 252, 63, 149, 205 — raport poprawek z 3.11). To wymaga osobnej prezentacji (§4.5), nie tylko plakietki. |
| `inRotation=false` (11) | Pytania **wyłączone z treningu w tej aplikacji** — to decyzja aplikacji, nie CEM. |
| `sources` (KSRG, ERC 2025, UPRM, MED…) | Podstawa odpowiedzi widoczna przy wyjaśnieniu. Filtr po źródle tylko w przeglądarce bazy (przydatny instruktorowi) — kursantowi nazwa dokumentu nie mówi, z czym ma problem. |
| Brak pola z działem tematycznym | **Dodać `topic`** (§8) — to warunek nauki działami i „słabych obszarów”, wchodzi do MVP. |

**Persona główna:** kursant KPP (strażak PSP/OSP, policjant, ratownik WOPR, GOPR), uczy się na telefonie, często na służbie lub w dojazdach, **z częstymi przerwami** i słabym zasięgiem. Cel: zdać egzamin teoretyczny za pierwszym razem, zwykle w znanym terminie.
**Persona poboczna:** instruktor KPP — przegląda bazę, weryfikuje klucz, zgłasza poprawki.

---

## 2. Zasady projektowe

1. **Utrwalenie > wynik.** Każde pytanie ma stan wiedzy; celem jest „cała baza utrwalona”, a nie pojedynczy test.
2. **Fakty zamiast prognoz.** Pulpit pokazuje to, co zmierzono (pokrycie bazy, powtórki, wyniki egzaminów), a nie wyliczone prawdopodobieństwo zdania.
3. **Przerwanie w każdej chwili.** Zamknięcie aplikacji, telefon, wezwanie — po powrocie wszystko jest tam, gdzie było (§4.4).
4. **Jedna ręka.** Główne akcje w dolnej części ekranu, cele dotyku ≥ 48 px.
5. **Offline-first.** Pełna baza i postęp lokalnie; brak sieci nie jest stanem błędu.
6. **Uczciwość klucza.** Gdy przyjęta odpowiedź różni się od aktualnych wytycznych, pokazujemy obie i nazywamy je wprost. Nie nazywamy klucza bazy „oficjalnym” ani „urzędowym”.
7. **Feedback nie tylko kolorem.** Ikona ✓/✗ + tekst + kolor (WCAG 2.2 AA, daltonizm).
8. **Zero konta na start.** Postęp zabezpiecza eksport do pliku; konto (jeśli powstanie) tylko do synchronizacji. Jedyne dane wysyłane poza urządzenie to anonimowe zdarzenia analityczne (TelemetryDeck, §7.5) i — z inicjatywy użytkownika — zgłoszenia błędów w pytaniach.

---

## 3. Architektura informacji

```
Aplikacja
├── Start (pulpit)
│   ├── Karta wznowienia (jeśli jest przerwana sesja lub egzamin)
│   ├── Utrwalone: 208 / 267 pytań bazy · nie widziane: 12
│   ├── Na dziś: 23 powtórki · CTA „Kontynuuj naukę”
│   ├── Ostatnie egzaminy: 26 · 28 · 27
│   └── Do egzaminu: 12 dni (jeśli ustawiono datę)
├── Nauka
│   ├── Sesja dzienna (powtórki + nowe + błędne, §4.2)
│   ├── Wg tematu (topic)
│   ├── Tylko błędne / oznaczone
│   └── Przeglądaj bazę (lista + wyszukiwarka, wszystkie 278)
│       └── Szczegóły pytania
├── Egzamin próbny
│   ├── Ekran zasad → Egzamin (30 pytań) → Wynik → Przegląd odpowiedzi
│   └── Historia egzaminów
├── Postępy
│   ├── Tematy: utrwalone / w nauce / nie widziane (słabe obszary na górze)
│   ├── Historia egzaminów (wykres wyników z linią 27)
│   └── Statystyki (dni nauki, trafność)
└── Więcej
    ├── Ustawienia (data egzaminu, długość sesji, motyw, rozmiar tekstu, haptyka)
    ├── Kopia postępu (eksport / import pliku, reset)
    └── O bazie (źródło CEM, data aktualizacji, legenda źródeł, zasady klucza)
```

**Nawigacja:** dolny pasek z 4 zakładkami — *Start · Nauka · Egzamin · Postępy* („Więcej” jako ikona w nagłówku Startu). W trakcie sesji/egzaminu pasek znika (tryb skupienia); „✕” wychodzi z zachowaniem stanu (§4.4).

---

## 4. Tryby i przepływy

### 4.1 Sesja nauki (tryb główny)

```
[Start] → „Kontynuuj naukę” → karta pytania → wybór odpowiedzi
        → feedback + wyjaśnienie → „Dalej” → … → podsumowanie sesji
```

- Domyślnie 15 pytań (ustawialne 10/15/25). Skład opisuje §4.2.
- Po odpowiedzi: zaznaczona poprawna opcja, **Wyjaśnienie** (domyślnie otwarte przy błędzie), **Podstawa** (chipy źródeł z pełną nazwą po dotknięciu), a przy rozbieżności klucza — panel z §4.5.
- Plakietki pewności i rozbieżności pojawiają się **dopiero po odpowiedzi** — wcześniej podpowiadałyby, że pytanie jest „podchwytliwe”.
- Akcje drugorzędne: ☆ Oznacz, ⚑ Zgłoś błąd.
- Pytanie, na które odpowiedziano źle, wraca na końcu tej samej sesji jako **ćwiczenie** (nie zmienia pudełka, §4.2).
- Podsumowanie: wynik sesji, ile pytań awansowało, ile czeka jutro.

### 4.2 Powtórki — zasady awansu (model pudełek Leitnera)

| Pudełko | Stan pokazywany użytkownikowi | Interwał do następnej powtórki |
|---|---|---|
| 0 | Nie widziane | — |
| 1 | W nauce | 1 dzień |
| 2 | W nauce | 3 dni |
| 3 | W nauce | 7 dni |
| 4 | **Utrwalone** | 14 dni |
| 5 | Utrwalone | 30 dni |

Reguły:

1. **Awansuje tylko odpowiedź w terminie** (pytanie nowe albo `due ≤ dziś`), udzielona w sesji nauki przed zobaczeniem wyjaśnienia. Pierwsza poprawna odpowiedź na nowe pytanie → pudełko 2; zgadywanie przy 5 opcjach ma ok. 20% szans, więc nie daje od razu wyższego stanu.
2. **Poprawna odpowiedź przed terminem** (np. drugi raz tego samego dnia, w trybie „wg tematu”, w egzaminie) nie zmienia pudełka — trafia tylko do statystyk.
3. **Błąd w dowolnym trybie, także w egzaminie**, cofa pytanie do pudełka 1 (termin: jutro).
4. **Powtórzenie po błędzie w tej samej sesji to ćwiczenie** — nie awansuje.
5. Skutek: „Utrwalone” wymaga co najmniej 3 poprawnych odpowiedzi w terminie, w 3 różnych dniach, rozłożonych na ≥ 10 dni.

**Skład sesji dziennej:**

- Kolejność: zaległe powtórki (najdłużej zaległe najpierw) → nowe pytania → błędne z poprzednich dni.
- Dopóki są pytania nie widziane, **min. 30% sesji to nowe pytania** — powtórki nie mogą ich całkiem wyprzeć.
- Gdy zaległości przekraczają 3 sesje, aplikacja mówi o tym wprost („Masz 52 zaległe powtórki — dziś bez nowych pytań?” [Tak] [Zostaw nowe]) zamiast po cichu zmieniać proporcje.

**Data egzaminu (opcjonalna):**

- Dzienna liczba nowych pytań = nie widziane ÷ (dni do egzaminu − 3 dni zapasu), zaokrąglone w górę.
- Interwały skracane tak, by każde pytanie miało co najmniej jedną powtórkę w ostatnich 7 dniach przed egzaminem.
- Pulpit pokazuje plan w liczbach: „Do egzaminu 12 dni · dziś 18 nowych, 23 powtórki”. Bez daty — stałe tempo z ustawień.

### 4.3 Egzamin próbny

- Ekran zasad: „30 losowych pytań, wymagane 27 poprawnych, bez podpowiedzi”. Limit czasu — **do ustalenia z regulaminem**; w aplikacji opcjonalny (domyślnie wyłączony), widoczny jako dyskretny licznik.
- Losowanie wyłącznie z `inRotation=true`; kolejność opcji wg §4.6, ustalana raz przy losowaniu.
- Brak feedbacku i plakietek w trakcie. Pasek postępu 1–30, pominięcie i „wróć później”, siatka pytań do przeskakiwania. Powrót do pytania pokazuje **tę samą kolejność opcji** i zaznaczoną wcześniej odpowiedź.
- Przed oddaniem: „Masz 2 pytania bez odpowiedzi” → [Wróć] [Oddaj].
- Wynik: **28/30 — ZDANY** (lub NIEZDANY, brakło 1), lista błędów z wyjaśnieniami i panelami rozbieżności. Błędne pytania automatycznie trafiają do pudełka 1 (§4.2).
- Jednocześnie może być **jeden** aktywny egzamin; rozpoczęcie nowego wymaga porzucenia przerwanego („Porzucić egzamin 12/30? Wynik nie zostanie zapisany”).

### 4.4 Przerwanie i wznowienie

Stan aktywnej sesji i egzaminu zapisuje się **po każdej akcji** (odpowiedź, pominięcie, przejście). Po zamknięciu aplikacji, utracie zasięgu czy restarcie telefonu przetrwa:

- tryb i zestaw pytań (ID) w ustalonej kolejności,
- kolejność opcji każdego pytania,
- udzielone odpowiedzi, oznaczenia „wróć później”, bieżąca pozycja,
- dla egzaminu z limitem: pozostały czas (czas biegnie tylko przy otwartej aplikacji — to trening, nie egzamin urzędowy; informacja na ekranie zasad),
- kopia treści pytań z chwili rozpoczęcia (§7.2).

Na Starcie, nad wszystkim innym:

```
┌──────────────────────────────┐
│ Przerwany egzamin próbny     │
│ Odpowiedziano 12 / 30        │
│ rozpoczęty dziś, 14:05       │
│ [Wznów egzamin]   Porzuć     │
└──────────────────────────────┘
```

- Wznowienie otwiera dokładnie to pytanie, na którym przerwano.
- Przerwana sesja nauki: „Wznów naukę — 7 / 15”. Odpowiedzi już udzielone zostały zaliczone do postępu na bieżąco, więc porzucenie niczego nie cofa. Sesja nauki starsza niż 24 h jest zamykana automatycznie z krótką informacją; egzamin czeka do decyzji użytkownika.
- „✕” w trakcie egzaminu: „Wyjść? Egzamin zostanie zapisany, wrócisz do niego z ekranu Start” [Wyjdź] [Zostań].

### 4.5 Pytania z rozbieżnością klucza

Dotyczy pytań, w których przyjęta odpowiedź jest zgodna z kluczami zewnętrznymi (gov.pl 2024, dlaratownikaimedyka 2025), a aktualne wytyczne wskazują co innego. Po odpowiedzi, zamiast zwykłej notatki:

```
┌──────────────────────────────┐
│ ✓ Odpowiedź przyjęta w bazie │
│   C — od 1. miesiąca do 1. r.│
│   Zgodna z kluczami          │
│   zewnętrznymi do egzaminu.  │
├──────────────────────────────┤
│ ⓘ Różnica względem aktualnych│
│   wytycznych (ERC 2025 PLS)  │
│   Niemowlę: od urodzenia do  │
│   1. roku życia — wg nowych  │
│   wytycznych poprawne byłoby │
│   E.                         │
└──────────────────────────────┘
```

- Obie sekcje rozwinięte, równorzędne wizualnie; żadna nie nosi nazwy „oficjalna odpowiedź egzaminacyjna”.
- Zaliczenie odpowiedzi liczone wg odpowiedzi przyjętej w bazie; wybór wariantu zgodnego z aktualnymi wytycznymi pokazuje komunikat „Twoja odpowiedź odpowiada wytycznym ERC 2025, ale w bazie przyjęto C” zamiast samego ✗. Pytanie i tak trafia do pudełka 1.
- W „O bazie” krótko opisane zasady: skąd pochodzi klucz, kiedy litera jest zmieniana, a kiedy zostaje z notatką.
- Pytania `inRotation=false` w przeglądarce bazy mają baner: **„Wyłączone z treningu w tej aplikacji — treść lub odpowiedź budzi wątpliwości”**. Nie pojawiają się w sesjach ani egzaminie i nie wliczają się do 267.

### 4.6 Kolejność opcji

- `all` — wszystkie opcje losowane; `partial` — losowane wszystkie poza `pinnedOptions`, które zostają na swoim miejscu (zwykle na końcu); `none` — kolejność jak w arkuszu CEM.
- Po przemieszaniu litery A–E nadawane są od nowa wg pozycji na ekranie. Odpowiedź zapisywana jest jako **oryginalna litera z bazy**, więc postęp, zgłoszenia i raporty odnoszą się do numeracji CEM.
- Kolejność losowana raz na pytanie na sesję/egzamin i zapisywana (§4.4) — nie zmienia się przy powrocie do pytania ani po wznowieniu.
- Wyjaśnienia nie odwołują się do liter A–E (sprawdzone w bieżącej bazie: brak takich odwołań); odwołania do numerów kroków są poprawne, bo kroki nie są mieszane.

### 4.7 Przeglądarka bazy

- Lista wszystkich 278 pytań: numer CEM, początek treści, stan (nie widziane / w nauce / utrwalone), znaczniki ☆ / ⓘ rozbieżność / ⚠ niska pewność / „wyłączone z treningu”.
- Wyszukiwarka pełnotekstowa (treść + opcje, ignorowanie polskich znaków: „oparzen” znajduje „oparzeń”).
- Filtry: stan, temat, oznaczone, rozbieżność klucza, pewność, wyłączone; **źródło** jako filtr zaawansowany (dla instruktora).
- Szczegóły pytania pokazują oryginalną kolejność opcji A–E (jak w PDF CEM) — przydatne przy porównaniu z kluczem.
- Plakietki są tu widoczne od razu — przeglądarka to tryb przeglądu, nie sprawdzania się.

---

## 5. Ekrany kluczowe (szkice)

### Start
```
┌──────────────────────────────┐
│ Testy KPP               ⚙   │
│                              │
│  Utrwalone pytania bazy      │
│  ███████████████░░░░         │
│  208 / 267 · nie widziane 12 │
│                              │
│  Do egzaminu: 12 dni         │
│  Dziś: 23 powtórki, 18 nowych│
│  ┌──────────────────────────┐│
│  │   Kontynuuj naukę  ▶     ││
│  └──────────────────────────┘│
│  Ostatnie egzaminy           │
│  26 ✗   28 ✓   27 ✓          │
│  [Nowy egzamin próbny]       │
├──────────────────────────────┤
│ Start  Nauka  Egzamin Postępy│
└──────────────────────────────┘
```

Pierwsze uruchomienie (bez postępu) — patrz scenariusz §11.1.

### Karta pytania — po odpowiedzi
```
┌──────────────────────────────┐
│ ✕   7 / 15   ━━━━━░░░░░  ☆ ⚑ │
│ Pytanie nr 58                │
│                              │
│ Rany otwarte klatki          │
│ piersiowej zabezpieczamy…    │
│                              │
│ ┌──────────────────────────┐ │
│ │ A  opatrunkiem …       ✗ │ │  ← wybrana, błędna
│ └──────────────────────────┘ │
│ ┌──────────────────────────┐ │
│ │ B  …                     │ │
│ └──────────────────────────┘ │
│ ┌──────────────────────────┐ │
│ │ E  …        ✓ Poprawna   │ │
│ └──────────────────────────┘ │
│ ▼ Wyjaśnienie                │
│   …                          │
│   Podstawa: [ERC 2025 – FA]  │
│            (treść przewijana)│
├──────────────────────────────┤
│ ┌──────────────────────────┐ │  ← stały pasek akcji
│ │         Dalej  →         │ │    poza obszarem treści
│ └──────────────────────────┘ │
└──────────────────────────────┘
```

„Dalej” siedzi w stałym pasku pod obszarem przewijania (nie nad treścią), więc nigdy nie zasłania ostatniej opcji ani wyjaśnienia.

### Karta z krokami (`steps`)
```
│ Wskaż prawidłową kolejność…  │
│ ┌──────────────────────────┐ │
│ │ 1 zamknąć dopływ gazu…   │ │  ← panel kroków, stały,
│ │ 2 ocenić czynności…      │ │    numeracja nie mieszana
│ │ 3 wynieść poszkodowanego │ │
│ │ 4 wezwać ZRM i PSP…      │ │
│ │ 5 opuścić pomieszczenie… │ │
│ └──────────────────────────┘ │
│  A  3, 2, 4                  │
│  B  1, 2, 3, 4               │
```
Panel kroków domyślnie rozwinięty; na wąskich ekranach można go zwinąć, a przy zwinięciu dotknięcie numeru w opcji pokazuje treść kroku.

### Wynik egzaminu
```
│          26 / 30             │
│        ✗ NIEZDANY            │
│   Do zaliczenia brakło 1     │
│  ───────────────────────     │
│  Błędne (4) — dodane do      │
│  powtórek na jutro           │
│  › nr 132 Uciskanie kl. … ⓘ  │
│  › nr 102 Kolejność …        │
│  [Przejrzyj błędy]  [Nowy]   │
```

---

## 6. Biblioteka komponentów

| Komponent | Warianty / stany |
|---|---|
| `QuestionCard` | standard · z krokami · z obrazem; stany: oczekuje, odpowiedziano-dobrze, odpowiedziano-źle, egzamin (bez feedbacku); treść przewijana niezależnie od paska akcji |
| `OptionButton` | domyślny, wybrany, poprawny, błędny, poprawny-niewybrany, zgodny-z-wytycznymi, nieaktywny; etykieta litery + tekst; klawisze 1–5 / A–E |
| `StepsPanel` | rozwinięty / zwinięty |
| `ExplanationPanel` | wyjaśnienie, chipy źródeł, notatka, plakietka pewności |
| `KeyDiscrepancyPanel` | dwie sekcje: „Odpowiedź przyjęta w bazie” / „Różnica względem aktualnych wytycznych” |
| `ConfidenceBadge` | wysoka (ukryta), średnia „do weryfikacji”, niska „niepewna odpowiedź” |
| `ExcludedBanner` | „Wyłączone z treningu w tej aplikacji…” |
| `ActionBar` | stały dolny pasek: Dalej / Oddaj / Sprawdź |
| `SessionHeader`, `ProgressBar` | postęp sesji, wyjście z zapisem, oznacz, zgłoś |
| `ResumeCard` | przerwany egzamin / przerwana sesja nauki |
| `CoverageSummary` | utrwalone / w nauce / nie widziane, liczby bezwzględne + pasek |
| `ExamHistory` | ostatnie 3 wyniki (pulpit), wykres z linią 27 (Postępy) |
| `TopicList` | tematy z pokryciem, sortowane od najsłabszego |
| `ExamNavigator` | siatka 30 pól: bez odp., odpowiedziano, „wróć później” |
| `ReportSheet` | dolny arkusz zgłoszenia: kategoria (klucz/treść/wyjaśnienie/literówka) + komentarz |
| `EmptyState`, `Toast`, `ConfirmDialog` | — |

### Tokeny wizualne (szkic)

- **Kolor bazowy:** zieleń ratownicza jako akcent, neutralne szarości; sukces = zieleń + ✓, błąd = czerwień + ✗, rozbieżność = niebieski + ⓘ (informacja, nie ostrzeżenie), niska pewność = bursztyn + ⚠. Pełny motyw jasny i ciemny.
- **Typografia:** systemowa (szybkość, polskie znaki), treść pytania 18–20 px, opcje 17 px, obsługa powiększenia tekstu systemowego do 200% bez łamania układu.
- **Odstępy:** siatka 4/8 px, margines boczny 16 px.
- **Ruch:** krótkie przejścia 150–200 ms, wyłączane przy `prefers-reduced-motion`; haptyka przy odpowiedzi (opcjonalna).

---

## 7. Architektura stanu (warstwa UX ↔ dane)

```
kpp.json (statyczny, wersjonowany)        Dane użytkownika (IndexedDB)
 ├─ questions[] (+ topic, revision,        ├─ progress[id] = {box, due, seen,
 │   discrepancy)                          │    correct, wrong, revision}
 ├─ sources{}                              ├─ activeSession | null
 ├─ exam{questions, passMin}               ├─ activeExam | null
 └─ sourceUpdated                          ├─ exams[] {date, ids, answers, score}
          │                                ├─ flags[id] (☆)
          ▼                                ├─ reports[] (kolejka zgłoszeń)
   QuestionRepository                      └─ settings {examDate, sessionSize…}
   (filtrowanie, losowanie,
    kolejność opcji)  ──►  SessionEngine (Nauka / Egzamin)  ──►  UI
                            Scheduler (Leitner, §4.2)
                            SessionStore (zapis po każdej akcji)
```

### 7.1 Aktywna sesja

```
activeExam = {
  id, startedAt, timeLeftSec | null,
  items: [{ qid, revision, optionOrder: ["C","A","E","B","D"],
            answer: "A" | null, later: false }],
  position: 12,
  snapshot: { [qid]: { question, options, steps, answer, … } }
}
```

`activeSession` ma ten sam kształt (bez limitu czasu). Zapis transakcyjny po każdej akcji; odczyt przy starcie aplikacji decyduje o wyświetleniu `ResumeCard`.

### 7.2 Aktualizacja bazy

- Każde pytanie ma `revision`; `progress[id].revision` pamięta wersję, na której zdobyto postęp.
- Zmiana `answer` lub treści → pytanie wraca do pudełka 1 z komunikatem „Zaktualizowano odpowiedź do tego pytania — powtórz je”. Bez tego kursant mógłby „utrwalić” nieaktualny klucz (raport z 3.11 zmienił m.in. 58, 239, 259).
- Sama zmiana wyjaśnienia lub notatki nie resetuje postępu, tylko oznacza pytanie jako „zaktualizowane wyjaśnienie” do obejrzenia.
- Aktywny egzamin/sesja kończy się na **kopii z chwili rozpoczęcia** (`snapshot`); reset zmienionych pytań następuje po jej zakończeniu.

### 7.3 Wskaźniki pulpitu

- **Utrwalone / 267** (pudełko ≥ 4), **w nauce** (1–3), **nie widziane** (0) — liczby bezwzględne, procent opisany jako „utrwalone pytania bazy”.
- **Powtórki na dziś**, **nowe na dziś** (z planu przy dacie egzaminu).
- **Ostatnie 3 egzaminy próbne** z oznaczeniem zdany/niezdany.
- Bez prognozy „szansy zdania”. Ewentualny model predykcyjny dopiero wtedy, gdy będzie można go skalibrować na danych (np. porównując stan pudełek z wynikami egzaminów próbnych) — patrz §10, „Później”.

### 7.4 Offline i kopia postępu

- PWA z service workerem; baza (~kilkaset kB) + 2 obrazy w pamięci podręcznej przy pierwszym uruchomieniu.
- Zgłoszenia błędów kolejkowane i wysyłane po odzyskaniu sieci.
- Eksport postępu do pliku JSON (udostępnij / zapisz) i import na innym urządzeniu; przypomnienie o kopii po 7 dniach nauki bez eksportu. Bez konta to jedyne zabezpieczenie przed utratą danych przy czyszczeniu przeglądarki.

### 7.5 Analityka (TelemetryDeck)

- Aplikacja iOS używa TelemetryDeck (SDK Swift, przestrzeń nazw `com.michalbgames`) do anonimowych statystyk użycia. Brak kont, brak identyfikatora reklamowego (IDFA), brak śledzenia między aplikacjami.
- Identyfikator użytkownika jest domyślnie haszowany po stronie urządzenia; TelemetryDeck nie dostaje danych pozwalających zidentyfikować osobę.
- Wysyłane zdarzenia: `studyStarted` (tryb nauki), `studyFinished` (liczba pytań i poprawnych odpowiedzi), `examStarted`, `examFinished` (wynik i zdany/niezdany), `purchaseCompleted`, a także automatyczne sygnały SDK (start sesji, wersja aplikacji, model urządzenia, system).
- Nie wysyłamy treści pytań, odpowiedzi użytkownika ani zawartości eksportu postępu. Postęp nauki zostaje na urządzeniu (§7.4).
- Analityka nie blokuje działania offline: zdarzenia są kolejkowane przez SDK i wysyłane po odzyskaniu sieci.

---

## 8. Rekomendowane zmiany w danych

1. **Pole `topic`** (jeden dział na pytanie), np.: *Organizacja systemu i prawo · Ocena poszkodowanego · BLS/AED dorośli · Resuscytacja dzieci i noworodków · Drogi oddechowe i sprzęt · Wstrząs i krwotoki · Urazy (głowa, kręgosłup, klatka, kończyny) · Oparzenia i termiczne · Zatrucia i środowiskowe · Stany nagłe (udar, zawał, cukrzyca, drgawki) · Poród i położnictwo · Segregacja i zdarzenia masowe · Ewakuacja i bezpieczeństwo*. Warunek MVP.
2. **Pole `discrepancy`** dla pytań ze świadomie zachowaną rozbieżnością, oddzielone od ogólnej `note`:
   ```json
   "discrepancy": {
     "basis": "Zgodne z kluczami gov.pl 2024 i dlaratownikaimedyka 2025",
     "current": "E",
     "guideline": "ERC2025-PLS",
     "text": "Niemowlę: od urodzenia do ukończenia 1. roku życia."
   }
   ```
   Dziś ta informacja jest wymieszana z innymi notatkami w `note` i nie da się jej niezawodnie wyświetlić w osobnym panelu.
3. **`revision` per pytanie** — prostsze i pewniejsze niż porównywanie skrótów przy wykrywaniu zmian klucza.
4. **Wyjaśnienia bez odwołań do liter A–E** — utrzymać jako regułę redakcyjną (obecnie spełniona).
5. **Odbiorca zgłoszeń** — ustalić (e-mail, formularz, issue w repo); bez tego ⚑ tylko zapisuje lokalnie.

---

## 9. Dostępność i jakość

- WCAG 2.2 AA: kontrast ≥ 4,5:1, widoczny focus, pełna obsługa klawiaturą (1–5 / A–E wybór, Enter dalej, S pomiń), czytnik ekranu ogłasza „Poprawnie” / „Błędnie, poprawna odpowiedź: …”.
- **Test długich kart:** prototyp sprawdzony na pytaniach o największej długości (101, 204, 130, 129, 199) na ekranie 360×640 przy 100% i 200% tekstu; kryteria: wszystkie opcje osiągalne przewijaniem, „Dalej” nie zasłania treści, panel kroków nie wypycha opcji poza 2 ekrany przy zwinięciu.
- Rozmiar tekstu do 200%, orientacja pionowa i pozioma; tablet: karta max 640 px, wyjaśnienie obok na ≥ 1024 px.
- Wydajność: pierwsze wyświetlenie < 1,5 s na średnim Androidzie, przejście między pytaniami bez opóźnienia, zapis stanu po akcji niezauważalny (< 50 ms).
- Zastrzeżenie stale dostępne w „O bazie”: odpowiedzi są opracowaniem własnym opartym na kluczach zewnętrznych i wytycznych, nie urzędowym kluczem CEM.

---

## 10. Kolejność wdrożenia

| Etap | Zakres | Wartość |
|---|---|---|
| **Prototyp** | Karta pytania (3 warianty) na najdłuższych pytaniach, panel rozbieżności, scenariusze z §11 | Weryfikacja decyzji przed kodem produkcyjnym |
| **MVP** | Sesja nauki, Leitner wg §4.2, egzamin próbny 30/27, przerwanie i wznowienie, `topic` + nauka wg tematu, panel rozbieżności, eksport/import postępu, offline | Pełny, bezpieczny cykl nauka → egzamin |
| **v1** | Pulpit z pokryciem i historią egzaminów, data egzaminu i plan dzienny, przeglądarka + wyszukiwarka, oznaczanie, zgłaszanie błędów, ciemny motyw | Planowanie nauki i zaufanie |
| **v1.1** | Słabe obszary wg tematu, wykres historii egzaminów, filtry zaawansowane (źródło, pewność) | Nauka celowana |
| **Później** | Siatka opanowania wszystkich pytań, skalibrowany model gotowości, konto i synchronizacja, panel instruktora, przypomnienia push | Skala |

Proponowany stos (lekki, statyczny): Vite + React lub Svelte, PWA, IndexedDB (np. przez `idb`), brak backendu w MVP.

---

## 11. Scenariusze do przejścia przed kodowaniem

### 11.1 Pierwsze uruchomienie bez postępu

1. Ekran powitalny (jeden): czym jest aplikacja, że odpowiedzi to opracowanie własne, link „Zasady klucza”. [Zaczynam]
2. Opcjonalne pytanie: „Kiedy masz egzamin?” [wybierz datę] [Nie wiem / pomiń].
3. Od razu pierwsza sesja: 10 nowych pytań z różnych tematów — bez testu wstępnego i bez konta.
4. Po sesji pulpit pokazuje realne liczby („10 widzianych, 0 utrwalonych — pierwsze powtórki jutro”), a nie puste wykresy.

Do rozstrzygnięcia: czy proponować egzamin próbny „na rozgrzewkę” na starcie (ryzyko demotywacji przy wyniku ~10/30).

### 11.2 Powrót do przerwanego egzaminu

1. Kursant odpowiada na 12/30, dostaje wezwanie, aplikacja ląduje w tle; system ją zamyka.
2. Po 3 godzinach: Start pokazuje `ResumeCard` „Przerwany egzamin — 12/30” nad wszystkimi innymi elementami.
3. [Wznów] → pytanie 13, siatka pokazuje 12 odpowiedzianych; powrót do pytania 4 pokazuje tę samą kolejność opcji i zaznaczoną odpowiedź.
4. W międzyczasie przyszła aktualizacja bazy zmieniająca klucz pytania 7 — egzamin kończy się na kopii, po wyniku komunikat „1 pytanie z tego egzaminu zostało zaktualizowane”.

Do rozstrzygnięcia: czy przerwany egzamin z limitem czasu powinien po 24 h wygasać.

### 11.3 Odpowiedź na pytanie z rozbieżnością klucza

1. Sesja nauki, pytanie 226 (definicja niemowlęcia). Kursant zna ERC 2025 i wybiera E.
2. Feedback: nie zwykłe ✗, tylko „Twoja odpowiedź odpowiada wytycznym ERC 2025, ale w bazie przyjęto C” + `KeyDiscrepancyPanel` z obiema sekcjami.
3. Pytanie trafia do pudełka 1, bo celem jest egzamin, którego klucz (wg kluczy zewnętrznych) wskazuje C.
4. W przeglądarce bazy pytanie ma znacznik ⓘ i daje się odfiltrować („Rozbieżność klucza”), żeby powtórzyć wszystkie takie przypadki naraz.

Do rozstrzygnięcia: czy wybór wariantu zgodnego z wytycznymi powinien cofać do pudełka 1, czy np. tylko o jedno pudełko — to decyzja o tym, ile aplikacja „karze” za wiedzę aktualniejszą niż klucz.
