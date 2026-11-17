# Laboratorium 1 — IAM i prywatne dane w S3

[Instrukcja główna](README.md) · [Następne laboratorium](lab-02-api-dynamodb.md)

## Cel

Umieść próbkę wybranego projektu w prywatnym Amazon S3. Sprawdź, że przydzielona rola może pobrać właściwy plik, a próba odczytu znanego obiektu spoza jej zakresu kończy się odmową. Tę samą próbkę wykorzystasz po uruchomieniu przetwarzania zleceń.

## Co warto wiedzieć

S3 przechowuje obiekty pod kluczami, np. `student-17/input/zlecenia.csv`. Fragment klucza `student-17/input/` to prefiks. Uprawnienie do czytania obiektu i uprawnienie do listowania kluczy to osobne operacje. Zacznij od pytania, czy rola musi znaleźć plik przez listowanie, czy wystarczy odczyt pod znanym kluczem. Jeżeli dodajesz listowanie, sprawdź także ograniczenie prefiksów dostępnych w tej operacji. [Polityki i uprawnienia S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html)

Rola użyta do przygotowania danych może mieć więcej uprawnień niż rola użyta do próby odczytu. Zapisz, którą tożsamością wykonujesz każdą próbę. Wyniki porównuj dla tej samej tożsamości odczytującej.

## Zadanie

1. Wybierz temat. W swoim repozytorium zanotuj wariant, region i sposób oznaczania własnych zasobów. Ustal z prowadzącym, czy używasz własnego bucketu, czy przydzielonego prefiksu w buckecie kursowym.
2. Zapisz odpowiednią próbkę jako plik CSV z nagłówkiem. Umieść go w prywatnym S3 pod własnym prefiksem. Sprawdź ustawienie Block Public Access.
3. Przygotuj z prowadzącym drugi, istniejący obiekt testowy w tym samym buckecie, pod osobnym prefiksem poza zakresem odczytu. Do tej próby użyj wydzielonego pliku kontrolnego.
4. Ustaw lub odbierz rolę z odczytem ograniczonym do własnego prefiksu. Pobierz swój plik z użyciem tej roli, a następnie spróbuj pobrać wskazany obiekt kontrolny. Zachowaj oba wyniki.

## Próbka wejściowa

`caseId` to etykieta wiersza CSV. W lab. 4 nadasz każdemu wierszowi osobny `requestId` i zachowasz mapowanie między nimi. Pole `group` służy do analizy wyników. Pole `scenario` steruje próbą techniczną.

### Wariant zamówień

```csv
caseId,group,scenario,sku,quantity
O01,standard,normal,SKU-A,1
O02,standard,normal,SKU-A,3
O03,standard,normal,SKU-B,2
O04,standard,normal,SKU-B,3
O05,limit,normal,SKU-A,5
O06,limit,normal,SKU-A,6
O07,limit,normal,SKU-B,4
O08,limit,normal,SKU-X,1
O09,retry,transient_once,SKU-A,1
O10,retry,transient_once,SKU-A,4
O11,retry,transient_once,SKU-B,1
O12,retry,transient_once,SKU-B,2
```

### Wariant płatności

```csv
caseId,group,scenario,amountMinor,currency
P01,standard,normal,1500,PLN
P02,standard,normal,3000,PLN
P03,standard,normal,5000,EUR
P04,standard,normal,5500,EUR
P05,limit,normal,10000,PLN
P06,limit,normal,10001,PLN
P07,limit,normal,5001,EUR
P08,limit,normal,1000,USD
P09,retry,transient_once,1000,PLN
P10,retry,transient_once,8000,PLN
P11,retry,transient_once,2000,EUR
P12,retry,transient_once,5000,EUR
```

## Sprawdzenie i oddanie

- Własny plik pobiera się i zawiera nagłówek oraz 12 rekordów. Odczyt obiektu kontrolnego kończy się `AccessDenied`.
- Oddaj link do commita oraz jeden dowód obejmujący oba odczyty. Podpisz, której roli użyto i czym różnią się prefiksy.

**Typowy błąd.** Próba odmowy dotyczy błędnego klucza albo jest wykonana inną tożsamością. Najpierw potwierdź z prowadzącym, że obiekt kontrolny istnieje. Przy problemach z konsolą sprawdź uprawnienia odczytu i listowania jako osobne operacje.

**Zachowaj na kolejne laboratoria.** Repozytorium, ustalone konto i region, przydzielony prefiks oraz prywatny plik z próbką.
