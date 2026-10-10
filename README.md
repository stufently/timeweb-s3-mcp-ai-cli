# Timeweb Cloud S3 MCP Server & AI CLI

![Version](https://img.shields.io/badge/version-0.1.0-blue) ![License](https://img.shields.io/badge/license-MIT-green)

Timeweb Cloud S3 Object Storage через MCP и CLI: Claude Code, Cursor и Codex помогают выполнять бэкап и работать с файлами, а AI-агент получает ограниченный доступ и подтверждение опасных действий.

## English

A single Go binary for Timeweb Cloud Object Storage: a stdio MCP server, streaming CLI and installable agent skill. Read-only MCP access is the default; independent write/delete/admin levels, bucket/prefix scope, human approval and confined local file access protect data. Large files travel directly through presigned URLs or multipart uploads. See the [full English guide](README.en.md).

## Быстрый старт

Для cold/premium создайте бакет в панели; `mb` использует класс по умолчанию. Классы `standard/hot/cold` в профиле — тарифный каталог, не заголовок StorageClass объекта. Цена 0 означает неизвестную ставку за ГБ. PUT конфигурации Object Lock выключен до подтверждения; блокировки облака описаны в [ресерче](docs/research/timeweb-s3.md).

Возьми бинарь своей платформы из Releases, когда версия будет опубликована, либо собери с Go 1.27.2:

```bash
bash scripts/go.sh go build -o timeweb-s3 ./cmd/timeweb-s3
./timeweb-s3 version
./timeweb-s3 auth set
./timeweb-s3 auth status
./timeweb-s3 ls --json
```

auth set спрашивает key ID и secret без эха; без TTY читает две строки stdin. Не передавай секрет в argv. Файл ~/.config/timeweb-s3/credentials создаётся атомарно с 0600; более широкие права отклоняются.

Приоритет: флаги > env > файл. Переменные: TIMEWEB_S3_ACCESS_KEY_ID, TIMEWEB_S3_SECRET_ACCESS_KEY, TIMEWEB_S3_POOL (ru-1), TIMEWEB_S3_ENDPOINT. HTTP требует TIMEWEB_S3_ALLOW_INSECURE=1.

После публикации образ: ghcr.io/stufently/timeweb-s3-mcp-ai-cli:0.1.0; локальная сборка: docker build -t timeweb-s3 . Образ запускает MCP от uid 65532, содержит CA-сертификаты. Передай креды через приватное окружение/файл.

## Возможности

- Потоковые object CRUD, copy, multipart и presigned URL в пределах профиля.
- Версии, теги, CORS, политика и Object Lock configuration регистрируются при наличии поддержки.
- Ограниченные сканы, диагностика, JSON, установка MCP и встроенного skill.

## Подключение к Claude Code, Cursor, Codex

```bash
timeweb-s3 mcp install --client cursor --dry-run
timeweb-s3 mcp install --client cursor
timeweb-s3 mcp install --client claude-desktop
timeweb-s3 mcp install --client codex
timeweb-s3 mcp install --client claude-code --scope project
timeweb-s3 mcp install --client claude-code --scope user
```

Последняя команда только печатает claude mcp add. Путь бинаря абсолютный; ключи в конфиг не пишутся. Cursor: ~/.cursor/mcp.json, Claude Code project: ./.mcp.json; Desktop — каталог конфигурации своей ОС. JSON сохраняет чужие ключи, .bak и идемпотентность.

Codex: ~/.codex/config.toml. Существующая таблица требует --force; заменяется только блок между # >>> timeweb-s3 (managed) / # <<< timeweb-s3 (managed). Таблица без маркеров не заменяется даже с --force; остальные байты TOML целы.

Уровни включаются независимо: timeweb-s3 mcp --allow-write --allow-delete --allow-admin либо TIMEWEB_S3_ALLOW=write,delete,admin. Флаги доступны и в mcp install. FILE_ROOT включается через TIMEWEB_S3_FILE_ROOT.

## Инструменты MCP

| Level | Tools |
|---|---|
| read | s3_list_buckets, s3_list_objects, s3_head_object, s3_get_object, s3_preview_object, s3_presign_get, s3_list_object_versions, s3_list_multipart_uploads, s3_list_parts, s3_get_bucket_config, s3_largest_objects, s3_prefix_usage, s3_unfinished_uploads, s3_get_object_tagging |
| read + FILE_ROOT | s3_download_file |
| write | s3_put_object, s3_create_folder, s3_copy_object, s3_presign_put, s3_multipart_create, s3_multipart_presign_part, s3_multipart_complete, s3_put_object_tagging |
| write + FILE_ROOT | s3_upload_file |
| delete | s3_delete_object, s3_delete_objects, s3_multipart_abort |
| admin | s3_create_bucket, s3_put_bucket_versioning, s3_put_bucket_cors, s3_put_bucket_policy, s3_delete_bucket_policy |
| admin + delete | s3_delete_bucket |

Ресурсы: `timeweb-s3://config`, `timeweb-s3://capabilities`, `timeweb-s3://pools`. Неподдерживаемые возможности исключают инструменты.

Списки — до 1000 элементов с continuation_token; версии/multipart имеют второй маркер. Inline get/put — 262144 байт, preview — 65536; большие файлы идут через presigned URL или FILE_ROOT. Ответ содержит structuredContent и JSON-текст; текст свыше 256 КиБ заменяется краткой сводкой. Ошибки: code/message/retryable; 429/503 SDK повторяет адаптивно.

Аналитика сканирует максимум max_scan (по умолчанию 10000); partial/token отмечают неполный результат. Prefix usage выдаёт страницы агрегатов через next_result_offset. Это не биллинг.

## CLI

```bash
timeweb-s3 mb backups
timeweb-s3 cp ./archive.tar s3://backups/daily/archive.tar --part-size 5242880 --overwrite --yes
timeweb-s3 ls backups/daily/ --max-keys 100 --json
timeweb-s3 stat backups/daily/archive.tar
timeweb-s3 cp s3://backups/daily/archive.tar ./restored.tar
timeweb-s3 cp ./directory s3://backups/tree/ --recursive --overwrite --yes
timeweb-s3 cp s3://backups/tree/ ./restored-directory --recursive
timeweb-s3 cp s3://backups/daily/archive.tar s3://backups/copy.tar --overwrite --yes
timeweb-s3 cat backups/daily/notes.txt
timeweb-s3 versions backups/daily/ --json
timeweb-s3 du backups/daily/ --max-scan 10000 --json
timeweb-s3 presign backups/daily/archive.tar --expires 3600
timeweb-s3 rm backups/copy.tar --yes
timeweb-s3 rb empty-bucket --yes
timeweb-s3 doctor --json
timeweb-s3 compat --markdown
```

cp принимает обычные локальные пути и S3 URI s3://bucket/key либо timeweb-s3://bucket/key. Большие загрузки используют manager/multipart, --recursive копирует каталоги/префиксы. Перезапись требует --overwrite и подтверждения; выгрузка атомарна. rm/rb/mv без --yes на не-TTY дают код 2; на TTY спрашивают. --json общий, ошибки — stderr и ненулевой код.

doctor проверяет источник/права кредов, DNS/TLS, часы по Date и ListBuckets: >5 минут — предупреждение, >15 — ошибка.

## Skill для AI-агентов

```bash
timeweb-s3 skill install
timeweb-s3 skill install --target ./agent-skills/timeweb-s3
```

По умолчанию ~/.claude/skills/timeweb-s3. Идемпотентно устанавливает SKILL.md и шесть references: backup/restore, share, least privilege, cost hygiene, migration, incident response. Триггеры: «timeweb s3», «бакет Timeweb Cloud», «загрузи в s3», «presigned», «бэкап в s3».

## Безопасность

Если `conditional_writes:false`, запись без перезаписи отказывает до S3-запросов: HEAD не даёт атомарной защиты. Для согласованной записи используй `--overwrite --yes` в CLI либо `overwrite:true, confirm:true` в MCP, сохраняя политику подтверждения. Если `conditional_delete:false`, `mv` и `s3_move_object` отключены.



MCP по умолчанию регистрирует только read; неподдерживаемые возможности и отсутствующие уровни исключают инструмент.

TIMEWEB_S3_DESTRUCTIVE_POLICY: confirm использует elicitation при поддержке клиента, иначе нужен confirm:true после согласия человека; elicit требует elicitation и не допускает обхода confirm:true; block всегда запрещает, allow выполняет без гейта. Отказ предшествует любым S3-запросам. Гейт покрывает удаления, move, presigned PUT, политику/Object Lock, suspension versioning и перезаписи.

При conditional_writes:true операции put/copy/move/multipart complete/upload file используют overwrite:false, HEAD цели и If-None-Match: *; при false запись по умолчанию отказывает. overwrite:true требует тот же гейт.

Скоуп: TIMEWEB_S3_ALLOWED_BUCKETS=backups и TIMEWEB_S3_ALLOWED_PREFIXES=backups/daily/. Префикс учитывает границу сегмента; daily-evil не подходит. Сегменты '..' отклоняются. Частичный скоуп запрещает целый бакет и широкие списки даже при наличии его в ALLOWED_BUCKETS. Copy/move проверяет источник и цель.

Файловые MCP-инструменты используют os.Root: только относительные пути внутри FILE_ROOT; '..', абсолютные пути и симлинки наружу отклоняются. На каждый вызов одна JSON-строка в stderr; TIMEWEB_S3_AUDIT_FILE дополнительно пишет в 0600. Секреты и X-Amz-Signature/Credential/Security-Token редактируются в логах/ошибках. Полная подписанная ссылка — только результат операции. Ограничивай также права самого S3-ключа.

s3_download_file по умолчанию публикует скачанный файл без замены существующего, в том числе созданного во время скачивания. Для замены передай overwrite:true: применяется разрушительная политика с подтверждением через форму либо confirm:true после согласия человека.

## Пулы и классы хранения Timeweb Cloud

Пулы, эндпоинты и классы — в [таблице совместимости](docs/compat.md); каждый факт, цены и ошибки с первоисточниками — в [ресерче](docs/research/timeweb-s3.md). Пул по умолчанию: `ru-1`; регион SigV4 равен выбранному пулу. В профиле `verified:false`: проверка на versitygw не подтверждает API облака. Неподтверждённые сведения помечены **[предположение]**; цены в профиле не являются калькулятором биллинга.

Lifecycle, ACL, bucket tagging и object retention могут поддерживаться облаком, но эта версия не предоставляет инструментов для их настройки; данные профиля описывают совместимость S3 API.

## FAQ

**Что такое MCP-сервер?** Программа предоставляет агентам инструменты и ресурсы через Model Context Protocol; здесь stdio.

**Как подключить MCP-сервер к Claude Code?** Сохрани ключи auth set, затем используй project install либо напечатанную user-команду.

**Безопасно ли давать агенту доступ к S3?** Ограничь ключи, скоуп и уровни; сохрани confirm/block и одобряй разрушительные эффекты осознанно.

**Чем это отличается от aws-cli и rclone?** Бинарь объединяет MCP, гейт, аудит, аналитику, skill и CLI; aws-cli/rclone остаются самостоятельными средствами передачи/автоматизации.

**Какие возможности поддерживаются?** См. capabilities и [compat](docs/compat.md). Инструменты регистрируются по профилю; lifecycle, ACL, bucket tagging и object retention эта версия не настраивает.


## Другие провайдеры

Семейство: Selectel, Yandex Cloud, VK Cloud, Cloud.ru, Timeweb Cloud, MWS, Beget. Все используют [общий kit](https://github.com/stufently/s3-mcp-ai-cli-kit).

## Лицензия

[MIT](LICENSE). [Проверенные факты и предположения](docs/research/timeweb-s3.md).

Проверки в Docker: bash scripts/go.sh go test -race ./..., bash scripts/conformance.sh, bash scripts/mcp_smoke.sh, bash scripts/mutation_gate.sh, bash scripts/check_docs.sh. Conformance использует versitygw v1.8.0 и тестовые ключи.

Общие MCP/CLI, защиты и проверки поставляет [s3-mcp-ai-cli-kit v0.1.0](https://github.com/stufently/s3-mcp-ai-cli-kit/tree/v0.1.0); в этом репозитории остаются встроенный профиль, тонкий main и документы Timeweb Cloud.
