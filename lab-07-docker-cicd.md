# Laboratorium 7 — Docker i wdrożenie z GitHub Actions

[Poprzednie laboratorium](lab-06-reguly-analiza.md) · [Instrukcja główna](README.md) · [Następne laboratorium](lab-08-kubernetes-odbior.md)

## Cel

Uruchom testy przy każdym pull requeście. Po zmianie na `main` i udanych testach wdróż nowy kod do istniejącej funkcji Lambda API i opublikuj obraz w prywatnym Amazon ECR. Sprawdź wdrożenie na tym samym adresie API, którego używałeś wcześniej.

## Co warto wiedzieć

CI wykrywa błędy przed wdrożeniem. CD przenosi sprawdzoną zmianę do działającego środowiska. Na tym etapie pipeline wytwarza dwa artefakty z tego samego commita. Pierwszy ma format wymagany przez istniejącą Lambdę API. Drugi to obraz aplikacji HTTP używany na następnym laboratorium.

GitHub Actions może uzyskać krótkotrwałe poświadczenia AWS przez **OIDC**. Polityka zaufania określa, z którego repozytorium i kontekstu uruchomienia zadanie może przyjąć rolę. Uprawnienia tej roli określają, co zadanie może zmienić w AWS po jej przyjęciu. [OIDC GitHub Actions w AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)

## Zadanie

1. Dodaj prosty interfejs HTTP do lokalnego wywołania modułu reguł i przygotuj dla niego obraz Docker. Wywołanie ma zwrócić decyzję oraz identyfikator wersji, bez potrzeby dostępu do SQS i DynamoDB. Sprawdź na jednym wejściu, czy kontener zwraca tę samą decyzję co moduł testowany w lab. 4.
2. Dodaj w GitHub Actions uruchamianie testów dla pull requestów i zmian na `main`. Przed pierwszym wdrożeniem sprawdź, jak wygląda przebieg z nieudanym testem. Wdrożenie i publikacja obrazu mają zależeć od przejścia testów i uruchamiać się dla zmian na `main`.
3. Przygotuj lub odbierz od prowadzącego rolę AWS dla GitHub Actions. Skonfiguruj dostęp przez OIDC dla własnego repozytorium i gałęzi `main` albo zatwierdzonego środowiska wdrożeniowego z regułą dopuszczającą tę gałąź. Przyznaj roli operacje potrzebne do aktualizacji wskazanej funkcji i publikacji w docelowym repozytorium ECR. Oddziel rolę wdrożeniową od ról używanych przez kod funkcji.
4. Przygotuj docelowe prywatne repozytorium ECR zgodnie z ustaleniami dla konta. Automatyzacja ma zaktualizować kod **istniejącej** Lambdy API i opublikować obraz. Oznacz obraz identyfikatorem pozwalającym powiązać go z commitem, np. skrótem SHA. Wybierz architekturę obrazu obsługiwaną przez Twój klaster lokalny. [Publikacja obrazów Docker w ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html)
5. Przed scaleniem zmiany odczytaj `apiVersion` dla znanego ID. Po udanym wdrożeniu odczytaj to samo zlecenie pod tym samym adresem i sprawdź, czy zwraca nową wersję API oraz dotychczasowy wynik. Przygotuj dwie kolejne wersje obrazu z rozróżnialną odpowiedzią HTTP na następne spotkanie. Pierwszy przebieg na `main` może opublikować obraz bazowy, a następny obraz ze zmianą wersji. Zachowaj ich tagi lub digesty.

## Sprawdzenie i oddanie

- Pull request uruchamia testy. Przebieg wdrożeniowy po zmianie na `main` ma udane testy, aktualizację Lambdy i publikację obrazu w ECR.
- Stary i nowy odczyt tego samego zlecenia pokazują zmianę `apiVersion`. W ECR widać obraz powiązany z commitem użytym do wdrożenia.
- Oddaj link do commita i dwa dowody. Pierwszy to link do udanego przebiegu GitHub Actions wraz z tagiem lub digestem obrazu. Drugi to odpowiedzi HTTP przed wdrożeniem i po nim.

**Typowy błąd.** GitHub Actions przechodzi testy, ale AWS odrzuca przyjęcie roli. Porównaj repozytorium, gałąź lub środowisko wdrożeniowe oraz dane tokena z warunkami zaufania. Jeżeli rola została już przyjęta, sprawdź osobno jej uprawnienia do operacji na Lambdzie i ECR. Gdy workflow kończy się powodzeniem, a API zwraca starą wersję, sprawdź integrację trasy z funkcją lub jej aliasem.

**Zachowaj na kolejny etap.** Działające API, wyniki testów, dwa identyfikowalne obrazy i informację, jak pobrać je z ECR lub udostępnić lokalnemu klastrowi.
