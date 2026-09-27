# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is 1

`kaspiInteg.bsl` is the object module of a 1C:Enterprise external processing (ВнешняяОбработка) named `CivilKaspiCards`, registered under the identifier `KASPI_CARD_INTEGRATION`. It runs inside the "Розница" (Retail) 1C configuration, edition 2.2.13.8, and syncs Civil's clothing catalog to the Kaspi.kz marketplace via Kaspi's product-import API, pulling missing product data (photos, fabric composition, descriptions, collar/attribute details) from Civil's own PIM system along the way.

`civil_retail.json` is a raw metadata export of the host "Розница" 1C configuration (справочники, документы, регистры, перечисления, константы). It is a **reference for looking up object structure**, not something this module ships or edits — it's ~3.5 MB, so query it with `jq` rather than reading it whole (e.g. `jq '.Объекты.РегистрыСведений[] | select(.Имя=="мп_ДанныеКарточек")' civil_retail.json`).

`../../kaspiDocumentation.md` (one directory up) documents the actual Kaspi Import API request/response shapes (`/shop/api/products/import`, `/shop/api/products/classification/*`) — read it before changing anything that builds the Kaspi HTTP payload.

## Build / lint / test

There is none, locally. This code has no compiler, linter, or test runner outside 1C itself — it's edited as plain text here and then pasted into the 1C Configurator or EDT as the module body of the `CivilKaspiCards` external processing, where 1C's own syntax checker validates it on save/compile. **There is no way to run or test this code from the shell.** Watch for 1C-specific syntax pitfalls that a text editor won't flag: mismatched `Если`/`ИначеЕсли`/`Иначе`/`КонецЕсли` nesting, keyword casing that 1C accepts but is easy to typo (`Тогда`/`тогда`/`ТОГДА` are all valid), and the requirement that every `Функция`/`Процедура` closes before the next one starts — a copy-paste that leaves a branch's `Тогда` orphaned outside its own function is a real, silent failure mode here (it compiles as a dangling `ИначеЕсли` with no matching `Если` and 1C reports the error at the *following* function's closing keyword, not at the actual mistake).

## Entry points

`СведенияОВнешнейОбработке()` registers eight commands with the 1C platform; `ВыполнитьКоманду(ИмяКоманды)` dispatches to them:

| Command | Purpose |
|---|---|
| `GET_DESCRIPTION`, `GET_COMPOSITION`, `GET_COLLAR`, `GET_PHOTO` | Pull missing product data from Civil's PIM gateway (`gateway.civil.com.tr:8080`) and cache it into 1C info registers (`мп_АтрибутыНоменклатур`, `мп_СоставНоменклатур`, `СоответствиеРеквизитовИМ`). |
| `ADD_NEW_ITEMS` | Seeds `РегистрСведений.ИнтернетМагазинКарточки` with (Номенклатура, Характеристика) rows for in-stock items not yet queued for Kaspi. |
| `CHECK_READEY_ITEMS` | Flags queued rows `ГотовКСоздании = Истина` once all required fields are present (size, gender, color, season, title, composition, category, ≥1 photo). |
| `CARD_CREAT` | Builds Kaspi product-import JSON for ready items and `POST`s them in batches of 10 to `https://kaspi.kz/shop/api/products/import`. |
| `REPORT` | Emails an Excel report of items still missing data, broken down by which field is missing. |

## CARD_CREAT: category → attributes dispatch table

Kaspi requires a different, hand-specified set of `attributes` (code/value pairs) per leaf category (`Master - Baby shirts and blouses`, `Master - Girl jeans`, ...). This is handled by a data-driven dispatch, in the `#Область КАТЕГОРИИ_KASPI` region right after `CARD_CREAT`:

- `ПолучитьТаблицуОбработчиковКатегорий()` returns a `Соответствие` mapping each Kaspi category string to the name of a handler function (`Кат_BabyShirtsAndBlouses`, `Кат_GirlJeans`, ...) — currently ~55 handlers covering ~60 categories.
- The main loop in `CARD_CREAT` looks up the category, then invokes the handler by name via `Выполнить("Результат = " + ИмяОбработчика + "(...)")` — 1C has no first-class function references, so this string-eval call is the standard idiom for table-driven dispatch here. Every handler shares one fixed parameter signature (`Товары, Категория, Категрия, attributes, Цвет, РазмерКапи, Пол, Сезон, СтранаИзготовитель, Number_in_sets, Рост, МассивСостава, Additional, ПолБейби, ПолБейби2`) so the `Выполнить` call string is uniform.
- A handler returns `Истина` to keep the item, `Ложь` to skip it (equivalent to the old chain's `Продолжить`).
- **When adding a category**: write a new `Кат_*` function in that region using the same signature, then register it in `ПолучитьТаблицуОбработчиковКатегорий()`. Don't reintroduce a giant `Если/ИначеЕсли` chain in `CARD_CREAT` itself — that's what this table replaced.
- **Known non-obvious quirk to preserve**: several "Boy X" categories deliberately emit attributes under a *different* category's code prefix — e.g. `Master - Boy coats` emits `Girl coats*...` attribute codes, `Master - Boy pants` emits `Girl pants*...`, `Master - Boy jeans` emits `Girl jeans*...`. This mirrors Kaspi's own classification, where some boy/girl leaf categories share one attribute set on Kaspi's side. Don't "fix" a prefix mismatch without checking `/shop/api/products/classification/attributes?c=<category>` first.

## Auth / caching

- Civil's PIM token (`ПолучитьТокенCivil`) is cached in `ХранилищеОбщихНастроек` for 24h under the `rmadmin` user key — don't call the token endpoint directly in a loop; go through `ДопПараметрыCivil()`.
- Kaspi's own request headers are built in `wrapfor()` (region `Setting`); `X-Auth-Token` / `X-Merchant-Uid` are currently set to empty strings there — this looks unresolved, confirm where (or whether) they get populated before relying on live Kaspi calls.
