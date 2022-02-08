# Laravel 9 · Пакет Laravel Envoy

- [Введение](#introduction)
- [Установка](#installation)
- [Написание задач](#writing-tasks)
    - [Определение задач](#defining-tasks)
    - [Множество серверов](#multiple-servers)
    - [Предстартовая подготовка](#setup)
    - [Переменные](#variables)
    - [Истории](#stories)
    - [Хуки](#completion-hooks)
- [Выполнение задач](#running-tasks)
    - [Подтверждение выполнения задачи](#confirming-task-execution)
- [Уведомления](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## Введение

[**Laravel Envoy**](https://github.com/laravel/envoy) – это инструмент для выполнения общих задач, запускаемых на ваших удаленных серверах. Используя синтаксис в стиле [Blade](blade.md), вы можете легко настроить задачи для развертывания, команд Artisan и многое другое. В настоящее время Envoy поддерживает только операционные системы Mac и Linux. Однако поддержка Windows достижима с помощью [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>
## Установка

Для начала установите Envoy с помощью менеджера пакетов Composer в свой проект:

```shell
composer require laravel/envoy --dev
```

После установки исполняемый файл Envoy будет доступен в каталоге вашего приложения `vendor/bin`:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## Написание задач

<a name="defining-tasks"></a>
### Определение задач

Задачи – это основной «строительный блок» Envoy. Задачи определяются командами оболочки, выполняемыми на ваших удаленных серверах при вызове задачи. Например, вы можете определить задачу, которая выполнит команду `php artisan queue:restart` обработчика очереди на серверах вашего приложения.

Все ваши задачи Envoy должны быть определены в файле `Envoy.blade.php` в корне вашего приложения. Например:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Как видите, в верхней части файла объявлен массив `@servers`, что позволяет вам ссылаться на эти серверы с помощью параметра `on` в определениях ваших задач. Объявление `@servers` всегда следует размещать в одной строке. В определениях `@task` вы должны поместить команды оболочки, которые должны выполняться на ваших серверах при вызове задачи.

<a name="local-tasks"></a>
#### Локальные задачи

Вы можете принудительно запустить сценарий на вашем локальном компьютере, указав IP-адрес сервера как `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Импорт задач Envoy

Используя директиву `@import`, вы можете импортировать другие файлы Envoy для добавления дополнительных историй и задач. После того, как файлы были импортированы, вы можете выполнять задачи, содержащиеся в них, как если бы они были определены в вашем собственном файле Envoy:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### Множество серверов

Envoy позволяет легко запускать задачу на нескольких серверах. Во-первых, добавьте необходимые серверы в объявление `@servers`. Каждому серверу должно быть присвоено уникальное имя. После определения дополнительных серверов, вы можете использовать каждый из них в массиве задачи `on`:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>
#### Параллельное выполнение

По умолчанию задачи будут выполняться на каждом сервере поочередно. Другими словами, задача должна завершится на первом сервере, прежде чем будет выполнена на втором. Если вы хотите запустить задачу на нескольких серверах параллельно, то добавьте параметр `parallel` в определение задачи:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>
### Предстартовая подготовка

По желанию можно выполнить произвольный PHP-код перед запуском ваших задач Envoy. Вы можете использовать директиву `@setup` для определения блока PHP-кода, который должен быть выполнен перед вашими задачами:

```php
@setup
    $now = new DateTime;
@endsetup
```

Если вам нужны другие файлы PHP перед выполнением вашей задачи, то вы можете использовать директиву `@include` в верхней части вашего файла `Envoy.blade.php`:

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>
### Переменные

При необходимости вы можете передать аргументы задачам Envoy, указав их в командной строке при вызове Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Вы можете получить доступ к параметрам ваших задач, используя [синтаксис «вывода» Blade](blade.md#displaying-data). Вы также можете определять операторы `if` и циклы Blade в своих задачах. Например, давайте проверим наличие переменной `$branch` перед выполнением команды `git pull`:

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### Истории

Истории группируют набор задач под одним удобным названием. Например, вы можете сгруппировать запуск задач `update-code` и `install-dependencies`, перечислив их имена в определении истории `deploy`:

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

После написания история, вы можете запустить ее так же, как вы запускаете отдельную задачу:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### Хуки

При запуске задач и историй инициализируется ряд хуков. Envoy поддерживает следующие типы хуков: `@before`,`@after`, `@error`, `@success` и `@finished`. Весь код в этих хуках интерпретируется как PHP и выполняется локально, а не на удаленных серверах, с которыми взаимодействуют ваши задачи.

Вы можете определить столько хуков, сколько захотите. Они будут выполняться в том порядке, в котором они указаны в вашем скрипте Envoy.

<a name="hook-before"></a>
#### Директива хука`@before`

Перед выполнением каждой задачи будут выполняться все хуки `@before`, зарегистрированные в вашем сценарии Envoy. Хуки `@before` получат имя задачи, которая будет выполняться:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### Директива хука `@after`

После выполнения каждой задачи будут выполняться все хуки `@after`, зарегистрированные в вашем сценарии Envoy. Хуки `@after` получат имя запущенной задачи:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### Директива хука `@error`

После каждого сбоя задачи (выход с кодом состояния больше `0`) будут выполняться все хуки `@error`, зарегистрированные в вашем сценарии Envoy. Хуки `@error` получат имя запущенной задачи:

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### Директива хука `@success`

Если все задачи выполнены без ошибок, то все хуки `@success`, зарегистрированные в вашем сценарии Envoy, будут выполнены:

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### Директива хука `@finished`

После выполнения всех задач (независимо от статуса выхода) будут выполнены все хуки `@finished`. Хуки `@finished` получат код состояния завершенной задачи, который может быть `null` или `integer`, большим или равным `0`:

```blade
@finished
    if ($exitCode > 0) {
        // В одной из задач произошли ошибки ...
    }
@endfinished
```

<a name="running-tasks"></a>
## Выполнение задач

Чтобы запустить задачу или историю, которая определена в файле `Envoy.blade.php` вашего приложения, выполните команду `run` Envoy, передав имя задачи или истории, которую вы хотите выполнить. Envoy выполнит задачу и отобразит вывод с ваших удаленных серверов во время выполнения задачи:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### Подтверждение выполнения задачи

Если вы хотите получить запрос на подтверждение перед запуском конкретной задачи на своих серверах, вам следует добавить параметр `confirm` в директиву определения задачи. Этот параметр особенно полезен для деструктивных операций:

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## Уведомления

<a name="slack"></a>
### Slack

Envoy поддерживает отправку уведомлений в [Slack](https://slack.com) после выполнения каждой задачи. Директива `@slack` принимает WebHook URL и имя канала / пользователя. Вы можете получить WebHook URL, создав интеграцию «Incoming WebHooks» в панели управления Slack.

Вы должны передать полный WebHook URL в качестве первого аргумента директивы `@slack`. Вторым аргументом, передаваемым директиве `@slack`, должно быть имя канала `#channel` или имя пользователя `@user`:

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

По умолчанию уведомления Envoy отправляют сообщение в канал уведомлений с описанием выполненной задачи. Однако вы можете назначить свое сообщение, передав третий аргумент директиве `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy также поддерживает отправку уведомлений в [Discord](https://discord.com) после выполнения каждой задачи. Директива `@discord` принимает WebHook URL и сообщение. Вы можете получить WebHook URL, создав «Webhook» в настройках сервера и выбрав канал, на который WebHook должен публиковать сообщения. Вы должны передать полный WebHook URL в директиву `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### Telegram

Envoy также поддерживает отправку уведомлений в [Telegram](https://telegram.org) после выполнения каждой задачи. Директива `@telegram` принимает идентификатор бота Telegram и идентификатор чата. Вы можете получить свой идентификатор бота, создав нового бота в [BotFather](https://t.me/botfather). Вы можете получить действительный идентификатор чата, используя [`@username_to_id_bot`](https://t.me/username_to_id_bot). Вы должны передать полный идентификатор бота и идентификатор чата в директиву `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy также поддерживает отправку уведомлений в [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) после выполнения каждой задачи. Директива `@microsoftTeams` принимает веб-хук Teams (обязательно), сообщение, цвет темы (по типу сообщения: успешно, информация, предупреждение, ошибка) и массив параметров. Вы можете получить веб-хук Teams, создав новый [входящий веб-хук](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook). В Teams API есть множество других атрибутов для настройки сообщения, например заголовок, сводка и разделы. Дополнительную информацию можно найти в [документации Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Вы должны передать полный URL-адрес веб-хука в директиву `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
