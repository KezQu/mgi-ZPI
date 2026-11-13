# Pitch projektu: ZwrotLog

**Platforma do obsługi logistyki opakowań kaucyjnych**
_Przedmiot: Zarządzanie projektem informatycznym_

## Problem

Od 1 października 2025 r. w Polsce działa system kaucyjny. Sklepy, restauracje i firmy przyjmują zwracane butelki i puszki, ale:

- opakowania zalegają na zapleczach i zajmują cenne miejsce,
- nie ma prostego sposobu, aby zamówić odbiór, śledzić jego status i rozliczyć zebrane ilości,
- mniejsze punkty są pomijane, bo pojedynczy kurs do nich jest nieopłacalny.

## Rozwiązanie

System informatyczny obejmujący cały proces – od zgłoszenia firmy do przekazania opakowań operatorowi systemu kaucyjnego:

1. **Rejestracja partnera:** firma zgłasza punkt, akceptuje umowę i podaje adres oraz typy opakowań.
2. **Zgłaszanie odbiorów:** partner zamawia odbiór jednorazowo lub cyklicznie i śledzi jego status.
3. **Planowanie tras i flota:** system grupuje zgłoszenia w trasy, przydziela ciężarówki i kierowców, a kierowca ma aplikację z listą przystanków.
4. **Magazyn:** przyjęcie dostawy, liczenie, sortowanie (PET, puszki, szkło) i wydanie do operatora systemu kaucyjnego.
5. **Rozliczenia i raporty:** ile opakowań odebrano od kogo, rozliczenia z partnerami i operatorem oraz raporty wymagane prawem.

## Wizja

> „Żaden worek z kaucją nie czeka na zapleczu dłużej niż 7 dni.”

## Wartość

- **Dla partnerów:** odzyskane miejsce, oszczędność czasu, przejrzyste rozliczenia.
- **Dla operatora systemu kaucyjnego:** przewidywalny strumień posortowanych opakowań.
- **Społecznie:** wyższy poziom zbiórki, który ustawa wymaga od operatorów, oraz mniej odpadów.

---

## Część informatyczna – wstępna koncepcja

Poniższe propozycje są punktem wyjścia do dyskusji i będą uzasadniane w dokumentach driverów architektonicznych (atrybuty jakościowe, ograniczenia, założenia).

### Typy aplikacji

| Aplikacja                         | Użytkownicy                                   | Typ                                                                  | Uzasadnienie                                            |
| --------------------------------- | --------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------- |
| Portal partnera                   | Sklepy, restauracje, firmy                    | Aplikacja webowa (SPA, responsywna)                                  | Dostęp z komputera i telefonu bez instalacji            |
| Panel dyspozytora i administracji | Dyspozytorzy, administratorzy, księgowość     | Aplikacja webowa (SPA)                                               | Praca biurowa na dużym ekranie, mapy, tabele            |
| Aplikacja kierowcy                | Kierowcy                                      | Aplikacja mobilna (np. React Native / Flutter lub PWA), tryb offline | Praca w terenie, słaby zasięg, GPS, zdjęcia potwierdzeń |
| Aplikacja magazynowa              | Magazynierzy                                  | Aplikacja webowa na tablet / terminal ze skanerem                    | Szybkie przyjęcia, skanowanie kodów worków              |
| API integracyjne                  | Operator systemu kaucyjnego, systemy księgowe | REST API + webhooki                                                  | Wymiana danych o ilościach i rozliczeniach              |

### Architektura

- **Start: modularny monolit** – jeden deployowalny backend podzielony na moduły (partnerzy, odbiory, trasy, magazyn, rozliczenia). Niższy koszt i prostsze wdrożenie przy małym zespole i budżecie MVP.
- **Ewolucja: wydzielanie usług** – moduły o innym profilu obciążenia (np. optymalizacja tras, telemetria GPS) mogą zostać wydzielone później jako osobne usługi.
- **Dokumentacja architektury:** diagramy C4 (kontekst, kontenery, komponenty) oraz rejestr decyzji architektonicznych (ADR).

### Komunikacja: synchroniczna czy asynchroniczna

| Przypadek                                                    | Typ komunikacji                                | Uzasadnienie                                                       |
| ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------------ |
| Interfejs użytkownika ↔ backend (formularze, listy, statusy) | Synchroniczna (REST/HTTP)                      | Użytkownik czeka na natychmiastową odpowiedź                       |
| Nowe zgłoszenie odbioru → planowanie tras                    | Asynchroniczna (kolejka, np. RabbitMQ)         | Planowanie jest kosztowne obliczeniowo, nie może blokować partnera |
| Pozycje GPS z aplikacji kierowcy                             | Asynchroniczna (strumień zdarzeń)              | Duża liczba drobnych komunikatów, tolerancja na opóźnienia         |
| Synchronizacja aplikacji kierowcy po odzyskaniu zasięgu      | Asynchroniczna (kolejka offline na urządzeniu) | Brak zasięgu nie może blokować pracy                               |
| Powiadomienia (e-mail, SMS, push)                            | Asynchroniczna                                 | Ponawianie przy awarii dostawcy zewnętrznego                       |
| Rozliczenia z operatorem systemu kaucyjnego                  | Asynchroniczna (zadania wsadowe + webhooki)    | Zewnętrzny system o nieznanej dostępności                          |

### Baza danych: SQL czy NoSQL

| Dane                                                        | Proponowane rozwiązanie                                | Uzasadnienie                                                |
| ----------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------- |
| Partnerzy, umowy, zgłoszenia, stany magazynowe, rozliczenia | **SQL** – PostgreSQL                                   | Transakcyjność (ACID), relacje, spójność danych finansowych |
| Lokalizacje punktów, trasy                                  | PostgreSQL + **PostGIS**                               | Zapytania przestrzenne (odległości, obszary)                |
| Telemetria GPS, logi zdarzeń                                | **NoSQL / szeregi czasowe** (np. TimescaleDB, MongoDB) | Duży wolumen zapisów, dane append-only                      |
| Cache, sesje, kolejki pomocnicze                            | **NoSQL klucz-wartość** – Redis                        | Szybki odczyt, dane tymczasowe                              |
| Zdjęcia potwierdzeń odbioru, dokumenty                      | Object storage (S3-kompatybilny)                       | Tani magazyn plików                                         |

### Dostępność: 24/7 czy z oknem serwisowym

Różne części systemu mają różne wymagania – nie wszystko musi działać 24/7:

| Element                       | Docelowa dostępność                                     | Uzasadnienie                                          |
| ----------------------------- | ------------------------------------------------------- | ----------------------------------------------------- |
| Portal partnera (zgłoszenia)  | 99,5%, okno serwisowe w nocy (np. 1 h, model „23/7”)    | Zgłoszenia głównie w godzinach pracy sklepów          |
| Aplikacja kierowcy            | Praca offline-first – działa nawet przy awarii backendu | Odbiory nie mogą stanąć przez brak zasięgu lub awarię |
| Panel dyspozytora             | 99,5% w godzinach 6:00–22:00                            | Praca zmianowa floty                                  |
| API integracyjne z operatorem | 99,9% (24/7)                                            | Wymagania zewnętrznego partnera i terminy rozliczeń   |

Mechanizmy: health-checki, automatyczny restart kontenerów, co najmniej 2 instancje backendu za load balancerem, monitoring i alerty (np. Prometheus + Grafana), wdrożenia bez przestojów (rolling / blue-green).

### Backup i odtwarzanie

- **Baza danych:** codzienny pełny backup + ciągła archiwizacja WAL (Point-in-Time Recovery).
- **Cele:** RPO ≤ 15 min (maks. utrata danych), RTO ≤ 4 h (maks. czas przywrócenia) – do doprecyzowania w scenariuszach jakościowych.
- **Zasada 3-2-1:** 3 kopie, 2 różne nośniki, 1 kopia w innej lokalizacji (w granicach EOG – RODO).
- **Pliki:** wersjonowanie w object storage.
- **Testy odtwarzania:** cykliczne przywracanie backupu na środowisku testowym.

### Infrastruktura

- **Chmura z regionem w EOG** (np. Azure / AWS / GCP – region Polska lub Niemcy) – wymóg RODO.
- **Konteneryzacja:** Docker; na start zarządzane usługi kontenerowe, ewentualnie Kubernetes przy wzroście skali.
- **Infrastructure as Code:** Terraform – powtarzalne środowiska (dev, test, prod).
- **CI/CD:** GitHub Actions – testy, budowanie obrazów, automatyczne wdrożenia.
- **Bezpieczeństwo:** uwierzytelnianie OAuth2/OIDC, role (partner, kierowca, magazynier, dyspozytor, admin), szyfrowanie w tranzycie (TLS) i w spoczynku, logi audytowe.
- **Koszty:** optymalizacja pod budżet MVP (mniejsze instancje, skalowanie w dół poza godzinami pracy).

### Design (UX/UI)

- Wspólny design system (komponenty, kolory, typografia) dla wszystkich aplikacji webowych.
- Prototypy klikalne w Figmie testowane z użytkownikami przed implementacją.
- Aplikacja kierowcy i magazynowa: duże przyciski, obsługa w rękawicach, minimum kroków.
- Responsywność i dostępność (WCAG 2.1 AA) w portalu partnera.

---

## Podział ról w zespole (5 osób)

Projekt jest realizowany wspólnie, ale zgodnie z uwagą w treści zadania każda osoba przygotowuje dokument driverów architektonicznych dla **innego, istotnie odmiennego celu biznesowego** – z własnymi personami, historyjkami, atrybutami jakościowymi, ograniczeniami i założeniami. Każda osoba odpowiada też za jeden obszar techniczny całego zespołu (z mapy myśli).

| Osoba | Moduł                                                                              | Przykładowy cel SMART                                                            | Główne persony                                            | Najważniejsze atrybuty jakościowe                                | Obszar techniczny w zespole                                      |
| ----- | ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------- |
| 1     | **Pozyskiwanie partnerów** (portal rejestracji, umowy, panel partnera)             | Pozyskanie 300 aktywnych punktów w ciągu 6 miesięcy od uruchomienia MVP          | Właściciel sklepu osiedlowego, menedżer sieci restauracji | Użyteczność, bezpieczeństwo (RODO), dostępność                   | Design (UX/UI), design system, bezpieczeństwo i uwierzytelnianie |
| 2     | **Zgłaszanie i harmonogram odbiorów** (zamówienia, odbiory cykliczne, statusy)     | 95% odbiorów zrealizowanych w ciągu 48 h od zgłoszenia w ciągu 4 miesięcy        | Kierownik zmiany w markecie, dyspozytor                   | Dostępność, wydajność, niezawodność powiadomień                  | Dostępność (24/7 vs okno serwisowe), monitoring                  |
| 3     | **Planowanie tras i flota** (optymalizacja tras, aplikacja kierowcy, GPS)          | Obniżenie kosztu kursu w przeliczeniu na 1000 opakowań o 20% w ciągu 6 miesięcy  | Dyspozytor floty, kierowca                                | Wydajność obliczeń, praca offline (mobile), skalowalność         | Komunikacja synchroniczna/asynchroniczna, aplikacja mobilna      |
| 4     | **Magazyn i sortowanie** (przyjęcia, liczenie, stany, wydania do operatora)        | Rozbieżność między deklaracją a stanem magazynowym poniżej 2% w ciągu 5 miesięcy | Magazynier, kierownik magazynu                            | Integralność danych, testowalność, wydajność przyjęć             | Baza danych (SQL vs NoSQL), model danych                         |
| 5     | **Rozliczenia i integracja z operatorem** (rozliczenia, faktury, raporty ustawowe) | Skrócenie czasu rozliczenia z partnerem z 30 do 7 dni w ciągu 6 miesięcy         | Księgowa partnera, analityk operatora systemu kaucyjnego  | Bezpieczeństwo, audytowalność, interoperacyjność (API operatora) | Backup i odtwarzanie, infrastruktura, CI/CD                      |

### Jak uniknąć powielania treści

- **Ograniczenia** – każdy wybiera te najistotniejsze dla swojego modułu:
  - Osoba 1: RODO i przechowywanie danych w EOG,
  - Osoba 2: budżet infrastruktury MVP,
  - Osoba 3: limity zewnętrznych API map i routingu,
  - Osoba 4: sprzęt magazynowy (skanery, wagi),
  - Osoba 5: ustawa kaucyjna i format raportów dla operatora.
- **Założenia** – również różne, np.:
  - „Partnerzy zgłaszają odbiór z wyprzedzeniem co najmniej 24 h” (Osoba 2),
  - „Darmowe API do routingu obsłuży wymaganą liczbę zapytań” (Osoba 3),
  - „Operator systemu kaucyjnego udostępnia API do rozliczeń” (Osoba 5).
- **Wspólny kontekst** – problem rynkowy, ogólną wizję i ramy techniczne (stos technologiczny, styl architektury) zespół ustala razem, ale każdy opisuje je własnymi słowami i uzasadnia z perspektywy swojego modułu.

---

## Harmonogram pracy

| Etap                             | Zakres                                                                                                                                                                                                                                                                                         | Kto                                            |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| 1. Uzgodnienia zespołu           | Akceptacja tematu przez prowadzącego, przydział modułów, celów biznesowych i obszarów technicznych, wspólny słownik pojęć                                                                                                                                                                      | Cały zespół                                    |
| 2. Wizja i zakres                | Problem, wizja, cel SMART, KPI, persony (sekcja 1)                                                                                                                                                                                                                                             | Każdy indywidualnie                            |
| 3. Wymagania i MVP               | Historyjki użytkownika, tabela priorytetyzacji, uzasadnienie MVP (sekcja 2)                                                                                                                                                                                                                    | Każdy indywidualnie                            |
| 4. Wspólne ramy techniczne       | Typy aplikacji, styl architektury (modularny monolit), wstępny stos technologiczny, diagram C4 – poziom kontekstu i kontenerów                                                                                                                                                                 | Cały zespół (prowadzi Osoba 3)                 |
| 5. Analiza obszarów technicznych | Rekomendacje dla obszarów z mapy myśli – każda zapisana jako ADR:<br>• baza danych SQL vs NoSQL (Osoba 4)<br>• komunikacja sync vs async (Osoba 3)<br>• dostępność 24/7 vs okno serwisowe (Osoba 2)<br>• backup, infrastruktura, CI/CD (Osoba 5)<br>• design system i bezpieczeństwo (Osoba 1) | Każdy w swoim obszarze, przegląd zespołowy     |
| 6. Atrybuty jakościowe           | Wybór atrybutów, scenariusze TOP 3, analiza kompromisów (sekcja 3) – z wykorzystaniem decyzji z etapu 5                                                                                                                                                                                        | Każdy indywidualnie                            |
| 7. Ograniczenia i założenia      | Ograniczenia z wpływem na architekturę, założenia z planem walidacji (sekcja 4)                                                                                                                                                                                                                | Każdy indywidualnie                            |
| 8. Prototypy i weryfikacja       | Prototyp UI w Figmie, PoC ryzykownych elementów (np. optymalizacja tras, API map, tryb offline)                                                                                                                                                                                                | Właściciele obszarów                           |
| 9. Przegląd spójności            | Sprawdzenie zgodności dokumentów z ADR i diagramami C4, brak powielania celów i treści, korekta językowa                                                                                                                                                                                       | Cały zespół                                    |
| 10. Oddanie                      | Eksport do PDF i wysłanie e-maila „ZPI – Drivery architektoniczne – Imię i Nazwisko”                                                                                                                                                                                                           | Każdy indywidualnie (termin: ostatnie zajęcia) |
