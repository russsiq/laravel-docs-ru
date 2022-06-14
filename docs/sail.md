# Laravel 9 · Пакет Laravel Sail

- [Введение](#introduction)
- [Установка и настройка](#installation)
    - [Установка Sail в существующие приложения](#installing-sail-into-existing-applications)
    - [Настройка псевдонима Bash](#configuring-a-bash-alias)
- [Запуск и остановка Sail](#starting-and-stopping-sail)
- [Выполнение команд](#executing-sail-commands)
    - [Выполнение команд PHP](#executing-php-commands)
    - [Выполнение команд Composer](#executing-composer-commands)
    - [Выполнение команд Artisan](#executing-artisan-commands)
    - [Выполнение команд Node / NPM](#executing-node-npm-commands)
- [Взаимодействие с базами данных](#interacting-with-sail-databases)
    - [Взаимодействие с MySQL](#mysql)
    - [Взаимодействие с Redis](#redis)
    - [Взаимодействие с MeiliSearch](#meilisearch)
- [Файловое хранилище](#file-storage)
- [Запуск тестов](#running-tests)
    - [Использование Laravel Dusk в Sail](#laravel-dusk)
- [Предварительный просмотр писем](#previewing-emails)
- [Контейнер CLI](#sail-container-cli)
- [Выбор версии PHP](#sail-php-versions)
- [Выбор версии Node](#sail-node-versions)
- [Совместный доступ к вашему сайту](#sharing-your-site)
- [Отладка с помощью Xdebug](#debugging-with-xdebug)
  - [Использование Xdebug CLI](#xdebug-cli-usage)
  - [Использование Xdebug Browser](#xdebug-browser-usage)
- [Изменение конфигурационных файлов Docker](#sail-customization)

<a name="introduction"></a>
## Введение

[Laravel Sail](https://github.com/laravel/sail) – это легкий интерфейс командной строки для взаимодействия со средой разработки Docker для Laravel. Sail обеспечивает отличную отправную точку для создания приложения Laravel с использованием PHP, MySQL и Redis без предварительного опыта работы с Docker.

По сути, Sail – это файл `docker-compose.yml` и сценарий `sail`, который хранится в корне вашего проекта. Сценарий `sail` предоставляет интерфейс командной строки с удобными методами для взаимодействия с контейнерами Docker, определенными файлом `docker-compose.yml`.

Laravel Sail поддерживается в macOS, Linux и Windows (через [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Установка и настройка

Laravel Sail автоматически устанавливается со всеми новыми приложениями Laravel, поэтому вы можете сразу же начать его использовать. Чтобы узнать, как создать новое приложение Laravel, обратитесь к [документации по установке](installation.md) Laravel для вашей операционной системы. Во время установки вам будет предложено выбрать, с какими службами, поддерживаемыми Sail, ваше приложение будет взаимодействовать.

<a name="installing-sail-into-existing-applications"></a>
### Установка Sail в существующие приложения

Если вы заинтересованы в использовании Sail в существующем приложением Laravel, то вы можете просто установить Sail с помощью менеджера пакетов Composer. Конечно, эти шаги предполагают, что ваша существующая локальная среда разработки позволяет вам устанавливать зависимости Composer:

```shell
composer require laravel/sail --dev
```

После установки Sail вы можете запустить команду `sail:install` Artisan. Эта команда опубликует файл `docker-compose.yml` Sail в корне вашего приложения:

```shell
php artisan sail:install
```

Наконец, вы можете запустить Sail. Чтобы продолжить изучение использования Sail, продолжайте читать оставшуюся часть этой документации:

```shell
./vendor/bin/sail up
```

<a name="using-devcontainers"></a>
#### Использование Devcontainer

Если вы хотите разработать в [Devcontainer](https://code.visualstudio.com/docs/remote/containers), то вы можете указать флаг `--devcontainer` для команды `sail:install`. Флаг `--devcontainer` проинструктирует команду `sail:install` опубликовать файл `.devcontainer/devcontainer.json` в корне вашего приложения:

```shell
php artisan sail:install --devcontainer
```

<a name="configuring-a-bash-alias"></a>
### Настройка псевдонима Bash

По умолчанию команды Sail вызываются с помощью скрипта `vendor/bin/sail`, который включен во все новые приложения Laravel:

```shell
./vendor/bin/sail up
```

Но вместо многократно повторяющегося набора `vendor/bin/sail` для выполнения команд Sail, вы можете задать псевдоним Bash, который позволит вам легче выполнять команды Sail:

```shell
alias sail='[ -f sail ] && bash sail || bash vendor/bin/sail'
```

После задания псевдонима Bash вы можете выполнять команды Sail, просто набрав `sail`. В остальных примерах документации предполагается, что вы настроили этот псевдоним:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Запуск и остановка Sail

Файл `docker-compose.yml` Laravel Sail определяет множество работающих вместе контейнеров Docker, которые помогут вам создавать приложения Laravel. Каждый из этих контейнеров является записью в конфигурации `services` вашего файла `docker-compose.yml`. Контейнер `laravel.test` – это основной контейнер приложения, который будет обслуживать ваше приложение.

Перед запуском Sail вы должны убедиться, что на вашем локальном компьютере не работают другие веб-серверы или базы данных. Чтобы запустить все контейнеры Docker, определенные в файле `docker-compose.yml` вашего приложения, вы должны выполнить команду `up`:

```shell
sail up
```

Чтобы запустить все контейнеры Docker в фоновом режиме, вы можете запустить Sail в режиме `detached`:

```shell
sail up -d
```

После запуска контейнеров приложения вы можете получить доступ к проекту в своем веб-браузере по адресу: http://localhost.

Чтобы остановить все контейнеры, вы можете просто нажать <kbd>Control + C</kbd>, чтобы остановить их выполнение. Или, если контейнеры работают в фоновом режиме, вы можете использовать команду `down`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Выполнение команд

При использовании Laravel Sail ваше приложение выполняется в контейнере Docker и изолировано от вашего локального компьютера. Однако Sail предоставляет удобный способ запускать различные команды для вашего приложения, такие как произвольные команды PHP, команды Artisan, команды Composer и команды Node / NPM.

**При чтении документации Laravel вы часто увидите ссылки на команды Composer, Artisan и Node / NPM, которые не ссылаются на Sail**. В этих примерах предполагается, что эти инструменты установлены на вашем локальном компьютере. Если вы используете Sail для своей локальной среды разработки Laravel, то вам следует выполнить эти команды с помощью Sail:

```shell
# Локальный запуск команд Artisan ...
php artisan queue:work

# Запуск команд Artisan в Laravel Sail ...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Выполнение команд PHP

Команды PHP могут быть выполнены с помощью команды `php`. Конечно, эти команды будут выполняться с использованием версии PHP вашего приложения. Чтобы узнать больше о версиях PHP, доступных для Laravel Sail, обратитесь к разделу [Выбор версии PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Выполнение команд Composer

Команды Composer могут быть выполнены с помощью команды `composer`. Контейнер приложения Laravel Sail содержит Composer 2.x:

```shell
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### Установка зависимостей Composer для существующих приложений

Существует вероятность, что при разработке в команде, вы не будете являться создателем приложения на Laravel. Следовательно, что ни одна из зависимостей Composer в приложении, включая рассматриваемый Sail, не будет установлена после клонирования репозитория приложения на локальный компьютер.

Вы можете установить зависимости приложения, перейдя в каталог приложения и выполнив следующую команду. Эта команда использует небольшой контейнер Docker, содержащий PHP и Composer, для установки зависимостей приложения:

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v $(pwd):/var/www/html \
    -w /var/www/html \
    laravelsail/php81-composer:latest \
    composer install --ignore-platform-reqs
```

При использовании образа `laravelsail/phpXX-composer` вам следует использовать ту же версию PHP, которую вы планируете использовать для своего приложения (`74`, `80` или `81`).

<a name="executing-artisan-commands"></a>
### Выполнение команд Artisan

Команды Artisan могут быть выполнены с помощью команды `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Выполнение команд Node / NPM

Команды Node могут быть выполнены с помощью команды `node`, в то время как команды NPM могут быть выполнены с помощью команды `npm`:

```shell
sail node --version

sail npm run prod
```

Если вы хотите, то можете использовать Yarn вместо NPM:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Взаимодействие с базами данных

<a name="mysql"></a>
### Взаимодействие с MySQL

Как вы могли заметить, файл `docker-compose.yml` вашего приложения содержит запись для контейнера MySQL. Этот контейнер использует [Docker volume](https://docs.docker.com/storage/volumes/), чтобы данные, хранящиеся в вашей базе данных, сохранялись даже при остановке и перезапуске ваших контейнеров.

Кроме того, при первом запуске контейнер MySQL создаст для вас две базы данных. Первая база данных будет названа с использованием значения переменной окружения `DB_DATABASE` и предназначена для вашей локальной разработки. Вторая — это база данных, предназначенная для тестирования и, именуемая как `testing`, которая гарантирует, что ваши тесты не будут мешать вашим данным при разработки.

После того, как вы запустили свои контейнеры, вы можете подключиться к MySQL в своем приложении, установив для переменной окружения `DB_HOST` значение `mysql` в файле `.env` вашего приложения.

Чтобы подключиться к базе данных MySQL вашего приложения с локального компьютера, вы можете использовать графическое приложение для управления базами данных, такое как [TablePlus](https://tableplus.com). По умолчанию база данных MySQL доступна на `localhost` с портом `3306`.

<a name="redis"></a>
### Взаимодействие с Redis

Файл docker-compose.yml вашего приложения также содержит запись для контейнера [Redis](https://redis.io). Этот контейнер использует [Docker volume](https://docs.docker.com/storage/volumes/), чтобы данные, хранящиеся в вашей базе данных Redis, сохранялись даже при остановке и перезапуске ваших контейнеров. После того, как вы запустили свои контейнеры, вы можете подключиться к Redis в своем приложении, установив для переменной окружения `REDIS_HOST` значение `redis` в файле `.env` вашего приложения.

Чтобы подключиться к базе данных Redis вашего приложения с локального компьютера, вы можете использовать графическое приложение для управления базами данных, такое как [TablePlus](https://tableplus.com). По умолчанию база данных Redis доступна на `localhost` с портом `6379`.

<a name="meilisearch"></a>
### Взаимодействие с MeiliSearch

Если вы выбрали [MeiliSearch](https://www.meilisearch.com) при установке Sail, то файл `docker-compose.yml` вашего приложения будет содержать запись для этой мощной поисковой системы, которая [совместима](https://github.com/meilisearch/meilisearch-laravel-scout) с [Laravel Scout](scout.md). После того, как вы запустили свои контейнеры, вы можете подключиться к MeiliSearch в своем приложении, установив для переменной окружения `MEILISEARCH_HOST` значение `http://meilisearch:7700`.

Со своего локального компьютера вы можете получить доступ к веб-панели администрирования MeiliSearch, перейдя по адресу `http://localhost:7700` в своем браузере.

<a name="file-storage"></a>
## Файловое хранилище

Если вы планируете использовать Amazon S3 для хранения файлов при запуске приложения в эксплуатационном окружении, то вы можете выбрать [MinIO](https://min.io) при установке Sail. MinIO предоставляет API, совместимый с S3, который вы можете использовать для локальной разработки с помощью драйвера `s3` afqkjdjuj хранилища Laravel без создавая «тестовых» корзин в эксплуатационном окружении S3. Если вы выберете MinIO при установке Sail, то раздел конфигурации MinIO будет добавлен в файл `docker-compose.yml` вашего приложения.

По умолчанию конфигурационный файл `filesystems` вашего приложения уже содержит конфигурацию для диска `s3`. Помимо использования этого диска для взаимодействия с Amazon S3, вы можете использовать его для взаимодействия с любой S3-совместимой службой хранения файлов, такой как MinIO, путем простого изменения соответствующих переменных окружения, которые управляют его конфигурацией. Например, при использовании MinIO, конфигурация переменной окружения файловой системы должна быть определена следующим образом:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## Запуск тестов

Laravel provides amazing testing support out of the box, and you may use Sail's `test` command to run your applications [feature and unit tests](testing.md). Any CLI options that are accepted by PHPUnit may also be passed to the `test` command:

```shell
sail test

sail test --group orders
```

The Sail `test` command is equivalent to running the `test` Artisan command:

```shell
sail artisan test
```

По умолчанию Sail создаст специальную базу данных `testing`, чтобы ваши тесты не мешали текущему состоянию вашей базы данных. При установке Laravel по умолчанию Sail также настроит ваш файл `phpunit.xml` для использования этой базы данных при выполнении ваших тестов:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Использование Laravel Dusk в Sail

[Laravel Dusk](dusk.md) предоставляет выразительный, простой в использовании API тестирования с использованием браузера. Благодаря Sail вы можете запускать эти тесты, даже не устанавливая Selenium или другие инструменты на свой локальный компьютер. Для того, чтобы начать работу, раскомментируйте службу Selenium в файле `docker-compose.yml` приложения:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Затем убедитесь, что служба `laravel.test` в файле `docker-compose.yml` вашего приложения имеет запись `selenium` в `depends_on`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

Наконец, вы можете запустить свой набор тестов Dusk, запустив Sail с командой `dusk`:

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Selenium на Apple Silicon

Если ваш локальный компьютер содержит микросхему Apple Silicon, то ваша служба `selenium` должна использовать образ `seleniarm/standalone-chromium`:

```yaml
selenium:
    image: 'seleniarm/standalone-chromium'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## Предварительный просмотр писем

По умолчанию файл `docker-compose.yml` Laravel Sail содержит служебную запись для [MailHog](https://github.com/mailhog/MailHog). MailHog перехватывает электронные письма, отправляемые вашим приложением во время локальной разработки, и предоставляет удобный веб-интерфейс, чтобы вы могли просматривать свои электронные письма в браузере. При использовании Sail хостом по умолчанию для MailHog является `mailhog` и он доступен через порт `1025`:

```ini
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Когда Sail запущен, вы можете получить доступ к веб-интерфейсу MailHog по адресу: http://localhost:8025

<a name="sail-container-cli"></a>
## Контейнер CLI

По желанию можно запустить сеанс Bash в контейнере вашего приложения. Вы можете использовать команду `shell` для подключения к контейнеру вашего приложения, что позволяет проверять файлы и установленные службы, а также выполнять произвольные команды оболочки внутри контейнера:

```shell
sail shell

sail root-shell
```

Чтобы начать новый сеанс [Laravel Tinker](https://github.com/laravel/tinker), вы можете выполнить команду `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## Выбор версии PHP

В настоящее время Sail поддерживает обслуживание вашего приложения с использованием PHP версий 8.1 / 8.0 / 7.4. Версия PHP, используемая по умолчанию Sail, в настоящее время – 8.1. Чтобы изменить версию PHP, которая используется для обслуживания вашего приложения, вы должны обновить определение `build` контейнера `laravel.test` в файле `docker-compose.yml` вашего приложения:

```yaml
# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0

# PHP 7.4
context: ./vendor/laravel/sail/runtimes/7.4
```

Кроме того, вы можете изменить `image`, чтобы отразить версию PHP, используемую вашим приложением. Этот параметр также определен в файле `docker-compose.yml` вашего приложения:

```yaml
image: sail-8.1/app
```

После обновления файла `docker-compose.yml` вашего приложения вы должны перестроить образы контейнеров:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Выбор версии Node

Sail по умолчанию устанавливает Node 16. Чтобы изменить версию Node, установленную при создании образов, вы можете обновить определение `build.args` службы `laravel.test` в файле `docker-compose.yml` вашего приложения:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '14'
```

После обновления файла `docker-compose.yml` вашего приложения вы должны перестроить образы контейнеров:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Совместный доступ к вашему сайту

Иногда требуется предоставить общий доступ к своему сайту для его предварительного просмотра коллеге или для проверки интеграции веб-перехватчиков с вашим приложением. Чтобы поделиться своим сайтом, вы можете использовать команду `share`. После выполнения этой команды вам будет выдан случайный URL-адрес `laravel-sail.site`, который вы можете использовать для доступа к своему приложению:

```shell
sail share
```

При совместном доступе к вашему сайту с помощью команды `share` вы должны настроить доверенные прокси вашего приложения в посреднике `TrustProxies`. В противном случае глобальные помощники генерации URL, такие как `url` ​​и `route`, не смогут определить правильный хост HTTP, который следует использовать во время генерации URL:

    /**
     * Доверенные прокси этого приложения.
     *
     * @var array|string|null
     */
    protected $proxies = '*';

Если вы хотите выбрать поддомен для вашего сайта, то вы можете указать опцию `subdomain` при выполнении команды `share`:

```shell
sail share --subdomain=my-sail-site
```

> {tip} Команда `share` поддерживается [Expose](https://github.com/beyondcode/expose), службой туннелирования с открытым исходным кодом от [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Отладка с помощью Xdebug

Конфигурация Docker для Laravel Sail включает поддержку [Xdebug](https://xdebug.org/), популярного и мощного отладчика для PHP. Чтобы задействовать Xdebug, вам нужно будет добавить несколько переменных [конфигурации Xdebug](https://xdebug.org/docs/step_debug#mode) в файл `.env` вашего приложения. Чтобы задействовать Xdebug, вы должны установить соответствующие режимы перед запуском Sail:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### Конфигурация IP-адреса хоста Linux

Внутренне переменная окружения `XDEBUG_CONFIG` определяется как `client_host=host.docker.internal`, так что Xdebug будет правильно настроен для Mac и Windows (WSL2). Если на вашем локальном компьютере работает Linux, убедитесь, что вы используете Docker Engine 17.06.0+ и Compose 1.16.0+. В противном случае вам нужно будет вручную определить эту переменную окружения, как показано ниже.

Во-первых, вы должны определить правильный IP-адрес хоста для добавления в переменную окружения, выполнив следующую команду. Как правило, `<container-name>` должно быть именем контейнера, который обслуживает ваше приложение, и чаще всего заканчиваться на `_laravel.test_1`:

```shell
docker inspect -f {{range.NetworkSettings.Networks}}{{.Gateway}}{{end}} <container-name>
```

После того, как вы получили правильный IP-адрес хоста, вы должны определить переменную `SAIL_XDEBUG_CONFIG` в файле `.env` вашего приложения:

```ini
SAIL_XDEBUG_CONFIG="client_host=<host-ip-address>"
```

<a name="xdebug-cli-usage"></a>
### Использование Xdebug CLI

Команда `sail debug` может быть использована для запуска сеанса отладки при запуске команды Artisan:

```shell
# Запустить команды Artisan без Xdebug ...
sail artisan migrate

# Запустить команды Artisan с Xdebug ...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Использование Xdebug Browser

Для отладки приложения при взаимодействии с приложением через веб-браузер следуйте [инструкциям Xdebug](https://xdebug.org/docs/step_debug#web-application) для запуска сеанса Xdebug из веб-браузера.

Если вы используете PhpStorm, то просмотрите документацию JetBrain относительно [отладки с нулевой конфигурацией](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> {note} Laravel Sail полагается на `artisan serve` для обслуживания вашего приложения. Команда `artisan serve` принимает только переменные `XDEBUG_CONFIG` и `XDEBUG_MODE` начиная с версии Laravel 8.53.0. Старые версии Laravel (8.52.0 и ниже) не поддерживают эти переменные и не принимают отладочные соединения.

<a name="sail-customization"></a>
## Изменение конфигурационных файлов Docker

Поскольку Sail – это просто Docker, то вы можете настроить почти все в нем. Чтобы опубликовать Docker-файлы Sail, вы можете выполнить команду `sail:publish`:

```shell
sail artisan sail:publish
```

После запуска этой команды файлы Docker и другие конфигурационные файлы, используемые Laravel Sail, будут помещены в каталог `docker` корневого каталога вашего приложения. После настройки Sail вы можете изменить имя образа для контейнера приложения в файле `docker-compose.yml` вашего приложения. После этого пересоберите контейнеры вашего приложения с помощью команды `build`. Присвоение уникального имени образу приложения особенно важно, если вы используете Sail для разработки нескольких приложений Laravel на одном компьютере:

```shell
sail build --no-cache
```
