Poczta: v.cerniavski@uwb.edu.pl

# Lab 2 · Jedna strona, dwa sposoby udostępnienia

Pracujesz indywidualnie, w swoim repozytorium. Przygotuj prostą stronę „Raport koszyków” i otwórz ją w przeglądarce pod dwoma adresami. Pod adresem S3 zobaczysz gotowy plik. Pod adresem funkcji Lambda zobaczysz stronę z nagłówkiem dopasowanym do parametru w adresie. Obie usługi uruchomisz na swoim komputerze w LocalStack.
Pliki dockera, polecenia wykonane w konsoli lstk proszę zapisać jako udowodnienia w oddzielnym pliku tekstowym oraz umieścić na swój github.

## 1. Przygotuj środowisko

Uruchom LocalStack przez Docker Compose. Zapisz konfigurację w `compose.yaml`. Udostępnij port tylko lokalnie i sprawdź w dokumentacji, czego LocalStack potrzebuje, aby uruchamiać funkcje Lambda w Dockerze. Jeśli obraz wymaga tokenu, zapisz go w lokalnym pliku `.env` i dodaj ten plik do `.gitignore`. Polecenia AWS CLI kieruj do lokalnego emulatora. [Instalacja LocalStack](https://docs.localstack.cloud/aws/getting-started/installation/), [Lambda w LocalStack](https://docs.localstack.cloud/aws/services/lambda/)

## 2. Udostępnij statyczną stronę w S3

W katalogu `www/` przygotuj `index.html` z nagłówkiem „Raport koszyków”, krótkim opisem i osobnym arkuszem `style.css`. Wybierz zmianę stylu, którą łatwo dostrzec w przeglądarce.

1. Utwórz bucket w LocalStack i prześlij oba pliki. Włącz hosting statyczny, ustaw `index.html` jako dokument startowy i skonfiguruj odczyt plików ćwiczeniowych przez przeglądarkę.
2. Otwórz główny adres **S3 Website** i sprawdź, czy strona ładuje się razem z CSS.
3. Zmień jedno zdanie w HTML, prześlij poprawiony plik i sprawdź wynik pod tym samym adresem.

[Hosting strony statycznej w LocalStack](https://docs.localstack.cloud/aws/tutorials/s3-static-website-terraform/)

## 3. Udostępnij wariant tej samej strony przez Lambdę

Na podstawie swojej strony przygotuj samodzielny plik `template.html`. Jeśli chcesz zachować styl, przenieś CSS do tego pliku. W nagłówku wpisz `Raport dla {{IMIE}}`. Prześlij szablon do osobnego bucketu w S3 i pozostaw jego domyślne ustawienia dostępu.

Napisz funkcję w Pythonie i udostępnij ją lokalnie przez **Lambda Function URL**. Funkcja ma:

- odczytać parametr `imie` z adresu. Jeśli brakuje parametru lub jego wartość jest pusta, zwrócić HTTP `400` z krótką podpowiedzią,
- przy poprawnym żądaniu pobrać szablon z S3 i zastąpić `{{IMIE}}` podaną wartością. Znaki specjalne w imieniu powinny wyświetlać się jako tekst,
- zwrócić gotowy HTML z kodem `200` i nagłówkiem `Content-Type: text/html; charset=utf-8`.

Szablon zostaw w S3. W paczce wdrożeniowej umieść kod funkcji. [Lambda w LocalStack](https://docs.localstack.cloud/aws/services/lambda/), [Boto3 w LocalStack](https://docs.localstack.cloud/aws/connecting/aws-sdks/python-boto3/), [Format żądania i odpowiedzi Function URL](https://docs.aws.amazon.com/lambda/latest/dg/urls-invocation.html)

Otwórz adres funkcji z `?imie={Podaj imie}` i `?imie={Podaj inne ime}`. Sprawdź oba nagłówki oraz odpowiedź po otwarciu adresu bez parametru. Następnie dopisz nowe zdanie w szablonie, prześlij go ponownie do S3 i odśwież stronę bez ponownego wdrażania funkcji. Odczytaj szablon z S3 i sprawdź, czy w obiekcie nadal jest `{{IMIE}}`.

## Co oddajesz

Prześlij prowadzącemu **link do commita na GitHubie**. Dołącz konfigurację Compose, pliki strony statycznej, kod funkcji, końcową wersję szablonu i raport `dowody/02-s3-lambda.md`. Jeśli konfigurujesz dostęp do statycznej strony za pomocą pliku polityki bucketu, dołącz go do commita. W raporcie podaj nazwy bucketów i funkcji oraz polecenia użyte do uruchomienia środowiska i wdrożenia obu wariantów.

W raporcie pokaż:

- **S3 Website.** Zrzut z paskiem adresu, widocznym stylem CSS i zdaniem zmienionym po publikacji.
- **Lambda Function URL.** Dwa zrzuty z paskiem adresu i nagłówkami dla Oli i Jana. Na przynajmniej jednym pokaż zdanie dodane później do szablonu i napisz, który obiekt w S3 został podmieniony.
- **Odpowiedzi HTTP i źródło HTML.** Pokaż status `200` i nagłówek `Content-Type`, status `400` z komunikatem oraz fragment obiektu odczytanego z S3 z widocznym `{{IMIE}}`. Do odpowiedzi HTTP wystarczy wynik z terminala albo zrzut z zakładki **Sieć** w przeglądarce.
- **Wniosek.** Napisz, kto dostarcza HTML w każdym wariancie, kiedy wystarczy strona statyczna i po co w drugim wariancie trzymać szablon osobno od kodu funkcji.
