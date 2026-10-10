# Timeweb Cloud S3 compatibility

Generated from the embedded provider profile.

| Feature | Supported |
|---|---|
| acl | true |
| bucket_policy | true |
| bucket_tagging | true |
| conditional_delete | false |
| conditional_writes | false |
| cors | true |
| lifecycle | true |
| multipart | true |
| object_lock_config | false |
| object_retention | true |
| object_tagging | true |
| presign | true |
| versioning | true |

| Pool | Endpoint | Location | Multi-AZ | Classes |
|---|---|---|---|---|
| ru-1 | https://s3.twcstorage.ru | Россия, ru-1 | false | standard, hot, cold |

| Class | Name | RUB/GB/month (from) | Verified | Checked |
|---|---|---|---|---|
| standard | Стандартный (каталог тарифов бакета) | 0.00 | false | 2026-10-10 |
| hot | Премиальный (каталог тарифов бакета) | 0.00 | false | 2026-10-10 |
| cold | Холодный (каталог тарифов бакета) | 2.20 | false | 2026-10-10 |

Presign maximum: 604800 seconds; verified: false. До 7 дней по AWS CLI документации; endpoint без собственного домена. standard/hot/cold — каталог тарифов бакета, не значения X-Amz-Storage-Class. Условия записи/удаления и PUT ObjectLock config требуют живой проверки.
