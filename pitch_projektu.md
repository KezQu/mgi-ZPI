# Pitch projektu: ZwrotLog

**Platforma do obsługi logistyki opakowań kaucyjnych**

## Problem

Od 1 października 2025 r. w Polsce działa system kaucyjny. Sklepy, restauracje i firmy przyjmują zwracane butelki i puszki, ale:

- opakowania zalegają na zapleczach i zajmują cenne miejsce,
- nie ma prostego sposobu, aby zamówić odbiór, śledzić jego status i rozliczyć zebrane ilości,
- mniejsze punkty są pomijane, bo pojedynczy kurs do nich jest nieopłacalny.

## Rozwiązanie

Jedna platforma obejmująca cały proces – od zgłoszenia firmy do przekazania opakowań operatorowi systemu kaucyjnego:

1. **Rejestracja partnera:** firma zgłasza punkt, akceptuje umowę i podaje adres oraz typy opakowań.
2. **Zgłaszanie odbiorów:** partner zamawia odbiór jednorazowo lub cyklicznie i śledzi jego status.
3. **Planowanie tras i flota:** system grupuje zgłoszenia w trasy, przydziela ciężarówki i kierowców, a kierowca ma aplikację z listą przystanków.
4. **Magazyn:** przyjęcie dostawy, liczenie, sortowanie (PET, puszki, szkło) i wydanie do operatora systemu kaucyjnego.
5. **Rozliczenia i raporty:** ile opakowań odebrano od kogo, rozliczenia z partnerami i operatorem oraz raporty wymagane prawem.

## Wizja

> „Żaden worek z kaucją nie czeka na zapleczu dłużej niż 48 godzin.”

## Wartość

- **Dla partnerów:** odzyskane miejsce, oszczędność czasu, przejrzyste rozliczenia.
- **Dla operatora systemu kaucyjnego:** przewidywalny strumień posortowanych opakowań.
- **Społecznie:** wyższy poziom zbiórki, który ustawa wymaga od operatorów, oraz mniej odpadów.

## Dlaczego to dobry temat na ZPI

- System składa się z kilku wyraźnie odrębnych modułów, więc każdy członek zespołu może przyjąć inny cel biznesowy.
- Ma realne ograniczenia: ustawę kaucyjną, RODO, budżet MVP, integrację z operatorem i usługi map/GPS.
- Moduły mają różne kluczowe atrybuty jakościowe (np. dostępność, wydajność, bezpieczeństwo, praca offline).

---

## Podział ról w zespole (5 osób)

Projekt jest realizowany wspólnie, ale zgodnie z uwagą w treści zadania każda osoba przygotowuje dokument driverów architektonicznych dla **innego, istotnie odmiennego celu biznesowego** – z własnymi personami, historyjkami, atrybutami jakościowymi, ograniczeniami i założeniami.

| Osoba | Moduł | Przykładowy cel SMART | Główne persony | Najważniejsze atrybuty jakościowe |
| --- | --- | --- | --- | --- |
| 1 | **Pozyskiwanie partnerów** (portal rejestracji, umowy, panel partnera) | Pozyskanie 300 aktywnych punktów w ciągu 6 miesięcy od uruchomienia MVP | Właściciel sklepu osiedlowego, menedżer sieci restauracji | Użyteczność, bezpieczeństwo (RODO), dostępność |
| 2 | **Zgłaszanie i harmonogram odbiorów** (zamówienia, odbiory cykliczne, statusy) | 95% odbiorów zrealizowanych w ciągu 48 h od zgłoszenia w ciągu 4 miesięcy | Kierownik zmiany w markecie, dyspozytor | Dostępność, wydajność, niezawodność powiadomień |
| 3 | **Planowanie tras i flota** (optymalizacja tras, aplikacja kierowcy, GPS) | Obniżenie kosztu kursu w przeliczeniu na 1000 opakowań o 20% w ciągu 6 miesięcy | Dyspozytor floty, kierowca | Wydajność obliczeń, praca offline (mobile), skalowalność |
| 4 | **Magazyn i sortowanie** (przyjęcia, liczenie, stany, wydania do operatora) | Rozbieżność między deklaracją a stanem magazynowym poniżej 2% w ciągu 5 miesięcy | Magazynier, kierownik magazynu | Integralność danych, testowalność, wydajność przyjęć |
| 5 | **Rozliczenia i integracja z operatorem** (rozliczenia, faktury, raporty ustawowe) | Skrócenie czasu rozliczenia z partnerem z 30 do 7 dni w ciągu 6 miesięcy | Księgowa partnera, analityk operatora systemu kaucyjnego | Bezpieczeństwo, audytowalność, interoperacyjność (API operatora) |

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
- **Wspólny kontekst** – problem rynkowy i ogólną wizję zespół ustala razem, ale każdy opisuje je własnymi słowami z perspektywy swojego modułu.

---

## Harmonogram pracy

| Etap | Zakres | Kto |
| --- | --- | --- |
| 1. Uzgodnienia zespołu | Akceptacja tematu przez prowadzącego, przydział modułów i celów biznesowych, wspólny słownik pojęć | Cały zespół |
| 2. Wizja i zakres | Problem, wizja, cel SMART, KPI, persony (sekcja 1) | Każdy indywidualnie |
| 3. Wymagania i MVP | Historyjki użytkownika, tabela priorytetyzacji, uzasadnienie MVP (sekcja 2) | Każdy indywidualnie |
| 4. Atrybuty jakościowe | Wybór atrybutów, scenariusze TOP 3, analiza kompromisów (sekcja 3) | Każdy indywidualnie |
| 5. Ograniczenia i założenia | Ograniczenia z wpływem na architekturę, założenia z planem walidacji (sekcja 4) | Każdy indywidualnie |
| 6. Przegląd spójności | Sprawdzenie, czy cele, persony i treści się nie powielają; korekta językowa | Cały zespół |
| 7. Oddanie | Eksport do PDF i wysłanie e-maila „ZPI – Drivery architektoniczne – Imię i Nazwisko” | Każdy indywidualnie (termin: ostatnie zajęcia) |
