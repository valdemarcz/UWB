# Laboratorium 4 — reguły biznesowe i analiza wyników

[Poprzednie laboratorium](lab-03-sqs-lambda.md) · [Instrukcja główna](README.md) · [Następne laboratorium](lab-05-docker-cicd.md)

## Cel

Uzupełnij reguły przetwarzania i sprawdź je na syntetycznej próbce z prywatnego S3. Porównaj odsetki odrzuceń w trzech grupach. Dla jednej próby pokaż, że zlecenie kończy się poprawnie po awarii przejściowej.

## Co warto wiedzieć

Poprawność formatu, decyzja biznesowa i dostępność techniczna to trzy różne rzeczy. Zła wartość wejściowa daje `400`. Poprawne zlecenie przekraczające limit kończy się normalnym wynikiem biznesowym `COMPLETED` z negatywną decyzją. Awaria przejściowa przerywa próbę wykonania i pozostawia możliwość ponowienia.

Moduł obliczający decyzję powinien dać się wywołać w teście z samymi danymi wejściowymi. Kod obsługi zdarzenia AWS może przekazać dane do modułu, a potem zapisać wynik. Ten sam moduł uruchomisz również w kontenerze. Przy scenariuszu przejściowym pamiętaj, że kolejne wywołanie Lambdy może trafić do innego środowiska wykonawczego. Dobierz sygnał lub stan, który pozwoli rozróżnić pierwszą i kolejną próbę dla danego zlecenia. [Ponowienia dla zdarzeń SQS](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-errorhandling.html)

## Reguły dla próbki

| Wariant | Reguła |
| --- | --- |
| **Zamówienia** | `SKU-A` ma dostępność 5 sztuk, a `SKU-B` 2 sztuk. Ilość do limitu włącznie daje `APPROVED`. Większa ilość lub nieznany kod produktu daje `REJECTED`. Każde zlecenie rozpatrz względem tej samej migawki dostępności. |
| **Płatności** | Limit dla `PLN` wynosi 10 000 w najmniejszych jednostkach waluty, a dla `EUR` 5 000. Kwota do limitu włącznie daje `APPROVED`. Przekroczenie limitu lub inny poprawnie zapisany kod waluty daje `DECLINED`. |

W obu wariantach `scenario=transient_once` powoduje kontrolowany błąd pierwszej próby, po którym kolejna próba stosuje zwykłe reguły. Dla każdej decyzji zapisz krótką przyczynę, np. `OUT_OF_STOCK` lub `LIMIT_EXCEEDED`. Przyczyna powinna pozwolić ustalić, dlaczego konkretny wiersz otrzymał swoją decyzję.

## Zadanie

1. Wydziel logikę decyzji z kodu odczytu SQS i zapisu do DynamoDB. Dodaj lokalne testy granicy limitu, przekroczenia limitu i nieznanego produktu lub nieobsługiwanej waluty.
2. Uruchom w AWS po jednym kontrolowanym przypadku zatwierdzenia, odrzucenia i awarii przejściowej. Dla awarii pokaż w logach kolejne próby z tym samym ID oraz końcowy wynik w API.
3. Pobierz próbkę z S3. Sama próbka zawiera dane wejściowe. Wyślij jej 12 wierszy do działającego API, nadając im świeże UUID. Zachowaj dla każdego wiersza `caseId`, `group` i `requestId`, aby później połączyć dane wejściowe z odpowiedziami `GET`.
4. Zbierz decyzje końcowe i oblicz odsetek odrzuceń w każdej grupie. Podziel liczbę `REJECTED` lub `DECLINED` przez liczbę zakończonych zleceń w danej grupie. Każdy identyfikator licz raz. Do porównania z tabelą referencyjną każda grupa powinna mieć cztery zakończone zlecenia. Jeżeli któregoś wyniku brakuje, ustal etap, na którym utknął.
5. Wyjaśnij, która grupa odstaje i które dane wejściowe wpływają na wynik. Obliczenia możesz wykonać w krótkim skrypcie, arkuszu lub zapytaniu. Zachowaj tabelę z liczebnościami oraz krótki wniosek.

## Sprawdzenie i oddanie

| Grupa | Zakończone | Odrzucone biznesowo | Odsetek |
| --- | ---: | ---: | ---: |
| `standard` | 4 | 1 | 25% |
| `limit` | 4 | 3 | 75% |
| `retry` | 4 | 0 | 0% |

Wyniki referencyjne są takie same dla obu tematów. Sprawdź jeden przypadek z grupy `limit` ręcznie na podstawie reguły. Oddaj link do commita, tabelę z wnioskiem oraz fragment logu i wynik `GET` dla jednego ID ze scenariusza `transient_once`.

**Typowy błąd.** Raport liczy próby wykonania zamiast zleceń albo zalicza błąd przejściowy jako odrzucenie. Odczytaj jedną decyzję końcową dla każdego UUID i grupuj po wartości `group` z CSV.

**Zachowaj na kolejny etap.** Testy, moduł decyzji, działający adres API, identyfikator zakończonego zlecenia i zapis analizy.
