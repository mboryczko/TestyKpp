# Monetyzacja — „Testy KPP”

3 listopada 2026 r. Cel: jak największa liczba płacących (nie maksymalny przychód na osobę).

## Cena: 19,99 zł jednorazowo (non-consumable `pl.testykpp.app.full`, Chmura rodzinna)

| Konkurent | Model | Cena |
|---|---|---|
| KPP Pro: Egzamin i Testy 2026 (iOS) | jednorazowo, „KPP Lifetime” | 29,99 zł |
| Ratownik KPP (Centrum Ratownictwa), Testy KPP (Wojnarowski), quizy WWW | darmowe | 0 zł |
| Prawo Jazdy 2025 (iOS, rynek pokrewny) | jednorazowo | 9,99–29,99 zł |
| Zdamyto – prawo jazdy (rynek pokrewny) | pakiety czasowe | 15,99 zł / 7 dni … 89,99 zł / rok |
| Patent strzelecki (nasza aplikacja) | jednorazowo | 29,99 zł |

Uzasadnienie:
- **Poniżej jedynego płatnego konkurenta** (29,99 zł) i poniżej psychologicznego progu 20 zł — przy darmowych alternatywach cena musi być „impulsowa”.
- **Jednorazowo, nie subskrypcja**: kursant uczy się 1–3 tygodnie do znanego terminu. Subskrypcja przy jednym egzaminie budzi nieufność i rezygnacje; „płacisz raz” to argument przeciw konkurencji.
- **Dożywotnio**: tytuł ratownika KPP jest ważny 3 lata — przy recertyfikacji aplikacja nadal działa.
- Do przetestowania (A/B w App Store Connect – Product Page Optimization albo kolejne wersje): 14,99 zł vs 19,99 zł vs 24,99 zł. Jeśli konwersja przy 14,99 zł wzrośnie o > 33%, niższa cena wygrywa także przychodem.

## Wersja darmowa

| Funkcja | Darmowa | Pełna |
|---|---|---|
| Pytania wprowadzane w nauce | 50 (≈ 3–4 dni nauki) | 267 |
| Powtórki poznanych pytań, plan do daty egzaminu | ✓ | ✓ |
| Egzaminy próbne | 1 (porzucony też się liczy) | bez limitu |
| Nauka wg tematu, powtórka błędów | – | ✓ |
| Odpowiedzi w przeglądarce bazy | tylko poznane | wszystkie |

Reset postępu ani import pliku nie odnawiają darmowej puli. Zapisy sprzed wersji darmowej zachowują dostęp do wszystkiego, co już widziały.

## Miejsca paywalla (od najwyższej intencji)

1. **„Kontynuuj naukę” po wyczerpaniu puli** – miękki paywall raz dziennie z wyjściem „Na razie tylko powtórki”; po zakupie sesja startuje od razu z nowymi pytaniami.
2. **Wynik egzaminu próbnego** – karta pod wynikiem („Egzamin losuje z całej bazy”); moment największej motywacji, zwłaszcza przy wyniku tuż pod progiem.
3. **Drugi egzamin próbny** – paywall z automatycznym startem egzaminu po zakupie.
4. **„Wg tematu” i „Błędne pytania”** – plakietka „Pełna” w menu Nauka.
5. **Przeglądarka bazy** – zablokowana odpowiedź z przyciskiem „Pokaż odpowiedź”.
6. **Koniec sesji** – karta z licznikiem, gdy zostaje ≤ 1 sesja darmowych pytań.
7. **Start** – karta „Wersja darmowa X / 50” (od pierwszej odpowiedzi, poza trwającą sesją).
8. **Więcej → Pełny dostęp** + „Przywróć zakup”.

Celowo **bez paywalla w onboardingu**: zasada „zero konta na start” i darmowi konkurenci — użytkownik musi najpierw zobaczyć działanie powtórek.

## Do zrobienia poza kodem

- Utworzyć produkt `pl.testykpp.app.full` w App Store Connect (Non-Consumable, cena 19,99 zł, Family Sharing włączone).
- Analityka lejka (paywall pokazany / kliknięty / kupiony wg powodu) – TelemetryDeck jest już zintegrowany (`purchaseCompleted`, zdarzenia nauki i egzaminu; szczegóły w „Architektura UX/UI” §7.5). Brakuje jeszcze zdarzeń paywalla (`paywallShown` z powodem, `paywallTapped`, `restoreTapped`) — bez nich nie zmierzymy konwersji wg miejsca z listy powyżej ani testu cen.
- Uzupełnić w App Store Connect ankietę „App Privacy”: *Usage Data → Product Interaction*, niepowiązane z tożsamością użytkownika, bez śledzenia (zgodnie z [polityką prywatności](polityka-prywatnosci.html)).
