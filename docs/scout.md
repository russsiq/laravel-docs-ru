# Laravel 9 · Пакет Laravel Scout

- [Введение](#introduction)
- [Установка](#installation)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
    - [Постановка в очередь](#queueing)
- [Конфигурирование](#configuration)
    - [Настройка индекса модели](#configuring-model-indexes)
    - [Настройка индексируемых сведений](#configuring-searchable-data)
    - [Настройка идентификатора модели](#configuring-the-model-id)
    - [Настройка поисковых систем для каждой модели](#configuring-search-engines-per-model)
    - [Идентификация пользователей](#identifying-users)
- [Поисковые системы `database` / `collection`](#database-and-collection-engines)
    - [Поисковая система `database`](#database-engine)
    - [Поисковая система `collection`](#collection-engine)
- [Индексирование](#indexing)
    - [Пакетный импорт записей](#batch-import)
    - [Добавление записей](#adding-records)
    - [Обновление записей](#updating-records)
    - [Удаление записей](#removing-records)
    - [Приостановка индексирования](#pausing-indexing)
    - [Условное индексирование экземпляров модели](#conditionally-searchable-model-instances)
- [Поиск](#searching)
    - [Выражения Where](#where-clauses)
    - [Постраничная навигация](#pagination)
    - [Программное удаление](#soft-deleting)
    - [Изменение техники поиска](#customizing-engine-searches)
- [Добавление собственных поисковых систем](#custom-engines)
- [Макрокоманды построителя поискового запроса](#builder-macros)

<a name="introduction"></a>
## Введение

[Laravel Scout](https://github.com/laravel/scout) предлагает простое решение на основе драйверов для добавления полнотекстового поиска вашим [моделям Eloquent](eloquent.md). Используя наблюдателей моделей, Scout будет автоматически синхронизировать ваши поисковые индексы с вашими записями Eloquent.

В настоящее время Scout поставляется с драйверами [Algolia](https://www.algolia.com/), [MeiliSearch](https://www.meilisearch.com) и MySQL / PostgreSQL (`database`). Кроме того, Scout включает в себя драйвер `collection`, предназначенный для использования при локальной разработке и не требующий каких-либо внешних зависимостей или сторонних сервисов. Более того, написать свой собственный драйвер просто, и вы можете расширить Scout собственной реализацией поиска.

<a name="installation"></a>
## Установка

Для начала установите Scout с помощью менеджера пакетов Composer в свой проект:

```shell
composer require laravel/scout
```

После установки Scout вы должны опубликовать конфигурационный файл Scout с помощью команды `vendor:publish` Artisan. Эта команда опубликует конфигурационный файл `scout.php` в каталоге `config` вашего приложения:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Наконец, добавьте трейт `Laravel\Scout\Searchable` модели, которую вы хотите сделать доступной для поиска. Этот трейт зарегистрирует наблюдателя модели, который будет автоматически синхронизировать модель с вашим драйвером поиска:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="driver-prerequisites"></a>
### Предварительная подготовка драйверов

<a name="algolia"></a>
#### Algolia

При использовании драйвера Algolia вы должны указать свои учетные данные Algolia `id` и `secret` в конфигурационном файле `config/scout.php`. После того, как ваши учетные данные будут указаны, вам также необходимо будет установить Algolia PHP SDK с помощью менеджера пакетов Composer:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
#### MeiliSearch

[MeiliSearch](https://www.meilisearch.com) это невероятно быстрая поисковая система с открытым исходным кодом. Если вы не знаете, как установить MeiliSearch на свой локальный компьютер, то вы можете использовать [Laravel Sail](sail.md#meilisearch), официально поддерживаемая среда разработки Docker.

При использовании драйвера MeiliSearch вам необходимо установить PHP SDK MeiliSearch с помощью менеджера пакетов Composer:

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Затем укажите переменную окружения `SCOUT_DRIVER`, а также ваши учетные данные `host` и `key` MeiliSearch в файле `.env` вашего приложения:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Для получения дополнительной информации о MeiliSearch, пожалуйста, обратитесь к [документации MeiliSearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Кроме того, вам следует убедиться, что вы установили версию `meilisearch/meilisearch-php`, совместимую с вашей версией MeiliSearch, просмотрев [документацию MeiliSearch о совместимости](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).

> {note} При обновлении Scout в приложении, которое использует MeiliSearch, вы всегда должны [просматривать любые дополнительные критические изменения](https://github.com/meilisearch/MeiliSearch/releases) в самой службе MeiliSearch.

<a name="queueing"></a>
### Постановка в очередь

Хотя это и не является строго обязательным, но вам следует подумать о настройке [драйвера очереди](queues.md) перед использованием пакета Scout. Запуск обработчика очереди позволит Scout ставить в очередь все операции, которые синхронизируют информацию вашей модели с вашими поисковыми индексами, что обеспечивает гораздо лучшее время отклика для веб-интерфейса вашего приложения.

После того, как вы настроили драйвер очереди, установите значение `true` для параметра `queue` в вашем конфигурационном файле `config/scout.php`:

    'queue' => true,

Даже когда для параметра `queue` установлено значение `false`, важно помнить, что некоторые драйверы Scout, такие как Algolia и MeiliSearch, всегда индексируют записи асинхронно. Это означает, что даже если операция индексирования завершена в вашем приложении Laravel, то сама поисковая система может не сразу отображать новые и обновленные записи.

<a name="configuration"></a>
## Конфигурирование

<a name="configuring-model-indexes"></a>
### Настройка индекса модели

Каждая модель Eloquent синхронизируется с конкретным поисковым «индексом», который содержит все доступные для поиска записи данной модели. Другими словами, вы можете думать о каждом индексе как о таблице MySQL. По умолчанию каждая модель будет сохранена в индексе, соответствующем типичному «табличному» имени модели. Обычно это форма множественного числа от названия модели; однако вы можете изменить индекс модели, переопределив метод `searchableAs` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Получить имя индекса, связанного с моделью.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Настройка индексируемых сведений

По умолчанию все результаты метода `toArray` модели будут сохранены в поисковом индексе. Если вы хотите изменить синхронизируемые с поисковым индексом данные, то вы можете переопределить метод `toSearchableArray` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * Получить индексируемый массив данных модели.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Изменение массива данных ...

            return $array;
        }
    }

<a name="configuring-the-model-id"></a>
### Настройка идентификатора модели

По умолчанию Scout будет использовать первичный ключ модели в качестве уникального идентификатора / ключа модели для сохранения в поисковом индексе. Если вам нужно изменить это поведение, то вы можете переопределить методы `getScoutKey` и `getScoutKeyName` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Получить значение ключа, используемое для индексации модели.
         *
         * @return mixed
         */
        public function getScoutKey()
        {
            return $this->email;
        }

        /**
         * Получить имя ключа, используемое для индексации модели.
         *
         * @return mixed
         */
        public function getScoutKeyName()
        {
            return 'email';
        }
    }

<a name="configuring-search-engines-per-model"></a>
### Настройка поисковых систем для каждой модели

При поиске Scout обычно использует поисковую систему по умолчанию, указанную в конфигурационном файле `scout` вашего приложения. Однако поисковую систему для конкретной модели можно изменить, переопределив метод `searchableUsing` модели:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\EngineManager;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * Получить поисковую систему, используемую для индексации модели.
         *
         * @return \Laravel\Scout\Engines\Engine
         */
        public function searchableUsing()
        {
            return app(EngineManager::class)->engine('meilisearch');
        }
    }

<a name="identifying-users"></a>
### Идентификация пользователей

Scout также позволяет автоматически идентифицировать пользователей при использовании [Algolia](https://algolia.com). Связывание аутентифицированного пользователя с операциями поиска может быть полезно при просмотре аналитики поиска в панели управления Algolia. Вы можете задействовать идентификацию пользователя, определив значение `true` для переменной окружения `SCOUT_IDENTIFY` в файле `.env` вашего приложения:

```ini
SCOUT_IDENTIFY=true
```

С помощью данного функционала, дополнительно будет переданы IP-адрес запроса и основной идентификатор вашего аутентифицированного пользователя в Algolia, таким образом, эти данные будут связаны с любым поисковым запросом, сделанным пользователем.

<a name="database-and-collection-engines"></a>
## Поисковые системы `database` / `collection`

<a name="database-engine"></a>
### Поисковая система `database`

> {note} Поисковая система `database` в настоящее время поддерживает MySQL и PostgreSQL.

Если ваше приложение взаимодействует с базами данных малого и среднего размера или имеет небольшую рабочую нагрузку, то вам может быть удобнее начать работу с поисковой системой `database` Scout. Поисковая система базы данных будет использовать выражения `where like` и полнотекстовые индексы при фильтрации результатов из вашей существующей базы данных, чтобы определить применимые результаты поиска для вашего запроса.

Чтобы использовать поисковую систему базы данных, вы можете просто установить значение переменной окружения `SCOUT_DRIVER` в значение `database` или указать драйвер `database` непосредственно в конфигурационном файле `scout` вашего приложения:

```ini
SCOUT_DRIVER=database
```

После того как вы указали поисковую систему базы данных в качестве предпочитаемого драйвера, вы должны [настроить доступные для поиска данные](#configuring-searchable-data). Затем вы можете начать [выполнение поисковых запросов](#searching) по вашим моделям. Индексация поисковой системы, которая необходима для заполнения индексов Algolia или MeiliSearch, не требуется при использовании поисковой системы базы данных.

#### Изменение техники поиска с использованием драйвера `database`

По умолчанию поисковая система базы данных будет выполнять запрос `where like` для каждого атрибута модели, который вы [настроили как доступный для поиска](#configuring-searchable-data). Однако в некоторых ситуациях это может привести к снижению производительности. Таким образом, стратегия поиска поисковой системы базы данных может быть настроена таким образом, чтобы некоторые указанные столбцы использовали запросы полнотекстового поиска или использовали только ограничения `where like` для поиска префиксов строк (`example%`) вместо поиска по всей строке (`%example%`).

Чтобы определить это поведение, вы можете назначить атрибуты PHP методу `toSearchableArray` вашей модели. Любые столбцы, которым не назначено дополнительное поведение стратегии поиска, будут по-прежнему использовать стратегию `where like` по умолчанию:

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Получить индексируемый массив данных модели.
 *
 * @return array
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray()
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> {note} Прежде чем указать, что столбец должен использовать ограничения полнотекстового запроса, убедитесь, что столбцу назначен [полнотекстовый индекс](migrations.ms#available-index-types).

<a name="collection-engine"></a>
### Поисковая система `collection`

Хотя вы можете использовать поисковые системы Algolia или MeiliSearch во время локальной разработки, вам может быть удобнее начать работу с поисковой системой `collection`. Поисковая система будет использовать выражения `where` и фильтрацию набора результатов из вашей существующей базы данных, чтобы определить подходящие результаты поиска по вашему запросу. При использовании этого системы нет необходимости «индексировать» ваши доступные для поиска модели, поскольку они будут просто извлечены из вашей локальной базы данных.

Чтобы использовать данную поисковую систему, вы можете просто установить для переменной окружения `SCOUT_DRIVER` значение `collection` или указать драйвер `collection` непосредственно в конфигурационном файле `scout` вашего приложения:

```ini
SCOUT_DRIVER=collection
```

После того, как вы указали драйвер `collection` в качестве предпочтительного драйвера, вы можете начать [выполнение поисковых запросов](#searching) по вашим моделям. Индексирование поисковой системы, которое необходимо для заполнения Algolia или MeiliSearch, не требуется при использовании поисковой системы `collection`.

#### Различия поисковых систем `database` и `collection`

На первый взгляд, поисковые системы `database` и `collection` очень похожи. Оба они взаимодействуют непосредственно с вашей базой данных для получения результатов поиска. Однако механизм `collection` не использует полнотекстовые индексы или выражения `LIKE` для поиска записей. Вместо этого он извлекает все возможные записи и использует помощник `Str::is` Laravel, чтобы определить, существует ли строка поиска в значениях атрибута модели.

Поисковая система `collection` является наиболее портативной поисковой системой, поскольку она работает со всеми реляционными базами данных, поддерживаемыми Laravel (включая SQLite и SQL Server); однако она менее эффективна, чем поисковая система `database` Scout.

<a name="indexing"></a>
## Индексирование

<a name="batch-import"></a>
### Пакетный импорт записей

Если вы устанавливаете Scout в существующий проект, то возможно, у вас уже есть записи базы данных, которые необходимо импортировать в соответствующие индексы. Scout содержит команду `scout:import` Artisan, которую вы можете использовать для импорта всех ваших существующих записей в соответствующие поисковые индексы:

```shell
php artisan scout:import "App\Models\Post"
```

Команду `flush` можно использовать для удаления всех записей модели из поисковых индексов:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Изменение запроса при импорте записей

Если вы хотите изменить запрос, используемый для получения всех ваших моделей при пакетном импорте, то вы можете определить метод `makeAllSearchableUsing` своей модели. Это отличное место для добавления любой [нетерпеливой загрузки отношений](eloquent-relationships.md#eager-loading), которая может потребоваться перед импортом ваших моделей:

    /**
     * Изменить запрос, используемый для получения всех индексируемых моделей.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    protected function makeAllSearchableUsing($query)
    {
        return $query->with('author');
    }

<a name="adding-records"></a>
### Добавление записей

После добавления в модель трейта `Laravel\Scout\Searchable`, все, что вам нужно сделать, это вызвать методы `save` или `create` экземпляра модели, и она будет автоматически добавлена в ваш поисковый индекс. Если вы настроили [использование очередей](#queueing), то эта операция будет выполняться в фоновом режиме вашим обработчиком очереди:

    use App\Models\Order;

    $order = new Order;

    // ...

    $order->save();

<a name="adding-records-via-query"></a>
#### Добавление записей через запрос

Если вы хотите добавить коллекцию моделей в поисковый индекс с помощью запроса Eloquent, то вы можете связать метод `searchable` с запросом Eloquent. Метод `searchable` [разбивает результаты](eloquent.md#chunking-results) запроса и добавляет записи в поисковый индекс. Опять же, если вы настроили использование очередей, то все подмножества моделей будут импортированы в фоновом режиме вашим обработчиком очереди:

    use App\Models\Order;

    Order::where('price', '>', 100)->searchable();

Вы также можете вызвать метод `searchable` экземпляра отношения Eloquent:

    $user->orders()->searchable();

Или, если у вас уже имеется в наличии коллекция моделей Eloquent, то вы можете вызвать метод `searchable` экземпляра коллекции, чтобы добавить экземпляры модели в соответствующий индекс:

    $orders->searchable();

> {tip} Метод `searchable` можно считать операцией «обновления-вставки». Другими словами, если запись модели уже есть в вашем индексе, то она будет обновлена. Если записи нет в поисковом индексе, то она будет добавлен в индекс.

<a name="updating-records"></a>
### Обновление записей

Чтобы обновить модель в поиске, то вам нужно только обновить свойства экземпляра модели и вызвать метод `save` модели. Scout автоматически сохранит изменения в вашем поисковом индексе:

    use App\Models\Order;

    $order = Order::find(1);

    // Обновление заказа ...

    $order->save();

Вы также можете вызвать метод `searchable` экземпляра запроса Eloquent, чтобы обновить коллекцию моделей. Если моделей нет в поисковом индексе, то они будут созданы:

    Order::where('price', '>', 100)->searchable();

Если вы хотите обновить записи поискового индекса для всех моделей отношения, то вы можете вызвать `searchable` экземпляра отношения:

    $user->orders()->searchable();

Или, если у вас уже имеется в наличии коллекция моделей Eloquent, то вы можете вызвать метод `searchable` экземпляра коллекции, чтобы обновить экземпляры модели в соответствующем индексе:

    $orders->searchable();

<a name="removing-records"></a>
### Удаление записей

Чтобы удалить запись из поискового индекса с сопутствующим удалением из базы данных, то вы можете просто вызвать метод `delete` модели. Это можно сделать, даже если вы используете [программное удаление](eloquent.md#soft-deleting) моделей:

    use App\Models\Order;

    $order = Order::find(1);

    $order->delete();

Если вы не хотите извлекать модель перед удалением записи, то вы можете использовать метод `unsearchable` экземпляра запроса Eloquent:

    Order::where('price', '>', 100)->unsearchable();

Если вы хотите удалить записи поискового индекса для всех моделей в отношении, то вы можете вызвать `unsearchable` экземпляра отношения:

    $user->orders()->unsearchable();

Или, если у вас уже имеется в наличии коллекция моделей Eloquent, то вы можете вызвать метод `unsearchable` экземпляра коллекции, чтобы удалить экземпляры модели из соответствующего индекса:

    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Приостановка индексирования

Иногда требуется выполнить пакет действий Eloquent с моделью без синхронизации данных модели с поисковым индексом. Вы можете сделать это, используя метод `withoutSyncingToSearch`. Этот метод принимает замыкание, которое будет немедленно выполнено. Любые операции модели, выполняемые внутри замыкания, не будут синхронизированы с поисковым индексом модели:

    use App\Models\Order;

    Order::withoutSyncingToSearch(function () {
        // Выполнение действий с моделью ...
    });

<a name="conditionally-searchable-model-instances"></a>
### Условное индексирование экземпляров модели

Иногда требуется сделать модель доступной для поиска только при определенных условиях. Например, представьте, что у вас есть модель поста `App\Models\Post`, который может находиться в одном из двух состояний: «черновик» и «опубликован». Вы можете разрешить поиск только «опубликованных» постов. Для этого вы можете определить метод `shouldBeSearchable` модели:

    /**
     * Определить, должна ли модель индексироваться.
     *
     * @return bool
     */
    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

Метод `shouldBeSearchable` учитывается при манипулировании моделями через запросы, отношения или методы `save` и `create`. Непосредственное указание доступности моделей или коллекций для поиска с помощью метода `searchable` переопределит результат метода `shouldBeSearchable`.

> {note} Метод `shouldBeSearchable` неприменим при использовании поисковой системы `database` Scout, поскольку все доступные для поиска данные всегда хранятся в базе данных. Чтобы добиться аналогичного поведения при использовании поисковой системы `database`, вместо этого следует использовать [выражения `where`](#where-clauses).

<a name="searching"></a>
## Поиск

Вы можете начать поиск модели, используя метод `search`. Метод `search` принимает одну строку, которая будет использоваться для поиска ваших моделей. Затем вы должны связать метод `get` с поисковым запросом, чтобы получить модели Eloquent, соответствующие указанному поисковому запросу:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->get();

Поскольку поисковые запросы Scout возвращают коллекцию моделей Eloquent, то вы даже можете возвращать результаты непосредственно из маршрута или контроллера, и они будут автоматически преобразованы в JSON:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return Order::search($request->search)->get();
    });

Если вы хотите получить необработанные результаты поиска до того, как они будут преобразованы в модели Eloquent, то вы можете использовать метод `raw`:

    $orders = Order::search('Star Trek')->raw();

<a name="custom-indexes"></a>
#### Изменение индекса

Поисковые запросы обычно выполняются по индексу, указанному в методе [`searchableAs`](#configuring-model-indexes) модели. Однако вы можете использовать метод `within`, чтобы указать необходимый поисковый индекс:

    $orders = Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Выражения Where

Scout позволяет добавлять к поисковым запросам простые выражения `WHERE`. В настоящее время эти выражения поддерживают только базовые проверки числового равенства и в первую очередь полезны для определения области поисковых запросов по идентификатору владельца:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->where('user_id', 1)->get();

Вы можете использовать метод `whereIn`, чтобы ограничить результаты заданным набором значений:

    $orders = Order::search('Star Trek')->whereIn(
        'status', ['paid', 'open']
    )->get();

Поскольку поисковый индекс не является реляционной базой данных, более сложные выражения `WHERE` в настоящее время не поддерживаются.

<a name="pagination"></a>
### Постраничная навигация

Помимо получения коллекции моделей, вы можете разбить результаты поиска постранично, используя метод `paginate`. Этот метод вернет экземпляр `Illuminate\Pagination\LengthAwarePaginator`, как если бы это был [обычный постраничный запрос Eloquent](pagination.md):

    use App\Models\Order;

    $orders = Order::search('Star Trek')->paginate();

Вы можете указать, сколько моделей необходимо извлекать на каждой странице, передав их количество методу `paginate`:

    $orders = Order::search('Star Trek')->paginate(15);

Получив результаты, вы можете отобразить их, а также ссылки на страницы с помощью [Blade](blade.md), как если бы это был обычный постраничный запрос Eloquent:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Конечно, если вы хотите получить результаты постраничной разбивки в виде JSON, то вы можете вернуть экземпляр пагинатора прямо из маршрута или контроллера:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        return Order::search($request->input('query'))->paginate(15);
    });

<a name="soft-deleting"></a>
### Программное удаление

Если ваши индексируемые модели поддерживают [программное удаление](eloquent.md#soft-deleting) и вам необходимо выполнять поиск по таким моделям, то установите значение `true` для параметра `soft_delete` конфигурационного файла `config/scout.php`:

    'soft_delete' => true,

Когда этот параметр конфигурации имеет значение `true`, тогда Scout не будет удалять такие модели из поискового индекса. Вместо этого он установит скрытый атрибут `__soft_deleted` для проиндексированной записи. Затем вы можете использовать методы `withTrashed` или `onlyTrashed` для взаимодействия с программно удаленными записями при поиске:

    use App\Models\Order;

    // Добавить удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->withTrashed()->get();

    // Вывести только удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->onlyTrashed()->get();

> {tip} Если модель будет окончательно удалена с помощью `forceDelete`, то Scout автоматически удалит ее из поискового индекса.

<a name="customizing-engine-searches"></a>
### Изменение техники поиска

Если вам нужно выполнить расширенный поиск, доступный для поисковой системы, то вы можете передать замыкание в качестве второго аргумента методу `search`. Например, вы можете использовать это замыкание, чтобы добавить данные о геолокации в параметры поиска до того, как поисковый запрос будет передан в Algolia:

    use Algolia\AlgoliaSearch\SearchIndex;
    use App\Models\Order;

    Order::search(
        'Star Trek',
        function (SearchIndex $algolia, string $query, array $options) {
            $options['body']['query']['bool']['filter']['geo_distance'] = [
                'distance' => '1000km',
                'location' => ['lat' => 36, 'lon' => 111],
            ];

            return $algolia->search($query, $options);
        }
    )->get();

<a name="customizing-the-eloquent-results-query"></a>
#### Настройка запроса результатов Eloquent

После того, как Scout получит список соответствующих запросу моделей Eloquent из поисковой системы вашего приложения, Eloquent используется для получения всех соответствующих запросу моделей по их первичным ключам. Вы можете тонко настроить этот запрос, вызвав метод `query`. Метод `query` принимает замыкание, которое получит в качестве аргумента экземпляр построителя запросов Eloquent:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')
    ->query(fn ($query) => $query->with('invoices'))
    ->get();
```

Поскольку это замыкание вызывается после того, как соответствующие модели уже получены из поисковой системы вашего приложения, метод `query` не следует использовать для «фильтрации» результатов. Вместо этого вы должны использовать [выражения «where» Scout](#where-clauses).

<a name="custom-engines"></a>
## Добавление собственных поисковых систем

<a name="writing-the-engine"></a>
#### Написание поисковой системы

Если ни одна из встроенных поисковых систем Scout не соответствует вашим потребностям, то вы можете написать свою собственную поисковую систему и зарегистрировать ее в Scout. Ваша система должна расширять абстрактный класс `Laravel\Scout\Engines\Engine`. Этот абстрактный класс содержит восемь методов, которые должны быть реализованы в вашей поисковой системе:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

Вы можете просмотреть реализации этих методов в классе `Laravel\Scout\Engines\AlgoliaEngine`. Этот класс предоставит вам хорошую отправную точку в изучении того, как можно реализовать каждый из этих методов в своей поисковой системе.

<a name="registering-the-engine"></a>
#### Регистрация поисковой системы

После реализации поисковой системы вы можете зарегистрировать ее в Scout, используя метод `extend` менеджера поисковых систем Scout. Менеджер поисковых систем Scout может быть извлечен из контейнера служб Laravel. Вы должны вызвать метод `extend` в методе `boot` поставщика `App\Providers\AppServiceProvider` или любого другого поставщика служб, используемого вашим приложением:

    use App\ScoutExtensions\MySqlSearchEngine
    use Laravel\Scout\EngineManager;

    /**
     * Загрузка любых служб приложения.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

После регистрации вашей поисковой системы вы можете указать ее в параметре `driver` конфигурационного файла `config/scout.php` вашего приложения в качестве драйвера, используемого по умолчанию в Scout:

    'driver' => 'mysql',

<a name="builder-macros"></a>
## Макрокоманды построителя поискового запроса

Если вы хотите определить собственный метод построителя поискового запроса Scout, то вы можете использовать метод `macro` класса `Laravel\Scout\Builder`. Как правило, вызов этого метода осуществляется в методе `boot` одного из [поставщиков служб](providers.md) вашего приложения:

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Scout\Builder;

    /**
     * Загрузка любых служб приложения.
     *
     * @return void
     */
    public function boot()
    {
        Builder::macro('count', function () {
            return $this->engine()->getTotalCount(
                $this->engine()->search($this)
            );
        });
    }

Метод `macro` принимает имя макрокоманды как свой первый аргумент и замыкание – как второй аргумент. Замыкание макрокоманды будет выполнено при вызове имени макрокоманды из реализации `Laravel\Scout\Builder`:

    use App\Models\Order;

    Order::search('Star Trek')->count();
