# Instalacja narzędzi – instrukcja dla studentów

**Termin: przed zajęciami w tygodniu 2.** Jeśli coś nie zadziała – nie szkodzi, na początku tych zajęć wspólnie rozwiążemy problemy z instalacją.
Czas: ok. 30–60 minut (w tym pobieranie plików).

## Co instalujemy i po co

| Narzędzie | Do czego służy |
|---|---|
| **Git** | System kontroli wersji – wysyłanie rozwiązań na GitHub |
| **Visual Studio Code** | Edytor kodu |
| **Rozszerzenie Dev Containers** | Uruchamia gotowe środowisko laboratoryjne w kontenerze |
| **Docker Desktop** | Uruchamia kontenery – w nich działa „chmura” (LocalStack), Terraform, AWS CLI i Python |

Nie musicie instalować AWS CLI, Terraform ani Pythona – wszystko to jest już w kontenerze laboratoryjnym.

## Wymagania

- Windows 10/11 (64-bit), macOS 13 lub nowszy albo Linux
- **min. 8 GB RAM** (zalecane 16 GB), **ok. 15 GB wolnego miejsca** na dysku
- **uprawnienia administratora** na komputerze
- najlepiej **prywatny laptop** – Docker Desktop jest darmowy do nauki i użytku prywatnego, ale na komputerach dużych firm wymaga płatnej licencji

> Nie spełniacie wymagań lub nie macie uprawnień administratora? Nic nie szkodzi – przejdźcie do sekcji **„Plan B: Google Cloud Shell”** na końcu.

---

## Windows 10 / 11

### Krok 1 – WSL2 (wymagany przez Docker)
1. Kliknij Start, wpisz **PowerShell**, kliknij prawym przyciskiem → **Uruchom jako administrator**.
2. Wpisz:
   ```powershell
   wsl --install
   ```
3. **Uruchom komputer ponownie.**
4. Jeśli po restarcie otworzy się okno Ubuntu i poprosi o nazwę użytkownika i hasło – ustaw je (możecie je zapamiętać, rzadko będą potrzebne).

> Jeśli WSL jest już zainstalowany, zaktualizuj go: `wsl --update`

### Krok 2 – Git
1. Pobierz z <https://git-scm.com/download/win> i zainstaluj.
2. Wszystkie opcje instalatora możecie zostawić domyślne.

### Krok 3 – Visual Studio Code
1. Pobierz z <https://code.visualstudio.com> i zainstaluj.
2. Podczas instalacji zaznacz **„Add to PATH”** i **„Add 'Open with Code' action”**.

### Krok 4 – Docker Desktop
1. Pobierz z <https://www.docker.com/products/docker-desktop/> (wersja dla Windows).
2. Podczas instalacji zostaw zaznaczone **„Use WSL 2 instead of Hyper-V”**.
3. Po instalacji uruchom ponownie komputer, potem uruchom **Docker Desktop** i zaakceptuj warunki.
   (Logowanie do konta Docker jest opcjonalne – możecie kliknąć „Skip”.)
4. Poczekaj, aż ikona wieloryba w zasobniku systemowym przestanie się animować (Docker działa).

### Krok 5 – rozszerzenie Dev Containers
W VS Code: ikona **Extensions** po lewej (lub `Ctrl+Shift+X`) → wyszukaj **Dev Containers** (autor: Microsoft) → **Install**.

Przejdź do sekcji **„Konfiguracja Git”**.

---

## macOS

### Krok 1 – Git
Otwórz **Terminal** i wpisz:
```bash
xcode-select --install
```
Potwierdź instalację w oknie, które się pojawi.

### Krok 2 – Visual Studio Code
1. Pobierz z <https://code.visualstudio.com>, rozpakuj i przenieś do **Aplikacji**.
2. Uruchom VS Code → `Cmd+Shift+P` → wpisz **„Shell Command: Install 'code' command in PATH”** → Enter.

### Krok 3 – Docker Desktop
1. Pobierz z <https://www.docker.com/products/docker-desktop/>.
   **Wybierz właściwą wersję:** menu Apple  → **Ten Mac** – jeśli procesor to *Apple M1/M2/M3/M4* → **Apple Silicon**, jeśli *Intel* → **Intel chip**.
2. Przeciągnij Docker do Aplikacji, uruchom go, zaakceptuj warunki (logowanie opcjonalne).

### Krok 4 – rozszerzenie Dev Containers
W VS Code: **Extensions** (`Cmd+Shift+X`) → **Dev Containers** (Microsoft) → **Install**.

---

## Linux (Ubuntu)

```bash
sudo apt update && sudo apt install -y git
```
- Docker: <https://docs.docker.com/engine/install/ubuntu/> (Docker Engine wystarczy), potem:
  `sudo usermod -aG docker $USER` i wyloguj się / zaloguj ponownie.
- VS Code: <https://code.visualstudio.com> (paczka `.deb`) + rozszerzenie **Dev Containers**.

---

## Konfiguracja Git (wszystkie systemy)

W terminalu (Windows: **Git Bash** lub PowerShell):
```bash
git config --global user.name "Imię Nazwisko"
git config --global user.email "email-uzyty-na-github@example.com"
```
Użyjcie tego samego e-maila co na GitHubie.

Logowanie do GitHub: przy pierwszym `git push` otworzy się okno przeglądarki – zalogujcie się i zatwierdźcie.

---

## Sprawdzenie instalacji ✅

Upewnij się, że **Docker Desktop jest uruchomiony**, otwórz nowy terminal i wpisz po kolei:

```bash
git --version
code --version
docker --version
docker run hello-world
```

Ostatnia komenda powinna wypisać m.in.:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Co oddać
Zrzut ekranu terminala z wynikiem **wszystkich czterech komend** → wyślij e-mailem na **[Twój e-mail]**.
Temat wiadomości: **`[Chmura] Lab 1 – Imię Nazwisko – Grupa A/B`**.
To część **Laboratorium 1**.

---

## Najczęstsze problemy

| Problem | Rozwiązanie |
|---|---|
| `wsl --install` – błąd o wirtualizacji / Docker: *„Virtualization support not detected”* | Wirtualizacja jest wyłączona w BIOS. Sprawdź: Menedżer zadań → Wydajność → CPU → *Wirtualizacja*. Jeśli „Wyłączona” – włącz w BIOS/UEFI opcję **Intel VT-x** / **AMD-V (SVM)**. Nazwa zależy od producenta – wyszukaj „model laptopa + enable virtualization”. |
| Docker: *„WSL update required”* / *„WSL 2 installation is incomplete”* | W PowerShell (administrator): `wsl --update`, potem restart. |
| `docker: command not found` / *„Cannot connect to the Docker daemon”* | Docker Desktop nie jest uruchomiony – uruchom go i poczekaj ok. minuty. |
| `code: command not found` | Windows: zainstaluj VS Code ponownie z opcją „Add to PATH”. Mac: krok 2 w sekcji macOS. |
| Brak uprawnień administratora | Skorzystaj z **Planu B** poniżej. |
| Komputer bardzo zwalnia | Zamknij inne programy; w Docker Desktop → Settings → Resources możesz ograniczyć pamięć. |

Nie udało się? **Nie przejmuj się** – przyjdź na zajęcia w tygodniu 2, rozwiążemy to razem. Opisz problem i dołącz zrzut ekranu błędu w e-mailu.

---

## Plan B: Google Cloud Shell (bez instalacji)

Jeśli instalacja na Twoim komputerze nie jest możliwa:
1. Wejdź na <https://shell.cloud.google.com> i zaloguj się kontem Google.
2. Dostajesz darmowy terminal Linux w przeglądarce (ok. 50 godzin tygodniowo) z zainstalowanymi Git, Docker, Terraform i Pythonem.
   Do samego Cloud Shell nie trzeba podawać karty płatniczej.
3. Wpisz w terminalu:
   ```bash
   git --version
   docker --version
   docker run hello-world
   ```
4. Zrzut ekranu wyślij e-mailem tak jak wyżej – napisz w nim, że korzystasz z Cloud Shell.

Instrukcję uruchamiania laboratoriów w Cloud Shell dostaniecie razem z każdym zadaniem.
