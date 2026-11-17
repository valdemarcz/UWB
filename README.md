# Projekt AWS — asynchroniczne przetwarzanie zleceń

W ciągu sześciu laboratoriów zbudujesz indywidualnie usługę, która przyjmuje zlecenie, wykonuje je w tle i udostępnia wynik. Na końcu pokażesz działającą aplikację w AWS, zmianę wdrożoną z GitHub Actions i wycofanie aktualizacji kontenera w lokalnym Kubernetesie.

## Wybierz temat

| Wariant | Dane zlecenia | Decyzja modułu przetwarzającego |
| --- | --- | --- |
| **Zamówienia** | Kod produktu i liczba sztuk | `APPROVED` lub `REJECTED` na podstawie fikcyjnej dostępności produktu. |
| **Symulowane płatności** | Kwota w najmniejszych jednostkach waluty i kod waluty | `APPROVED` lub `DECLINED` na podstawie fikcyjnej listy walut i limitów. |

Wybierz jeden wariant do końca pierwszego spotkania. Używaj wyłącznie danych syntetycznych. Wariant płatniczy kończy się na obliczeniu decyzji z ustalonych reguł. Oba warianty korzystają z tego samego API, stanów, usług AWS i kryteriów oceny.

## Jak działa projekt

1. Klient wysyła zlecenie do **API Gateway HTTP API**. Funkcja **Lambda API** sprawdza format, zapisuje zlecenie w **DynamoDB** i przekazuje komunikat do **SQS**.
2. Osobna **Lambda przetwarzająca** odbiera komunikat, podejmuje decyzję i zapisuje wynik w DynamoDB. Klient sprawdza stan przez API, używając identyfikatora zlecenia.
3. **Prywatne S3** przechowuje próbkę wejściową do analizy. **CloudWatch Logs** pozwala prześledzić zlecenie po identyfikatorze. **IAM** określa, co może zrobić każda rola.
4. **GitHub Actions** testuje kod, aktualizuje Lambdę API i publikuje obraz w **ECR**. Obraz uruchamia tę samą logikę biznesową z prostym interfejsem HTTP w lokalnym Kubernetesie.

Każde laboratorium rozbudowuje ten sam projekt. Możesz wcześniej testować kod lokalnie, także w LocalStacku. 

## Wspólny kontrakt API

### Dane wejściowe

`POST /requests` przyjmuje obiekt JSON z trzema polami.

| Pole | Znaczenie |
| --- | --- |
| `requestId` | UUID w postaci tekstowej, nadany przez klienta. Użyj nowego ID dla nowego zlecenia. Przy ponowieniu tej samej operacji zachowaj ID. |
| `scenario` | `normal` lub `transient_once`. Druga wartość służy do kontrolowanej próby ponowienia. Na etapie 2 używaj `normal`. |
| `payload` | Obiekt zawierający pola wybranego wariantu. |

- **Zamówienia** używają `sku`, czyli niepustego tekstu, oraz `quantity`, czyli dodatniej liczby całkowitej.
- **Płatności** używają `amountMinor`, czyli dodatniej liczby całkowitej, oraz `currency`, czyli kodu składającego się z trzech wielkich liter ASCII. Na przykład `2500` przy `PLN` oznacza 25 zł.

**Przykładowe zamówienie**

```json
{
  "requestId": "8b4efb83-a38e-45ba-9604-f2ab89f833e4",
  "scenario": "normal",
  "payload": {"sku": "SKU-A", "quantity": 2}
}
```

**Przykładowa symulowana płatność**

```json
{
  "requestId": "f8a1766e-02f3-4c22-a181-246c6263c983",
  "scenario": "normal",
  "payload": {"amountMinor": 2500, "currency": "PLN"}
}
```

Poprawny format pozwala przyjąć zlecenie do przetworzenia. Dostępność produktu, lista obsługiwanych walut i limity to reguły biznesowe wykonywane później. Na przykład `quantity: 0` to błąd wejścia, a poprawna ilość przekraczająca dostępność może dać biznesowe odrzucenie.

### Odpowiedzi i stany

| Operacja | Oczekiwane zachowanie |
| --- | --- |
| `POST /requests` z nowym ID podczas lab. 2 | `201 Created`. Zapisany stan `DRAFT`. Odpowiedź zawiera `requestId` i `statusUrl` wskazujący trasę odczytu. |
| `POST /requests` od lab. 3 | `202 Accepted` po potwierdzeniu przyjęcia komunikatu przez SQS. Odpowiedź zawiera `requestId` i `statusUrl`. |
| `GET /requests/{id}` dla istniejącego ID | `200 OK` z polami `requestId`, `status` i `apiVersion`. Dla `COMPLETED` dochodzi obiekt `result` z polami `decision` i `reason`. |
| `POST` z niepoprawnym JSON-em lub danymi | `400 Bad Request` z czytelnym opisem błędu. |
| `GET` dla poprawnego, nieznanego UUID | `404 Not Found`. |
| `POST` ze znanym ID i zmienionym `payload` lub `scenario` | `409 Conflict`. Zachowaj dotychczasowe dane. |
| `POST`, gdy wysłanie do SQS się nie powiodło lub brak potwierdzenia | `503 Service Unavailable`. Klient może ponowić operację z tym samym ID i tymi samymi danymi. |

`DRAFT` to zapis ćwiczeniowy z lab. 2. Od lab. 3 `PENDING` oznacza zlecenie oczekujące na wynik. W krótkim okresie między zapisem a wysłaniem do kolejki rekord może już mieć stan `PENDING`. Potwierdzeniem przyjęcia komunikatu na etapie zlecania jest odpowiedź `202`. Stan `COMPLETED` oznacza zapisaną decyzję końcową, w tym odrzucenie biznesowe. Ustal w swoim README typ i przykładową wartość `apiVersion` oraz kody `reason`.

Przy powtórnym `POST` z tym samym ID i tymi samymi danymi istniejący szkic na lab. 2 daje `200` i pozostaje szkicem. Od lab. 3 takie żądanie może przesłać szkic do kolejki lub ponowić wysyłkę zlecenia oczekującego. Zwróć `202` po potwierdzeniu wysyłki dla tego ID. Jeśli zlecenie jest już zakończone, zwróć `200` z istniejącym wynikiem. Porównuj wartości pól, aby kolejność kluczy JSON nie zmieniała znaczenia żądania.

## Laboratoria

| Karta | Wynik spotkania |
| --- | --- |
| [1. IAM i prywatne S3](lab-03-iam-s3.md) | Próbka danych na S3, odczyt dozwolony i odczyt odmówiony. |
| [2. HTTP API i DynamoDB](lab-04-api-dynamodb.md) | Utworzenie szkicu i odczyt stanu przez działający adres w AWS. |
| [3. SQS i Lambda przetwarzająca](lab-05-sqs-lambda.md) | Przejście od oczekiwania do wyniku i sprawdzenie powtórnego dostarczenia. |
| [4. Reguły i analiza danych](lab-06-reguly-analiza.md) | Trzy sprawdzone scenariusze i analiza wyników dla próbki z S3. |
| [5. Docker i GitHub Actions](lab-07-docker-cicd.md) | Wersja wdrożona automatycznie na AWS i obraz w ECR. |
| [6. Kubernetes i odbiór projektu](lab-08-kubernetes-odbior.md) | Aktualizacja i rollback kontenera oraz pokaz kompletnego projektu w AWS. |

## Praca na koncie kursowym

- Przed pierwszym wdrożeniem ustal , region, swój prefiks nazw i dopuszczone operacje. Regionalne zasoby aplikacji trzymaj w ustalonym regionie. Zakres usług i operacji zależy od planu konta oraz przyznanych uprawnień. [Plany kont AWS](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html)
- Pracuj na przydzielonej tożsamości AWS. Rozróżniaj rolę osoby wdrażającej, role wykonawcze obu funkcji i rolę wdrożeniową GitHub Actions.
- Utrzymuj prywatne S3 z włączonym **Block Public Access**. Jeżeli trasy korzystają z autoryzacji IAM, klient musi podpisywać żądania SigV4 i mieć uprawnienie do wywołania trasy. [Autoryzacja IAM w HTTP API](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-access-control-iam.html)
- Zapisuj w Git kod, testy, workflow i manifesty. Dane dostępowe przechowuj poza repozytorium. Anonimizuj dowody widoczne publicznie. W logach aplikacji wystarczą identyfikator zlecenia, etap i wynik operacji.
- Używaj małych porcji danych i utrzymuj listę własnych zasobów. Zachowaj zasoby potrzebne na następny etap, a po odbiorze zastosuj instrukcję sprzątania z ostatniej karty.

## Oddawanie pracy

Na każdym etapie oddaj link do konkretnego commita i jeden lub dwa dowody wymienione w karcie. Fragment odpowiedzi HTTP lub krótki zapis poleceń z wynikiem zwykle wystarczy. Na odbiorze liczy się działanie projektu, zgodność ze wspólnym kontraktem i umiejętność wyjaśnienia własnych decyzji. Możesz korzystać z AI. Przygotuj przykład testu, którym sprawdziłeś zmianę, i wyjaśnij, jaki błąd ten test wykrywa.
