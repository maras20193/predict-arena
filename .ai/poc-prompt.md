# Prompt dla generatora Proof of Concept - PredictArena

## Kontekst projektu

Tworzysz Proof of Concept (PoC) dla webowej aplikacji PredictArena - systemu do prywatnych typowań wyników meczów w gronie znajomych. PoC ma zweryfikować **podstawową funkcjonalność** aplikacji: branie udziału w turnieju, dołączanie do niego w odpowiednich ramach czasowych oraz widoki dashboardów zmieniające się w zależności od fazy turnieju.

## WAŻNE: Proces pracy

**PRZED rozpoczęciem implementacji musisz:**

1. **Przygotować plan pracy** - rozbić zadanie na etapy (szkielet projektu, schemat bazy danych, podstawowe komponenty, logika biznesowa, testy)
2. **Przedstawić plan użytkownikowi** - wyświetlić listę zadań z oszacowaniem zakresu
3. **Uzyskać akceptację** - kontynuować dopiero po potwierdzeniu planu przez użytkownika

## Stack technologiczny

### Frontend

- **Astro 5** - framework główny
- **React 19** - komponenty interaktywne
- **TypeScript 5** - typowanie statyczne
- **Tailwind 4** - stylowanie
- **Shadcn/ui** - komponenty UI
- **Recharts 3** - wykresy słupkowe (leaderboard)
- **Tanstack-query** - zarządzanie danymi, cache
- **Tanstack-table** - wirtualizowana macierz typów
- **react-hook-form + zod** - formularze z walidacją

### Backend i dane

- **DOWOLNE rozwiązanie** - użyj tego co jest najszybsze (localStorage, JSON mock data, prosty Express/Node, etc.)
- **NIE używaj Supabase** - PoC ma być szybki, wybierz najprostsze rozwiązanie
- Dane przechowuj w formacie UTC, UI prezentuje w strefie **Europe/Warsaw**

### Symulacja roli i stanu (KRYTYCZNE dla PoC)

- **NIE implementuj autoryzacji Google/OAuth**
- **Dodaj switche/select w UI** do symulacji:
  - **Rola**: Admin / User (dropdown/switch w górnej części aplikacji)
  - **Stan fazy**: Przed lock_at / Po lock_at, przed start_at / Po start_at (dropdown/select)
- Symulacja pozwoli szybko sprawdzić jak widoki zmieniają się w zależności od roli i fazy
- Możesz też dodać "czas symulacji" (manualny datetime picker) do testowania różnych momentów w czasie

## Zakres PoC - Funkcjonalności WŁĄCZONE

### 1. Turniej i dołączanie

- **Jeden turniej w UI** (DB może wspierać wiele, ale UI jeden)
- Dołączanie przez wniosek użytkownika → akceptacja przez admina
- **Blokada dołączania po starcie turnieju** (start = start pierwszej fazy)
- Admin może również uczestniczyć w turnieju (jego typy i punkty są uwzględniane)

### 2. Fazy i mecze

- Faza ma `start_at` (ujawnienie typów) i `lock_at` (koniec edycji typów)
- **Domyślnie w PoC: start_at = lock_at** (tryb prosty)
- Admin tworzy fazę z ustawionymi czasami i dodaje mecze
- Mecz: drużyna A vs drużyna B, data/godzina meczu
- Flagi drużyn (ISO codes) z CDN/fallback

### 4. Typowanie

- Formularz typowania obejmuje **wszystkie mecze w aktywnej fazie**
- Edycja typów możliwa **tylko do lock_at**
- Po `lock_at` - **blokada edycji** (weryfikacja stanu fazy na frontendzie)
- Formularz używa `react-hook-form + zod`
- Przy błędzie zapisu formularz pozostaje zamontowany z danymi (brak draftu)

### 5. Widoczność typów i prywatność (KRYTYCZNE)

- **Przed `start_at`**: użytkownicy widzą **tylko własne typy**
- **Po `start_at`**: typy **wszystkich uczestników** są widoczne
- **Frontend filtruje dane** zgodnie z wybranym stanem fazy (switch/select)
- Logika weryfikuje stan fazy i pokazuje odpowiednie dane

### 6. Dashboardy zmieniające się w zależności od fazy

#### A. Przed `start_at` (faza nie rozpoczęta)

- **Formularz typowania** (jeśli `lock_at` jeszcze nie minął)
- Widok "Moje typy" - tylko własne typy
- Licznik odliczający do `lock_at` (w strefie Europe/Warsaw)

#### B. Po `lock_at`, przed `start_at` (faza zamknięta do edycji, typy jeszcze ukryte)

- Formularz edycji **zablokowany** (komunikat o zamknięciu)
- Widok "Moje typy" - tylko własne typy
- Licznik do `start_at` (ujawnienia typów)

#### C. Po `start_at` (faza ujawniona)

- **Macierz typów** (tabela: wiersze = mecze, kolumny = użytkownicy)
  - Kolorowanie komórek: 5 pkt = zielony, 3 pkt = niebieski, inne = neutralne
  - Sticky headers, wirtualizacja (Tanstack-table)
  - Grupowanie po fazach
- **Leaderboard** (wykres słupkowy Recharts)
  - Suma punktów wszystkich faz
  - Avatary uczestników
  - Ranking ex aequo z lukami (np. 1,1,3)
  - Tooltip z pozycją i sumą punktów
- Dostęp do historii - przełączanie między fazami (typy i punkty po ujawnieniu)

### 7. Wyniki i punktacja

- Admin ręcznie wprowadza wyniki meczów (po 90 minutach)
- **Automatyczne naliczanie punktów**:
  - 5 pkt za dokładny wynik (np. 2:1 = 2:1)
  - 3 pkt za poprawną stronę (np. 2:1 vs 1:2 - zwycięzca trafiony, wynik nie)
  - Konfiguracja punktacji przy tworzeniu turnieju (domyślnie 5/3)
- Po wprowadzeniu wyniku punkty aktualizują się automatycznie

### 8. Czas i strefy czasowe

- Backend: wszystkie czasy w **UTC**
- UI: prezentacja w **Europe/Warsaw** z uwzględnieniem DST
- Liczniki czasu (odliczanie do `lock_at`/`start_at`) w UI

## Zakres PoC - Funkcjonalności WYKLUCZONE

❌ **Nie implementuj:**

- Analityka (page_view, join_request, etc.) - US-021
- Reschedule fazy (zmiana `start_at`/`lock_at` po utworzeniu) - US-012
- Kafelki highlight (Best Predictor, etc.) - opcjonalne z PRD
- Batch dodawanie meczów (wystarczy pojedyncze dodawanie)
- Idempotencja API z request_id (wystarczy podstawowa walidacja)
- Zaawansowane zarządzanie profilami
- Import/eksport CSV
- Notifications/powiadomienia
- CI/CD i deployment (tylko lokalny development)
- Zaawansowane walidacje i edge cases

## Kluczowe wymagania techniczne

### Symulacja i widoczność (frontend)

- **Filtrowanie danych na frontendzie**:
  - Przed `start_at`: pokazuj tylko typy obecnego użytkownika (zgodnie z wybraną rolą)
  - Po `start_at`: pokazuj typy wszystkich uczestników
- **Blokada edycji**: po `lock_at` formularz jest nieaktywny (weryfikacja stanu fazy)
- **Autoryzacja admina**: tylko gdy rola = "Admin" pokazuj przyciski/akcje admina (akceptacja wniosków, tworzenie faz, dodawanie meczów, wprowadzanie wyników)

### Wydajność (opcjonalne, ale zalecane dla PoC)

- Wirtualizacja macierzy typów (Tanstack-table)
- Cache'owanie danych (Tanstack-query)

### Struktura danych (schemat sugerowany)

```
tournaments
  - id, name, scoring_config (json: {exact: 5, winner: 3}), created_at, started_at

phases
  - id, tournament_id, name, start_at (UTC), lock_at (UTC), created_at

matches
  - id, phase_id, team_a, team_b, team_a_flag (ISO), team_b_flag (ISO),
    match_date, result_a, result_b, created_at

tournament_participants
  - id, tournament_id, user_id, role (user/admin), status (pending/approved/rejected), created_at

predictions (typy)
  - id, match_id, user_id, team_a_score, team_b_score, points_earned, created_at, updated_at
```

## User Stories krytyczne dla PoC (z symulacją)

**US-SIM**: Symulacja roli i stanu (switch/select dla roli i fazy)  
**US-003**: Wysłanie prośby o dołączenie (mock, bez realnej autoryzacji)  
**US-004**: Akceptacja/odrzucenie przez admina (gdy rola = Admin)  
**US-005**: Widok fazy z licznikami czasu (symulowany stan)  
**US-006**: Składanie typów do lock_at (blokada gdy stan = "Po lock_at")  
**US-007**: Blokada edycji po lock_at (weryfikacja stanu)  
**US-008**: Prywatność typów przed start_at (filtrowanie na frontendzie)  
**US-013**: Wprowadzanie wyników przez admina (gdy rola = Admin)  
**US-014**: Automatyczne naliczanie punktów  
**US-015**: Leaderboard (wykres)  
**US-016**: Macierz typów (widoczna po start_at)  
**US-024**: Prezentacja czasu (UTC → Europe/Warsaw)

## Success criteria dla PoC

PoC jest udany jeśli:

1. ✅ **Symulacja roli działa**: switch/select pozwala przełączać między Admin/User
2. ✅ **Symulacja stanu fazy działa**: switch/select pozwala wybrać stan (Przed lock_at / Po lock_at, przed start_at / Po start_at)
3. ✅ Użytkownik może wysłać wniosek o dołączenie (mock)
4. ✅ Gdy rola = Admin: może zaakceptować wniosek, utworzyć fazę z meczami, wprowadzić wyniki
5. ✅ Uczestnik może złożyć typy (gdy stan = "Przed lock_at")
6. ✅ Gdy stan = "Po lock_at": edycja typów jest zablokowana w UI
7. ✅ Gdy stan = "Przed start_at": uczestnik widzi tylko własne typy (filtrowanie frontend)
8. ✅ Gdy stan = "Po start_at": uczestnik widzi typy wszystkich (macierz)
9. ✅ Punkty naliczają się automatycznie (5/3) po wprowadzeniu wyniku
10. ✅ Leaderboard wyświetla ranking z punktami (wykres słupkowy)
11. ✅ Macierz typów wyświetla typy z kolorami (5=zielony, 3=niebieski)
12. ✅ **Dashboardy zmieniają się dynamicznie** w zależności od wybranego stanu fazy

## Kolejność implementacji (sugerowana)

1. **Setup projektu**: Astro + React + TypeScript + Tailwind + podstawowe konfiguracje
2. **Mock dane i storage**: przygotuj strukturę danych (JSON/mock state/localStorage) z przykładowymi danymi
3. **Symulacja roli i stanu**: dodaj switche/selecty w UI (rola: Admin/User, stan fazy: 3 opcje)
4. **Struktura danych**: typy TypeScript, mock data z turniejem, fazami, meczami, uczestnikami, typami
5. **Dołączanie do turnieju**: formularz wniosku (mock), panel admina z akceptacją (widoczny tylko gdy rola=Admin)
6. **Fazy i mecze**: tworzenie fazy przez admina, dodawanie meczów (tylko gdy rola=Admin)
7. **Typowanie**: formularz, zapis typów (mock storage), walidacja stanu (blokada gdy stan="Po lock_at")
8. **Filtrowanie widoczności**: logika pokazywania typów (tylko własne przed start_at, wszystkie po)
9. **Wyniki i punktacja**: wprowadzanie wyników przez admina, automatyczne liczenie punktów (5/3)
10. **Dashboardy**: macierz typów, leaderboard, widoki zależne od stanu fazy
11. **Czas i strefy**: konwersja UTC → Europe/Warsaw, liczniki (symulowane w zależności od stanu)
12. **Testy scenariuszy**: przełączanie ról i stanów, weryfikacja widoków

## Uwagi końcowe

- **Priorytet: szybkość implementacji** - użyj najprostszych rozwiązań (localStorage, mock data, prosty backend)
- **NIE używaj Supabase ani OAuth** - symulacja roli i stanu przez switche/selecty
- **Priorytet: weryfikacja widoków** - kluczowe jest sprawdzenie jak dashboardy zmieniają się w różnych fazach i rolach
- Użyj prostych, ale czytelnych komponentów (Shadcn/ui)
- **Symulacja stanu fazy** jest kluczowa - użytkownik musi móc szybko przełączać między stanami aby zobaczyć różnice
- Obsługa błędów: podstawowe komunikaty
- Dokumentacja: podstawowy README z instrukcją setupu, uruchomienia i używania switchey symulacji

---

**Zacznij od stworzenia planu pracy i przedstawienia go użytkownikowi przed rozpoczęciem implementacji.**
