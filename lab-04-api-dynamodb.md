# Laboratorium 3 — HTTP API i stan w DynamoDB

[Poprzednie laboratorium](lab-03-iam-s3.md) · [Instrukcja główna](README.md) · [Następne laboratorium](lab-04-sqs-lambda.md)

## Cel

Wdróż HTTP API i funkcję Lambda, które przyjmują poprawne zlecenie, utrwalają szkic w DynamoDB i pozwalają go odczytać po ID. W kolejnym laboratorium do tej samej aplikacji dołączysz kolejkę.

## Co warto wiedzieć

API Gateway kieruje żądanie HTTP do funkcji Lambda. Kod funkcji odpowiada za walidację danych i znaczenie zwracanego statusu HTTP. DynamoDB przechowuje stan między wywołaniami, dlatego osobne żądanie `GET` może później odczytać szkic zapisany przez `POST`. Skoro odczyt odbywa się po jednym ID, `requestId` jest naturalnym kandydatem na klucz partycji.

Pomyśl, co może się stać, jeśli ten sam klient wyśle dwa żądania z tym samym ID jednocześnie. Warunki zapisu DynamoDB pozwalają sprawdzić stan rekordu w ramach operacji zapisu, np. utworzyć rekord tylko pod wolnym kluczem. [Zapisy warunkowe w DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)

## Zadanie

1. Przygotuj w AWS tabelę DynamoDB, funkcję Lambda API, jej rolę wykonawczą i dwie trasy w API Gateway HTTP API. Zastosuj ustalony na kursie sposób dostępu do tras. Rola funkcji powinna obejmować operacje potrzebne na tej tabeli oraz zapis logów.
2. Zaimplementuj `POST /requests`. Sprawdź format JSON, identyfikator, scenariusz i pola wariantu według [kontraktu](README.md#wspólny-kontrakt-api). Dla nowego poprawnego zlecenia zwróć `201` po zapisaniu rekordu w stanie `DRAFT`.
3. Zaimplementuj `GET /requests/{id}`. Odczytaj stan po identyfikatorze i zwróć `apiVersion` w odpowiedzi dla istniejącego rekordu. Po `POST` wykonaj osobne `GET`. Jeśli odczyt od razu po zapisie ma dać aktualny stan, sprawdź ustawienie spójności odczytu. [Spójność odczytu DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
4. Określ zachowanie dla ponowienia z tym samym ID. Ten sam zestaw danych ma zachować jeden szkic. Zmiana danych przy tym samym ID kończy się `409` i zachowuje wcześniejszy rekord.
5. Zapisuj w CloudWatch Logs identyfikator zlecenia i wynik operacji, np. utworzenie szkicu. Odszukaj wpis wygenerowany przez próbę na działającym API. [Logi funkcji Lambda w CloudWatch](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html)

## Sprawdzenie i oddanie

| Próba | Oczekiwany wynik |
| --- | --- |
| Poprawny `POST`, potem `GET` pod zwróconym adresem | `201`, następnie `200`. To samo ID, stan `DRAFT` i wersja w odpowiedzi odczytu. |
| Drugi `POST` z tymi samymi danymi | `200`. Istnieje jeden szkic. |
| `POST` ze zmienionymi danymi i tym samym ID | `409`. Dotychczasowy szkic pozostaje bez zmian. |
| `POST` z błędem składni JSON lub wartością spoza kontraktu | `400`. Dane takiego zlecenia nie są zapisane. |
| `GET` dla innego, poprawnego UUID | `404`. |

Oddaj link do commita oraz jeden zapis prób HTTP i opcjonalnie jeden odpowiadający mu fragment logu. Pokaż przynajmniej udany zapis i odczyt oraz jedną odpowiedź błędną.

**Typowy błąd.** Żądanie trafia do innej trasy lub Lambda odwołuje się do tabeli w innym regionie. Sprawdź metodę i adres wywołania, a następnie rozróżnij odpowiedź Twojego kodu od błędu zgłoszonego przez API Gateway. Przy odmowie dostępu sprawdź rolę wykonywania funkcji.

**Zachowaj na kolejny etap.** Adres API i sposób wywoływania tras, działającą funkcję, tabelę oraz przykładowe żądanie, któremu możesz nadać nowe ID.
