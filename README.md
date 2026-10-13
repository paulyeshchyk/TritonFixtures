# TritonFixtures

Единый источник данных для **TritonFront** и **TritonBack**.

## Что здесь лежит

- `version.json` — версия контракта фикстур.
- `catalog.json` — дерево каталога сущностей (структура `ICatalogTree` на фронте).
- `layouts/<actionId>.json` — готовые к рендеру layout'ы (`ILoadedLayout` на фронте).

## Правила версионирования (single-tier)

**Одна мажорная версия на всю систему.** TritonFront, TritonBack и TritonFixtures всегда имеют одинаковый `major`.

- Совпадают `.major` во всех трёх → система совместима.
- Не совпадают → приложение **не стартует** (frontend и backend падают с явной ошибкой).

### Когда bump

| Изменение | Действие |
|---|---|
| Добавили поле (опциональное) | `minor` +1 |
| Добавили новый layout-файл | `minor` +1 |
| Добавили новый тип сущности | `minor` +1 |
| Убрали поле | **`major` +1** |
| Переименовали поле | **`major` +1** |
| Поменяли тип поля | **`major` +1** |
| Переструктурировали JSON | **`major` +1** |

При `major`-bump — **обязательно** синхронно менять (полный список из 7 мест):

1. `TritonFixtures/version.json` → `major`
2. `TritonFront/package.json` → `version`
3. `TritonFront/public/runtime-config.js` → `expectedFixturesMajor`
4. `TritonBack/src/BackendServer.Api/BackendServer.Api.csproj` → `<Version>`
5. `TritonBack/src/BackendServer.Api/appsettings.json` → `Backend:ExpectedFixturesMajor`
6. `TritonBack/build-and-package.ps1` → `$imageTag`
7. `TritonFront/tools/backend-server/docker-compose.yml` → `image`

Иначе один из компонентов откажется стартовать — и это **правильно**, лучше явный отказ, чем тихая работа с несовместимыми данными.

## Как обновляются JSON-файлы

Файлы в этой папке — **сгенерированные**. Не редактируй их руками.

Источник истины — TS-моки в `TritonFront/src/app/mocks/data/`. Генератор — `TritonFront/tools/export-fixtures.ts`.

Пересобрать:

```powershell
cd ..\TritonFront
npm run export:fixtures
```

Скрипт перезапишет `catalog.json` и `layouts/*.json` в этой папке.

## Как фикстуры попадают в проекты

**TritonFront:** копируются в `public/fixtures/` при `prestart` / `prebuild` (через `tools/sync-fixtures.ps1`).

**TritonBack:** копируются в `bin/**/Fixtures/` через MSBuild-правило в `BackendServer.Api.csproj` (`CopyToOutputDirectory="PreserveNewest"`).

Ни один из проектов не читает `TritonFixtures/` напрямую в рантайме — только через свою локальную копию.

