# Copilot - Instrukcje Architektoniczne dla Projektu
*   **Wersja:** 1.0
*   **Data:** 2025-11-15 11:26:18
*   **Właściciel:** @pliszkaPL

## 1. Filozofia Architektury

Ten dokument opisuje architekturę aplikacji opartej na frameworku Laravel. Celem jest stworzenie modularnego monolitu, który jest łatwy w utrzymaniu, testowaniu i ewolucji w kierunku mikroserwisów. Podstawowe wzorce to:
1.  **Domain-Driven Design (DDD):** Dzielimy system na logiczne moduły (Bounded Contexts) z własnym, spójnym modelem domenowym.
2.  **Vertical Slice Architecture:** Grupujemy kod według funkcjonalności (przypadków użycia), a nie według typów technicznych.
3.  **Ports & Adapters (Hexagonal Architecture):** Izolujemy logikę biznesową (domenę i aplikację) od zewnętrznych zależności (framework, baza danych, API).
4.  **CQRS (Command Query Responsibility Segregation):** Używamy dedykowanych obiektów (Command, Query) do reprezentowania intencji użytkownika.

## 2. Struktura Katalogów

Główna logika aplikacji znajduje się w katalogu `src/`, aby oddzielić ją od struktury Laravela. Katalog `app/` służy głównie jako "klej" konfiguracyjny.

```
src/
├── Content/                  # Bounded Context (np. Zarządzanie Treścią)
│   ├── Contracts/            # Publiczny Kontrakt Kontekstu (np. Zdarzenia Integracyjne)
│   ├── Features/             # Katalog na Vertical Slices
│   │   └── PublishBook/      # Przykład Vertical Slice
│   │       ├── PublishBookController.php
│   │       ├── PublishBookCommand.php
│   │       └── PublishBookHandler.php
│   ├── Domain/               # Rdzeń Biznesowy Kontekstu (framework-agnostic)
│   │   ├── Entity/
│   │   ├── Repository/       # Interfejsy Repo (Porty)
│   │   └── Event/            # Zdarzenia Domenowe (wewnętrzne)
│   ├── Infrastructure/       # Implementacje (Adaptery)
│   │   ├── Persistence/
│   │   │   ├── Eloquent/
│   │   │   │   ├── BookModel.php
│   │   │   │   └── EloquentBookRepository.php
│   │   │   └── Mappers/
│   │   │       └── BookMapper.php
│   │   └── Providers/        # Service Provider dla tego Kontekstu
│   └── Shared/               # (Opcjonalnie) Kod współdzielony wewnątrz tego Kontekstu
│
├── Sales/                    # Inny Bounded Context
│   └── ...
│
└── Shared/                     # Globalny Kod Techniczny (horyzontalny)
    ├── Application/
    │   └── Bus/              # Interfejsy dla Command/Query/Event Bus
    └── Infrastructure/
        └── Bus/              # Implementacje Busów
```

## 3. Definicje Warstw i Komponentów

### 3.1. Warstwa `Domain`
*   **Cel:** Zawiera logikę i reguły biznesowe. Musi być "czysta" - **zero zależności od Laravela** i innych frameworków.
*   **Komponenty:**
    *   **Entity (Encja):** Obiekt z tożsamością i cyklem życia, zawierający logikę biznesową (np. `Book::publish()`).
    *   **Value Object (Obiekt Wartości):** Niezmienny obiekt opisujący atrybut, bez tożsamości (np. `Price`, `ISBN`).
    *   **Repository Interface (Port):** Interfejs definiujący metody dostępu do danych (np. `BookRepositoryInterface`). Nie zawiera szczegółów implementacji.
    *   **Domain Event:** Obiekt opisujący zdarzenie, które miało miejsce w domenie (np. `BookTitleWasChanged`). Używany do komunikacji *wewnątrz* Bounded Contextu.

### 3.2. Warstwa `Application` (wewnątrz `Features/*`)
*   **Cel:** Orkiestruje przepływy (use cases). Nie zawiera logiki biznesowej - deleguje ją do domeny.
*   **Komponenty:**
    *   **Command/Query:** Proste obiekty DTO reprezentujące intencję (np. `PublishBookCommand`).
    *   **Handler:** Klasa, która wykonuje logikę aplikacji:
        1.  Pobiera encje z repozytorium.
        2.  Wywołuje na nich metody biznesowe.
        3.  Zapisuje zmiany za pomocą repozytorium.
        4.  Może dispatchować zdarzenia.

### 3.3. Warstwa `Presentation` (wewnątrz `Features/*`)
*   **Cel:** Punkt wejścia do aplikacji. Tłumaczy żądania (HTTP, CLI) na komendy/zapytania.
*   **Komponenty:**
    *   **Controller/Artisan Command:** Pobiera dane z `Request`, tworzy obiekt `Command`/`Query`, przekazuje go do Busa i zwraca `Response`. Powinien być jak najprostszy.

### 3.4. Warstwa `Infrastructure`
*   **Cel:** Implementacja ("Adaptery") interfejsów z warstwy `Domain` i `Application`. Tutaj znajdują się wszystkie zależności od zewnętrznych narzędzi.
*   **Komponenty:**
    *   **Eloquent Repository:** Implementacja interfejsu repozytorium używająca modeli Eloquent.
    *   **Eloquent Model:** Model bazy danych. Jest to detal implementacyjny, a nie obiekt domenowy.
    *   **Mapper:** Dedykowana klasa odpowiedzialna za mapowanie między Encją Domenową a Modelem Eloquent.
    *   **Service Clients:** Klienty do komunikacji z zewnętrznymi API (np. bramką płatniczą).

## 4. Zdarzenia: Domenowe vs. Integracyjne

### 4.1. Zdarzenia Domenowe (Domain Events)
*   **Cel:** Komunikacja **wewnątrz** jednego Bounded Contextu. Służą do redukcji sprzężeń między różnymi częściami tego samego modułu.
*   **Lokalizacja:** `src/{Context}/Domain/Event/`
*   **Przykład:** `BookTitleWasChanged` w `Content` BC. Gdy tytuł się zmienia, inny listener w tym samym BC może zaktualizować pole `slug`.

### 4.2. Zdarzenia Integracyjne (Integration Events)
*   **Cel:** Komunikacja **pomiędzy** Bounded Contexts. Stanowią publiczny, stabilny kontrakt.
*   **Lokalizacja:** `src/{Context}/Contracts/Events/`
*   **Charakterystyka:**
    *   Są prostymi, niezmiennymi obiektami DTO, zawierającymi tylko dane.
    *   Nie powinny zawierać żadnej logiki.
    *   Są wersjonowane.
*   **Przykład:** `OrderPlaced` w `Sales` BC. Jest emitowane, gdy zamówienie zostanie złożone, aby inne konteksty (np. `Content`) mogły na nie zareagować.

## 5. Komunikacja Między Bounded Contexts

Komunikacja między modułami musi być przemyślana, aby zachować ich autonomię.
*   **Preferowana metoda:** **Komunikacja asynchroniczna przez zdarzenia.**
    1.  Kontekst "Upstream" (np. `Sales`) emituje **Zdarzenie Integracyjne** (`OrderPlaced`).
    2.  Zdarzenie jest wysyłane na globalną, asynchroniczną szynę zdarzeń (np. Laravel Queue).
    3.  Kontekst "Downstream" (np. `Content`) posiada Listener, który nasłuchuje na to zdarzenie.
    4.  Listener w `Content` BC używa **Anti-Corruption Layer (ACL)** – dedykowanej klasy-tłumacza – aby przekonwertować dane ze zdarzenia `OrderPlaced` na komendę zrozumiałą dla `Content` BC (np. `GrantAccessCommand`). To zapobiega "skażeniu" `Content` BC modelem z `Sales`.

*   **Metoda alternatywna (dla zapytań):** **Komunikacja synchroniczna przez Fasady/Gateway.**
    *   Gdy jeden kontekst musi synchronicznie pobrać dane z innego, robi to przez dedykowany interfejs (Gateway) w swojej warstwie `Infrastructure`. Ta fasada hermetyzuje wywołanie API lub serwisu drugiego kontekstu.

## 6. Katalog `Shared`

*   **Cel:** Zawiera kod, który jest **technicznie** współdzielony przez wszystkie Bounded Contexts i nie zawiera żadnej logiki biznesowej.
*   **Lokalizacja:** `src/Shared/`
*   **Przykłady:**
    *   Interfejsy dla `CommandBus`, `QueryBus`, `EventBus`.
    *   Abstrakcyjne klasy bazowe (np. `DataTransferObject`).
    *   Generyczne helpery.

Unikamy wzorca **Shared Kernel** (współdzielenia logiki domenowej). Jeśli kod jest reużywany przez kilka funkcjonalności, ale tylko wewnątrz *jednego* Bounded Contextu, powinien trafić do `src/{Context}/Shared/`.
```