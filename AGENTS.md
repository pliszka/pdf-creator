# Agent & Developer Onboarding Instructions
*   **Wersja:** 1.0
*   **Data:** 2025-11-15 11:46:05
*   **Właściciel:** @pliszkaPL

Ten dokument zawiera kluczowe informacje dla każdego agenta AI lub dewelopera dołączającego do tego projektu.

## 1. Uruchamianie i Zarządzanie Projektem: `Makefile`

Wszystkie podstawowe skrypty i polecenia niezbędne do pracy z projektem zostały scentralizowane w pliku `Makefile` w głównym katalogu. Nie uruchamiaj poleceń `docker`, `composer` czy `npm` bezpośrednio.

Zamiast tego używaj poleceń `make`, takich jak:
*   `make install` - aby zainstalować wszystkie zależności projektu (Composer, NPM itp.).
*   `make up` - aby uruchomić środowisko deweloperskie Docker.
*   `make down` - aby zatrzymać środowisko deweloperskie.
*   `make test` - aby uruchomić zestaw testów.

Przed wykonaniem jakiejkolwiek akcji, zawsze sprawdź `Makefile` w poszukiwaniu odpowiedniego polecenia.

## 2. Wytyczne i Dokumentacja Projektu: `.github` i `.ai`

Wszystkie zasady dotyczące tworzenia kodu, opisy architektury, wymagania produktowe (PRD) oraz instrukcje dla agentów AI znajdują się w dedykowanych katalogach:

*   **Katalog `.github/`:** Zawiera szablony dla Pull Requestów, Issue, a także konfiguracje dla GitHub Actions (workflowy CI/CD). Jest to źródło prawdy na temat procesów deweloperskich i wkładu w projekt.

*   **Katalog `.ai/`:** Zawiera dokumentację przeznaczoną bezpośrednio dla agentów AI, w tym szczegółowe wymagania produktowe (`prd.md`) oraz inne dokumenty generowane w procesie projektowym.

Przed rozpoczęciem jakiejkolwiek pracy nad kodem, zapoznaj się z zawartością obu tych katalogów, aby zrozumieć kontekst biznesowy i wymagania techniczne projektu.