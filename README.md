## Документация Laravel

Вольный перевод [репозитория](https://github.com/laravel/docs/tree/8.x) документации **laravel/docs** ветка 8.x.

Ссылка на [лицензию](https://github.com/laravel/docs/blob/8.x/license.md) оригинала документации **laravel/docs**.

Перед чтением документации ознакомьтесь с [соглашением по переводу](#dictionary).

### Содержание документации

Перевод документации пополняется по мере обращения к соответствующим разделам.

- ## Prologue
    - [Release Notes](/docs/releases)
    - [Upgrade Guide](/docs/upgrade)
    - [Contribution Guide](/docs/contributions)
- ## Getting Started
    - [Installation](/docs/installation)
    - [Configuration](/docs/configuration)
    - [Directory Structure](/docs/structure)
    - [Homestead](/docs/homestead)
    - [Valet](/docs/valet)
    - [Deployment](/docs/deployment)
- ## Architecture Concepts
    - [Request Lifecycle](/docs/lifecycle)
    - [Service Container](/docs/container)
    - [Service Providers](/docs/providers)
    - [Facades](/docs/facades)
    - [Contracts](/docs/contracts)
- ## The Basics
    - [Routing](/docs/routing)
    - [Middleware](/docs/middleware)
    - [CSRF Protection](/docs/csrf)
    - [Controllers](/docs/controllers)
    - [Requests](/docs/requests)
    - [Responses](/docs/responses)
    - [Views](/docs/views)
    - [URL Generation](/docs/urls)
    - [Session](/docs/session)
    - [Validation](/docs/validation)
    - [Error Handling](/docs/errors)
    - [Logging](/docs/logging)
- ## Frontend
    - [Blade Templates](/docs/blade)
    - [Localization](/docs/localization)
    - [Compiling Assets](/docs/mix)
- ## Security
    - [Authentication](/docs/authentication)
    - [Authorization](/docs/authorization)
    - [Email Verification](/docs/verification)
    - [Encryption](/docs/encryption)
    - [Hashing](/docs/hashing)
    - [Password Reset](/docs/passwords)
- ## Digging Deeper
    - [Artisan Console](/docs/artisan)
    - [Broadcasting](/docs/broadcasting)
    - [Cache](/docs/cache)
    - [Collections](/docs/collections)
    - [Events](/docs/events)
    - [File Storage](/docs/filesystem)
    - [Helpers](/docs/helpers)
    - [HTTP Client](/docs/http-client)
    - [Mail](/docs/mail)
    - [Notifications](/docs/notifications)
    - [Package Development](/docs/packages)
    - [Queues](/docs/queues)
    - [Task Scheduling](/docs/scheduling)
- ## Database
    - [Getting Started](/docs/database)
    - [Query Builder](/docs/queries)
    - [Pagination](/docs/pagination)
    - [Migrations](/docs/migrations)
    - [Seeding](/docs/seeding)
    - [Redis](/docs/redis)
- ## Eloquent ORM
    - [Getting Started](/docs/eloquent)
    - [Relationships](/docs/eloquent-relationships)
    - [Collections](/docs/eloquent-collections)
    - [Mutators](/docs/eloquent-mutators)
    - [API Resources](/docs/eloquent-resources)
    - [Serialization](/docs/eloquent-serialization)
- ## Testing
    - [Getting Started](/docs/testing)
    - [HTTP Tests](/docs/http-tests)
    - [Console Tests](/docs/console-tests)
    - [Browser Tests](/docs/dusk)
    - [Database](/docs/database-testing)
    - [Mocking](/docs/mocking)
- ## Packages
    - [Cashier (Stripe)](/docs/billing)
    - [Cashier (Paddle)](/docs/cashier-paddle)
    - [Cashier (Mollie)](https://github.com/laravel/cashier-mollie)
    - [Dusk](/docs/dusk)
    - [Envoy](/docs/envoy)
    - [Horizon](/docs/horizon)
    - [Jetstream](https://jetstream.laravel.com)
    - [Passport](/docs/passport)
    - [Sanctum](/docs/sanctum)
    - [Scout](/docs/scout)
    - [Socialite](/docs/socialite)
    - [Telescope](/docs/telescope)
- [API Documentation](https://laravel.com/api/8.x/)

<a name="dictionary"></a>
### Соглашения по переводу

В следующей таблице перечислены варианты принятых переводов некоторых терминов и словосочетаний, используемых в переводе документации для общей согласованности.

**Оглавление**
0-9
[A](#dictionary-a)
B
[C](#dictionary-c)
[D](#dictionary-d)
[E](#dictionary-e)
[F](#dictionary-f)
[G](#dictionary-g)
[H](#dictionary-h)
I
[J](#dictionary-j)
K
[L](#dictionary-l)
[M](#dictionary-m)
[N](#dictionary-n)
[O](#dictionary-o)
[P](#dictionary-p)
[Q](#dictionary-q)
[R](#dictionary-r)
[S](#dictionary-s)
[T](#dictionary-t)
U
[V](#dictionary-v)
[W](#dictionary-w)
X
Y
Z

Исходный вариант  |  Вариации перевода  |  Замечания
----------------- | ------------------- | -------------
<a name="dictionary-a">**A**</a>
abilities  |  полномочия, компетенции  |  
accessors  |  аксессоры  |  
argument  |  аргумент  |  
asset  |  исходник  |  
assets  |  исходники  |  Директория исходных файлов проекта для фронтенда
attach a relation  |  назначить отношение  |  
attaching  |  прикрепление  |  
attempting  |  пытается  |  
authentication  |  аутентификация  |  
authorization  |  авторизация  |  
auto-incrementing ID  |  автоинкрементный идентификатор  |  
<a name="dictionary-b">**B**</a>
bag, message bag, attribute bag  |  коллекция, коллекция сообщений, коллекция атрибутов  |  
binding  |  связывание  |  
<a name="dictionary-c">**C**</a>
cache  |  кэш  |  
casting  |  типизация  |  
cast type  |  типизатор  |  
chunking  |  разбиение  |  
clauses  |  условия  |  
сlosure  |  замыкание  |  
command line interface (CLI)  |  интерфейс командной строки  |  
condition  |  условие  |  
conditional  |  кондиционный  |  
configuration  |  конфигурация, конфигурирование  |  
constraints  |  ограничения, условия  |  
contract  |  контракт  |  
custom  |  пользовательский, желаемый  |  
customization  |  настройка  |  
customizing  |  корректировка  |  
<a name="dictionary-d">**D**</a>
defining  |  определение  |  
deployment  |  развертывание  |  
detaching  |  отсоединение  |  
dump  |  вывод, отображение  |  
<a name="dictionary-e">**E**</a>
eager  |  нетерпеливый  |  
encrypter  |  шифровальщик  |  
encryption  |  шифрование  |  
environment  |  среда, окружение  |  
evaluated  |  проанализированы  |  
event  |  событие  |  
<a name="dictionary-f">**F**</a>
facade  |  фасад  |  
factory callbacks  |  хуки фабрики  |  
fake  |  фальсификация, фальшивка  |  
foreign key constraints  |  ограничения внешнего ключа  |  
flash data  |  флеш-данные  |  Данные, имеющие непродолжительный срок существования
flashed to the session  |  записаны (краткосрочно) в сессию  |  
<a name="dictionary-g">**G**</a>
given  |  переданный, указанный  |  
guard  |  охранник (аутентификатора)  |  
guarding  |  защита  |  
<a name="dictionary-h">**H**</a>
hashing  |  хеширование  |  
helpers  |  помощники, глобальные вспомогательные функции  |  
<a name="dictionary-j">**J**</a>
job  |  задание  |  
<a name="dictionary-l">**L**</a>
layer over  |  обертка над  |  
lazy  |  отложенный  |  
listener  |  слушатель  |  
log  |  журнал  |  
logger  |  регистратор  |  
logging  |  логирование  |  
localization  |  локализация  |  
<a name="dictionary-m">**M**</a>
making  |  инициализация  |  Например, метод фабрики `make` инициализирует, но не создает запись о модели в БД.
mass assignment  |  массовое присвоение  |  
middleware  |  посредник  |  
migration  |  миграция  |  
miscellaneous  |  разное  |  
mock  |  подставной объект  |  
mocking  |  имитация  |  
mutator  |  мутатор  |  
<a name="dictionary-n">**N**</a>
named error bag  |  массив именованных ошибок  |  
<a name="dictionary-o">**O**</a>
observers  |  наблюдатели  |  
options  |  параметры  |  
overriding  |  переопределение  |  
<a name="dictionary-p">**P**</a>
pagination  |  постраничная навигация  |  
placeholder  |  заполнитель, метка-заполнитель, символ-заполнитель, заменитель  |  
policy  |  политика  |  
production  |  эксплуатация  |  
<a name="dictionary-q">**Q**</a>
query builder  |  построитель запросов  |  
query constraints  |  ограничения запроса  |  
queue  |  очередь  |  
<a name="dictionary-r">**R**</a>
redirect  |  перенаправление  |  
redirector  |  перенаправитель  |  
refreshing  |  обновление  |  
reflection  |  рефлексия (отражение)  |  
relationships  |  отношения  |  
replicating  |  репликация, тиражирование  |  
request  |  запрос  |  
resource  |  ресурс  |  Источник информации, например, API-ресурс
resources  |  ресурсы  |  Директория исходных файлов проекта для фронтенда
response  |  ответ  |  
retrieving  |  получение, извлечение  |  
route  |  маршрут  |  
routing  |  маршрутизация  |  
<a name="dictionary-s">**S**</a>
scope  |  область, сфера, диапазон  |  То, что имеет ограничения
seeding  |  наполнение  |  
serialization  |  сериализация  |  
service container  |  контейнер служб  |  
service providers  |  поставщики служб  |  
session  |  сессия  |  
soft deleting  |  программное удаление  |  
spy on an object  |  наблюдать за объектом  |  
specific  |  конкретный, определенный  |  
statement  |  оператор  |  
stub  |  заглушка  |  
<a name="dictionary-t">**T**</a>
terms of service  |  условия использования  |  
timestamp  |  временная метка  |  
toggling  |  переключение  |  
<a name="dictionary-v">**V**</a>
validation  |  валидация, проверка  |  
validator  |  валидатор  |  
vendor  |  поставщик  |  Директория `vendor` с библиотеками от сторонних поставщиков
verb, HTTP  |  HTTP-метод  |  
verification  |  подтверждение  |  
view  |  шаблон, представление  |  
<a name="dictionary-w">**W**</a>
wildcard  |  метасимвол подстановки  |  
