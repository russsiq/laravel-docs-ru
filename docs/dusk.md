# Laravel 9 · Тестирование · Пакет Laravel Dusk

- [Введение](#introduction)
- [Установка](#installation)
    - [Управление установками ChromeDriver](#managing-chromedriver-installations)
    - [Использование других браузеров](#using-other-browsers)
- [Начало работы](#getting-started)
    - [Генерация тестов](#generating-tests)
    - [Миграции базы данных](#migrations)
    - [Запуск тестов](#running-tests)
    - [Обработка файла переменных окружения](#environment-handling)
- [Основы работы с браузером](#browser-basics)
    - [Создание браузеров](#creating-browsers)
    - [Навигация](#navigation)
    - [Изменение размера окна браузера](#resizing-browser-windows)
    - [Макрокоманды браузера](#browser-macros)
    - [Аутентификация](#authentication)
    - [Cookies](#cookies)
    - [Выполнение JavaScript](#executing-javascript)
    - [Получение снимка экрана](#taking-a-screenshot)
    - [Сохранение вывода консоли на диск](#storing-console-output-to-disk)
    - [Сохранение исходного кода страницы на диск](#storing-page-source-to-disk)
- [Взаимодействие с элементами](#interacting-with-elements)
    - [Селекторы Dusk](#dusk-selectors)
    - [Текст, значения и атрибуты](#text-values-and-attributes)
    - [Взаимодействие с формами](#interacting-with-forms)
    - [Прикрепление файлов](#attaching-files)
    - [Нажатие кнопок](#pressing-buttons)
    - [Кликанье по ссылкам](#clicking-links)
    - [Использование клавиатуры](#using-the-keyboard)
    - [Использование мыши](#using-the-mouse)
    - [Диалоговые окна JavaScript (Alert, Prompt, Confirm)](#javascript-dialogs)
    - [Сегментированное тестирование по селекторам](#scoping-selectors)
    - [Ожидание доступности элементов](#waiting-for-elements)
    - [Прокрутка элемента в область видимости пользователя](#scrolling-an-element-into-view)
- [Доступные утверждения](#available-assertions)
- [Тестовые страницы](#pages)
    - [Генерация тестовых страниц](#generating-pages)
    - [Конфигурирование тестовых страниц](#configuring-pages)
    - [Навигация по тестовым страницам](#navigating-to-pages)
    - [Псевдонимы селекторов](#shorthand-selectors)
    - [Методы тестовых страниц](#page-methods)
- [Компоненты для тестов](#components)
    - [Генерация компонентов](#generating-components)
    - [Использование компонентов](#using-components)
- [Непрерывная интеграция](#continuous-integration)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## Введение

[Laravel Dusk](https://github.com/laravel/dusk) предлагает выразительный и простой в использовании API для автоматизации поведения браузера и тестирования. По умолчанию Dusk не требует установки JDK или Selenium на ваш локальный компьютер. Вместо этого Dusk использует автономную установку [ChromeDriver](https://sites.google.com/chromium.org/driver). По желанию вы можете использовать любой другой драйвер, совместимый с Selenium.

<a name="installation"></a>
## Установка

Для начала установите [Google Chrome](https://www.google.com/chrome) и добавьте в зависимость `laravel/dusk` с помощью менеджера пакетов Composer в свой проект:

```shell
composer require --dev laravel/dusk
```

> {note} Если вы вручную регистрируете поставщика `DuskServiceProvider`, то вам **никогда** не следует регистрировать его в рабочем окружении, так как это может привести к тому, что случайные пользователи смогут пройти аутентификацию в вашем приложении.

После установки пакета Dusk выполните команду `dusk:install` Artisan. Команда `dusk:install` создаст каталог `tests/Browser` и пример теста Dusk:

```shell
php artisan dusk:install
```

Затем установите переменную окружения `APP_URL` в файле `.env` вашего приложения. Это значение должно соответствовать URL-адресу, который вы используете для доступа к вашему приложению в браузере.

> {tip} Если вы используете [Laravel Sail](sail.md) для управления своей локальной средой разработки, то обратитесь также к документации Sail по [настройке и запуску тестов Dusk](sail.md#laravel-dusk).

<a name="managing-chromedriver-installations"></a>
### Управление установками ChromeDriver

Если вы хотите установить версию ChromeDriver, отличную от той, которая включена в Laravel Dusk, то вы можете использовать команду `dusk:chrome-driver`:

```shell
# Установить последнюю версию ChromeDriver для вашей ОС ...
php artisan dusk:chrome-driver

# Установить конкретную версию ChromeDriver для вашей ОС ...
php artisan dusk:chrome-driver 86

# Установить конкретную версию ChromeDriver для всех поддерживаемых ОС ...
php artisan dusk:chrome-driver --all

# Установить версию ChromeDriver, которая соответствует обнаруженной версии Chrome / Chromium для вашей ОС ...
php artisan dusk:chrome-driver --detect
```

> {note} Dusk требует, чтобы файлы `chromedriver` были доступны для выполнения. Если у вас возникли проблемы с запуском Dusk, то вы должны убедиться, что файлы доступны для выполнения, используя следующую команду: `chmod -R 0755 vendor/laravel/dusk/bin/`.

<a name="using-other-browsers"></a>
### Использование других браузеров

По умолчанию Dusk использует Google Chrome и автономную установку [ChromeDriver](https://sites.google.com/chromium.org/driver) для запуска ваших браузерных тестов. Тем не менее, вы можете запустить свой собственный сервер Selenium и запускать тесты в любом браузере по желанию.

Для начала откройте файл `tests/DuskTestCase.php`, который является базовым тестовым классом Dusk вашего приложения. Внутри этого файла вы можете удалить вызов метода `startChromeDriver`. Это остановит Dusk от автоматического запуска ChromeDriver:

    /**
     * Подготовить Dusk для выполнения теста.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Затем вы можете изменить метод `driver` для подключения к URL-адресу и порту по вашему выбору. Кроме того, вы можете изменить «требуемые характеристики» через класс `DesiredCapabilities`, передаваемые экземпляру WebDriver:

    /**
     * Создать экземпляр RemoteWebDriver.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## Начало работы

<a name="generating-tests"></a>
### Генерация тестов

Чтобы сгенерировать тест Dusk, используйте команду `dusk:make` Artisan. Сгенерированный тест будет помещен в каталог `tests/Browser`:

```shell
php artisan dusk:make LoginTest
```

<a name="migrations"></a>
### Миграции базы данных

Большинство тестов, которые вы пишете, будут взаимодействовать со страницами, которые извлекают данные из базы данных вашего приложения; однако в ваших тестах Dusk никогда не следует использовать трейт `RefreshDatabase`. Трейт `RefreshDatabase` использует транзакции базы данных, которые не будут доступны и следовательно применяться через HTTP-запросы. Вместо этого используйте трейт `DatabaseMigrations`, который будет повторно выполнять миграцию базу данных для каждого теста:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

> {note} Базы данных SQLite, хранимые в памяти, нельзя использовать при выполнении тестов Dusk. Поскольку браузер выполняет свой собственный процесс, он не сможет получить доступ к базам данных, хранимых в памяти, других процессов.

<a name="running-tests"></a>
### Запуск тестов

Чтобы запустить браузерные тесты, выполните команду `dusk` Artisan:

```shell
php artisan dusk
```

Если при последнем запуске команды `dusk` у вас были ошибки тестирования, то вы можете сэкономить время, повторно запустив сначала неудачные тесты с помощью команды `dusk:fails`:

```shell
php artisan dusk:fails
```

Команда `dusk` принимает любой аргумент, который обычно принимается тестером PHPUnit, например, позволяет вам запускать тесты только для указанной [группы](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group):

```shell
php artisan dusk --group=foo
```

> {tip} Если вы используете [Laravel Sail](sail.md) для управления своей локальной средой разработки, обратитесь к документации Sail по [настройке и запуску тестов Dusk](sail.md#laravel-dusk).

<a name="manually-starting-chromedriver"></a>
#### Запуск ChromeDriver вручную

По умолчанию Dusk автоматически пытается запустить ChromeDriver. Если это не работает для вашей конкретной системы, вы можете вручную запустить ChromeDriver перед запуском команды `dusk`. Если вы решили запустить ChromeDriver вручную, то вы должны закомментировать следующую строку вашего файла `tests/DuskTestCase.php`:

    /**
     * Подготовить Dusk для выполнения теста.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Кроме того, если вы запускаете ChromeDriver на порту, отличном от `9515`, то вам следует изменить метод `driver` того же класса, чтобы указать необходимый порт:

    /**
     * Создать экземпляр RemoteWebDriver.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Обработка файла переменных окружения

Чтобы заставить Dusk использовать свой собственный файл окружения при запуске тестов, создайте файл `.env.dusk.{environment}` в корне вашего проекта. Например, если вы будете запускать команду `dusk` из вашей `local` (локальной) среды, то вы должны создать файл `.env.dusk.local`.

При запуске тестов Dusk создаст резервную копию вашего файла `.env` и переименует ваше окружение Dusk в файле `.env`. После завершения тестов ваш файл `.env` будет восстановлен.

<a name="browser-basics"></a>
## Основы работы с браузером

<a name="creating-browsers"></a>
### Создание браузеров

Для начала давайте напишем тест, который проверяет, можем ли мы войти в наше приложение. После создания теста мы можем изменить его, чтобы перейти на страницу входа, ввести некоторые учетные данные и нажать кнопку «Войти». Чтобы создать экземпляр браузера, вы можете вызвать метод `browse` из своего теста Dusk:

    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * Отвлеченный пример браузерного теста.
         *
         * @return void
         */
        public function test_basic_example()
        {
            $user = User::factory()->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'password')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Как видно в приведенном выше примере, метод `browse` принимает замыкание. Dusk автоматически передаст экземпляр браузера в это замыкание. Экземпляр браузера является основным объектом, используемым для взаимодействия с вашим приложением и создания утверждений.

<a name="creating-multiple-browsers"></a>
#### Создание нескольких браузеров

Иногда для правильного проведения теста требуется несколько браузеров. Например, для тестирования экрана чата, взаимодействующего с веб-сокетами, может потребоваться несколько браузеров. Чтобы создать несколько браузеров, просто добавьте больше аргументов браузера к сигнатуре замыкания, передаваемому методу `browse`:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

<a name="navigation"></a>
### Навигация

Метод `visit` используется для перехода к конкретному URI вашего приложения:

    $browser->visit('/login');

Вы можете использовать метод `visitRoute` для перехода к [именованному маршруту](routing.md#named-routes):

    $browser->visitRoute('login');

Вы можете перемещаться «назад» и «вперед», используя методы `back` и `forward`:

    $browser->back();

    $browser->forward();

Вы можете использовать метод `refresh` для обновления страницы:

    $browser->refresh();

<a name="resizing-browser-windows"></a>
### Изменение размера окна браузера

Вы можете использовать метод `resize` для настройки размера окна браузера:

    $browser->resize(1920, 1080);

Метод `maximize` используется для максимизации окна браузера:

    $browser->maximize();

Метод `fitContent` изменит размер окна браузера в соответствии с размером его содержимого:

    $browser->fitContent();

Если тест не пройден, то Dusk автоматически изменяет размер окна браузера в соответствии с его содержимым, прежде чем сделать снимок экрана. Вы можете отключить эту функцию, вызвав в своем тесте метод `disableFitOnFailure`:

    $browser->disableFitOnFailure();

Вы можете использовать метод `move`, чтобы переместить окно браузера в другое место на экране:

    $browser->move($x = 100, $y = 100);

<a name="browser-macros"></a>
### Макрокоманды браузера

Если вы хотите определить собственный метод браузера, который вы можете повторно использовать в различных ваших тестах, вы можете использовать метод `macro` класса `Browser`. Как правило, вызов этого метода осуществляется в методе `boot` [поставщика служб](providers.md):

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Browser;

    class DuskServiceProvider extends ServiceProvider
    {
        /**
         * Регистрация макрокоманд браузера Dusk.
         *
         * @return void
         */
        public function boot()
        {
            Browser::macro('scrollToElement', function ($element = null) {
                $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

                return $this;
            });
        }
    }

Метод `macro` принимает имя в качестве первого аргумента и замыкание в качестве второго. Замыкание будет выполнено при вызове макрокоманды в качестве метода экземпляра `Browser`:

    $this->browse(function ($browser) use ($user) {
        $browser->visit('/pay')
                ->scrollToElement('#credit-card-details')
                ->assertSee('Enter Credit Card Details');
    });

<a name="authentication"></a>
### Аутентификация

Часто вы будете тестировать страницы, требующие аутентификации. Вы можете использовать метод Dusk `loginAs`, чтобы избежать взаимодействия с экраном входа в систему вашего приложения во время каждого теста. Метод `loginAs` принимает первичный ключ аутентифицируемой модели или, непосредственно, экземпляр аутентифицируемой модели:

    use App\Models\User;

    $this->browse(function ($browser) {
        $browser->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} После использования метода `loginAs` сессия пользователя будет поддерживаться для всех тестов, находящихся в файле.

<a name="cookies"></a>
### Cookies

Вы можете использовать метод `cookie` для получения или установления зашифрованного значения cookie. По умолчанию все файлы cookie, созданные Laravel, зашифрованы:

    $browser->cookie('name');

    $browser->cookie('name', 'Taylor');

Вы можете использовать метод `plainCookie` для получения или установления незашифрованного значения cookie:

    $browser->plainCookie('name');

    $browser->plainCookie('name', 'Taylor');

Вы можете использовать метод `deleteCookie` для удаления конкретного файла cookie:

    $browser->deleteCookie('name');

<a name="executing-javascript"></a>
### Выполнение JavaScript

Вы можете использовать метод `script` для выполнения произвольных выражений JavaScript в браузере:

    $browser->script('document.documentElement.scrollTop = 0');

    $browser->script([
        'document.body.scrollTop = 0',
        'document.documentElement.scrollTop = 0',
    ]);

    $output = $browser->script('return window.location.pathname');

<a name="taking-a-screenshot"></a>
### Получение снимка экрана

Вы можете использовать метод `screenshot`, чтобы сделать снимок экрана и сохранить его с заданным именем файла. Все скриншоты будут храниться в каталоге `tests/Browser/screenshots`:

    $browser->screenshot('filename');

<a name="storing-console-output-to-disk"></a>
### Сохранение вывода консоли на диск

Вы можете использовать метод `storeConsoleLog` для записи вывода консоли текущего браузера на диск с заданным именем файла. Вывод консоли будет храниться в каталоге `tests/Browser/console`:

    $browser->storeConsoleLog('filename');

<a name="storing-page-source-to-disk"></a>
### Сохранение исходного кода страницы на диск

Вы можете использовать метод `storeSource` для записи исходного кода текущей страницы на диск с заданным именем файла. Исходный код страницы будет храниться в каталоге `tests/Browser/source`:

    $browser->storeSource('filename');

<a name="interacting-with-elements"></a>
## Взаимодействие с элементами

<a name="dusk-selectors"></a>
### Селекторы Dusk

Выбор универсальных селекторов CSS для взаимодействия с элементами – одна из самых сложных частей написания тестов Dusk. Со временем изменения клиентского интерфейса могут привести к тому, что селекторы CSS, подобные приведенным ниже, нарушат ваши тесты:

    // HTML-разметка ...

    <button>Login</button>

    // Выполнение теста ...

    $browser->click('.login-page .container div > button');

Селекторы Dusk позволяют сосредоточиться на написании эффективных тестов, а не на запоминании селекторов CSS. Чтобы определить селектор, добавьте к вашему элементу HTML-атрибут `dusk`. Затем, при взаимодействии с браузером Dusk, добавьте к селектору префикс `@`, чтобы управлять закрепленным элементом в вашем тесте:

    // HTML-разметка ...

    <button dusk="login-button">Login</button>

    // Выполнение теста ...

    $browser->click('@login-button');

<a name="text-values-and-attributes"></a>
### Текст, значения и атрибуты

<a name="retrieving-setting-values"></a>
#### Получение и установка значений

Dusk содержит несколько методов для взаимодействия с текущим значением, отображаемым текстом и атрибутами элементов на странице. Например, чтобы получить «значение» элемента, которое соответствует указанному CSS или Dusk селектору, используйте метод `value`:

    // Получить значение ...
    $value = $browser->value('selector');

    // Установить значение ...
    $browser->value('selector', 'value');

Вы можете использовать метод `inputValue` для получения «значения» элемента ввода, имеющего указанное имя поля:

    $value = $browser->inputValue('field');

<a name="retrieving-text"></a>
#### Получение текста

Метод `text` используется для получения отображаемого текста элемента, соответствующий указанному селектору:

    $text = $browser->text('selector');

<a name="retrieving-attributes"></a>
#### Получение атрибутов

Наконец, метод `attribute` может быть использован для получения значения атрибута элемента, соответствующий указанному селектору:

    $attribute = $browser->attribute('selector', 'value');

<a name="interacting-with-forms"></a>
### Взаимодействие с формами

<a name="typing-values"></a>
#### Ввод значений

Dusk содержит множество методов для взаимодействия с формами и элементами ввода. Во-первых, давайте взглянем на пример ввода текста в поле:

    $browser->type('email', 'taylor@laravel.com');

Обратите внимание, что, нам не требуется передавать селектор CSS в метод `type`, хотя метод принимает его при необходимости. Если селектор CSS не указан, то Dusk будет искать поле `input` или `textarea` с указанным атрибутом `name`.

Чтобы добавить текст в поле, не очищая его содержимое, вы можете использовать метод `append`:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Вы можете очистить значение поля с помощью метода `clear`:

    $browser->clear('email');

Вы можете указать Dusk печатать медленно, используя метод `typeSlowly`. По умолчанию Dusk будет делать паузу на `100` миллисекунд между нажатиями клавиш. Чтобы изменить время между нажатиями клавиш, вы можете передать соответствующее количество миллисекунд в качестве третьего аргумента метода:

    $browser->typeSlowly('mobile', '+1 (202) 555-5555');

    $browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);

Вы можете использовать метод `appendSlowly` для медленного добавления текста:

    $browser->type('tags', 'foo')
            ->appendSlowly('tags', ', bar, baz');

<a name="dropdowns"></a>
#### Выпадающие списки

Чтобы выбрать значение, доступное для выпадающего списка, вы можете использовать метод `select`. Как и метод `type`, метод `select` не требует полного селектора CSS. При передаче значения методу `select` вы должны передать значение параметра `value` вместо отображаемого текста:

    $browser->select('size', 'Large');

Вы можете выбрать случайный вариант, опустив второй аргумент:

    $browser->select('size');

Предоставляя массив в качестве второго аргумента метода `select`, вы можете указать методу мульти-выбор параметров:

    $browser->select('categories', ['Art', 'Music']);

<a name="checkboxes"></a>
#### Флажки

Чтобы «отметить» флажок, вы можете использовать метод `check`. Как и многие другие методы, связанные с вводом, полный селектор CSS не требуется. Если совпадение селектора CSS не найдено, то Dusk будет искать флажок с соответствующим атрибутом `name`:

    $browser->check('terms');

Метод `uncheck` используется для «снятия галочки» с флажка:

    $browser->uncheck('terms');

<a name="radio-buttons"></a>
#### Радиокнопки

Чтобы «выбрать» вариант из радиокнопок, вы можете использовать метод `radio`. Как и многие другие методы, связанные с вводом, полный селектор CSS не требуется. Если совпадение селектора CSS не найдено, то Dusk будет искать радиокнопку с соответствующими атрибутами `name` и `value`:

    $browser->radio('size', 'large');

<a name="attaching-files"></a>
### Прикрепление файлов

Метод `attach` используется для прикрепления файла к элементу выбора файлов. Как и многие другие методы, связанные с вводом, полный селектор CSS не требуется. Если совпадение селектора CSS не найдено, то Dusk будет искать элемент выбора файлов с соответствующим атрибутом `name`:

    $browser->attach('photo', __DIR__.'/photos/mountains.png');

> {note} Функционал прикрепления требует, чтобы на вашем сервере было установлено и включено расширение `Zip` PHP.

<a name="pressing-buttons"></a>
### Нажатие кнопок

Метод `press` используется для нажатия кнопки на странице. Аргумент, передаваемый методу `press`, может быть либо отображаемый текст кнопки, либо CSS / Dusk селектор:

    $browser->press('Login');

При отправке форм многие приложения отключают кнопку отправки формы после ее нажатия, а затем снова включают кнопку, когда HTTP-запрос отправки формы завершен. Чтобы нажать кнопку и дождаться ее повторного включения, вы можете использовать метод `pressAndWaitFor`:

    // Нажимаем кнопку и ждем ее активности не более 5 секунд ...
    $browser->pressAndWaitFor('Save');

    // Нажимаем кнопку и ждем ее активности не более 1 секунды ...
    $browser->pressAndWaitFor('Save', 1);

<a name="clicking-links"></a>
### Кликанье по ссылкам

Чтобы щелкнуть ссылку, вы можете использовать метод `clickLink` экземпляра браузера. Метод `clickLink` щелкнет ссылку с указанным видимым текстом:

    $browser->clickLink($linkText);

Вы можете использовать метод `seeLink`, чтобы определить, видна ли на странице ссылка с указанным видимым текстом:

    if ($browser->seeLink($linkText)) {
        // ...
    }

> {note} Эти методы взаимодействуют с библиотеками jQuery. Если jQuery недоступен на странице, то Dusk автоматически вставит его на страницу, чтобы он был доступен во время теста.

<a name="using-the-keyboard"></a>
### Использование клавиатуры

Метод `keys` позволяет передавать более сложные последовательности ввода для указанного элемента, чем это обычно доступно при использовании метода `type`. Например, при вводе значений можно поручить Dusk удерживать клавиши-модификаторы. В этом примере клавиша <kbd>shift</kbd> будет удерживаться, пока строка «taylor» вводится в элемент заданного селектора. После ввода «taylor», строка «swift» будет вводиться без модификаторов:

    $browser->keys('selector', ['{shift}', 'taylor'], 'swift');

Другой значимый пример использования метода `keys` – это отправка комбинации «горячих клавиш» основному селектору CSS вашего приложения:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Все модификаторы клавиш, такие как `{command}` заключены в символы `{}` и соответствуют константам, определенным в классе `Facebook\WebDriver\WebDriverKeys`, который можно [найти на GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Использование мыши

<a name="clicking-on-elements"></a>
#### Кликанье по элементам

Метод `click` используется для щелчка по элементу с указанным CSS / Dusk селектором:

    $browser->click('.selector');

Метод `clickAtXPath` используется для щелчка по элементу с указанным XPath-выражением:

    $browser->clickAtXPath('//div[@class = "selector"]');

Метод `clickAtPoint` используется для щелчка по самому верхнему элементу в точке с координатой, указанной относительно видимой области браузера:

    $browser->clickAtPoint($x = 0, $y = 0);

Метод `doubleClick` используется для имитации двойного щелчка мыши:

    $browser->doubleClick();

Метод `rightClick` используется для имитации щелчка правой кнопкой мыши:

    $browser->rightClick();

    $browser->rightClick('.selector');

Метод `clickAndHold` используется для имитации нажатия и удержания кнопки мыши. Последующий вызов метода `releaseMouse` отменяет это поведение и отпускает кнопку мыши:

    $browser->clickAndHold()
            ->pause(1000)
            ->releaseMouse();

<a name="mouseover"></a>
#### Наведение мыши

Метод `mouseover` используется, когда вам нужно навести указатель мыши на элемент с заданным CSS или Dusk селектором:

    $browser->mouseover('.selector');

<a name="drag-drop"></a>
#### Перетаскивания

Метод `drag` используется для перетаскивания элемента с указанным селектором, на другой элемент:

    $browser->drag('.from-selector', '.to-selector');

Или вы можете перетащить элемент в одном направлении:

    $browser->dragLeft('.selector', $pixels = 10);
    $browser->dragRight('.selector', $pixels = 10);
    $browser->dragUp('.selector', $pixels = 10);
    $browser->dragDown('.selector', $pixels = 10);

Наконец, вы можете перетащить элемент с указанным смещением:

    $browser->dragOffset('.selector', $x = 10, $y = 10);

<a name="javascript-dialogs"></a>
### Диалоговые окна JavaScript (Alert, Prompt, Confirm)

Dusk содержит различные методы для взаимодействия с диалогами JavaScript. Например, вы можете использовать метод `waitForDialog`, чтобы дождаться появления диалогового окна JavaScript. Этот метод принимает необязательный аргумент, указывающий, сколько секунд ждать до появления диалогового окна:

    $browser->waitForDialog($seconds = null);

Метод `assertDialogOpened` используется для утверждения того, что диалоговое окно было отображено и содержит указанное сообщение:

    $browser->assertDialogOpened('Dialog message');

Если диалоговое окно JavaScript содержит поле ввода, то вы можете использовать метод `typeInDialog`, чтобы ввести значение:

    $browser->typeInDialog('Hello World');

Чтобы закрыть открытое диалоговое окно JavaScript, нажав кнопку «ОК», вы можете вызвать метод `acceptDialog`:

    $browser->acceptDialog();

Чтобы закрыть открытое диалоговое окно JavaScript, нажав кнопку «Отмена», вы можете вызвать метод `dismissDialog`:

    $browser->dismissDialog();

<a name="scoping-selectors"></a>
### Сегментированное тестирование по селекторам

Иногда требуется выполнить несколько операций, принадлежащих конкретному селектору. Например, вы можете утверждать, что некоторый текст существует только в таблице, а затем щелкнуть кнопку в этой таблице. Для этого можно использовать метод `with`. Все операции, выполняемые в рамках замыкания, переданного методу `with`, будут привязаны к исходному селектору:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

Иногда требуется выполнить утверждения за пределами текущей области. Вы можете использовать для этого методы `elsewhere` и `elsewhereWhenAvailable`:

     $browser->with('.table', function ($table) {
        // Текущая область `body .table` ...

        $browser->elsewhere('.page-title', function ($title) {
            // Текущая область `body .page-title` ...
            $title->assertSee('Hello World');
        });

        $browser->elsewhereWhenAvailable('.page-title', function ($title) {
            // Текущая область `body .page-title` ...
            $title->assertSee('Hello World');
        });
     });

<a name="waiting-for-elements"></a>
### Ожидание доступности элементов

При тестировании приложений, широко использующих JavaScript, часто возникает необходимость «подождать», когда станут доступны определенные элементы или данные, прежде чем продолжить выполнение теста. Dusk сделает это легко. Используя различные методы, вы можете подождать, когда элементы станут видимыми на странице, или даже дождаться, когда указанное выражение JavaScript не станет «истинным».

<a name="waiting"></a>
#### Ожидание

Если вам нужно просто приостановить тест на определенное количество миллисекунд, используйте метод `pause`:

    $browser->pause(1000);

<a name="waiting-for-selectors"></a>
#### Ожидание конкретных селекторов

Метод `waitFor` используется для приостановки выполнения теста до тех пор, пока на странице не отобразится элемент с указанным CSS или Dusk селектором. По умолчанию это приостанавливает тест максимум на пять секунд перед выбросом исключения. При необходимости вы можете передать иной порог тайм-аута в качестве второго аргумента метода:

    // Ожидание селектора не более пяти секунд ...
    $browser->waitFor('.selector');

    // Ожидание селектора максимум одну секунду ...
    $browser->waitFor('.selector', 1);

Вы также можете подождать, пока элемент с указанным селектором не будет содержать необходимый текст:

    // Ожидание селектора, содержащего указанный текст, не более пяти секунд ...
    $browser->waitForTextIn('.selector', 'Hello World');

    // Ожидание селектора, содержащего указанный текст, не более одной секунды ...
    $browser->waitForTextIn('.selector', 'Hello World', 1);

Вы также можете подождать, пока элемент с указанным селектором не исчезнет со страницы:

    // Ожидание исчезновения селектора не более пяти секунд ...
    $browser->waitUntilMissing('.selector');

    // Ожидание исчезновения селектора не более одной секунды ...
    $browser->waitUntilMissing('.selector', 1);

Или вы можете подождать, пока элемент с указанным селектором не будет включен или отключен (заблокирован):

    // Ожидание включения селектора не более пяти секунд ...
    $browser->waitUntilEnabled('.selector');

    // Ожидание включения селектора не более одной секунды ...
    $browser->waitUntilEnabled('.selector', 1);

    // Ожидание отключения селектора не более пяти секунд ...
    $browser->waitUntilDisabled('.selector');

    // Ожидание отключения селектора не более одной секунды ...
    $browser->waitUntilDisabled('.selector', 1);

<a name="scoping-selectors-when-available"></a>
#### Сегментированное тестирование при доступности селекторов

Иногда требуется дождаться появления элемента с указанным селектором, а затем взаимодействовать с этим элементом. Например, вы можете подождать, пока не станет доступно модальное окно, а затем нажать кнопку «ОК» в модальном окне. Для этого можно использовать метод `whenAvailable`. Все операции с элементами, выполняемые в рамках замыкания, будут привязаны к исходному селектору:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

<a name="waiting-for-text"></a>
#### Ожидание видимости текста

Метод `waitForText` используется для ожидания видимости текста на странице:

    // Ожидание видимости текста максимум пять секунд ...
    $browser->waitForText('Hello World');

    // Ожидание видимости текста максимум одну секунду ...
    $browser->waitForText('Hello World', 1);

Вы можете использовать метод `waitUntilMissingText`, чтобы дождаться, пока отображаемый текст не будет удален со страницы:

    // Ожидание удаления текста не более пяти секунд ...
    $browser->waitUntilMissingText('Hello World');

    // Ожидание удаления текста не более одной секунды ...
    $browser->waitUntilMissingText('Hello World', 1);

<a name="waiting-for-links"></a>
#### Ожидание доступности ссылок

Метод `waitForLink` используется для ожидания появления текста указанной ссылки на странице:

    // Ожидание видимости ссылки не более пяти секунд ...
    $browser->waitForLink('Create');

    // Ожидание видимости ссылки не более одной секунды ...
    $browser->waitForLink('Create', 1);

<a name="waiting-on-the-page-location"></a>
#### Ожидание расположения страницы

При утверждения пути, например, `$browser->assertPathIs('/home')`, утверждение может завершиться ошибкой, если `window.location.pathname` обновляется асинхронно. Вы можете использовать метод `waitForLocation` для ожидания, когда расположение не станет необходимым значением:

    $browser->waitForLocation('/secret');

Метод `waitForLocation` также может использоваться для ожидания, когда текущее расположение окна не станет полным URL:

    $browser->waitForLocation('https://example.com/path');

Вы также можете дождаться расположения [именованного маршрута](routing.md#named-routes):

    $browser->waitForRoute($routeName, $parameters);

<a name="waiting-for-page-reloads"></a>
#### Ожидание перезагрузки страницы

Если вам нужно дождаться перезагрузки страницы после выполнения действия, то используйте метод `waitForReload`:

    use Laravel\Dusk\Browser;

    $browser->waitForReload(function (Browser $browser) {
        $browser->press('Submit');
    })
    ->assertSee('Success!');

Поскольку необходимость дождаться перезагрузки страницы обычно возникает после нажатия кнопки, для удобства можно использовать метод `clickAndWaitForReload`:

    $browser->clickAndWaitForReload('.selector')
            ->assertSee('something');

<a name="waiting-on-javascript-expressions"></a>
#### Ожидание выражений JavaScript

По желанию можно приостановить выполнение теста до тех пор, пока указанное выражение JavaScript не станет истинным. Вы можете легко сделать это, используя метод `waitUntil`. При передаче выражения в этот метод вам не нужно включать в него ни ключевое слово `return`, ни конечную точку с запятой:

    // Ожидание истинности выражения не более пяти секунд ...
    $browser->waitUntil('App.data.servers.length > 0');

    // Ожидание истинности выражения не более одной секунды ...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="waiting-on-vue-expressions"></a>
#### Ожидание выражений Vue

Методы `waitUntilVue` и `waitUntilVueIsNot` могут использоваться для ожидания, пока атрибут [компонента Vue](https://vuejs.org) не получит указанное значение:

    // Ожидание соответствия атрибута компонента указанному значению ...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // Ожидание несоответствия атрибута компонента указанному значению ...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

<a name="waiting-for-javascript-events"></a>
#### Ожидание событий JavaScript

Метод `waitForEvent` можно использовать для приостановки выполнения теста до тех пор, пока не произойдет событие JavaScript:

    $browser->waitForEvent('load');

Слушатель событий присоединен к текущей области видимости, которая по умолчанию является элементом `body`. При использовании селектора с областью действия слушатель событий будет прикреплен к соответствующему элементу:

    $browser->with('iframe', function ($iframe) {
        // Ожидание события загрузки элемента iframe ...
        $iframe->waitForEvent('load');
    });

Вы также можете предоставить селектор в качестве второго аргумента метода `waitForEvent`, чтобы прикрепить слушатель событий к определенному элементу:

    $browser->waitForEvent('load', '.selector');

Вы также можете дождаться событий для объектов `document` и `window`:

    // Ожидание прокрутки документа ...
    $browser->waitForEvent('scroll', 'document');

    // Ожидание не более пяти секунд изменений размера окна ...
    $browser->waitForEvent('resize', 'window', 5);

<a name="waiting-with-a-callback"></a>
#### Использование замыканий при ожидании

Многие из методов «ожидания» в Dusk основаны на методе `waitUsing`. Вы можете использовать этот метод напрямую, чтобы дождаться, пока переданное замыкание не вернет `true`. Метод `waitUsing` принимает максимальное количество секунд ожидания, интервал между выполнениями замыкания (паузу), само замыкание и необязательное сообщение об ошибке:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="scrolling-an-element-into-view"></a>
### Прокрутка элемента в область видимости пользователя

Иногда вы не можете щелкнуть элемент, потому что он находится за пределами области просмотра браузера. Метод `scrollIntoView` будет прокручивать окно браузера до тех пор, пока элемент с указанным селектором не окажется видимым:

    $browser->scrollIntoView('.selector')
            ->click('.selector');

<a name="available-assertions"></a>
## Доступные утверждения

Dusk содержит множество утверждений, которые вы можете использовать при тестировании вашего приложения. Все доступные утверждения представлены в списке ниже:

<!-- <style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style> -->

<!-- <div class="collection-method-list" markdown="1"> -->

- [assertTitle](#assert-title)
- [assertTitleContains](#assert-title-contains)
- [assertUrlIs](#assert-url-is)
- [assertSchemeIs](#assert-scheme-is)
- [assertSchemeIsNot](#assert-scheme-is-not)
- [assertHostIs](#assert-host-is)
- [assertHostIsNot](#assert-host-is-not)
- [assertPortIs](#assert-port-is)
- [assertPortIsNot](#assert-port-is-not)
- [assertPathBeginsWith](#assert-path-begins-with)
- [assertPathIs](#assert-path-is)
- [assertPathIsNot](#assert-path-is-not)
- [assertRouteIs](#assert-route-is)
- [assertQueryStringHas](#assert-query-string-has)
- [assertQueryStringMissing](#assert-query-string-missing)
- [assertFragmentIs](#assert-fragment-is)
- [assertFragmentBeginsWith](#assert-fragment-begins-with)
- [assertFragmentIsNot](#assert-fragment-is-not)
- [assertHasCookie](#assert-has-cookie)
- [assertHasPlainCookie](#assert-has-plain-cookie)
- [assertCookieMissing](#assert-cookie-missing)
- [assertPlainCookieMissing](#assert-plain-cookie-missing)
- [assertCookieValue](#assert-cookie-value)
- [assertPlainCookieValue](#assert-plain-cookie-value)
- [assertSee](#assert-see)
- [assertDontSee](#assert-dont-see)
- [assertSeeIn](#assert-see-in)
- [assertDontSeeIn](#assert-dont-see-in)
- [assertSeeAnythingIn](#assert-see-anything-in)
- [assertSeeNothingIn](#assert-see-nothing-in)
- [assertScript](#assert-script)
- [assertSourceHas](#assert-source-has)
- [assertSourceMissing](#assert-source-missing)
- [assertSeeLink](#assert-see-link)
- [assertDontSeeLink](#assert-dont-see-link)
- [assertInputValue](#assert-input-value)
- [assertInputValueIsNot](#assert-input-value-is-not)
- [assertChecked](#assert-checked)
- [assertNotChecked](#assert-not-checked)
- [assertRadioSelected](#assert-radio-selected)
- [assertRadioNotSelected](#assert-radio-not-selected)
- [assertSelected](#assert-selected)
- [assertNotSelected](#assert-not-selected)
- [assertSelectHasOptions](#assert-select-has-options)
- [assertSelectMissingOptions](#assert-select-missing-options)
- [assertSelectHasOption](#assert-select-has-option)
- [assertSelectMissingOption](#assert-select-missing-option)
- [assertValue](#assert-value)
- [assertValueIsNot](#assert-value-is-not)
- [assertAttribute](#assert-attribute)
- [assertAttributeContains](#assert-attribute-contains)
- [assertAriaAttribute](#assert-aria-attribute)
- [assertDataAttribute](#assert-data-attribute)
- [assertVisible](#assert-visible)
- [assertPresent](#assert-present)
- [assertNotPresent](#assert-not-present)
- [assertMissing](#assert-missing)
- [assertInputPresent](#assert-input-present)
- [assertInputMissing](#assert-input-missing)
- [assertDialogOpened](#assert-dialog-opened)
- [assertEnabled](#assert-enabled)
- [assertDisabled](#assert-disabled)
- [assertButtonEnabled](#assert-button-enabled)
- [assertButtonDisabled](#assert-button-disabled)
- [assertFocused](#assert-focused)
- [assertNotFocused](#assert-not-focused)
- [assertAuthenticated](#assert-authenticated)
- [assertGuest](#assert-guest)
- [assertAuthenticatedAs](#assert-authenticated-as)
- [assertVue](#assert-vue)
- [assertVueIsNot](#assert-vue-is-not)
- [assertVueContains](#assert-vue-contains)
- [assertVueDoesNotContain](#assert-vue-does-not-contain)

<!-- </div> -->

<a name="assert-title"></a>
#### assertTitle

Утверждение о том, что заголовок страницы соответствует переданному тексту:

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>
#### assertTitleContains

Утверждение о том, что заголовок страницы содержит переданный текст:

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>
#### assertUrlIs

Утверждение о том, что текущий URL (без строки запроса) соответствует переданной строке:

    $browser->assertUrlIs($url);

<a name="assert-scheme-is"></a>
#### assertSchemeIs

Утверждение о том, что схема текущего URL соответствует переданной схеме:

    $browser->assertSchemeIs($scheme);

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

Утверждение о том, что схема текущего URL не соответствует переданной схеме:

    $browser->assertSchemeIsNot($scheme);

<a name="assert-host-is"></a>
#### assertHostIs

Утверждение о том, что хост текущего URL соответствует переданному хосту:

    $browser->assertHostIs($host);

<a name="assert-host-is-not"></a>
#### assertHostIsNot

Утверждение о том, что хост текущего URL не соответствует переданному хосту:

    $browser->assertHostIsNot($host);

<a name="assert-port-is"></a>
#### assertPortIs

Утверждение о том, что порт текущего URL соответствует переданному порту:

    $browser->assertPortIs($port);

<a name="assert-port-is-not"></a>
#### assertPortIsNot

Утверждение о том, что порт текущего URL не соответствует переданному порту:

    $browser->assertPortIsNot($port);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

Утверждение о том, что путь текущего URL начинается с указанного пути:

    $browser->assertPathBeginsWith('/home');

<a name="assert-path-is"></a>
#### assertPathIs

Утверждение о том, что текущий путь соответствует переданному пути:

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>
#### assertPathIsNot

Утверждение о том, что текущий путь не соответствует переданному пути:

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>
#### assertRouteIs

Утверждение о том, что текущий URL соответствует переданному URL [именованного маршрута](routing.md#named-routes):

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

Утверждение о том, что переданный параметр строки запроса присутствует:

    $browser->assertQueryStringHas($name);

Утверждение о том, что переданный параметр строки запроса присутствует и имеет указанное значение:

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

Утверждение о том, что переданный параметр строки запроса отсутствует:

    $browser->assertQueryStringMissing($name);

<a name="assert-fragment-is"></a>
#### assertFragmentIs

Утверждение о том, что хеш-фрагмент текущего URL соответствует переданному фрагменту:

    $browser->assertFragmentIs('anchor');

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

Утверждение о том, что хеш-фрагмент текущего URL начинается с указанного фрагмента:

    $browser->assertFragmentBeginsWith('anchor');

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

Утверждение о том, что хеш-фрагмент текущего URL не соответствует переданному фрагменту:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

Утверждение о том, что переданный зашифрованный файл cookie присутствует:

    $browser->assertHasCookie($name);

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

Утверждение о том, что переданный незашифрованный файл cookie присутствует:

    $browser->assertHasPlainCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Утверждение о том, что переданный зашифрованный файл cookie отсутствует:

    $browser->assertCookieMissing($name);

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

Утверждение о том, что переданный незашифрованный файл cookie отсутствует:

    $browser->assertPlainCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

Утверждение о том, что зашифрованный файл cookie имеет указанное значение:

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

Утверждение о том, что незашифрованный файл cookie имеет указанное значение:

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>
#### assertSee

Утверждение о том, что переданный текст присутствует на странице:

    $browser->assertSee($text);

<a name="assert-dont-see"></a>
#### assertDontSee

Утверждение о том, что переданный текст отсутствует на странице:

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>
#### assertSeeIn

Утверждение о том, что переданный текст присутствует в селекторе:

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

Утверждение о том, что переданный текст отсутствует в селекторе:

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

Утверждение о том, что в селекторе присутствует какой-либо текст:

    $browser->assertSeeAnythingIn($selector);

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

Утверждение о том, что в селекторе отсутствует какой-либо текст:

    $browser->assertSeeNothingIn($selector);

<a name="assert-script"></a>
#### assertScript

Утверждение о том, что переданное выражение JavaScript возвращает указанное либо истинное значение:

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

<a name="assert-source-has"></a>
#### assertSourceHas

Утверждение о том, что переданный исходный код присутствует на странице:

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

Утверждение о том, что переданный исходный код отсутствует на странице:

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

Утверждение о том, что переданная ссылка присутствует на странице:

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

Утверждение о том, что переданная ссылка отсутствует на странице:

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

Утверждение о том, что переданное поле ввода имеет указанное значение:

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

Утверждение о том, что переданное поле ввода не имеет указанное значение:

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>
#### assertChecked

Утверждение о том, что переданный флажок отмечен:

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

Утверждение о том, что переданный флажок не отмечен:

    $browser->assertNotChecked($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

Утверждение о том, что переданная радиокнопка выбрана:

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

Утверждение о том, что переданная радиокнопка не выбрана:

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

Утверждение о том, что в переданном выпадающем списке выбрано указанное значение:

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

Утверждение о том, что в переданном выпадающем списке не выбрано указанное значение:

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

Утверждение о том, что переданный массив значений доступен для выбора:

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

Утверждение о том, что переданный массив значений недоступен для выбора:

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

Утверждение о том, что переданное значение доступно для выбора в указанном поле:

    $browser->assertSelectHasOption($field, $value);

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

Утверждение о том, что переданное значение недоступно для выбора в указанном поле:

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-value"></a>
#### assertValue

Утверждение о том, что элемент с указанным селектором, имеет переданное значение:

    $browser->assertValue($selector, $value);

<a name="assert-value-is-not"></a>
#### assertValueIsNot

Утверждение о том, что элемент с указанным селектором, не имеет переданное значение:

    $browser->assertValueIsNot($selector, $value);

<a name="assert-attribute"></a>
#### assertAttribute

Утверждение о том, что элемент с указанным селектором, имеет переданное значение атрибута:

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

Утверждение о том, что элемент с указанным селектором, частично содержит переданное значение атрибута:

    $browser->assertAttributeContains($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

Утверждение о том, что элемент с указанным селектором, имеет переданное значение `aria`-атрибута:

    $browser->assertAriaAttribute($selector, $attribute, $value);

Например, учитывая разметку `<button aria-label="Add"></button>`, вы можете выстроить утверждение относительно атрибута `aria-label` следующим образом:

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>
#### assertDataAttribute

Утверждение о том, что элемент с указанным селектором, имеет переданное значение `data`-атрибута:

    $browser->assertDataAttribute($selector, $attribute, $value);

Например, учитывая разметку `<tr id="row-1" data-content="attendees"></tr>`, вы можете выстроить утверждение относительно атрибута `data-content` следующим образом:

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>
#### assertVisible

Утверждение о том, что элемент с указанным селектором, видим:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

Утверждение о том, что элемент с указанным селектором, присутствует в исходном коде страницы:

    $browser->assertPresent($selector);

<a name="assert-not-present"></a>
#### assertNotPresent

Утверждение о том, что элемент с указанным селектором, отсутствует в исходном коде страницы:

    $browser->assertNotPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

Утверждение о том, что элемент с указанным селектором, не виден:

    $browser->assertMissing($selector);

<a name="assert-input-present"></a>
#### assertInputPresent

Утверждение о том, что поле ввода с указанным именем, присутствует:

    $browser->assertInputPresent($name);

<a name="assert-input-missing"></a>
#### assertInputMissing

Утверждение о том, что поле ввода с указанным именем, отсутствует в исходном коде страницы:

    $browser->assertInputMissing($name);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

Утверждение о том, что был открыт диалог JavaScript с указанным сообщением:

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

Утверждение о том, что переданное поле доступно для использования:

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

Утверждение о том, что переданное поле недоступно для использования:

    $browser->assertDisabled($field);

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

Утверждение о том, что переданная кнопка доступна для использования:

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

Утверждение о том, что переданная кнопка недоступна для использования:

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>
#### assertFocused

Утверждение о том, что переданное поле находится в фокусе:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

Утверждение о том, что переданное поле не находится в фокусе:

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>
#### assertAuthenticated

Утверждение о том, что пользователь аутентифицирован:

    $browser->assertAuthenticated();

<a name="assert-guest"></a>
#### assertGuest

Утверждение о том, что пользователь не аутентифицирован:

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Утверждение о том, что пользователь аутентифицирован как указанный пользователь:

    $browser->assertAuthenticatedAs($user);

<a name="assert-vue"></a>
#### assertVue

Dusk даже позволяет вам делать утверждения о состоянии данных [компонента Vue](https://vuejs.org). Например, представьте, что ваше приложение содержит следующий компонент Vue:

    // HTML-разметка ...

    <profile dusk="profile-component"></profile>

    // Определение компонента ...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

Вы можете утверждать о состоянии компонента Vue следующим образом:

    /**
     * Отвлеченный пример теста компонента Vue.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

Утверждение о том, что переданное свойство данных компонента Vue не соответствует указанному значению:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

Утверждение о том, что переданное свойство данных компонента Vue является массивом и содержит указанное значение:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-does-not-contain"></a>
#### assertVueDoesNotContain

Утверждение о том, что переданное свойство данных компонента Vue является массивом и не содержит указанное значения:

    $browser->assertVueDoesNotContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## Тестовые страницы

Иногда тесты требуют последовательного выполнения нескольких сложных действий. Это может затруднить чтение и понимание ваших тестов. Страницы Dusk позволяют вам выразительно определять действия, которые затем могут быть выполнены на данной странице с помощью одного метода. Страницы также позволяют вам определять псевдонимы для общих селекторов всего приложения или отдельной страницы.

<a name="generating-pages"></a>
### Генерация тестовых страниц

Чтобы сгенерировать класс страницы, выполните команду `dusk:page` Artisan. Все классы страниц будут помещены в каталог `tests/Browser/Pages` вашего приложения:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Конфигурирование тестовых страниц

По умолчанию страницы имеют три метода: `url`, `assert` и `elements`. Сейчас мы обсудим методы `url` и `assert`. Метод `elements` будет [более подробно описан ниже](#shorthand-selectors).

<a name="the-url-method"></a>
#### Метод `url`

Метод `url` должен возвращать путь URL-адреса, представляющего страницу. Dusk будет использовать этот URL-адрес при переходе на страницу в браузере:

    /**
     * Получить URL-адрес страницы.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

<a name="the-assert-method"></a>
#### Метод `assert`

Метод `assert` может делать любые утверждения, необходимые для подтверждения того, что браузер действительно находится на данной странице. На самом деле нет необходимости размещать что-либо в этом методе; однако вы можете сделать эти утверждения, если хотите. Эти утверждения будут запускаться автоматически при переходе на страницу:

    /**
     * Подтвердить, что браузер находится на странице.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Навигация по тестовым страницам

После того, как страница определена, вы можете посетить ее с помощью метода `visit`:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

Представим, что уже находясь на какой-либо странице, вам необходимо «загрузить» селекторы и методы страницы в текущий контекст теста. Это обычное явление, когда вы нажимаете кнопку и перенаправляетесь на указанную страницу без явного перехода к ней. В этой ситуации вы можете использовать метод `on` для загрузки страницы:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Псевдонимы селекторов

Метод `elements` внутри классов страниц позволяет вам определять быстрые, легко запоминающиеся псевдонимы для любого селектора CSS на вашей странице. Например, давайте определим псевдоним для поля ввода «электронная почта» на странице входа в приложение:

    /**
     * Получить псевдонимы элементов страницы.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

После того, как псевдоним был определен, вы можете использовать сокращенный селектор в любом месте, где вы обычно используете полный селектор CSS:

    $browser->type('@email', 'taylor@laravel.com');

<a name="global-shorthand-selectors"></a>
#### Глобальные псевдонимы селекторов

После установки Dusk базовый класс `Page` будет помещен в ваш каталог `tests/Browser/Pages`. Этот класс содержит метод `siteElements`, который можно использовать для определения глобальных псевдонимов селекторов, которые должны быть доступны на каждой странице вашего приложения:

    /**
     * Получить глобальные псевдонимы элементов сайта.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Методы тестовых страниц

В дополнение к методам по умолчанию, определенным на тестовых страницах, вы можете определить дополнительные методы, которые могут использоваться в ваших тестах. Например, представим, что мы создаем приложение для управления музыкой. Обычным действием для одной страницы приложения может быть создание списка воспроизведения. Вместо того чтобы переписывать логику создания списка воспроизведения в каждом тесте, вы можете определить пользовательский метод `createPlaylist` в классе страницы:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Другие методы страницы ...

        /**
         * Создать новый список воспроизведения.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Как только метод определен, вы можете использовать его в любом тесте, использующем данную страницу. Экземпляр браузера будет автоматически внедрен в качестве первого аргумента пользовательским методам страницы:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## Компоненты для тестов

Компоненты похожи на «классы страниц» Dusk, но предназначены для частей пользовательского интерфейса и функций, которые повторно используются в вашем приложении, таких как панель навигации или окно уведомлений. Таким образом, компоненты не привязаны к конкретным URL-адресам.

<a name="generating-components"></a>
### Генерация компонентов

Чтобы сгенерировать компонент, выполните команду `dusk:component` Artisan. Новые компоненты будут помещены в каталог `tests/Browser/Components`:

    php artisan dusk:component DatePicker

Компонент «выбора даты» является примером компонента, который может присутствовать в вашем приложении на различных страницах. Может оказаться обременительным вручную написать логику автоматизации браузера для выбора даты в десятках тестов. Вместо этого мы можем определить компонент Dusk для представления элемента выбора даты, что позволит нам инкапсулировать эту логику внутри компонента:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * Получить корневой селектор компонента.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * Подтвердить, что страница браузера содержит компонент.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Получить псевдонимы элементов компонента.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@year-list' => 'div > div.datepicker-years',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Выбрать дату.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $year
         * @param  int  $month
         * @param  int  $day
         * @return void
         */
        public function selectDate(Browser $browser, $year, $month, $day)
        {
            $browser->click('@date-field')
                    ->within('@year-list', function ($browser) use ($year) {
                        $browser->click($year);
                    })
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### Использование компонентов

Как только компонент определен, мы можем легко выбрать дату с помощью элемента выбора даты из любого теста. И, если логика, необходимая для выбора даты, изменится, нам нужно будет только обновить компонент:

    <?php

    namespace Tests\Browser;

    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        /**
         * Отвлеченный пример теста компонента.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(2019, 1, 30);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## Непрерывная интеграция

> {note} Большинство конфигураций непрерывной интеграции Dusk предполагают, что ваше приложение Laravel будет обслуживаться с помощью встроенного сервера разработки PHP на порту `8000`. Поэтому, прежде чем продолжить, вы должны убедиться, что ваша среда непрерывной интеграции имеет значение переменной окружения `APP_URL`, равное `http://127.0.0.1:8000`.

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

Чтобы запустить тесты Dusk на [Heroku CI](https://www.heroku.com/continuous-integration), добавьте следующий пакет сборки и скрипты Google Chrome в свой файл `app.json` Heroku:

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Чтобы запустить тесты Dusk на [Travis CI](https://travis-ci.org), используйте следующую конфигурацию `.travis.yml`. Поскольку Travis CI не является графической средой, то нам нужно будет предпринять некоторые дополнительные шаги, чтобы запустить браузер Chrome. Кроме того, мы будем использовать `php artisan serve` для запуска встроенного веб-сервера PHP:

```yaml
language: php

php:
  - 7.3

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Если вы используете [Github Actions](https://github.com/features/actions) для запуска тестов Dusk, то вы можете использовать следующий конфигурационный файл в качестве отправной точки. Как и в случае с TravisCI, мы будем использовать команду `php artisan serve` для запуска встроенного веб-сервера PHP:

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE 'my-database' character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver `/opt/google/chrome/chrome --version | cut -d " " -f3 | cut -d "." -f1`
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        env:
          APP_URL: "http://127.0.0.1:8000"
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: console
          path: tests/Browser/console
```
