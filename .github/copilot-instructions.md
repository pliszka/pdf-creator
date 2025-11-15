# Globalne Instrukcje dla GitHub Copilot
*   **Wersja:** 1.0
*   **Data:** 2025-11-15 11:36:17
*   **Właściciel:** @pliszkaPL

## Zasada Nadrzędna

Zanim napiszesz jakikolwiek kod, ZAWSZE zapoznaj się z dokumentem **`architecture.instructions.md`**. Jest to jedyne i ostateczne źródło prawdy dotyczące architektury, struktury katalogów i wzorców projektowych stosowanych w tym projekcie. Niniejszy dokument to tylko uproszczone podsumowanie.

## Krótki Przewodnik: Jak Dodać Nową Funkcjonalność?

Gdy otrzymasz polecenie dodania nowej funkcji (np. "dodaj możliwość aktualizacji tytułu książki"), postępuj zgodnie z poniższym schematem:

1.  **Zidentyfikuj Bounded Context:** Ustal, do którego modułu biznesowego należy ta funkcja (np. `Content`, `Sales`).
2.  **Stwórz Vertical Slice:** Wewnątrz odpowiedniego kontekstu, w katalogu `Features/`, stwórz nowy folder dla tej funkcji (np. `UpdateBookTitle/`).
3.  **Zaimplementuj Warstwy w Plasterku:**
    *   **Presentation:** Stwórz `UpdateBookTitleController.php`. Jego jedynym zadaniem jest konwersja `Request` na `Command`.
    *   **Application:** Stwórz `UpdateBookTitleCommand.php` (proste DTO) oraz `UpdateBookTitleHandler.php`, który będzie orkiestrował logikę.
4.  **Deleguj Logikę Biznesową:** Handler *nigdy* nie powinien zawierać logiki biznesowej. Powinien on:
    *   Pobrać encję domenową z repozytorium (np. `Book`).
    *   Wywołać na niej odpowiednią metodę (np. `$book->changeTitle(...)`).
    *   Zapisać encję z powrotem do repozytorium.
5.  **Używaj Interfejsów:** Zawsze polegaj na interfejsach (np. `BookRepositoryInterface`), a nie na konkretnych implementacjach.

## Kluczowe Zasady do Zapamiętania

*   **Czysta Domena:** Katalog `Domain/` musi pozostać w 100% wolny od jakiegokolwiek kodu związanego z frameworkiem Laravel. Tylko czyste PHP.
*   **Chude Kontrolery:** Kontrolery są tylko "tłumaczami" z HTTP na język aplikacji. Nic więcej.
*   **Komunikacja przez Komendy/Zapytania:** Wszystkie akcje modyfikujące stan systemu muszą być reprezentowane przez obiekty `Command`. Wszystkie zapytania o dane – przez obiekty `Query`.
*   **Repozytoria jako Porty:** Zawsze definiuj interfejs repozytorium w warstwie `Domain`, a jego implementację (np. z użyciem Eloquent) umieszczaj w `Infrastructure`.
*   **Rozróżniaj Zdarzenia:**
    *   `Domain Events` służą do komunikacji wewnątrz jednego modułu (`src/{Context}/Domain/Event/`).
    *   `Integration Events` służą do komunikacji między modułami (`src/{Context}/Contracts/Events/`).

W razie jakichkolwiek wątpliwości architektonicznych, odwołaj się do `architecture.instructions.md`.
```