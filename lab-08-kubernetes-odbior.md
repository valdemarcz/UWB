# Laboratorium 6 — Kubernetes, rollback i odbiór projektu

[Poprzednie laboratorium](lab-05-docker-cicd.md) · [Instrukcja główna](README.md)

## Cel

Uruchom kontener z logiką projektu w lokalnym Kubernetesie, zmień wersję obrazu i przywróć poprzednią. Następnie pokaż pełny przepływ nowego zlecenia w AWS.

## Co warto wiedzieć

**Deployment** opisuje oczekiwany stan aplikacji, w tym obraz uruchamiany w podach. Gdy zmienia się szablon poda, Kubernetes zapisuje nową rewizję i przeprowadza aktualizację. Rollback przywraca wcześniejszą rewizję. Osobne tagi lub digesty pozwalają ustalić, jaki obraz uruchomiła każda rewizja. Sprawdź odpowiedź aplikacji po każdym kroku, ponieważ gotowy pod świadczy o stanie wdrożenia, a odpowiedź HTTP pozwala rozpoznać wersję i wynik. [Aktualizacje i rollback Deploymentu](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

## Część A — lokalne wdrożenie

1. Wybierz klaster lokalny dostępny na zajęciach. Przed zmianą zasobów sprawdź aktywny kontekst Kubernetes. Przygotuj manifest Deploymentu oraz sposób dotarcia do testowego interfejsu HTTP, np. przez Service i przekierowanie portu.
2. Zapewnij klastrowi dostęp do obu obrazów z lab. 5. Możesz załadować je do środowiska wykonawczego klastra albo użyć uzgodnionego sposobu pobrania z prywatnego rejestru. Zachowaj powiązanie obrazów z commitami.
3. Uruchom pierwszą wersję. Sprawdź stan Deploymentu i wywołaj interfejs HTTP z wybranymi danymi. Zapisz wersję oraz decyzję.
4. Zmień obraz na drugą wersję i poczekaj na zakończenie aktualizacji. Powtórz to samo żądanie. Potem wykonaj rollback, sprawdź rewizję i ponownie wywołaj aplikację. W odpowiedzi powinna wrócić pierwsza wersja.

Oczekiwana obserwacja to sekwencja **wersja 1 → wersja 2 → wersja 1**. Dla tej samej próbki decyzja biznesowa powinna pozostać poprawna. Zachowaj manifesty w repozytorium.

Jeśli prowadzący udostępni wspólny klaster EKS, porównaj z pokazem uruchomienia tych samych manifestów na AWS. Sprawdź, które elementy konfiguracji zależą od środowiska, np. dostęp do rejestru i sposób udostępnienia portu. Zasoby EKS przygotowuje prowadzący po sprawdzeniu konta i kosztu. [Cennik Amazon EKS](https://aws.amazon.com/eks/pricing/)

## Część B — demonstracja na AWS

1. Wyślij poprawne zlecenie z nowym UUID do wdrożonego `POST /requests`. Pokaż odpowiedź `202` i odczytaj końcowy wynik z `GET` pod zwróconym adresem. Przetwarzanie może się skończyć przed pierwszym odczytem. Próbę `PENDING` pokaż wtedy na dowodzie z lab. 3.
2. Odszukaj to samo ID w logach funkcji przetwarzającej. Wyjaśnij, która rola wysłała wiadomość, która obsłużyła kolejkę i gdzie zapisano stan końcowy.
3. Ponów to samo zlecenie i pokaż zachowany wynik. Wyjaśnij, co dzieje się przy przerwaniu operacji między zapisem w DynamoDB a potwierdzeniem przez SQS.
4. Wskaż `apiVersion` działającej trasy i przebieg GitHub Actions, który ją wdrożył. Przedstaw jeden wynik analizy z lab. 4 oraz wybrany test, który daje Ci podstawę do uznania danej zmiany za poprawną.

## Sprawdzenie i oddanie

Oddaj link do końcowego commita i dwa krótkie dowody. Pierwszy pokazuje odpowiedzi kontenera w kolejności aktualizacji i rollbacku. Drugi pokazuje próbę na AWS z jednym ID w odpowiedziach API i w logach oraz wskazuje wdrożoną wersję. Wyniki wcześniejszych etapów, w tym analizę, miej pod ręką podczas odbioru.

**Typowy błąd.** Pod pozostaje w `ImagePullBackOff`. Sprawdź, czy obraz i tag istnieją, czy klaster ma dostęp do prywatnego rejestru i czy obraz jest zbudowany dla zgodnej architektury. Jeżeli rollback kończy się powodzeniem, ale odpowiedź ma złą wersję, porównaj obrazy przypisane do obu rewizji i sprawdź, do którego poda trafia żądanie.

## Rozszerzenie dla chętnych

Po uruchomieniu całego projektu w AWS możesz wybrać jedno ćwiczenie dodatkowe. Uzgodnij dostęp do zasobów z prowadzącym.

- **Odzyskiwanie zlecenia.** Dodaj kolejkę błędów (*dead-letter queue*, DLQ). Wywołaj kontrolowany trwały błąd i pokaż, że wiadomość trafia do DLQ po ustalonej liczbie prób. Usuń przyczynę awarii, ponów przetwarzanie i sprawdź wynik dla tego samego ID. [Obsługa błędów w Lambda–SQS](https://docs.aws.amazon.com/lambda/latest/dg/services-sqs-configure.html)
- **Analiza w AWS.** Zapisz syntetyczne wyniki z lab. 4 w prywatnym S3. Oblicz odsetki odrzuceń po grupach w Athena i porównaj je z obliczeniami ręcznymi. Pokaż zapytanie i tabelę wyników. [Pierwsze zapytanie w Amazon Athena](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html)

## Po odbiorze

1. Zachowaj commit i dowody potrzebne do oceny. Uzgodnij z prowadzącym moment sprzątania, jeśli będzie jeszcze wywoływać Twoje API.
2. Usuń zasoby utworzone na potrzeby ćwiczenia w lokalnym Kubernetesie. Na AWS zakończ przyjmowanie zleceń i wyłącz powiązanie kolejki z funkcją przed usunięciem zasobów aplikacji.
3. Przejrzyj własną listę zasobów, w tym API, obie funkcje, kolejkę, tabelę, logi, obiekty S3, obrazy ECR i role utworzone specjalnie dla projektu. Usuń przypisane Ci zasoby zgodnie z zasadami konta. Przy współdzielonym buckecie lub rejestrze ogranicz się do przydzielonych obiektów lub obrazów. Zasoby współdzielone usuwa ich właściciel.
