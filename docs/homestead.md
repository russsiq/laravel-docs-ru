# Laravel 9 · Пакет Laravel Homestead

- [Введение](#introduction)
- [Установка и настройка](#installation-and-setup)
    - [Первые шаги](#first-steps)
    - [Конфигурирование Homestead](#configuring-homestead)
    - [Конфигурирование сайтов Nginx](#configuring-nginx-sites)
    - [Конфигурирование служб](#configuring-services)
    - [Запуск образа Vagrant](#launching-the-vagrant-box)
    - [Индивидуальная установка для проекта](#per-project-installation)
    - [Установка дополнительного программного обеспечения](#installing-optional-features)
    - [Псевдонимы команд](#aliases)
- [Обновление Homestead](#updating-homestead)
- [Повседневное использование](#daily-usage)
    - [Подключение через SSH](#connecting-via-ssh)
    - [Добавление дополнительных сайтов](#adding-additional-sites)
    - [Переменные окружения](#environment-variables)
    - [Порты](#ports)
    - [Версии PHP](#php-versions)
    - [Подключение к базам данных](#connecting-to-databases)
    - [Резервное копирование базы данных](#database-backups)
    - [Конфигурирование расписаний Cron](#configuring-cron-schedules)
    - [Конфигурирование MailHog](#configuring-mailhog)
    - [Конфигурирование Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [Совместный доступ к вашему окружению](#sharing-your-environment)
- [Отладка и профилирование](#debugging-and-profiling)
    - [Отладка веб-запросов с помощью Xdebug](#debugging-web-requests)
    - [Отладка консольных приложений](#debugging-cli-applications)
    - [Профилирование приложений с Blackfire](#profiling-applications-with-blackfire)
- [Сетевые интерфейсы](#network-interfaces)
- [Расширение Homestead](#extending-homestead)
- [Настройки, специфичные для провайдера](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Введение

Laravel стремится сделать весь процесс разработки на PHP приятным, и ваша локальная среда разработки не является исключением. [Laravel Homestead](https://github.com/laravel/homestead) – это официальный предварительно упакованный образ Vagrant, который обеспечит окружение без требований установки PHP, веб-сервера и любого другого серверного программного обеспечения на вашем локальном компьютере.

[Vagrant](https://www.vagrantup.com) предлагает простой и элегантный способ подготовки и управления виртуальными машинами. Образы Vagrant полностью «одноразовые». Если что-то пойдет не так, вы можете уничтожить и воссоздать образ за считанные минуты!

Homestead работает в любой системе: Windows, macOS или Linux и содержит Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, а также другое программное обеспечение, необходимое для разработки потрясающих приложений Laravel.

> {note} Если вы используете Windows, то вам может потребоваться включить аппаратную виртуализацию (VT-x). Обычно это можно выполнить в BIOS. Если вы используете Hyper-V в системе UEFI, то вам может дополнительно потребоваться отключить Hyper-V, чтобы получить доступ к VT-x.

<a name="included-software"></a>
### Прилагаемое программное обеспечение

<!-- <style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style> -->

<!-- <div id="software-list" markdown="1"> -->

- Ubuntu 20.04
- Git
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 13
- Composer
- Node (включая Yarn, Bower, Grunt и Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

<!-- </div> -->

<a name="optional-software"></a>
### Дополнительное программное обеспечение

<!-- <style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style> -->

<!-- <div id="software-list" markdown="1"> -->

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Фреймворк Lucky (Crystal)
- Docker
- Elasticsearch
- EventStoreDB
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- RVM (Менеджер версий Ruby)
- Solr
- TimescaleDB
- Trader <small>(расширение PHP)</small>
- Утилиты Webdriver и Laravel Dusk

<!-- </div> -->

<a name="installation-and-setup"></a>
## Установка и настройка

<a name="first-steps"></a>
### Первые шаги

Перед запуском среды Homestead необходимо установить [Vagrant](https://www.vagrantup.com/downloads.html), а также одного из следующих поддерживаемых провайдеров:

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Downloads)
- [Parallels](https://www.parallels.com/products/desktop/)

Все эти программные пакеты предоставляют простые в использовании визуальные установщики для всех популярных операционных систем.

Чтобы использовать провайдер Parallels, вам необходимо установить [плагин Vagrant Parallels](https://github.com/Parallels/vagrant-parallels). Это бесплатно.

<a name="installing-homestead"></a>
#### Установка Homestead

Вы можете установить Homestead, клонировав репозиторий Homestead на свой хост-компьютер. Рассмотрите возможность клонирования репозитория в папку `Homestead` вашего «домашнего» каталога, поскольку виртуальная машина Homestead будет служить хостом для всех ваших приложений Laravel. В этой документации мы будем называть этот каталог вашим «каталогом Homestead»:

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

После клонирования репозитория Laravel Homestead вы должны проверить ветку `release`. Эта ветка всегда содержит последний стабильный релиз Homestead:

```shell
cd ~/Homestead

git checkout release
```

Затем выполните команду `bash init.sh` из каталога Homestead, чтобы создать конфигурационный файл `Homestead.yaml`. Файл `Homestead.yaml` – это то место, где вы настраиваете все параметры установки Homestead. Этот файл будет размещен в каталоге Homestead:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### Конфигурирование Homestead

<a name="setting-your-provider"></a>
#### Настройка вашего провайдера

Параметр `provider` файла `Homestead.yaml` указывает Vagrant, какой провайдер ему следует использовать: `virtualbox` или `parallels`:

    provider: virtualbox

> {note} Если вы используете Apple Silicon, то вам следует добавить `box: laravel/homestead-arm` в ваш файл `Homestead.yaml`. Для Apple Silicon требуется провайдер Parallels.

<a name="configuring-shared-folders"></a>
#### Конфигурирование общих папок

Параметр `folder` файла `Homestead.yaml` позволяет перечислить все общедоступные папки окружения Homestead. Изменения файлов в этих папках будут синхронизированы между вашим локальным компьютером и виртуальной средой Homestead. Вы можете указать любое количество общих папок:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> {note} Пользователи Windows не должны использовать синтаксис пути `~/`, вместо этого необходимо использовать полный путь к своему проекту, например, `C:\Users\user\Code\project1`.

Вы всегда должны индивидуально сопоставлять каждое приложение с собственной папкой приложения вместо сопоставления одного большого каталога, содержащего все ваши приложения. При сопоставлении папки виртуальная машина должна отслеживать все операции ввода-вывода диска для *каждого* файла в папке. При наличии множества файлов в папке, может возникнуть снижение производительности:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> {note} Вы никогда не должны монтировать текущий каталог `.` при использовании Homestead. Это приводит к тому, что Vagrant не отображает текущую папку в `/vagrant`, что нарушает работу дополнительных функций и приводит к неожиданным результатам во время подготовки.

Чтобы задействовать [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), вы можете добавить параметр `type` при сопоставлении папок:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> {note} При использовании NFS в Windows вам следует рассмотреть возможность установки плагина [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Этот плагин будет поддерживать корректные разрешения пользователя / группы для файлов и каталогов виртуальной машины Homestead.

Вы также можете передать любые параметры, поддерживаемые [синхронизируемыми папками](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) Vagrant, указав их под параметром `options`:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

<a name="configuring-nginx-sites"></a>
### Конфигурирование сайтов Nginx

Не знакомы с Nginx? Без проблем. Параметр `sites` вашего файла `Homestead.yaml` позволяет вам легко сопоставить «домен» с папкой в ​​окружении Homestead. Пример конфигурации сайта содержится в файле `Homestead.yaml`. Опять же, вы можете добавить столько сайтов в ​​окружение Homestead, сколько необходимо. Homestead может служить удобной виртуализированной средой для каждого приложения Laravel, над которым вы работаете:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

Если вы измените параметр `sites` после подготовки виртуальной машины Homestead, то вы должны выполнить команду `vagrant reload --provision` в своем терминале, чтобы обновить конфигурацию Nginx на виртуальной машине.

> {note} Скрипты Homestead созданы максимально идемпотентными. Однако, если у вас возникли проблемы во время подготовки, то вам следует уничтожить и пересоздать машину, выполнив команду `vagrant destroy && vagrant up`.

<a name="hostname-resolution"></a>
#### Разрешение имени хоста

Homestead публикует имена хостов, используя `mDNS` для автоматического разрешения хостов. Если вы укажите `hostname: homestead` в файле `Homestead.yaml`, то хост будет доступен по адресу `homestead.local`. Настольные дистрибутивы macOS, iOS и Linux по умолчанию содержат поддержку `mDNS`. Если вы используете Windows, то вы должны установить [Bonjour Print Services](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US) для Windows.

Использование автоматических имен хостов лучше всего подходит для [индивидуальных установок](#per-project-installation) Homestead под каждый проект. Если вы размещаете несколько сайтов на одном экземпляре Homestead, то вы можете добавить «домены» для своих веб-сайтов в файле `hosts` на вашем компьютере. Файл `hosts` будет перенаправлять запросы для ваших сайтов Homestead на вашу виртуальную машину Homestead. В macOS и Linux этот файл находится в `/etc/hosts`. В Windows он находится в `C:\Windows\System32\drivers\etc\hosts`. Строки, добавляемые вами в этот файл, будут выглядеть следующим образом:

    192.168.56.56  homestead.test

Убедитесь, что в списке указан IP-адрес, указанный в вашем файле `Homestead.yaml`. После того, как вы добавили домен в свой файл `hosts` и запустили окно Vagrant, вы сможете получить доступ к сайту через свой веб-браузер:

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### Конфигурирование служб

По умолчанию Homestead запускает несколько служб; однако вы можете указать, какие службы будут включены или отключены во время подготовки. Например, вы можете включить PostgreSQL и отключить MySQL, изменив параметр `services` в файле `Homestead.yaml`:

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

Указанные службы будут запущены или остановлены в зависимости от их порядка в директивах `enabled` и `disabled`.

<a name="launching-the-vagrant-box"></a>
### Запуск образа Vagrant

После редактирования файла `Homestead.yaml` по своему вкусу, запустите команду `vagrant up` из каталога Homestead. Vagrant загрузит виртуальную машину и автоматически настроит ваши общие папки и сайты Nginx.

Чтобы уничтожить машину, вы можете использовать команду `vagrant destroy`.

<a name="per-project-installation"></a>
### Индивидуальная установка для проекта

Вместо того, чтобы устанавливать Homestead глобально и использовать одну и ту же виртуальную машину Homestead для всех ваших проектов, вы можете настроить экземпляр Homestead для каждого проекта, которым вы управляете. Установка Homestead для каждого проекта может быть полезной, если вы хотите отправить `Vagrantfile` вместе с вашим проектом, позволяя другим разработчикам выполнить `vagrant up` сразу после клонирования репозитория проекта.

Для начала установите Homestead с помощью менеджера пакетов Composer в свой проект:

```shell
composer require laravel/homestead --dev
```

Как только Homestead будет установлен, вызовите команду `make` Homestead, чтобы сгенерировать файлы `Vagrantfile` и `Homestead.yaml` для вашего проекта. Эти файлы будут помещены в корень вашего проекта. Команда `make` автоматически настроит параметры `sites` и `folder` в файле `Homestead.yaml`:

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

Запустите команду `vagrant up` в терминале и перейдите по адресу `http://homestead.test` в вашем браузере. Помните, что вам также нужно будет добавить запись в файле `/etc/hosts` для домена `homestead.test` или любого другого необходимого домена, если вы не используете автоматическое [разрешение имени хоста](#hostname-resolution).

<a name="installing-optional-features"></a>
### Установка дополнительного программного обеспечения

Дополнительное программное обеспечение устанавливается с помощью параметра `features` в вашем файле `Homestead.yaml`. Большую часть ПО можно включить или отключить с помощью логического значения, но существует ПО, позволяющее использовать несколько параметров конфигурации:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - docker: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

<a name="elasticsearch"></a>
#### Elasticsearch

Вы можете указать поддерживаемую версию Elasticsearch, которая должна быть точным номером версии в формате `major.minor.patch`. При установке по умолчанию будет создан кластер с именем `homestead`. Вы никогда не должны выделять для Elasticsearch более половины памяти операционной системы, поэтому убедитесь, что на вашей виртуальной машине Homestead выделено как минимум вдвое больше памяти Elasticsearch.

> {tip} Ознакомьтесь с [документацией Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current), чтобы узнать, как настроить свою конфигурацию.

<a name="mariadb"></a>
#### MariaDB

Включение MariaDB удалит MySQL и установит MariaDB. MariaDB обычно служит заменой MySQL, поэтому вам все равно следует использовать драйвер базы данных `mysql` в конфигурации базы данных вашего приложения.

<a name="mongodb"></a>
#### MongoDB

При установке MongoDB для имени пользователя БД будет установлено значение `homestead` по умолчанию, а для пароля – `secret`.

<a name="neo4j"></a>
#### Neo4j

При установке Neo4j для имени пользователя БД будет установлено значение `homestead` по умолчанию, а для пароля – `secret`. Чтобы получить доступ к Neo4j, перейдите по адресу `http://homestead.test:7474` в вашем браузере. Порты `7687` (Bolt), `7474` (HTTP) и `7473` (HTTPS) готовы обслуживать запросы от клиента Neo4j.

<a name="aliases"></a>
### Псевдонимы команд

Вы можете добавить псевдонимы Bash для своей виртуальной машины Homestead, изменив файл `aliases` в каталоге Homestead:

```shell
alias c='clear'
alias ..='cd ..'
```

После обновления файла `aliases` вы должны повторно подготовить виртуальную машину Homestead с помощью команды `vagrant reload --provision`. Это обеспечит доступность ваших новых псевдонимов на машине.

<a name="updating-homestead"></a>
## Обновление Homestead

Перед обновлением Homestead, убедитесь, что вы уничтожили текущую виртуальную машину, выполнив следующую команду в каталоге Homestead:

```shell
vagrant destroy
```

Затем вам нужно обновить исходный код Homestead. Если вы клонировали репозиторий, то вы можете выполнить следующие команды в том месте, где вы изначально клонировали репозиторий:

```shell
git fetch

git pull origin release
```

С помощью этих команд будет подтянут последний код Homestead из репозитория GitHub, извлечены последние теги и будет выполнена проверка последнего релиза по тегам. Вы можете найти последнюю стабильную версию релиза Homestead на [странице GitHub](https://github.com/laravel/homestead/releases).

Если вы установили Homestead в свой проект с помощью Composer, то необходимо убедиться, что ваш файл `composer.json` содержит запись `"laravel/homestead": "^12"` с последующим обновлением ваших зависимостей:

```shell
composer update
```

Затем вы должны обновить образ Vagrant с помощью команды:

```shell
vagrant box update
```

После обновления образа Vagrant вы должны запустить команду `bash init.sh` из каталога Homestead, чтобы обновить дополнительные файлы конфигурации Homestead. При этом вас спросят, хотите ли вы перезаписать существующие файлы `Homestead.yaml`, `after.sh` и `aliases`:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

Наконец, вам нужно будет пересоздать вашу виртуальную машину Homestead, чтобы использовать последнюю установку Vagrant:

```shell
vagrant up
```

<a name="daily-usage"></a>
## Повседневное использование

<a name="connecting-via-ssh"></a>
### Подключение через SSH

Вы можете подключиться к вашей виртуальной машине через SSH, выполнив команду терминала `vagrant ssh` из вашего каталога Homestead.

<a name="adding-additional-sites"></a>
### Добавление дополнительных сайтов

После того, как ваша среда Homestead подготовлена ​​и запущена, вы можете добавить дополнительные сайты Nginx для других ваших проектов Laravel. Вы можете запускать столько проектов Laravel, сколько хотите, в одном окружении Homestead. Чтобы добавить дополнительный сайт, добавьте его в свой файл `Homestead.yaml`.

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> {note} Перед добавлением сайта убедитесь, что вы настроили [сопоставление папок](#configuring-shared-folders) для каталога проекта.

Если Vagrant не управляет вашим файлом `hosts` автоматически, то вам может потребоваться также добавить новый сайт в этот файл. В macOS и Linux этот файл находится в `/etc/hosts`. В Windows он находится в `C:\Windows\System32\drivers\etc\hosts`:

    192.168.56.56  homestead.test
    192.168.56.56  another.test

После добавления сайта выполните команду терминала `vagrant reload --provision` из каталога Homestead.

<a name="site-types"></a>
#### Типы сайтов

Homestead поддерживает несколько «типов» сайтов, которые позволяют легко запускать проекты, построенные не на Laravel. Например, мы можем легко добавить приложение Statamic в Homestead, используя тип сайта `statamic`:

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

Доступные типы сайтов: `apache`, `apigility`, `expressive`, `laravel` (по умолчанию), `proxy`, `silverstripe`, `statamic`, `symfony2`, `symfony4` и `zf`.

<a name="site-parameters"></a>
#### Параметры сайта

Вы можете добавить дополнительные значения параметра `fastcgi_param` Nginx на свой сайт с помощью директивы сайта `params`:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

<a name="environment-variables"></a>
### Переменные окружения

Вы можете определить глобальные переменные окружения, добавив их в свой файл `Homestead.yaml`:

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

После обновления файла `Homestead.yaml` не забудьте повторно подготовить машину, выполнив команду `vagrant reload --provision`. Это обновит конфигурацию PHP-FPM для всех установленных версий PHP, а также обновит окружение для пользователя `vagrant`.

<a name="ports"></a>
### Порты

По умолчанию в окружении Homestead перенаправляются следующие порты:

<!-- <div class="content-list" markdown="1"> -->

- **HTTP:** `8000` &rarr; `80`
- **HTTPS:** `44300` &rarr; `443`

<!-- </div> -->

<a name="forwarding-additional-ports"></a>
#### Перенаправление дополнительных портов

По желанию можно перенаправить дополнительные порты в образе Vagrant, указав конфигурационную запись `ports` в вашем файле `Homestead.yaml`. После обновления файла `Homestead.yaml` не забудьте повторно подготовить машину, выполнив команду `vagrant reload --provision`:

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

Ниже приведен список дополнительных портов служб Homestead в образе Vagrant, которые можно сопоставить со своей хост-машины:

<!-- <div class="content-list" markdown="1"> -->

- **SSH:** 2222 &rarr; 22
- **ngrok UI:** 4040 &rarr; 4040
- **MySQL:** 33060 &rarr; 3306
- **PostgreSQL:** 54320 &rarr; 5432
- **MongoDB:** 27017 &rarr; 27017
- **Mailhog:** 8025 &rarr; 8025
- **Minio:** 9600 &rarr; 9600

<!-- </div> -->

<a name="php-versions"></a>
### Версии PHP

В Homestead 6 появилась поддержка запуска нескольких версий PHP на одной виртуальной машине. Вы можете указать, какую версию PHP использовать для конкретного сайта в файле `Homestead.yaml`. Доступные версии PHP: «5.6», «7.0», «7.1», «7.2», «7.3», «7.4», «8.0» (по умолчанию) и «8.1»:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[На виртуальной машине Homestead](#connecting-via-ssh) вы можете использовать любую из поддерживаемых версий PHP через интерфейс командной строки:

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
```

Вы можете изменить версию PHP, используемую по умолчанию в CLI, выполнив следующие команды на своей виртуальной машине Homestead:

```shell
php56
php70
php71
php72
php73
php74
php80
php81
```

<a name="connecting-to-databases"></a>
### Подключение к базам данных

База данных `homestead` настроена из коробки как для MySQL, так и для PostgreSQL. Чтобы подключиться к вашей базе данных MySQL или PostgreSQL из клиента базы данных вашего хост-компьютера, вы должны подключиться к `127.0.0.1` через порт `33060` (MySQL) или `54320` (PostgreSQL). Имя пользователя и пароль для обеих баз данных – `homestead` / `secret`.

> {note} Вы должны использовать эти нестандартные порты только при подключении к базам данных с вашего хост-компьютера. Вы будете использовать по умолчанию порты `3306` и `5432` в конфигурационном файле `database` вашего приложения Laravel, поскольку Laravel работает _внутри_ виртуальной машины.

<a name="database-backups"></a>
### Резервное копирование базы данных

Homestead может автоматически создавать резервную копию вашей базы данных при уничтожении вашей виртуальной машины Homestead. Чтобы использовать этот функционал, вы должны использовать Vagrant версии 2.1.0 или выше. Если вы используете старую версию Vagrant, то вы должны установить плагин `vagrant-triggers`. Чтобы включить автоматическое резервное копирование базы данных, добавьте следующую строку в ваш файл `Homestead.yaml`:

    backup: true

Теперь Homestead будет экспортировать ваши базы данных в каталоги `.backup/mysql_backup` и `.backup/postgres_backup` при выполнении команды `vagrant destroy`. Эти каталоги можно найти в папке, в которую вы установили Homestead, или в корне вашего проекта, если вы используете метод [индивидуальных установок](#per-project-installation) Homestead под каждый проект.

<a name="configuring-cron-schedules"></a>
### Конфигурирование расписаний Cron

Laravel предоставляет удобный способ [планирования заданий Cron](scheduling.md) с ежеминутным запуском с помощью одной команды `schedule:run` Artisan. Команда `schedule:run` проверяет расписание заданий вашего класса `App\Console\Kernel`, чтобы определить, какие запланированные задачи нужно запустить.

Если вы хотите, чтобы команда `schedule:run` запускалась для сайта Homestead, то вы можете установить для параметра `schedule` значение `true` при определении сайта:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

Задание Cron для сайта будет определено в каталоге `/etc/cron.d` виртуальной машины Homestead.

<a name="configuring-mailhog"></a>
### Конфигурирование MailHog

[MailHog](https://github.com/mailhog/MailHog) позволяет вам перехватывать исходящую электронную почту без отправки ее получателям. Для начала обновите файл `.env` вашего приложения, чтобы использовать следующие настройки почты:

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

После этой настройки вы можете получить доступ к панели управления MailHog по адресу `http://localhost:8025`.

<a name="configuring-minio"></a>
### Конфигурирование Minio

[Minio](https://github.com/minio/minio) – это сервер хранения объектов с открытым исходным кодом и API, совместимым с Amazon S3. Чтобы установить Minio, обновите файл `Homestead.yaml`, указав следующий параметр конфигурации в разделе [`features`](#installing-optional-features):

    minio: true

По умолчанию Minio доступен через порт `9600`. Вы можете получить доступ к панели управления Minio, перейдя по адресу `http://localhost:9600`. Ключ доступа и секретный ключ по умолчанию – `homestead` / `secretkey`. При доступе к Minio вы всегда должны использовать регион `us-east-1`.

Чтобы использовать Minio, вам нужно будет настроить конфигурацию диска S3 в конфигурационном файле `config/filesystems.php` вашего приложения. Вам нужно будет добавить параметр `use_path_style_endpoint` в конфигурацию диска, а также изменить параметр `url` на `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true,
    ]

Наконец, убедитесь, что в вашем файле `.env` прописаны следующие параметры:

```ini
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

Чтобы подготовить корзину S3 на базе Minio, добавьте параметр `buckets` в ваш файл `Homestead.yaml`. После определения ваших корзин вы должны выполнить команду `vagrant reload --provision` в своем терминале:

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Поддерживаемые значения `policy`: `none`, `download`, `upload` и `public`.

<a name="laravel-dusk"></a>
### Laravel Dusk

Чтобы запустить тесты [Laravel Dusk](dusk.md) в Homestead, вы должны задействовать функционал [`webdriver`](#installing-optional-features) в вашей конфигурации Homestead:

```yaml
features:
    - webdriver: true
```

После определения параметра `webdriver` вы должны выполнить команду `vagrant reload --provision` в своем терминале.

<a name="sharing-your-environment"></a>
### Совместный доступ к вашему окружению

Иногда бывает необходимо поделиться тем, над чем вы сейчас работаете, с коллегами или клиентом. Vagrant имеет встроенную поддержку для этого с помощью команды `vagrant share`; однако это не сработает, если в вашем файле `Homestead.yaml` сконфигурировано несколько сайтов.

Чтобы решить эту проблему, Homestead содержит собственную команду `share`. Для начала [подключитесь к вашей виртуальной машине](#connecting-via-ssh) через `vagrant ssh` и выполните команду `share homestead.test`. Эта команда предоставит общий доступ к сайту `homestead.test` из вашего конфигурационного файла `Homestead.yaml`. Вы можете заменить `homestead.test` на любой из других сконфигурированных вами сайтов:

```shell
share homestead.test
```

После выполнения команды вы увидите экран Ngrok, который содержит журнал активности и общедоступные URL-адреса для вашего сайта. Если вы хотите указать регион, поддомен или другой параметр во время выполнения Ngrok, то вы можете добавить их в команду `share`:

```shell
share homestead.test -region=eu -subdomain=laravel
```

> {note} Помните, что Vagrant по своей сути небезопасен, и вы открываете свою виртуальную машину для доступа в Интернет, выполняя команду `share`.

<a name="debugging-and-profiling"></a>
## Отладка и профилирование

<a name="debugging-web-requests"></a>
### Отладка веб-запросов с помощью Xdebug

Homestead содержит поддержку пошаговой отладки с использованием [Xdebug](https://xdebug.org). Например, вы можете получить доступ к странице в своем браузере, и PHP подключится к вашей среде IDE, чтобы разрешить проверку и изменение выполняемого кода.

По умолчанию Xdebug уже запущен и готов принимать соединения. Если вам нужно задействовать Xdebug в CLI, выполните команду `sudo phpenmod xdebug` на вашей виртуальной машине Homestead. Затем следуйте инструкциям вашей IDE, чтобы включить отладку. Наконец, настройте свой браузер для запуска Xdebug с расширением или [букмарклетом](https://www.jetbrains.com/phpstorm/marklets/).

> {note} Xdebug заставляет PHP работать значительно медленнее. Чтобы отключить Xdebug, запустите `sudo phpdismod xdebug` на виртуальной машине Homestead и перезапустите службу FPM.

<a name="autostarting-xdebug"></a>
#### Автозапуск Xdebug

При отладке функциональных тестов, которые отправляют запросы к веб-серверу, проще автоматически запускать отладку, чем изменять тесты для прохождения через настраиваемый заголовок или файл cookie для запуска отладки. Чтобы заставить Xdebug запускаться автоматически, измените файл `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` внутри вашей виртуальной машины Homestead и добавьте следующую конфигурацию:

```ini
; Если Homestead.yaml содержит другую подсеть для IP-адреса, этот адрес может быть другим ...
xdebug.remote_host = 192.168.10.1
xdebug.remote_autostart = 1
```

<a name="debugging-cli-applications"></a>
### Отладка консольных приложений

Для отладки консольного приложения PHP, используйте псевдоним оболочки `xphp` внутри вашей виртуальной машины Homestead:

    xphp /path/to/script

<a name="profiling-applications-with-blackfire"></a>
### Профилирование приложений с Blackfire

[Blackfire](https://blackfire.io/docs/introduction) – это сервис для профилирования веб-запросов и консольных приложений. Он предлагает интерактивный пользовательский интерфейс, который отображает данные профиля в виде графиков вызовов и временных шкал. Он создан для использования в разработке, тестировании и эксплуатации без дополнительных затрат для конечных пользователей. Кроме того, Blackfire обеспечивает проверку производительности, качества и безопасности как кода, так и параметров конфигурации `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) – это приложение с открытым исходным кодом для веб-сканирования, веб-тестирования и веб-скрапинга, которое может работать совместно с Blackfire для создания сценариев профилирования.

Чтобы задействовать Blackfire, используйте параметр `features` в файле конфигурации Homestead:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Учетные данные сервера и клиента [требуют учетной записи Blackfire](https://blackfire.io/signup). Blackfire предлагает различные варианты профилирования приложения, включая инструмент командной строки и расширение браузера. Пожалуйста, [просмотрите документацию Blackfire](https://blackfire.io/docs/cookbooks/index) для получения дополнительной информации.

<a name="network-interfaces"></a>
## Сетевые интерфейсы

Параметр `network` файла `Homestead.yaml` конфигурирует сетевые интерфейсы для вашей виртуальной машины Homestead. Вы можете указать столько интерфейсов, сколько необходимо:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

Чтобы задействовать интерфейс [сетевого моста](https://www.vagrantup.com/docs/networking/public_network.html), измените тип сети на `public_network` и укажите параметр `bridge` для сети:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

Чтобы задействовать [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), просто удалите параметр `ip` из вашей конфигурации:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

<a name="extending-homestead"></a>
## Расширение Homestead

Вы можете расширить Homestead, используя сценарий `after.sh` в корне вашего каталога Homestead. В этот файл вы можете добавить любые команды оболочки, которые необходимы для правильной конфигурации и изменений вашей виртуальной машины.

Настраивая Homestead, Ubuntu может спросить вас, хотите ли вы сохранить исходную конфигурацию пакета или перезаписать ее новым файлом конфигурации. Чтобы избежать этого, вы должны использовать следующую команду при установке пакетов, чтобы избежать перезаписи любой конфигурации, ранее созданной Homestead:

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### Пользовательские настройки

При использовании Homestead со своей командой разработчиков вы можете настроить Homestead, чтобы он лучше соответствовал вашему личному стилю разработки. Для этого вы можете создать файл `user-customizations.sh` в корне вашего каталога Homestead (тот же каталог, где находится ваш файл `Homestead.yaml`). В этом файле вы можете сделать любую настройку, которую захотите; однако версионирование файла `user-customizations.sh` не должно выполняться.

<a name="provider-specific-settings"></a>
## Настройки, специфичные для провайдера

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

По умолчанию Homestead устанавливает для параметра `natdnshostresolver` значение `on`. Это позволяет Homestead использовать настройки DNS вашей операционной системы. Если вы хотите изменить это поведение, то добавьте следующие параметры конфигурации в ваш файл `Homestead.yaml`:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```

<a name="symbolic-links-on-windows"></a>
#### Символические ссылки в Windows

Если символические ссылки не работают должным образом на вашем компьютере с Windows, то вам может потребоваться добавить следующий блок в ваш `Vagrantfile`:

```ruby
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```
