# Laravel 8 · Пакет Laravel Scout

- [Введение](#introduction)
- [Установка](#installation)
    - [Предварительная подготовка драйверов](#driver-prerequisites)
    - [Постановка в очередь](#queueing)
- [Конфигурирование](#configuration)
    - [Настройка индекса модели](#configuring-model-indexes)
    - [Настройка индексируемых сведений](#configuring-searchable-data)
    - [Настройка идентификатора модели](#configuring-the-model-id)
    - [Идентификация пользователей](#identifying-users)
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

Laravel Scout provides a simple, driver based solution for adding full-text search to your [Eloquent models](eloquent.md). Using model observers, Scout will automatically keep your search indexes in sync with your Eloquent records.

Currently, Scout ships with an [Algolia](https://www.algolia.com/) driver; however, writing custom drivers is simple and you are free to extend Scout with your own search implementations.

<a name="installation"></a>
## Установка

First, install Scout via the Composer package manager:

    composer require laravel/scout

After installing Scout, you should publish the Scout configuration file using the `vendor:publish` Artisan command. This command will publish the `scout.php` configuration file to your application's `config` directory:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Finally, add the `Laravel\Scout\Searchable` trait to the model you would like to make searchable. This trait will register a model observer that will automatically keep the model in sync with your search driver:

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

When using the Algolia driver, you should configure your Algolia `id` and `secret` credentials in your `config/scout.php` configuration file. Once your credentials have been configured, you will also need to install the Algolia PHP SDK via the Composer package manager:

    composer require algolia/algoliasearch-client-php

<a name="meilisearch"></a>
#### MeiliSearch

MeiliSearch is a powerful, open source search-engine that may be run locally using [Laravel Sail](sail.md). MeiliSearch provides and maintains an [official MeiliSearch driver for Laravel Scout](https://github.com/meilisearch/meilisearch-laravel-scout). Please consult this package's documentation to learn how to use MeiliSearch with Laravel Scout.

<a name="queueing"></a>
### Постановка в очередь

While not strictly required to use Scout, you should strongly consider configuring a [queue driver](queues.md) before using the library. Running a queue worker will allow Scout to queue all operations that sync your model information to your search indexes, providing much better response times for your application's web interface.

Once you have configured a queue driver, set the value of the `queue` option in your `config/scout.php` configuration file to `true`:

    'queue' => true,

<a name="configuration"></a>
## Конфигурирование

<a name="configuring-model-indexes"></a>
### Настройка индекса модели

Each Eloquent model is synced with a given search "index", which contains all of the searchable records for that model. In other words, you can think of each index like a MySQL table. By default, each model will be persisted to an index matching the model's typical "table" name. Typically, this is the plural form of the model name; however, you are free to customize the model's index by overriding the `searchableAs` method on the model:

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

By default, the entire `toArray` form of a given model will be persisted to its search index. If you would like to customize the data that is synchronized to the search index, you may override the `toSearchableArray` method on the model:

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

By default, Scout will use the primary key of the model as model's unique ID / key that is stored in the search index. If you need to customize this behavior, you may override the `getScoutKey` and the `getScoutKeyName` methods on the model:

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

<a name="identifying-users"></a>
### Идентификация пользователей

Scout also allows you to auto identify users when using [Algolia](https://algolia.com). Associating the authenticated user with search operations may be helpful when viewing your search analytics within Algolia's dashboard. You can enable user identification by defining a `SCOUT_IDENTIFY` environment variable as `true` in your application's `.env` file:

    SCOUT_IDENTIFY=true

Enabling this feature this will also pass the request's IP address and your authenticated user's primary identifier to Algolia so this data is associated with any search request that is made by the user.

<a name="indexing"></a>
## Индексирование

<a name="batch-import"></a>
### Пакетный импорт записей

If you are installing Scout into an existing project, you may already have database records you need to import into your indexes. Scout provides a `scout:import` Artisan command that you may use to import all of your existing records into your search indexes:

    php artisan scout:import "App\Models\Post"

The `flush` command may be used to remove all of a model's records from your search indexes:

    php artisan scout:flush "App\Models\Post"

<a name="modifying-the-import-query"></a>
#### Изменение запроса при импорте записей

If you would like to modify the query that is used to retrieve all of your models for batch importing, you may define a `makeAllSearchableUsing` method on your model. This is a great place to add any eager relationship loading that may be necessary before importing your models:

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

Once you have added the `Laravel\Scout\Searchable` trait to a model, all you need to do is `save` or `create` a model instance and it will automatically be added to your search index. If you have configured Scout to [use queues](#queueing) this operation will be performed in the background by your queue worker:

    use App\Models\Order;

    $order = new Order;

    // ...

    $order->save();

<a name="adding-records-via-query"></a>
#### Добавление записей через запрос

If you would like to add a collection of models to your search index via an Eloquent query, you may chain the `searchable` method onto the Eloquent query. The `searchable` method will [chunk the results](eloquent.md#chunking-results) of the query and add the records to your search index. Again, if you have configured Scout to use queues, all of the chunks will be imported in the background by your queue workers:

    use App\Models\Order;

    Order::where('price', '>', 100)->searchable();

You may also call the `searchable` method on an Eloquent relationship instance:

    $user->orders()->searchable();

Or, if you already have a collection of Eloquent models in memory, you may call the `searchable` method on the collection instance to add the model instances to their corresponding index:

    $orders->searchable();

> {tip} The `searchable` method can be considered an "upsert" operation. In other words, if the model record is already in your index, it will be updated. If it does not exist in the search index, it will be added to the index.

<a name="updating-records"></a>
### Обновление записей

To update a searchable model, you only need to update the model instance's properties and `save` the model to your database. Scout will automatically persist the changes to your search index:

    use App\Models\Order;

    $order = Order::find(1);

    // Обновление заказа ...

    $order->save();

You may also invoke the `searchable` method on an Eloquent query instance to update a collection of models. If the models do not exist in your search index, they will be created:

    Order::where('price', '>', 100)->searchable();

If you would like to update the search index records for all of the models in a relationship, you may invoke the `searchable` on the relationship instance:

    $user->orders()->searchable();

Or, if you already have a collection of Eloquent models in memory, you may call the `searchable` method on the collection instance to update the model instances in their corresponding index:

    $orders->searchable();

<a name="removing-records"></a>
### Удаление записей

To remove a record from your index you may simply `delete` the model from the database. This may be done even if you are using [soft deleted](eloquent.md#soft-deleting) models:

    use App\Models\Order;

    $order = Order::find(1);

    $order->delete();

If you do not want to retrieve the model before deleting the record, you may use the `unsearchable` method on an Eloquent query instance:

    Order::where('price', '>', 100)->unsearchable();

If you would like to remove the search index records for all of the models in a relationship, you may invoke the `unsearchable` on the relationship instance:

    $user->orders()->unsearchable();

Or, if you already have a collection of Eloquent models in memory, you may call the `unsearchable` method on the collection instance to remove the model instances from their corresponding index:

    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Приостановка индексирования

Sometimes you may need to perform a batch of Eloquent operations on a model without syncing the model data to your search index. You may do this using the `withoutSyncingToSearch` method. This method accepts a single closure which will be immediately executed. Any model operations that occur within the closure will not be synced to the model's index:

    use App\Models\Order;

    Order::withoutSyncingToSearch(function () {
        // Выполнение действий с моделью ...
    });

<a name="conditionally-searchable-model-instances"></a>
### Условное индексирование экземпляров модели

Sometimes you may need to only make a model searchable under certain conditions. For example, imagine you have `App\Models\Post` model that may be in one of two states: "draft" and "published". You may only want to allow "published" posts to be searchable. To accomplish this, you may define a `shouldBeSearchable` method on your model:

    /**
     * Определить, должна ли модель индексироваться.
     *
     * @return bool
     */
    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

The `shouldBeSearchable` method is only applied when manipulating models through the `save` and `create` methods, queries, or relationships. Directly making models or collections searchable using the `searchable` method will override the result of the `shouldBeSearchable` method.

<a name="searching"></a>
## Поиск

You may begin searching a model using the `search` method. The search method accepts a single string that will be used to search your models. You should then chain the `get` method onto the search query to retrieve the Eloquent models that match the given search query:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->get();

Since Scout searches return a collection of Eloquent models, you may even return the results directly from a route or controller and they will automatically be converted to JSON:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return Order::search($request->search)->get();
    });

If you would like to get the raw search results before they are converted to Eloquent models, you may use the `raw` method:

    $orders = Order::search('Star Trek')->raw();

<a name="custom-indexes"></a>
#### Изменение индекса

Search queries will typically be performed on the index specified by the model's [`searchableAs`](#configuring-model-indexes) method. However, you may use the `within` method to specify a custom index that should be searched instead:

    $orders = Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Выражения Where

Scout allows you to add simple "where" clauses to your search queries. Currently, these clauses only support basic numeric equality checks and are primarily useful for scoping search queries by an owner ID. Since a search index is not a relational database, more advanced "where" clauses are not currently supported:

    use App\Models\Order;

    $orders = Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Постраничная навигация

In addition to retrieving a collection of models, you may paginate your search results using the `paginate` method. This method will return an `Illuminate\Pagination\LengthAwarePaginator` instance just as if you had [paginated a traditional Eloquent query](pagination.md):

    use App\Models\Order;

    $orders = Order::search('Star Trek')->paginate();

You may specify how many models to retrieve per page by passing the amount as the first argument to the `paginate` method:

    $orders = Order::search('Star Trek')->paginate(15);

Once you have retrieved the results, you may display the results and render the page links using [Blade](blade.md) just as if you had paginated a traditional Eloquent query:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Of course, if you would like to retrieve the pagination results as JSON, you may return the paginator instance directly from a route or controller:

    use App\Models\Order;
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        return Order::search($request->input('query'))->paginate(15);
    });

<a name="soft-deleting"></a>
### Программное удаление

If your indexed models are [soft deleting](eloquent.md#soft-deleting) and you need to search your soft deleted models, set the `soft_delete` option of the `config/scout.php` configuration file to `true`:

    'soft_delete' => true,

When this configuration option is `true`, Scout will not remove soft deleted models from the search index. Instead, it will set a hidden `__soft_deleted` attribute on the indexed record. Then, you may use the `withTrashed` or `onlyTrashed` methods to retrieve the soft deleted records when searching:

    use App\Models\Order;

    // Добавить удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->withTrashed()->get();

    // Вывести только удаленные записи при получении результатов ...
    $orders = Order::search('Star Trek')->onlyTrashed()->get();

> {tip} When a soft deleted model is permanently deleted using `forceDelete`, Scout will remove it from the search index automatically.

<a name="customizing-engine-searches"></a>
### Изменение техники поиска

If you need to perform advanced customization of the search behavior of an engine you may pass a closure as the second argument to the `search` method. For example, you could use this callback to add geo-location data to your search options before the search query is passed to Algolia:

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

<a name="custom-engines"></a>
## Добавление собственных поисковых систем

<a name="writing-the-engine"></a>
#### Написание поисковой системы

If one of the built-in Scout search engines doesn't fit your needs, you may write your own custom engine and register it with Scout. Your engine should extend the `Laravel\Scout\Engines\Engine` abstract class. This abstract class contains eight methods your custom engine must implement:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

You may find it helpful to review the implementations of these methods on the `Laravel\Scout\Engines\AlgoliaEngine` class. This class will provide you with a good starting point for learning how to implement each of these methods in your own engine.

<a name="registering-the-engine"></a>
#### Регистрация поисковой системы

Once you have written your custom engine, you may register it with Scout using the `extend` method of the Scout engine manager. Scout's engine manager may be resolved from the Laravel service container. You should call the `extend` method from the `boot` method of your `App\Providers\AppServiceProvider` class or any other service provider used by your application:

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

Once your engine has been registered, you may specify it as your default Scout `driver` in your application's `config/scout.php` configuration file:

    'driver' => 'mysql',

<a name="builder-macros"></a>
## Макрокоманды построителя поискового запроса

If you would like to define a custom Scout search builder method, you may use the `macro` method on the `Laravel\Scout\Builder` class. Typically, "macros" should be defined within a [service provider's](providers.md) `boot` method:

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

The `macro` function accepts a macro name as its first argument and a closure as its second argument. The macro's closure will be executed when calling the macro name from a `Laravel\Scout\Builder` implementation:

    use App\Models\Order;

    Order::search('Star Trek')->count();
