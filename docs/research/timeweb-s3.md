# Timeweb Cloud S3 — официальный ресерч

Проверено 2026-10-10 по официальному сайту и Markdown-версиям документации. Живой аккаунт не использовался, `verified:false`; **[предположение]** — непроверенный контракт/консервативный предел.

## Подключение

Endpoint `https://s3.twcstorage.ru`; регион подписи `ru-1`, SigV4 (также поддерживается V2, этот бинарь использует V4). Актуальные ключи и endpoint смотрите в Dashboard своего бакета. [AWS CLI](https://timeweb.cloud/docs/s3-storage/tools/aws-cli), [Принципы S3](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-guide).

Path-style подтверждён примерами `endpoint/bucket/key` и настройкой `host_base = host_bucket = s3.twcstorage.ru`; профиль выбирает его. **[предположение]** virtual-host для стандартного endpoint не подтверждён; собственный домен — отдельная настройка, presign с ним невозможен. [S3cmd](https://timeweb.cloud/docs/s3-storage/tools/s3cmd), [Presign](https://timeweb.cloud/docs/s3-storage/supported-features/presigned-url).

**[предположение]** несколько зон доступности для одного S3-пула не установлены; `multi_az:false` — отсутствие подтверждения, а не отказоустойчивости. Общий обзор говорит о трёх репликах, актуальная страница создания разделяет тройную репликацию premium и избыточное распределение standard. Используем описание конкретного класса. [Обзор](https://timeweb.cloud/docs/s3-storage), [Создание](https://timeweb.cloud/docs/s3-storage/manage-storage/create-bucket).

## Классы и тарификация

Стандартный, премиальный и холодный — **классы бакетов**. Холодный/премиальный бакет создаётся с `LocationConstraint=ru:ru-1-cold` / `ru:ru-1-standard-ia`; регион SigV4 остаётся `ru-1`. Текущий kit создаёт бакет с настройками по умолчанию; для cold/premium создайте бакет в панели, затем работайте с ним через CLI/MCP. [Создание](https://timeweb.cloud/docs/s3-storage/manage-storage/create-bucket), [Примеры Object Lock](https://timeweb.cloud/docs/s3-storage/supported-features/object-lock).

В официальном каталоге тарифов страницы сервиса (включая SSR-данные калькулятора) идентификаторы `storage_class`: `standard`, `hot`, `cold`; они сохранены как каталог в профиле и **не являются значениями S3-заголовка X-Amz-Storage-Class**. **[предположение]** выбор класса отдельного объекта не подтверждён, этот параметр не задавайте. [Официальный калькулятор S3](https://timeweb.cloud/services/s3-storage).

На дату проверки стандартные тарифы: 1/10/100/250 ГБ — 1/79/349/639 ₽ в месяц, свыше 250 — 4 ₽/ГБ; премиальные — 1/99/449/789 ₽, свыше 250 — 5 ₽/ГБ; холодный pay-as-you-go — 2,2 ₽/ГБ, минимум 1 ГБ. В профиле цена standard/hot `0` означает отсутствие фиксированной ставки за ГБ, **не бесплатное хранение**, cold — справочная ставка с `verified:false`. [Калькулятор](https://timeweb.cloud/services/s3-storage), [Автомасштабирование](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-autoscaling).

Трафик standard/premium 1,3 ₽/ГБ, cold 1,5 ₽/ГБ. Prime даёт 1000 ГБ/месяц; маркетинговая страница также обещает 100 ГБ без Prime, тогда как документация трафика говорит об оплате всего трафика без Prime. **[предположение]** бесплатный объём конкретного аккаунта не установлен и не закладывается в профиль. Расхождение не мешает подключению и API-профилю. [Трафик](https://timeweb.cloud/docs/s3-storage/tariffication), [Страница сервиса](https://timeweb.cloud/services/s3-storage).

## Условия и возможности

| Возможность | Подтверждение/решение | Источник |
|---|---|---|
| If-None-Match / If-Match | **[предположение]** PUT/Copy/Complete/Delete с защитными условиями не описаны; `conditional_writes:false`, `conditional_delete:false` | [Перечень методов](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-guide) |
| Object / bucket tags | Get/Put/DeleteTagging перечислены для обоих ресурсов | [Методы](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-guide) |
| Lifecycle | Чтение/настройка/удаление правил | [Lifecycle](https://timeweb.cloud/docs/s3-storage/supported-features/object-lifecycle) |
| ACL / policy | Object/bucket ACL и Get/Put/DeleteBucketPolicy | [Методы](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-guide), [Policy](https://timeweb.cloud/docs/s3-storage/supported-features/bucket-policies) |
| Versioning | Enable/Suspend, list/get/delete версии | [Версии](https://timeweb.cloud/docs/s3-storage/supported-features/s3-object-versioning) |
| Object Lock / retention | Get config, lock при создании, retention и legal hold documented; governance/compliance | [Object Lock](https://timeweb.cloud/docs/s3-storage/supported-features/object-lock) |
| PUT ObjectLock configuration | **[предположение]** отдельный метод изменения default-конфигурации не подтверждён примерами; `object_lock_config:false` исключает инструмент изменения. Это не означает отсутствия lock в облаке | [Object Lock](https://timeweb.cloud/docs/s3-storage/supported-features/object-lock) |
| CORS | Настройка через панель и S3 API | [CORS](https://timeweb.cloud/docs/s3-storage/supported-features/cors-setup) |
| Multipart | Create/Upload/List/Complete/Abort | [AWS CLI](https://timeweb.cloud/docs/s3-storage/tools/aws-cli) |

Default writes отказывают до сети; явный overwrite требует общего подтверждения. Move не регистрируется. Lifecycle, ACL, bucket tags и retention отражены как возможности API, без инструментов настройки в текущем kit.

## Presign и лимиты

GET/PUT presign поддерживаются; официальный AWS CLI гайд устанавливает максимум **7 дней (604800 секунд)**, по умолчанию час. Панель всегда создаёт ссылку на 60 минут, после истечения срока документация указывает HTTP 500; это описание панели, не доказательство ответа SigV4 API. Не используйте custom domain для presign. [AWS CLI](https://timeweb.cloud/docs/s3-storage/tools/aws-cli), [Presign](https://timeweb.cloud/docs/s3-storage/supported-features/presigned-url).

Документация S3cmd указывает части от 5 МБ до 5 ГБ. **[предположение]** `max_put_bytes:5000000000` и `max_parts:10000` — консервативные S3-клиентские пределы, полный серверный потолок объекта/частей не установлен. Числовые RPS и XML-коды превышения не найдены: `rate_limits:{}`, проверка с ключом обязательна. [S3cmd](https://timeweb.cloud/docs/s3-storage/tools/s3cmd), [Multipart](https://timeweb.cloud/docs/s3-storage/supported-features/multipart-upload).

При установленном лимите объёма pay-as-you-go бакет становится read-only; точный HTTP/XML-ответ на запись не приведён. При выключенном автоскейлинге превышение тарифа приостанавливает сервис. [Автомасштабирование](https://timeweb.cloud/docs/s3-storage/manage-storage/s3-autoscaling).

Превышение пользовательского лимита исходящего трафика блокирует доступ к объектам с HTTP **403**; проверка раз в час, промежуточный расход оплачивается. **[предположение]** XML-код этого отказа, ошибки RPS, превышения размера объекта и числа бакетов требуют живого теста. [Трафик](https://timeweb.cloud/docs/s3-storage/tariffication).

## Нужна живая проверка

Ключ владельца; endpoint/регион, условия PUT/Copy/Complete/Delete, PUT default ObjectLock config, object StorageClass и классы бакетов/цены; presign, квоты и точные HTTP/XML-коды. Versitygw проверяет общий код и регистрацию, не сервис Timeweb Cloud.
