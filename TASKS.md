# Статус

- Завершено: ресерч официальной документации и консервативный профиль; README/compat/Registry/skill/Docker/release/CI.
- Завершено: Docker build/vet/race/lint (0 issues), versitygw conformance, MCP smoke (все 8 сочетаний уровней), мутационные ворота 87/87 (100%) и self-tests, check_docs, actionlint, GoReleaser snapshot; образ uid 65532 запущен.
- Доставка завершена: main опубликован, описание/topics настроены; [CI реализации `23c0c82`](https://github.com/stufently/timeweb-s3-mcp-ai-cli/actions/runs/38065569611) зелёный.
- Нужна живая проверка: эндпоинты/регионы, условные заголовки, классы/цены, presign, API и ошибки на реальном тестовом бакете с ключом владельца.
- Релиз v0.1.0 и публикация OCI/Registry отложены до живой проверки.

- **[предположение]**: условия записи/удаления, PUT default ObjectLock config, object StorageClass, virtual-host/multi-AZ, серверные пределы/ошибки и льготы аккаунта; подробно в [ресерче](docs/research/timeweb-s3.md).
