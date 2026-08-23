# Как устроен скрипт

Загруженный скрипт — ещё не работающий: верхний уровень файла выполняется при загрузке, а обработчики и команды работают только пока скрипт включён. Всё, что даёт API, висит на неявном получателе файла, поэтому любой член с этой страницы вызывается без префикса.

```kotlin
name("Auto jump")
description("Jumps for you")
key(Key.G)

onEnable { chat.print("$id on") }
onDisable { chat.print("$id off") }
onUnload { storage.save() }

whenInGame { enabled = true }
```

## Загружен и включён

| Метод | Тип | Описание |
|---|---|---|
| `id` | `String` | id скрипта = имя `.kts`-файла без расширения |
| `enabled` | `Boolean` | состояние вкл/выкл, чтение и запись; запись уходит на клиентский поток |
| `toggle()` | `Unit` | переключает `enabled` |
| `onEnable(action)` | `Unit` | колбэк включения, можно несколько, исключения логируются |
| `onDisable(action)` | `Unit` | колбэк выключения, можно несколько, исключения логируются |
| `onUnload(action)` | `Unit` | выполняется один раз при выгрузке или перезагрузке (бросает `ScriptStateException`, если скрипт уже выгружен) |
| `whenInGame(body)` | `Unit` | выполняет `body`, только если есть игрок и мир |

Верхний уровень выполняется один раз, при загрузке. Пока скрипт выключен, его обработчики, команды, хоткеи и задачи отписаны; включение возвращает их обратно, файл заново не выполняется.

### Script

| Метод | Тип | Описание |
|---|---|---|
| `Script.setEnabled(value)` | `void` | Java-форма записи в `enabled`, уходит на клиентский поток |
| `Script.defaultKey(key)` | `Script` | Java-форма `key(key)`, возвращает скрипт |
| `Script.events()` | `Events` | реестр событий под `on<E> { }` — [Подписка на события](../events/basics.md) |
| `Script.client()` | `Client` | корень `client`, таблица ниже |
| `Script.game()` | `Game` | корень `game`, таблица ниже |

`Script` — это Java-объект, которому переадресует всё выше; `id()`, `name()`, `description()`, `enabled()`, `toggle()`, `bind()`, `onEnable()`, `onDisable()` и `onUnload()` носят те же имена, что уже есть на этой странице.
`Script` наследует `SettingHost`, поэтому каждая фабрика настроек со страницы [Виды настроек](../settings/types.md) — тоже его член.

## Имя, описание, бинд

| Метод | Тип | Описание |
|---|---|---|
| `name(value)` | `Unit` | имя в меню, обрезается по краям (бросает `ScriptException` при пустом или длиннее 32 символов) |
| `description(value)` | `Unit` | описание в меню, null становится `""` |
| `key(key)` | `Unit` | клавиша по умолчанию, ставится только пока бинд пуст |
| `bind` | `Bind` | собственный бинд скрипта — [Клавиши и бинды](../actions/keys.md) |

## Что доступно из скрипта

| Метод | Тип | Описание |
|---|---|---|
| `client` | `Client` | клиентские сервисы, таблица ниже |
| `game` | `Game` | текущая игровая сессия, таблица ниже |
| `user` | `User` | аккаунт Nursultan текущего пользователя — [Твой аккаунт](../extras/user.md) (API 2) |
| `chat` | `Chat` | печать и отправка в чат — [Сообщения](../ui/messages.md) |
| `text` | `Texts` | фабрика текстовых компонентов — [Оформленный текст](../ui/text.md) |
| `log` | `Logger` | логгер консоли скриптов — [Сообщения](../ui/messages.md) |
| `storage` | `Config` | конфиг скрипта по умолчанию — [Сохранение данных](../settings/storage.md) |
| `configs` | `Configs` | именованные конфиги этого скрипта — [Сохранение данных](../settings/storage.md) |
| `assets` | `Assets` | чтение из `scripts/assets` — [Папка assets](../extras/assets.md) (API 2) |
| `clipboard` | `Clipboard` | системный буфер обмена — [Сообщения](../ui/messages.md) (API 2) |
| `filters` | `EntityFilters` | готовые предикаты сущностей — [Сущности и фильтры](../game/entities.md) |
| `keys` | `Keys` | состояние клавиатуры и мыши — [Клавиши и бинды](../actions/keys.md) |
| `party` | `Party` | данные группы и шина сообщений скриптов — [Сообщения в группе](../extras/party.md) (API 4) |
| `player` | `SelfPlayer` | свой игрок — [Свой игрок](../game/player.md) |
| `world` | `World` | загруженный мир — [Мир и блоки](../game/world.md) |
| `inventory` | `Inventory` | инвентарь игрока — [Инвентарь и предметы](../game/inventory.md) |
| `container` | `Container` | открытый обработчик экрана — [Контейнеры](../game/containers.md) |
| `recipes` | `Recipes` | книга рецептов — [Контейнеры](../game/containers.md) (API 2) |
| `inGame` | `Boolean` | true, когда есть и игрок, и мир |
| `interaction` | `Interaction` | удар, использование, ломание — [Взаимодействие](../actions/interaction.md) |
| `raycast` | `Raycast` | лучи по блокам и сущностям — [Лучи и прицел](../game/raycast.md) |
| `control` | `Control` | флаги движения и прыжок — [Движение](../actions/control.md) |
| `gameSettings` | `GameSettings` | настройки Minecraft — [Движение](../actions/control.md#настройки-игры) (API 2) |
| `slots` | `Slots` | выбор слота хотбара — [Слоты и броня](../actions/slots.md) |
| `armor` | `Armor` | сравнение надетой брони — [Слоты и броня](../actions/slots.md) |
| `combat` | `Combat` | точка удара и пометка цели — [Взаимодействие](../actions/interaction.md) |
| `rotations` | `Rotations` | подмена yaw/pitch для сервера — [Повороты](../actions/rotations.md) |
| `prediction` | `Prediction` | предсказание движения и снарядов — [Предсказание](../actions/prediction.md) |
| `gpu` | `Gpu` | реестр мешей и пайплайнов — [Своя геометрия](../ui/gpu.md) (API 2) |

`player`, `world`, `inventory`, `container`, `recipes`, `interaction`, `raycast` и `control` бросают `ScriptStateException` вне мира; проверка — `inGame`.
Любой корень бросает `ScriptStateException` после выгрузки скрипта.

## Объект client

| Метод | Тип | Описание |
|---|---|---|
| `client.user()` | `User` | аккаунт Nursultan текущего пользователя (API 2) |
| `client.chat()` | `Chat` | печать и отправка в чат, с меткой id скрипта |
| `client.text()` | `Texts` | фабрика текстовых компонентов |
| `client.log()` | `Logger` | логгер консоли скриптов, работает и после выгрузки |
| `client.notifications()` | `Notifications` | стек уведомлений клиента — [Сообщения](../ui/messages.md) |
| `client.tasks()` | `Tasks` | планировщик тиков этого скрипта — [Таймеры и задачи](../extras/tasks.md) |
| `client.storage()` | `Config` | конфиг скрипта по умолчанию, имя файла `storage` |
| `client.configs()` | `Configs` | хранилище именованных конфигов скрипта |
| `client.assets()` | `Assets` | чтение файлов из `scripts/assets` (API 2) |
| `client.clipboard()` | `Clipboard` | чтение и запись системного буфера (API 2) |
| `client.commands()` | `Commands` | регистрация команд с префиксом `.` — [Свои команды](../extras/commands.md) |
| `client.modules()` | `Modules` | реестр модулей клиента — [Модули клиента](../extras/modules.md) |
| `client.waypoints()` | `Waypoints` | менеджер путевых точек — [Путевые точки](../extras/waypoints.md) |
| `client.party()` | `Party` | данные группы и шина сообщений скриптов — [Сообщения в группе](../extras/party.md) (API 4) |
| `client.rotations()` | `Rotations` | обработчик подменённых поворотов |
| `client.combat()` | `Combat` | точка удара, пометка цели |
| `client.slots()` | `Slots` | управление слотом хотбара и возвратом |
| `client.armor()` | `Armor` | доступ к надетой броне |
| `client.prediction()` | `Prediction` | предсказание движения и снарядов |
| `client.timer()` | `Timer` | новый независимый секундомер на каждый вызов — [Таймеры и задачи](../extras/tasks.md) |
| `client.filters()` | `EntityFilters` | фабрика предикатов сущностей, работает и после выгрузки |
| `client.keys()` | `Keys` | состояние клавиатуры, мыши и курсора |
| `client.fonts()` | `Fonts` | реестр шрифтов на сессию — [Рендер 2D](../ui/render-2d.md) |
| `client.shaders()` | `Shaders` | реестр шейдеров этого скрипта — [Шейдеры](../ui/shaders.md) |
| `client.gpu()` | `Gpu` | реестр GPU-буферов и пайплайнов скрипта (API 2) |
| `client.textures()` | `Textures` | реестр текстур этого скрипта — [Рендер 2D](../ui/render-2d.md) |

`client.fps()`, `tick()`, `millis()`, `nanos()`, `tickDelta()` и `onClientThread()` описаны на странице [Таймеры и задачи](../extras/tasks.md).

## Объект game

| Метод | Тип | Описание |
|---|---|---|
| `game.inGame()` | `boolean` | true, когда есть и игрок, и мир |
| `game.connected()` | `boolean` | true, пока существует сетевой обработчик игры |
| `game.player()` | `SelfPlayer` | свой игрок — [Свой игрок](../game/player.md) |
| `game.world()` | `World` | клиентский мир — [Мир и блоки](../game/world.md) |
| `game.inventory()` | `Inventory` | инвентарь игрока — [Инвентарь и предметы](../game/inventory.md) |
| `game.control()` | `Control` | управление состоянием движения — [Движение](../actions/control.md) |
| `game.interaction()` | `Interaction` | удар, использование, ломание — [Взаимодействие](../actions/interaction.md) |
| `game.raycast()` | `Raycast` | лучи по блокам и сущностям — [Лучи и прицел](../game/raycast.md) |
| `game.server()` | `Server` | данные текущего сервера — [Сервер, табло, таблист](../game/server.md) (бросает `ScriptStateException`, когда нет подключения) |
| `game.packets()` | `Packets` | отправка исходящих пакетов, никогда не бросает — [Пакеты](../actions/packets.md) |
| `game.screenOpen()` | `boolean` | true, когда открыт любой экран |
| `game.screenKind()` | `ScreenKind` | вид открытого экрана, `NONE` если его нет |
| `game.screen()` | `Screen?` | снимок открытого экрана, null если его нет — [Сервер, табло, таблист](../game/server.md) |
| `game.container()` | `Container` | открытый обработчик экрана — [Контейнеры](../game/containers.md) |
| `game.recipes()` | `Recipes` | книга рецептов — [Контейнеры](../game/containers.md) (API 2) |
| `game.settings()` | `GameSettings` | настройки игры, никогда не бросает — [Движение](../actions/control.md#настройки-игры) (API 2) |
| `game.translate(key, arguments)` | `String` | переведённый текст Minecraft, `""` для пустого ключа |
| `game.hasTranslation(key)` | `boolean` | true, когда ключ есть в загруженном языке |
| `game.language()` | `String` | код языка, например `en_us` |
| `game.item(itemId)` | `Item` | новый стак из 1 предмета, неймспейс по умолчанию `minecraft:` (бросает `ScriptException` при неразбираемом id) |
| `game.closeScreen()` | `void` | закрывает открытый экран на клиентском потоке |
| `game.disconnect()` | `void` | выходит с сервера на клиентском потоке |

`player()`, `world()`, `inventory()`, `control()`, `interaction()`, `raycast()`, `container()` и `recipes()` бросают `ScriptStateException` вне мира.

## Требование к версии

| Метод | Тип | Описание |
|---|---|---|
| `requireApi(minimum)` | `Unit` | не даёт загрузиться на старом клиенте (бросает `ScriptApiException`, когда `ApiVersion.CURRENT` < `minimum`) |

Этот клиент предоставляет версию API 5 — см. [Версии API](../extras/api-versions.md).
