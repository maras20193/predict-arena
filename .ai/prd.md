# Dokument wymagań produktu (PRD) - PredictArena (MVP)

## 1. Przegląd produktu

Produkt to webowa aplikacja do prywatnych typowań wyników meczów w gronie znajomych. Celem MVP jest zapewnienie bezbłędnej, moderowanej przez admina organizacji typowań dla jednego turnieju z fazami, z prostą i konfigurowalną punktacją oraz przejrzystymi dashboardami. Aplikacja działa bez realtime; dane odświeżają się po przeładowaniu strony.

Zakres MVP obejmuje:

- Logowanie przez Google (Supabase Auth) i podstawowy profil z avatarem.
- Jeden turniej z fazami, które mają start_at (otwarcie/ujawnienie typów) oraz lock_at (zamknięcie edycji typów); domyślnie w MVP start_at = lock_at (tryb prosty), z opcją rozdzielenia w trybie zaawansowanym. Czas w backendzie w UTC, w UI prezentowany w Europe/Warsaw.
- Dołączenie do turnieju na wniosek użytkownika, zatwierdzane przez jedynego admina turnieju; możliwość dołączenia tylko do momentu startu turnieju.
- Formularz typowania dostępny wyłącznie dla otwartych faz; po lock_at edycja jest niemożliwa i blokowana na backendzie (RLS/serwer).
- Konfigurowalna punktacja per turniej: domyślnie 5 punktów za dokładny wynik oraz 3 punkty za poprawne wskazanie zwycięzcy/strony; punktacja wyłącznie za wynik dla 90 minut.
- Dashboardy: macierz typów (jedyna tabela; wiersze = mecze, kolumny = użytkownicy), leaderboard prezentowany jako wykres słupkowy sumy punktów (+ avatary) bez odrębnej tabeli, flagi drużyn (ISO, CDN/fallback); opcjonalnie kafelki highlight (np. Best Predictor, najwięcej trafień 5 pkt).
- Widok Moje typy z przeglądem złożonych i otwartych typów oraz dostępem do historii poprzednich faz.

Najważniejsze decyzje projektowe:

- Lock_at utrzymywane na poziomie fazy; ujawnianie typów następuje po starcie fazy (start_at).
- Start_at steruje wyłącznie widocznością typów; lock_at steruje wyłącznie możliwością edycji.
- Relacja czasów: zazwyczaj lock_at ≤ start_at (najpierw koniec edycji, potem ujawnienie).
- Domyślnie w MVP start_at = lock_at (tryb prosty); model i UI wspierają rozdzielenie tych czasów (tryb zaawansowany).
- Backend przechowuje i rozlicza czas w UTC; UI prezentuje czasy w Europe/Warsaw.
- Ranking bez tie-breaków, z miejscami z lukami (np. 1,1,3; 1,2,3,3,3,6).
- Brak realtime w MVP; dane aktualizowane po odświeżeniu strony.

## 2. Problem użytkownika

Obecnie prywatne typowania są organizowane chaotycznie: w arkuszach kalkulacyjnych, bez wiarygodnych blokad czasowych, z ręcznym i podatnym na błędy liczeniem punktów oraz bez moderacji przez administratora. Istniejące serwisy rzadko pozwalają na konfigurację zasad punktowania i kontrolę dostępu w ramach zamkniętej grupy.

Konsekwencje:

- Błędy w punktacji i spory o wyniki.
- Brak pewności co do terminów zamknięcia edycji typów i uczciwości.
- Czasochłonność przygotowania turnieju i zarządzania uczestnikami.
- Ograniczona przejrzystość wyników i typów, szczególnie w większych grupach.

Aplikacja rozwiązuje te problemy przez: centralizację danych, egzekwowanie blokad po lock_at, moderację dostępu, automatyczne naliczanie punktów oraz wydajne wizualizacje (leaderboard i macierz typów).

## 3. Wymagania funkcjonalne

3.1. Uwierzytelnianie i profil

- Podstawowy system uwierzytelniania oraz Google via Supabase Auth.
- Profil użytkownika z avatarem (wyświetlany w listach, macierzy, leaderboardzie).

  3.2. Turniej i role

- Jeden turniej w UI (DB może wspierać wiele turniejów, ale UI obsługuje jeden).
- Dwie role: user i jeden admin na turniej.
- Dwie role: user i jeden admin na turniej.
- Admin może również brać udział w turnieju jako uczestnik; jego typy i punkty są uwzględniane w macierzy i rankingu.
- Dołączenie do turnieju wyłącznie do momentu jego startu; po starcie zapisy są zablokowane.

  3.3. Fazy i mecze

- Faza posiada start_at (moment ujawnienia typów) oraz lock_at (koniec możliwości edycji typów).
- Admin tworzy fazę i dodaje mecze; dla ergonomii dodawanie meczów odbywa się szybko (możliwy batch w formularzu) i może dziedziczyć domyślne czasy z fazy.
- Reschedule na poziomie fazy przez aktualizację start_at i/lub lock_at; UI prezentuje komunikat o zmianie.
- Faza może mieć podział na grupy (stosowane tylko do stylowania UI, grupy nie mają swoich osobnych czasów).

  3.4. Dołączanie do turnieju

- Użytkownik wysyła prośbę o dołączenie; admin akceptuje lub odrzuca.
- Dołączenie skuteczne tylko przed startem turnieju.

  3.5. Typowanie

- Formularz typowania obejmuje wszystkie mecze w danej fazie.
- Edycja typów do lock_at; po lock_at edycja jest niemożliwa i egzekwowana na backendzie (RLS/serwer).
- Lock_at nie ujawnia typów; ujawnienie następuje dopiero o start_at (jeśli różne).
- Brak konceptu draftu; przy błędzie POST formularz pozostaje zamontowany i zachowuje dane do ponownej próby.
- API powinno być idempotentne (request id) dla bezpiecznych powtórzeń.

  3.6. Widoczność typów i prywatność

- Przed startem fazy użytkownicy widzą tylko własne typy.
- Po starcie fazy typy wszystkich stają się widoczne dla uczestników.
- Domyślnie w MVP start_at = lock_at; przy rozdzieleniu czasów UI prezentuje oba liczniki i stany.
- RLS i logika backend egzekwują prywatność i blokady.

  3.7. Punktacja i wyniki

- Konfigurowalna punktacja per turniej: domyślnie 5 pkt za dokładny wynik, 3 pkt za poprawny wynik strony.
- Punktacja dotyczy wyniku po 90 minutach.
- Admin ręcznie wprowadza wyniki meczów.
- System automatycznie nalicza punkty zgodnie z konfiguracją.

  3.8. Dashboardy i raportowanie

- Leaderboard prezentowany jako wykres słupkowy sumy punktów (z avatarami), bez odrębnej tabeli rankingu; ranking ex aequo (z lukami) odwzorowany w etykietach/tooltipach.
- Macierz typów (jedyna tabela): wiersze = mecze, kolumny = użytkownicy, kolorowanie komórek (5 = accent1/zielony, 3 = accent2/niebieski), grupowanie po fazach/grupach, sticky headers, legenda.
- Kafelki highlight (opcjonalnie): np. Best Predictor (najwięcej trafień 5 pkt), najwyższy wynik w fazie, najdłuższa seria trafień.
- Dostęp do historii poprzednich faz (typy i punktacja po starcie danych faz).

  3.9. Dane referencyjne i zasoby

- Słownik drużyn/flag zgodny z ISO, flagi serwowane z CDN z fallbackiem.

  3.10. Analityka

- Lekka analityka: rejestrowane zdarzenia page_view, join_request, join_approved, picks_submitted, result_entered.
- Pomiar TTFB/LCP oraz czasów renderowania widoków kluczowych (leaderboard, macierz).

  3.11. Wydajność i niezawodność

- Leaderboard i macierz ładują się poniżej 2 s przy 30 graczach i kilkudziesięciu meczach (wirtualizacja listy, indeksy, ewentualne preagregacje leaderboardu).
- Stabilność zapisów bez draftu: idempotentne API i komunikaty błędów z możliwością łatwej ponownej próby.

  3.12. Czas i strefy czasowe

- Backend: UTC; UI: Europe/Warsaw; liczniki i prezentacja czasu spójne z DST.

  3.13. Bezpieczeństwo i RLS

- Brak możliwości edycji typów po lock_at.
- Brak dostępu do cudzych typów przed startem fazy.
- Autoryzacja roli admin do akceptacji zgłoszeń, tworzenia faz, dodawania meczów, wprowadzania wyników i konfiguracji punktacji.

## 4. Granice produktu

Poza zakresem MVP:

- Realtime/streaming wyników i punktów.
- Więcej niż jeden provider OAuth (na start tylko Google).
- Integracje z zewnętrznymi API terminarzy/wyników (admin wprowadza ręcznie).
- Zaawansowane reguły punktowania (bonusy, dogrywki/karne, mnożniki).
- Publiczne katalogi turniejów i UI wielo-turniejowe (DB może wspierać wiele, UI obsługuje jeden).
- Powiadomienia e-mail/push i rozbudowane systemy zaproszeń.
- Aplikacje mobilne; jedynie web.
- Import/eksport CSV.

Ograniczenia i założenia:

- Jeden admin na turniej (procedura awaryjnego przejęcia do doprecyzowania).
- Wynik meczów punktowany wyłącznie na podstawie stanu po 90 minutach.
- Dołączenie tylko przed startem turnieju.
- Ranking bez tie-breaków (ex aequo z lukami).

Otwarte kwestie do doprecyzowania:

- Szczegóły UI dla prezentacji dwóch liczników, gdy start_at ≠ lock_at.
- Zakres i forma komunikatu przy reschedule (banner/in-app).
- Wybór narzędzia analitycznego free (np. PostHog/GA4/Fathom) i polityka prywatności.
- Mechanizm awaryjnego transferu właściciela przy utracie dostępu przez jedynego admina.
- Polityka dla odwołanych/przerwanych meczów (np. void, brak punktów).
- Decyzja o wprowadzeniu batch add/klonowania faz w MVP dla ergonomii admina.

## 5. Historyjki użytkowników

US-001
Tytuł: Logowanie przez Google
Opis: Jako użytkownik chcę zalogować się jednym kliknięciem przez Google, aby szybko uzyskać dostęp do turnieju.
Kryteria akceptacji:

- Jeśli nie jestem zalogowany, widzę przycisk Zaloguj przez Google i po udanym logowaniu trafiam na stronę główną.
- Jeśli logowanie się nie powiedzie, widzę zrozumiały komunikat i mogę spróbować ponownie.

US-002
Tytuł: Profil i avatar
Opis: Jako użytkownik chcę mieć avatar widoczny przy moim nazwisku, aby łatwo identyfikować mnie na listach i w macierzy.
Kryteria akceptacji:

- Mogę ustawić lub zaktualizować avatar; jest wyświetlany w leaderboardzie i macierzy.
- W razie braku avatara stosowany jest bezpieczny domyślny placeholder.

US-003
Tytuł: Wysłanie prośby o dołączenie do turnieju
Opis: Jako użytkownik chcę wysłać prośbę o dołączenie, aby admin mógł mnie zaakceptować.
Kryteria akceptacji:

- Widzę formularz dołączenia, jeśli nie jestem uczestnikiem i turniej nie wystartował.
- Po wysłaniu prośby widzę status Oczekujące.
- Jeśli turniej wystartował, nie mogę wysłać prośby i otrzymuję komunikat, że zapisy zamknięte.

US-004
Tytuł: Akceptacja/odrzucenie prośby przez admina
Opis: Jako admin chcę akceptować lub odrzucać prośby o dołączenie, aby kontrolować dostęp.
Kryteria akceptacji:

- Widzę listę oczekujących wniosków z podstawowymi danymi użytkownika.
- Mogę zaakceptować lub odrzucić wniosek; po akceptacji użytkownik zyskuje dostęp do turnieju.
- Decyzja jest niedostępna po starcie turnieju (brak nowych akceptacji po starcie).

US-005
Tytuł: Widok aktywnej fazy z licznikami czasu
Opis: Jako uczestnik chcę widzieć start_at i lock_at fazy wraz z licznikami, aby wiedzieć, kiedy mogę edytować typy i kiedy będą ujawnione.
Kryteria akceptacji:

- Widzę odliczanie do start_at i lock_at w strefie Europe/Warsaw.
- Gdy start_at = lock_at, widzę jeden wspólny licznik; gdy różne, widzę dwa.
- Po osiągnięciu lock_at formularz edycji znika lub jest nieaktywny.

US-006
Tytuł: Składanie i edycja typów do lock_at
Opis: Jako uczestnik chcę wypełnić formularz typów dla wszystkich meczów fazy i edytować je do lock_at.
Kryteria akceptacji:

- Mogę wprowadzić typy i zapisać je wielokrotnie przed lock_at.
- Po błędzie zapisu widzę jasny komunikat, formularz pozostaje zamontowany, a dane nie znikają.
- Powtórna próba zapisu nie duplikuje danych (idempotencja).

US-007
Tytuł: Blokada edycji po lock_at
Opis: Jako uczestnik nie mogę edytować typów po lock_at.
Kryteria akceptacji:

- Próba zapisu po lock_at kończy się odmową na backendzie; UI wyświetla informację o zamknięciu edycji.
- Nie widzę możliwości edycji w UI po lock_at.

US-008
Tytuł: Prywatność typów przed startem fazy
Opis: Jako uczestnik widzę wyłącznie własne typy przed startem fazy.
Kryteria akceptacji:

- Nie mam możliwości podejrzenia cudzych typów przed start_at.
- Po start_at typy wszystkich są widoczne w macierzy i historii.

US-009
Tytuł: Historia faz i typów
Opis: Jako uczestnik chcę mieć dostęp do poprzednich faz, aby sprawdzić własne i cudze typy po ich ujawnieniu oraz uzyskane punkty.
Kryteria akceptacji:

- Mogę przełączyć się na wcześniejsze fazy i zobaczyć typy oraz naliczane punkty.

US-010
Tytuł: Konfiguracja punktacji przez admina
Opis: Jako admin chcę ustawić preset punktacji (np. 5/3) przy tworzeniu turnieju.
Kryteria akceptacji:

- Widzę opcję ustawienia punktów za dokładny wynik i poprawną stronę.
- Ustawiona konfiguracja wpływa na automatyczne naliczanie punktów.

US-011
Tytuł: Dodanie fazy i meczów przez admina
Opis: Jako admin chcę szybko utworzyć fazę z parametrami start_at i lock_at oraz dodać do niej mecze.
Kryteria akceptacji:

- Mogę utworzyć fazę, ustawić oba czasy i dodać listę meczów (ergonomia: batch w formularzu lub dziedziczenie domyślnych czasów).
- Czynności przygotowania fazy i meczów mieszczą się w 15 minutach.

US-012
Tytuł: Reschedule fazy
Opis: Jako admin chcę móc przesunąć start_at i/lub lock_at fazy i poinformować uczestników w aplikacji.
Kryteria akceptacji:

- Po zmianie czasu uczestnicy widzą banner/in-app komunikat o zmianie i zaktualizowane liczniki.
- Możliwe rozdzielenie lub ponowne zsynchronizowanie start_at i lock_at.
- Nowe czasy są respektowane przez logikę widoczności i edycji typów.

US-013
Tytuł: Wprowadzanie wyników meczów przez admina
Opis: Jako admin chcę szybko wprowadzać wyniki, by system naliczał punkty automatycznie.
Kryteria akceptacji:

- Wprowadzenie pojedynczego wyniku zajmuje do 15 sekund (UX, walidacje, focus, skróty).
- Po wprowadzeniu wyników uczestnicy widzą zaktualizowane punkty po odświeżeniu.

US-014
Tytuł: Automatyczne naliczanie punktów
Opis: Jako uczestnik chcę, by punkty były naliczane automatycznie zgodnie z konfiguracją, abym znał swój wynik bez ręcznych wyliczeń.
Kryteria akceptacji:

- Punkty naliczane są zgodnie z regułami 5/3 (lub innymi ustawionymi przez admina) i pokrywają 100 procent przetestowanych przypadków.

US-015
Tytuł: Leaderboard z rankingiem ex aequo
Opis: Jako uczestnik chcę zobaczyć ranking w formie wykresu słupkowego z miejscami ex aequo i sumą punktów (bez odrębnej tabeli rankingu).
Kryteria akceptacji:

- Ranking wyświetla pozycje z lukami (np. 1,1,3) i jest poprawny dla remisów.
- Leaderboard wczytuje się w mniej niż 2 sekundy dla 30 graczy i kilkudziesięciu meczów.

US-016
Tytuł: Macierz typów
Opis: Jako uczestnik chcę przejrzeć macierz typów (tabela: wiersze = mecze, kolumny = użytkownicy) z kolorami punktów, z grupowaniem po fazach i sticky headers.
Kryteria akceptacji:

- Komórki są kolorowane: 5 punktów (zielony/accent1), 3 punkty (niebieski/accent2), inne stany neutralne.
- Lista jest wirtualizowana i działa płynnie przy 100–150 meczach i 30+ graczach.

US-017
Tytuł: Odświeżanie danych bez realtime
Opis: Jako uczestnik po odświeżeniu strony widzę najnowsze dane.
Kryteria akceptacji:

- Zmiana wyników i punktów jest widoczna po reloadzie strony.

US-018
Tytuł: Bezpieczny dostęp do danych (RLS)
Opis: Jako właściciel danych chcę mieć pewność, że przed startem fazy nikt nie zobaczy moich typów i że po lock_at nie da się ich zmienić.
Kryteria akceptacji:

- Zapytania do cudzych typów przed start_at są odrzucane przez RLS; po start_at dostęp jest przyznany.
- Próby zapisu po lock_at są odrzucane przez RLS/serwer.

US-019
Tytuł: Ograniczenie dołączenia po starcie turnieju
Opis: Jako użytkownik nie mogę dołączyć do turnieju po jego starcie.
Kryteria akceptacji:

- Próba dołączenia po starcie wyświetla komunikat o zamkniętych zapisach i nie tworzy wniosku.

US-020
Tytuł: Stabilność zapisu typów bez draftu
Opis: Jako uczestnik po błędzie zapisu mogę bezpiecznie powtórzyć wysyłkę bez ryzyka duplikacji.
Kryteria akceptacji:

- Idempotentne API akceptuje ponowne żądanie bez tworzenia duplikatów; UI zachowuje dane formularza.

US-021
Tytuł: Analityka kluczowych zdarzeń
Opis: Jako zespół produktowy chcemy mierzyć kluczowe zdarzenia i wydajność renderowania.
Kryteria akceptacji:

- Rejestrowane są zdarzenia: page_view, join_request, join_approved, picks_submitted, result_entered.
- Zbierane są metryki TTFB i LCP dla kluczowych ekranów.

US-022
Tytuł: Flagi i drużyny z ISO
Opis: Jako uczestnik widzę poprawne flagi i nazwy drużyn zgodne z ISO.
Kryteria akceptacji:

- Flagi ładują się z CDN z mechanizmem fallback.

US-023
Tytuł: Dostęp do turnieju tylko dla zaakceptowanych
Opis: Jako niezaakceptowany użytkownik nie widzę danych turnieju, mogę tylko złożyć prośbę o dołączenie.
Kryteria akceptacji:

- Widoki turniejowe są chronione; bez akceptacji widzę wyłącznie ekran dołączenia.

US-024
Tytuł: Spójność czasu i strefy
Opis: Jako uczestnik chcę, aby czasy były spójne niezależnie od DST.
Kryteria akceptacji:

- Prezentowane czasy wynikają z UTC i są poprawnie konwertowane do Europe/Warsaw, w tym podczas przejść DST.

US-025
Tytuł: Widoczność typów po starcie fazy
Opis: Jako uczestnik chcę zobaczyć typy innych po starcie fazy.
Kryteria akceptacji:

- Natychmiast po start_at fazy cudze typy są widoczne w macierzy i historii.

## 6. Metryki sukcesu

- 20–30 aktywnych graczy dołącza i jest zaakceptowanych w turnieju.
- Co najmniej 85 procent graczy składa typy dla pełnej fazy przed lock_at.
- Zero możliwości edycji typów po lock_at (testy RLS i brak odrzuconych zapisów w logach produkcyjnych dla poprawnych ścieżek).
- Dashboardy (tabela i wykres) ładują się poniżej 2 sekund przy 30 graczach i kilkudziesięciu meczach.
- Admin tworzy fazę i dodaje mecze w czasie nie dłuższym niż 15 minut.
- Admin wprowadza pojedynczy wynik meczu w czasie nie dłuższym niż 15 sekund.
- Automatyczne naliczanie punktów zgodne z konfiguracją w 100 procentach przetestowanych przypadków.
- Zdarzenia analityczne rejestrowane dla page_view, join_request, join_approved, picks_submitted, result_entered; pomiary TTFB i LCP zbierane dla kluczowych ekranów.
