# Product Requirements Document: PDF Books Generator (MVP)
*   **Wersja:** 1.0
*   **Data:** 2025-11-14
*   **Autor:** @pliszkaPL, @copilot

## 1. Wprowadzenie

### 1.1. Problem do rozwiązania
Twórcy treści cyfrowych (autorzy ebooków, trenerzy, marketerzy) potrzebują prostego i zintegrowanego narzędzia do tworzenia, recenzowania, generowania profesjonalnych plików PDF oraz ich sprzedaży bez konieczności korzystania z wielu rozproszonych i skomplikowanych aplikacji.

### 1.2. Cel produktu
Celem produktu jest dostarczenie aplikacji webowej (MVP), która umożliwia kompleksowe zarządzanie cyklem życia książki cyfrowej – od napisania treści, przez proces recenzji, aż po sprzedaż i dystrybucję gotowego pliku PDF.

## 2. Zakres MVP

### 2.1. Główne funkcjonalności
*   Możliwość tworzenia i edycji książek/poradników za pomocą modułowego edytora.
*   Możliwość generowania plików PDF na podstawie utworzonej treści z użyciem predefiniowanego szablonu.
*   System zapraszania recenzentów za pomocą jednorazowych kodów dostępu.
*   Możliwość sprzedaży książek poprzez zamockowaną bramkę płatniczą.
*   Podstawowe panele dla autora, kupującego i recenzenta.

### 2.2. Co NIE wchodzi w zakres MVP
*   Zaawansowane funkcje zarządzania kontem (np. zmiana e-maila).
*   Możliwość tworzenia kodów rabatowych.
*   Personalizacja szablonów PDF przez użytkownika.
*   Możliwość podpięcia własnej domeny.
*   Zaawansowana analityka sprzedaży.
*   Powiadomienia e-mail o aktywnościach (np. nowe recenzje).
*   Możliwość tworzenia wielu profili autorskich w ramach jednego konta.

## 3. Użytkownicy i role
*   **Autor:** Tworzy, edytuje i publikuje książki. Zarządza procesem recenzji, zaprasza recenzentów, przegląda ich uwagi. Zarządza metadanymi SEO i ceną.
*   **Recenzent:** Otrzymuje dostęp do książki za pomocą jednorazowego kodu. Może przeglądać treść i dodawać komentarze do poszczególnych bloków.
*   **Kupujący (Użytkownik):** Może kupować książki i uzyskiwać do nich dostęp w swoim panelu. Może dodawać publiczne opinie (ocena + komentarz) do zakupionych produktów.
*   **Superadmin:** Ma dostęp do panelu administracyjnego z uprawnieniami do zarządzania użytkownikami, książkami i opiniami.

## 4. Kryteria sukcesu i KPI
*   **Główny cel:** Osiągnięcie sprzedaży na poziomie 1000 ebooków miesięcznie (cel długoterminowy).
*   **KPI dla pozyskania twórców:**
    *   Po 1 miesiącu: 2 aktywnych twórców.
    *   Po 1 kwartale: 10 aktywnych twórców.
    *   Po 1 roku: 100 aktywnych twórców.
    *   *Definicja aktywnego twórcy: opublikował co najmniej jedną książkę w ciągu ostatnich 3 miesięcy.*
*   **Zbierane zdarzenia:** `pdf_generated`, `pdf_published`, `pdf_downloaded`, `sale_created`, `review_submitted`.

## 5. Wymagania funkcjonalne

### 5.1. Tworzenie i edycja książki
*   Edytor oparty na modułowych, przemieszczanych blokach (drag-and-drop) z wykorzystaniem biblioteki `vue-draggable`.
*   Dodawanie nowych bloków odbywa się poprzez przycisk `+`, który otwiera modal z wizualną siatką dostępnych typów bloków (Tekst, Obraz).
*   Każdy blok posiada ikonę kosza. Usunięcie bloku wymaga potwierdzenia w popupie ("Czy na pewno chcesz usunąć ten blok?"). Po usunięciu pojawia się powiadomienie z opcją "Cofnij".
*   Automatyczne zapisywanie zmian (aktualizacja pola `last_modified_at`).
*   Wprowadzono walidację po stronie frontendu i backendu:
    *   Max. 5000 znaków na segment tekstowy.
    *   Max. 100 segmentów na dokument.
    *   Max. 50 000 znaków na cały dokument.
    *   Przekroczenie limitu jest sygnalizowane wizualnie (np. czerwona ramka).
*   Autor ma możliwość zdefiniowania metadanych SEO (tytuł, opis) oraz FAQ dla strony sprzedażowej.

### 5.2. Struktura danych dokumentu (JSON)
*   Dokument jest reprezentowany przez obiekt JSON, który jest przesyłany do mikroserwisu generującego PDF.
*   Główny obiekt zawiera metadane (`authors`, `title`, itp.) oraz tablicę `content` z blokami.
*   Każdy blok w tablicy to obiekt z polami `type` (`image` lub `text`) oraz `context` zawierającym dane specyficzne dla typu.
*   Dokument posiada pola `version` (inkrementowane przy publikacji) i `last_modified_at`.
*   **Przykładowa struktura:**
    ```json
    {
      "document_meta": {
        "title": "Tytuł książki",
        "author_name": "Imię Autora",
        "version": "1.0.0",
        "publication_date": "2025-11-14"
      },
      "content": [
        {
          "type": "image",
          "context": { "url": "https://example.com/image.jpg" }
        },
        {
          "type": "text",
          "context": {
            "items": [
              { "type": "h1", "content": "Nagłówek pierwszego stopnia" },
              { "type": "p", "content": "Treść akapitu." }
            ]
          }
        }
      ]
    }
    ```

### 5.3. Generowanie PDF
*   Generowanie odbywa się synchronicznie (request-response) z limitem czasu 30s.
*   Wykorzystana biblioteka: `pdfmake`.
*   Backend wykorzystuje wzorzec Chain of Responsibility do renderowania poszczególnych segmentów, co ułatwi rozszerzalność.
*   Stosowany jest jeden predefiniowany szablon wizualny:
    *   **Czcionka:** Open Sans.
    *   **H1:** 28pt, Bold, margines dolny 20px.
    *   **H2:** 22pt, Bold, margines dolny 15px.
    *   **H3:** 18pt, Semi-bold, margines dolny 10px.
    *   **Paragraf:** 11pt, Regular, interlinia 1.5.
*   Opcjonalnie autor może włączyć dynamiczny znak wodny w stopce PDF z e-mailem kupującego.

### 5.4. Proces recenzowania
*   Autor generuje w panelu unikalne, jednorazowe kody dostępu dla recenzentów.
*   Autor kopiuje gotowy szablon zaproszenia z linkiem i kodem.
*   Kod jest ważny przez 7 dni. Po tym czasie dostęp wygasa.
*   Recenzent po przejściu pod link i podaniu kodu widzi treść książki i może dodawać komentarze do poszczególnych bloków.
*   Recenzent widzi tylko swoje komentarze. Autor widzi wszystkie komentarze i może zmieniać ich status (`nowy`, `do zrobienia`, `gotowy`, `odrzucony`) za pomocą menu przy każdym komentarzu.

### 5.5. Sprzedaż i płatności
*   Bramka płatnicza w MVP jest zamockowana.
*   Autor ustala cenę i walutę (PLN/EUR) w ustawieniach książki.
*   Do zamockowanej bramki przesyłany jest obiekt "intencji płatności" o zdefiniowanej strukturze, gotowy do integracji z prawdziwą bramką.
*   Kupujący po "zakupie" otrzymuje dostęp do książki w swoim panelu.
*   Dostęp do przyszłych, zaktualizowanych wersji książki jest darmowy i dożywotni dla każdego, kto ją zakupił.

### 5.6. Onboarding i wsparcie
*   Nowy autor przechodzi przez krótki, interaktywny samouczek pokazujący kluczowe funkcje.
*   Użytkownicy mogą zgłaszać problemy przez formularz kontaktowy. Każde zgłoszenie otrzymuje unikalny numer UUID do śledzenia.

## 6. Wymagania niefunkcjonalne

### 6.1. Przechowywanie plików
*   Przesyłane obrazy (do 5MB) są optymalizowane (kompresja, zmiana rozmiaru do max 1MB) i przechowywane. Oryginał jest również zachowywany.
*   Wygenerowane pliki PDF są usuwane po 2 dniach od ostatniego pobrania (zadanie cron). Przez 30 dni utrzymywany jest `soft-delete`.

### 6.2. Bezpieczeństwo
*   Uwierzytelnianie dla autorów i kupujących oparte o sesję (e-mail + hasło).
*   Dostęp dla recenzentów za pomocą jednorazowych, ograniczonych czasowo tokenów.
*   Usunięcie konta wymaga podwójnego potwierdzenia i skutkuje trwałym usunięciem danych.

### 6.3. Infrastruktura i CI/CD
*   Aplikacja zdokeryzowana i przygotowana do wdrożenia na Dokku.
*   Workflow GitHub Actions obejmuje:
    1.  Instalację zależności.
    2.  Uruchomienie testów PEST (backend).
    3.  Uruchomienie testów Playwright (smoke tests).
    4.  Build obrazu Docker.
    5.  Deploy na środowisko.
*   Automatyczne migracje bazy danych.
*   Prosty monitoring (healthcheck) i alerty na Slack/e-mail w razie niepowodzenia wdrożenia.

## 7. Stos technologiczny
*   **Frontend:** Vue.js 3.0
*   **Backend:** Laravel 12 (REST API)
*   **Baza danych:** (do ustalenia, np. PostgreSQL)
*   **Generowanie PDF:** biblioteka `pdfmake` (w osobnym mikroserwisie)
*   **Testy:** PEST (backend), Playwright (E2E/smoke tests)
*   **Infrastruktura:** Docker, Dokku
*   **CI/CD:** GitHub Actions

## 8. Plany po MVP
*   Integracja z prawdziwą bramką płatniczą (np. Stripe) i systemem wypłat dla autorów.
*   Możliwość tworzenia kodów rabatowych.
*   Rozbudowa analityki dla autorów.
*   System powiadomień e-mail.
*   Możliwość personalizacji szablonów PDF.