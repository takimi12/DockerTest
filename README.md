Konteneryzacja aplikacji Node.js
Wymagania wstępne
Zainstalowano najnowszą wersję Docker Desktop.

Posiadasz klienta Git. Przykłady w tej sekcji używają klienta Git wiersza poleceń, ale możesz użyć dowolnego innego klienta.

Przegląd
Ta sekcja przeprowadzi Cię przez proces konteneryzacji i uruchomienia aplikacji Node.js.

Pobierz przykładową aplikację
Sklonuj przykładową aplikację, której będziemy używać w tym przewodniku. Otwórz terminal, przejdź do katalogu, w którym chcesz pracować i uruchom następujące polecenie, aby sklonować repozytorium:


git clone https://github.com/docker/docker-nodejs-sample && cd docker-nodejs-sample
Inicjalizacja zasobów Dockera
Teraz, gdy masz aplikację, możesz utworzyć niezbędne pliki Dockera, aby ją skonteneryzować. Możesz skorzystać z wbudowanej funkcji Docker Init w Docker Desktop, aby ułatwić ten proces, albo utworzyć pliki ręcznie.

Użyj Docker Init
Wewnątrz katalogu docker-nodejs-sample uruchom polecenie docker init w terminalu. Polecenie to wygeneruje domyślną konfigurację, ale będziesz musiał odpowiedzieć na kilka pytań dotyczących Twojej aplikacji. Oto przykładowe odpowiedzi:


docker init
Wynik:


Witamy w Docker Init CLI!

To narzędzie pomoże Ci utworzyć następujące pliki z domyślną konfiguracją:
  - .dockerignore
  - Dockerfile
  - compose.yaml
  - README.Docker.md

Zaczynamy!

? Jakiej platformy aplikacyjnej używa Twój projekt? Node  
? Jakiej wersji Node chcesz użyć? 18.0.0  
? Którego menedżera pakietów chcesz użyć? npm  
? Jakiego polecenia chcesz użyć do uruchomienia aplikacji: node src/index.js  
? Na jakim porcie nasłuchuje Twój serwer? 3000  
Po zakończeniu, katalog docker-nodejs-sample powinien zawierać przynajmniej następujące pliki:


├── docker-nodejs-sample/
│   ├── spec/
│   ├── src/
│   ├── .dockerignore
│   ├── .gitignore
│   ├── compose.yaml
│   ├── Dockerfile
│   ├── package-lock.json
│   ├── package.json
│   └── README.md
Aby dowiedzieć się więcej o tych plikach, zobacz:

Dockerfile

.dockerignore

compose.yaml

Uruchom aplikację
Wewnątrz katalogu docker-nodejs-sample uruchom następujące polecenie w terminalu:

docker compose up --build
Otwórz przeglądarkę i przejdź do: http://localhost:3000. Powinna się wyświetlić prosta aplikacja typu "todo".

Aby zatrzymać aplikację, naciśnij Ctrl+C w terminalu.

Uruchom aplikację w tle
Możesz uruchomić aplikację w tle (z odłączeniem od terminala) dodając opcję -d. W katalogu docker-nodejs-sample użyj polecenia:


docker compose up --build -d
Aplikacja będzie dostępna pod adresem: http://localhost:3000.

Aby zatrzymać aplikację, użyj:

docker compose down
Więcej informacji o poleceniach Compose znajdziesz w dokumentacji Compose CLI.



Używanie kontenerów do rozwoju aplikacji Node.js

Wymagania wstępne
Zrealizuj zadanie "Konteneryzacja aplikacji Node.js".

Przegląd
W tej sekcji nauczysz się, jak skonfigurować środowisko deweloperskie dla swojej konteneryzowanej aplikacji. Obejmuje to:

Dodanie lokalnej bazy danych i utrwalanie danych

Konfigurację kontenera do uruchomienia środowiska deweloperskiego

Debugowanie konteneryzowanej aplikacji

Dodaj lokalną bazę danych i utrwalaj dane
Możesz użyć kontenerów do konfiguracji lokalnych usług, takich jak baza danych. W tej sekcji zaktualizujesz plik compose.yaml, aby zdefiniować usługę bazy danych oraz wolumen do utrwalania danych.

Otwórz plik compose.yaml w IDE lub edytorze tekstu.

Odkomentuj instrukcje związane z bazą danych. Poniżej znajduje się zaktualizowany plik compose.yaml.

Ważne
W tej sekcji nie uruchamiaj komendy docker compose up, dopóki nie zostaniesz do tego poinstruowany. Uruchomienie tej komendy w trakcie procesu może błędnie zainicjować Twoją bazę danych.

# Komentarze znajdują się w całym pliku, aby pomóc Ci w rozpoczęciu.
# Jeśli potrzebujesz więcej pomocy, odwiedź przewodnik po Docker Compose pod adresem
# https://docs.docker.com/go/compose-spec-reference/

# Tutaj instrukcje definiują Twoją aplikację jako usługę o nazwie "server".
# Usługa ta jest budowana z Dockerfile w bieżącym katalogu.
# Możesz dodać inne usługi, od których zależy Twoja aplikacja, takie jak
# baza danych lub pamięć podręczna. Przykłady znajdują się w repozytorium Awesome Compose:
# https://github.com/docker/awesome-compose
services:
  server:
    build:
      context: .
    environment:
      NODE_ENV: production
    ports:
      - 3000:3000

    # Poniższa odkomentowana sekcja to przykład, jak zdefiniować bazę danych PostgreSQL,
    # którą Twoja aplikacja może wykorzystać. `depends_on` mówi Docker Compose, aby
    # uruchomił bazę danych przed aplikacją. Wolumen `db-data` utrwala dane bazy danych
    # między restartami kontenerów. Sekret `db-password` jest używany do ustawienia
    # hasła bazy danych. Musisz utworzyć plik `db/password.txt` i dodać wybrane przez
    # siebie hasło, zanim uruchomisz `docker compose up`.

    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
Uwaga
Aby dowiedzieć się więcej o instrukcjach w pliku Compose, zapoznaj się z przewodnikiem po pliku Compose.

Otwórz plik src/persistence/postgres.js w IDE lub edytorze tekstu. Zauważysz, że aplikacja używa bazy danych Postgres i wymaga kilku zmiennych środowiskowych do połączenia się z bazą danych. Plik compose.yaml jeszcze nie zawiera tych zmiennych.

Dodaj zmienne środowiskowe określające konfigurację bazy danych. Poniżej znajduje się zaktualizowany plik compose.yaml.



services:
  server:
    build:
      context: .
    environment:
      NODE_ENV: production
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db-password
      POSTGRES_DB: example
    ports:
      - 3000:3000
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
Dodaj sekcję secrets pod usługą serwera, aby Twoja aplikacja bezpiecznie obsługiwała hasło bazy danych. Poniżej znajduje się zaktualizowany plik compose.yaml.

services:
  server:
    build:
      context: .
    environment:
      NODE_ENV: production
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db-password
      POSTGRES_DB: example
    ports:
      - 3000:3000
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
W katalogu docker-nodejs-sample stwórz katalog o nazwie db.

W katalogu db utwórz plik o nazwie password.txt. Ten plik będzie zawierał Twoje hasło do bazy danych.

Powinieneś teraz mieć przynajmniej następującą strukturę katalogów w swoim projekcie:


├── docker-nodejs-sample/
│ ├── db/
│ │ └── password.txt
│ ├── spec/
│ ├── src/
│ ├── .dockerignore
│ ├── .gitignore
│ ├── compose.yaml
│ ├── Dockerfile
│ ├── package-lock.json
│ ├── package.json
│ └── README.md
Otwórz plik password.txt w edytorze tekstu i określ wybrane przez siebie hasło. Twoje hasło musi znajdować się w jednym wierszu bez dodatkowych linii. Upewnij się, że plik nie zawiera żadnych znaków nowej linii ani ukrytych znaków.

Zapisz zmiany w plikach, które edytowałeś.

Uruchom poniższą komendę, aby uruchomić aplikację:


docker compose up --build
Otwórz przeglądarkę i zweryfikuj, czy aplikacja działa pod adresem http://localhost:3000.

Dodaj kilka elementów do listy zadań, aby przetestować utrwalanie danych.

Po dodaniu kilku elementów, naciśnij ctrl+c w terminalu, aby zatrzymać aplikację.

W terminalu uruchom docker compose rm, aby usunąć kontenery:


docker compose rm
Uruchom ponownie aplikację, używając docker compose up:


docker compose up --build
Odśwież stronę http://localhost:3000 w swojej przeglądarce i zweryfikuj, czy elementy na liście zadań zostały utrwalone, nawet po usunięciu kontenerów i ich ponownym uruchomieniu.

Skonfiguruj i uruchom kontener deweloperski
Możesz użyć montażu wiązań, aby zamontować kod źródłowy do kontenera. Dzięki temu kontener będzie natychmiast widział zmiany wprowadzane w kodzie po zapisaniu pliku. Oznacza to, że możesz uruchomić procesy, takie jak nodemon, w kontenerze, które będą monitorować zmiany w systemie plików i reagować na nie. Aby dowiedzieć się więcej o montażach wiązań, zapoznaj się z przeglądem przechowywania.

Oprócz dodania montażu wiązań, możesz skonfigurować swój Dockerfile i plik compose.yaml, aby zainstalować zależności deweloperskie i uruchomić narzędzia deweloperskie.

Zaktualizuj swój Dockerfile dla środowiska deweloperskiego
Otwórz Dockerfile w edytorze tekstu. Zauważ, że Dockerfile nie instaluje zależności deweloperskich i nie uruchamia nodemon. Musisz zaktualizować swój Dockerfile, aby zainstalować zależności deweloperskie i uruchomić nodemon.

Zamiast tworzyć jeden Dockerfile dla produkcji, a drugi dla rozwoju, możesz używać jednego Dockerfile wieloetapowego dla obu środowisk.

Zaktualizuj Dockerfile do poniższego wieloetapowego Dockerfile:

# syntax=docker/dockerfile:1

ARG NODE_VERSION=18.0.0

FROM node:${NODE_VERSION}-alpine as base
WORKDIR /usr/src/app
EXPOSE 3000

FROM base as dev
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
USER node
COPY . .
CMD npm run dev

FROM base as prod
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev
USER node
COPY . .
CMD node src/index.js
W Dockerfile najpierw dodajesz etap podstawowy, oznaczony jako base, który odnosi się do obrazu node:${NODE_VERSION}-alpine. Następnie dodajesz nowy etap, oznaczony jako dev, który instaluje zależności deweloperskie i uruchamia kontener z npm run dev. Na końcu dodajesz etap prod, który pomija zależności deweloperskie i uruchamia aplikację przy użyciu node src/index.js. Aby dowiedzieć się więcej o budowaniu wieloetapowym, zapoznaj się z dokumentacją dotyczącą Multi-stage builds.

Zaktualizuj plik Compose dla środowiska deweloperskiego
Aby uruchomić etap deweloperski z Compose, musisz zaktualizować plik compose.yaml. Otwórz plik compose.yaml w edytorze tekstu, a następnie dodaj instrukcję target: dev, aby wybrać etap deweloperski z wieloetapowego Dockerfile.

Dodaj również nowy wolumen do usługi serwera, aby zamontować źródła kodu. Dla tej aplikacji zamontujesz ./src z lokalnej maszyny do /usr/src/app/src w kontenerze.

Na końcu opublikuj port 9229 do debugowania.

Poniżej znajduje się zaktualizowany plik Compose:


services:
  server:
    build:
      context: .
      target: dev
    ports:
      - 3000:3000
      - 9229:9229
    environment:
      NODE_ENV: production
      POSTGRES_HOST: db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD_FILE: /run/secrets/db-password
      POSTGRES_DB: example
    depends_on:
      db:
        condition: service_healthy
    secrets:
      - db-password
    volumes:
      - ./src:/usr/src/app/src
  db:
    image: postgres
    restart: always
    user: postgres
    secrets:
      - db-password
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
    expose:
      - 5432
    healthcheck:
      test: ["CMD", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
secrets:
  db-password:
    file: db/password.txt
Uruchom swój kontener deweloperski i debuguj aplikację
Uruchom aplikację z poniższą komendą, aby uwzględnić zmiany w Dockerfile i pliku compose.yaml:

docker compose up --build
Otwórz przeglądarkę i zweryfikuj, czy aplikacja działa pod adresem http://localhost:3000.

Wszystkie zmiany w plikach źródłowych aplikacji na Twojej maszynie lokalnej będą teraz natychmiast widoczne w działającym kontenerze.

Otwórz plik docker-nodejs-sample/src/static/js/app.js i zaktualizuj tekst przycisku w linii 109 z "Add Item" na "Add".

Odśwież stronę http://localhost:3000 w przeglądarce i upewnij się, że tekst został zaktualizowany.

Teraz możesz połączyć klienta inspektora z aplikacją, aby debugować kod. Więcej szczegółów znajdziesz w dokumentacji Node.js na temat klientów inspektora.

Podsumowanie
W tej sekcji zapoznałeś się z konfiguracją pliku Compose w celu dodania fikcyjnej bazy danych i utrwalenia danych. Nauczyłeś się także, jak utworzyć wieloetapowy Dockerfile i skonfigurować montaż wiązań do pracy deweloperskiej.

------------------------------------
Uruchamianie testów Node.js w kontenerze

Wymagania wstępne
Ukończ wszystkie poprzednie sekcje tego przewodnika, począwszy od "Containerize a Node.js application".

Przegląd
Testowanie jest kluczowym elementem nowoczesnego rozwoju oprogramowania. Testowanie może oznaczać różne rzeczy dla różnych zespołów deweloperskich. Istnieją testy jednostkowe, testy integracyjne oraz testowanie end-to-end. W tym przewodniku zapoznasz się z uruchamianiem testów jednostkowych w Dockerze podczas pracy nad projektem oraz przy jego budowie.

Uruchamianie testów podczas lokalnego rozwoju
Przykładowa aplikacja już zawiera pakiet Jest do uruchamiania testów oraz testy w katalogu "spec". Podczas lokalnego rozwoju możesz użyć Compose do uruchomienia testów.

Uruchom poniższe polecenie, aby uruchomić skrypt testowy z pliku package.json wewnątrz kontenera:


docker compose run server npm run test
Aby dowiedzieć się więcej o tym poleceniu, zapoznaj się z dokumentacją "docker compose run".

Powinieneś zobaczyć wynik podobny do poniższego:


docker-nodejs@1.0.0 test
jest

PASS  spec/routes/deleteItem.spec.js
PASS  spec/routes/getItems.spec.js
PASS  spec/routes/addItem.spec.js
PASS  spec/routes/updateItem.spec.js
PASS  spec/persistence/sqlite.spec.js
● Console

console.log
  Using sqlite database at /tmp/todo.db
  at Database.log (src/persistence/sqlite.js:18:25)
...
Test Suites: 5 passed, 5 total
Tests:       9 passed, 9 total
Snapshots:   0 total
Time:        2.008 s
Ran all test suites.
Uruchamianie testów podczas budowy
Aby uruchomić testy podczas budowania obrazu, należy zaktualizować Dockerfile, dodając nowy etap testowy.

Poniżej znajduje się zaktualizowany Dockerfile:


# syntax=docker/dockerfile:1

ARG NODE_VERSION=18.0.0

FROM node:${NODE_VERSION}-alpine as base
WORKDIR /usr/src/app
EXPOSE 3000

FROM base as dev
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
USER node
COPY . .
CMD npm run dev

FROM base as prod
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev
USER node
COPY . .
CMD node src/index.js

FROM base as test
ENV NODE_ENV test
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/root/.npm \
    npm ci --include=dev
USER node
COPY . .
RUN npm run test
Zamiast używać instrukcji CMD w etapie testowym, użyj RUN, aby uruchomić testy. Powodem jest to, że instrukcja CMD uruchamia się, gdy kontener jest uruchamiany, podczas gdy instrukcja RUN uruchamia się, gdy obraz jest budowany, a budowa nie powiedzie się, jeśli testy się nie powiodą.

Uruchom poniższe polecenie, aby zbudować nowy obraz, używając etapu testowego jako celu, i zobaczyć wyniki testów. Dołącz --progress=plain, aby wyświetlić wynik budowy, --no-cache, aby upewnić się, że testy zawsze zostaną uruchomione, oraz --target test, aby celować w etap testowy:


docker build -t node-docker-image-test --progress=plain --no-cache --target test .
Powinieneś zobaczyć wynik zawierający następujące informacje:


...
11 [test 3/3] RUN npm run test
11 1.058
11 1.058 > docker-nodejs@1.0.0 test
11 1.058 > jest
11 1.058
11 3.765 PASS spec/routes/getItems.spec.js
11 3.767 PASS spec/routes/deleteItem.spec.js
11 3.783 PASS spec/routes/updateItem.spec.js
11 3.806 PASS spec/routes/addItem.spec.js
11 4.179 PASS spec/persistence/sqlite.spec.js
11 4.207
11 4.208 Test Suites: 5 passed, 5 total
11 4.208 Tests:       9 passed, 9 total
11 4.208 Snapshots:   0 total
11 4.208 Time:        2.168 s
11 4.208 Ran all test suites.
11 4.265 npm notice
11 4.265 npm notice New major version of npm available! 8.6.0 -> 9.8.1
11 4.265 npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.8.1>
11 4.265 npm notice Run `npm install -g npm@9.8.1` to update!
11 4.266 npm notice
11 DONE 4.3s
...
Podsumowanie
W tej sekcji dowiedziałeś się, jak uruchomić testy podczas lokalnego rozwoju za pomocą Compose oraz jak uruchomić testy podczas budowania obrazu.
-------------------------------------------------
Skonfiguruj CI/CD dla swojej aplikacji Node.js


Wymagania wstępne
Ukończ wszystkie poprzednie sekcje tego przewodnika, zaczynając od "Konteneryzacja aplikacji Node.js". Musisz mieć konto na GitHubie oraz na Dockerze, aby ukończyć tę sekcję.

Podsumowanie
W tej sekcji nauczysz się, jak skonfigurować i używać GitHub Actions do budowania i testowania swojego obrazu Docker oraz przesyłania go na Docker Hub. Wykonasz następujące kroki:

Utwórz nowe repozytorium na GitHubie.

Zdefiniuj workflow GitHub Actions.

Uruchom workflow.

Krok pierwszy: Utwórz repozytorium
Utwórz repozytorium na GitHubie, skonfiguruj dane logowania do Docker Hub i prześlij swój kod źródłowy.

Utwórz nowe repozytorium na GitHubie.

Otwórz ustawienia repozytorium i przejdź do sekcji Secrets and variables > Actions.

Utwórz nową zmienną repozytorium o nazwie DOCKER_USERNAME i jako wartość wpisz swoje ID na Dockerze.

Utwórz nowy Personal Access Token (PAT) dla Docker Hub. Możesz nadać temu tokenowi nazwę docker-tutorial. Upewnij się, że uprawnienia dostępu obejmują opcje Read oraz Write.

Dodaj PAT jako sekret repozytorium w swoim repozytorium GitHub, pod nazwą DOCKERHUB_TOKEN.

W lokalnym repozytorium na swoim komputerze, uruchom poniższe polecenie, aby zmienić adres zdalny na repozytorium, które właśnie stworzyłeś. Upewnij się, że zamienisz your-username na swoją nazwę użytkownika na GitHubie, a your-repository na nazwę repozytorium, które stworzyłeś.


git remote set-url origin https://github.com/your-username/your-repository.git
Uruchom poniższe polecenia, aby dodać pliki, zatwierdzić zmiany i wysłać swoje lokalne repozytorium na GitHub:


git add -A
git commit -m "my commit"
git push -u origin main
Krok drugi: Skonfiguruj workflow
Skonfiguruj workflow GitHub Actions do budowania, testowania i przesyłania obrazu na Docker Hub.

Przejdź do swojego repozytorium na GitHubie, a następnie wybierz zakładkę Actions.

Wybierz set up a workflow yourself.

To przeniesie Cię do strony, na której będziesz mógł stworzyć nowy plik workflow GitHub Actions w swoim repozytorium, domyślnie w lokalizacji .github/workflows/main.yml.

W oknie edytora, skopiuj i wklej poniższą konfigurację YAML:


name: ci

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and test
        uses: docker/build-push-action@v6
        with:
          target: test
          load: true

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          target: prod
          tags: ${{ vars.DOCKER_USERNAME }}/${{ github.event.repository.name }}:latest
Aby uzyskać więcej informacji na temat składni YAML dla docker/build-push-action, zapoznaj się z dokumentacją GitHub Action.

Krok trzeci: Uruchom workflow
Zapisz plik workflow i uruchom zadanie.

Wybierz Commit changes... i prześlij zmiany do gałęzi main.

Po wysłaniu zatwierdzenia, workflow rozpocznie się automatycznie.

Przejdź do zakładki Actions. Zobaczysz tam wykres przedstawiający postęp workflow.

Po zakończeniu workflow, przejdź do swojego repozytorium na Docker Hub.

Jeśli widzisz nowe repozytorium na tej liście, oznacza to, że GitHub Actions pomyślnie przesłał obraz na Docker Hub.# DockerTest
