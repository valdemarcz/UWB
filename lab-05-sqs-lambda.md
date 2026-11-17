# Laboratorium 5 — SQS i Lambda przetwarzająca

[Poprzednie laboratorium](lab-04-api-dynamodb.md) · [Instrukcja główna](README.md) · [Następne laboratorium](lab-06-reguly-analiza.md)

## Cel

Rozdziel przyjęcie zlecenia od jego wykonania. Pokaż na działającym API, że zlecenie można przyjąć, kiedy konsument jest wstrzymany, a wynik pojawia się po wznowieniu przetwarzania.

## Co warto wiedzieć

`202 Accepted` potwierdza przyjęcie do realizacji. Klient poznaje decyzję końcową dopiero przez `GET`. Powiązanie SQS z funkcją Lambda, czyli *event source mapping*, pobiera wiadomości z kolejki i przekazuje je do funkcji. Po odebraniu wiadomość jest przez pewien czas ukryta. Jeżeli przetwarzanie się nie powiedzie, SQS może udostępnić ją ponownie po upływie tego czasu. AWS nazywa to *visibility timeout*. [Jak Lambda korzysta z SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)

SQS może przekazać tę samą wiadomość więcej niż raz. **Idempotencja** oznacza tutaj, że ponowna obsługa tego samego zlecenia zachowuje jeden wynik końcowy. Wyobraź sobie, że funkcja zapisała wynik, ale wiadomość została dostarczona ponownie. Zastanów się, po czym następne wywołanie rozpozna zakończoną pracę i jak zabezpieczysz się przed równoległymi zapisami.

## Zadanie

1. Przygotuj standardową kolejkę SQS i drugą funkcję Lambda w regionie aplikacji. Skonfiguruj powiązanie kolejki z funkcją. Rola API potrzebuje wysyłania wiadomości. Rola przetwarzająca potrzebuje obsługi wiadomości z własnej kolejki, dostępu do stanu i zapisu logów. Sprawdź relację między czasem działania funkcji a czasem ukrycia wiadomości. [Konfiguracja powiązania SQS z Lambdą](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)
2. Rozszerz `POST`. Zapisz zlecenie jako `PENDING` i wyślij komunikat umożliwiający jego przetworzenie. Ustal i zapisz w README minimalną zawartość komunikatu. Używaj tego samego `requestId` w API, wiadomości, bazie i logach.
3. Zwracaj `202` po potwierdzonej wysyłce do SQS. Rozważ przypadek, w którym zapis do DynamoDB się udał, ale API nie otrzymało potwierdzenia wysłania. Zwróć `503`, a zapisane zlecenie pozostaw do bezpiecznego ponowienia z tym samym ID. Takie ponowienie ma umożliwić dalszą pracę także wtedy, gdy istnieje już rekord `PENDING`. Sprawdź je w kontrolowanej próbie albo w teście z zasymulowanym błędem wysyłki.
4. W funkcji przetwarzającej wprowadź jedną prostą, przewidywalną decyzję dla poprawnej próbki. Zapisz wynik w DynamoDB i udostępnij go przez `GET` jako `COMPLETED`. Pełny zestaw reguł dodasz na następnym spotkaniu.
5. Zabezpiecz wynik końcowy przed powtórnym dostarczeniem. Stan powinien przechodzić w kierunku `DRAFT` → `PENDING` → `COMPLETED` albo `PENDING` → `COMPLETED` dla nowego zlecenia. Powtórny zapis w API i powtórna obsługa przez funkcję zachowują końcowy wynik. Sprawdź również kontrakt dla tego samego ID i zmienionych danych.
6. Loguj identyfikator i etap pracy, np. przyjęcie, rozpoczęcie przetwarzania i zapis decyzji. Jeśli funkcja odbiera kilka rekordów w jednej partii, ustal, co ma się stać z pozostałymi wiadomościami po błędzie jednego rekordu. [Ponowienia i błędy partii w Lambda–SQS](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)

## Sprawdzenie i oddanie

1. Wyłącz powiązanie kolejki z funkcją i zaczekaj, aż zmiana stanu powiązania będzie widoczna. Wyślij **nowe** zlecenie. Zapisz odpowiedź `202` oraz odczyt `PENDING` dla tego samego ID.
2. Wznów odbiór i odczytuj stan w rozsądnych odstępach. Po zakończeniu zapisz `COMPLETED` z decyzją i znajdź odpowiadający mu wpis w CloudWatch Logs.
3. Powtórz `POST` dla zakończonego zlecenia i sprawdź odpowiedź `200` z wynikiem. Osobno odtwórz tę samą wiadomość za pomocą uprawnionego narzędzia w AWS lub wykorzystaj kontrolowany test modułu przetwarzającego. Sprawdź, że drugie wykonanie zachowuje ten sam wynik i pomija drugi zapis końcowy. Powtórzenie `POST` i ponowne dostarczenie przez kolejkę sprawdzają dwa odrębne miejsca systemu.

Oddaj link do commita i dwa dowody. Pierwszy ma pokazać oba stany dla jednego ID. Drugi ma pokazać zachowanie przy powtórnym dostarczeniu, np. wynik testu albo log przetwarzania z odczytem zachowanego wyniku.

**Typowy błąd.** Zlecenie pozostaje w `PENDING` po wznowieniu. Sprawdź stan powiązania, logi i uprawnienia roli wykonawczej do SQS oraz DynamoDB. Zwróć uwagę, czy komunikat ma ten sam identyfikator, którego szukasz w tabeli.

**Zachowaj na kolejny etap.** Obie funkcje, kolejkę, tabelę, trasy API i próbkę z S3. Pozostaw powiązanie kolejki z funkcją włączone.
