# Stash — справочник параметров конфигурации

Этот файл описывает все параметры JSON-конфигов мода **AR_Stash** и даёт готовые примеры настроек
для типа тайника.

## Где лежат файлы на сервере

- `$profile:\AR_Settings\AR_Main_Stash_Settings.json` — общие настройки мода (один файл на всех).
- `$profile:\AR_Settings\AR_Stash_Configs\<ИмяКласса>_Config.json` — настройки одного типа тайника
(по файлу на каждый placeable-класс из `AR_Stash_PUBLIC/stash/config.cpp`).

---

## 1. MainStashSettingsConfig — общие настройки (`AR_Main_Stash_Settings.json`)


| Параметр               | Тип             | Описание                                                                                                                                                                                                                                                                                                      |
| ---------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `checkDistance`        | float           | Радиус в метрах вокруг **открытого** тайника, в котором сервер раз в 120 сек проверяет присутствие игроков. Используется только вместе с `enableRemove`.                                                                                                                                                      |
| `enableRemove`         | int (0/1)       | 1 — если в радиусе `checkDistance` долго нет ни одного игрока, открытый тайник со всем содержимым удаляется. 0 — тайник никогда не удаляется автоматически.                                                                                                                                                   |
| `toolDamageMultiplier` | float (0.0–1.0) | Множитель прочности предметов, заспавненных внутри, если тайник открыт **альтернативным инструментом** (а не ключом и не руками). Пример: 0.1 — предметы получат только 10% от рассчитанной прочности (эффект "неаккуратного вскрытия"). При открытии ключом или руками множитель не применяется (равен 1.0). |
| `toolWearAmount`       | float (0.0–1.0) | Общий износ инструмента за одно открытие — действует одинаково на **все** тайники, у которых задан `alternativeTool`. Например, 0.5 = минус 50% прочности инструмента за одно использование.                                                                                                                  |


---



## 2. StashConfig — настройки одного типа тайника (`<Класс>_Config.json`)


| Параметр                          | Тип                          | Описание                                                                                                                                                                    |
| --------------------------------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `items`                           | map<string, ItemStashConfig> | Пул предметов, которые могут появиться внутри при открытии. Ключ карты — базовый classname предмета (см. раздел 3).                                                         |
| `minSpawnCount` / `maxSpawnCount` | int                          | Минимальное/максимальное количество **разных** предметов (не стаков), которые заспавнятся за одно открытие. Реальное число каждый раз выбирается случайно в этом диапазоне. |
| `requiredKey`                     | string                       | Classname ключа, которым можно открыть тайник. Пустая строка `""` — ключ не требуется.                                                                                      |
| `keyWearAmount`                   | float (0.0–1.0)              | Износ ключа за одно открытие. `0.0` — ключ не изнашивается. Работает только если `requiredKey` не пустой.                                                                   |
| `alternativeTool`                 | string                       | Classname альтернативного инструмента, которым тоже можно открыть тайник. Пустая строка `""` — альтернативы нет.                                                            |


**Логика открытия** (`ActionOpenStash` + `AR_Stash_Base`):

- `requiredKey == ""` и `alternativeTool == ""` → тайник открывается голыми руками (любым предметом в руке или вообще без предмета).
- Задан только `requiredKey` → открыть может **только** предмет этого класса.
- Задан только `alternativeTool` → открыть может **только** предмет этого класса.
- Заданы оба → открыть может **любой** из двух — ключ **или** инструмент.
- Если открывали ключом — изнашивается ключ (`keyWearAmount`), прочность добычи не страдает.
- Если открывали инструментом — изнашивается инструмент (`toolWearAmount` из главных настроек) и добыча получает урон (`toolDamageMultiplier` из главных настроек).

---



## 3. ItemStashConfig — один предмет внутри `items`


| Параметр                      | Тип             | Описание                                                                                                                                                                                                                     |
| ----------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `chance`                      | float           | Шанс выпадения предмета относительно остальных в пуле.                                                        |
| `variants`                    | array           | Альтернативные classname (например, других цветов/раскрасок). При спавне случайно берётся либо базовый classname, либо один из `variants` — так исключается появление двух одинаковых предметов разного цвета за один спавн. |
| `attachments`                 | array           | Аттачменты, которые могут быть подвешены к предмету при спавне (см. раздел 4).                                                                                                                                               |
| `quantityMin` / `quantityMax` | float           | Диапазон наполнения. Для обычных предметов — проценты (0–100) от максимальной вместимости (заряд, жидкость и т.п.). Для магазинов (`Magazine`) — это уже абсолютное количество патронов (округляется до целого).             |
| `minHP` / `maxHP`             | float (0.0–1.0) | Диапазон прочности предмета при спавне (`1.0` = 100%). К итоговому значению ещё умножается `toolDamageMultiplier`, если тайник открыт инструментом.                                                                          |




## 4. StashAttachmentConfig — один аттачмент внутри `attachments`


| Параметр                      | Тип             | Описание                                                                                    |
| ----------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| `attachment`                  | string          | Classname аттачмента.                                                                       |
| `chance`                      | float (0.0–1.0) | Вероятность, что аттачмент заспавнится (проверяется отдельно для каждого аттачмента).       |
| `minHP` / `maxHP`             | float (0.0–1.0) | Диапазон прочности аттачмента.                                                              |
| `quantityMin` / `quantityMax` | float           | Диапазон наполнения аттачмента (то же правило: проценты либо число патронов для магазинов). |


---



## 5. Готовые конфиги (кладутся в `AR_Stash_Configs\<Класс>_Config.json`) ПРИМЕРЫ:



### 🖐 Открываются руками

**AR_Stash_Army_Crate_Config.json** — армейский ящик с боеприпасами

```json
{
    "AR_Stash_Army_Crate": {
        "items": {	
            "AmmoBox_00buck_10rnd": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.8,
                "maxHP": 1.0
            },
            "AmmoBox_22_50Rnd": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.8,
                "maxHP": 1.0
            },
            "AmmoBox_308Win_20Rnd": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "",
        "keyWearAmount": 0.0,
        "alternativeTool": ""
    }
}
```

**AR_Stash_Army_Weapon_Crate_Config.json** — ящик с оружием

```json
{
    "AR_Stash_Army_Weapon_Crate": {
        "items": {	
            "AugShort": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [
                    { 
                        "attachment": "Mag_Aug_30Rnd",
                        "chance": 1.0,
                        "minHP": 0.4,
                        "maxHP": 1.0,
                        "quantityMin": 5,
                        "quantityMax": 15
                    }
                ],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.4,
                "maxHP": 1.0
            },
            "FAMAS": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [
                    { 
                        "attachment": "Mag_FAMAS_25Rnd",
                        "chance": 1.0,
                        "minHP": 0.4,
                        "maxHP": 1.0,
                        "quantityMin": 4,
                        "quantityMax": 13
                    }
                ],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.4,
                "maxHP": 1.0
            },
            "M16A2": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [
                    { 
                        "attachment": "Mag_CMAG_30Rnd",
                        "chance": 1.0,
                        "minHP": 0.4,
                        "maxHP": 1.0,
                        "quantityMin": 5,
                        "quantityMax": 15
                    }
                ],
                "quantityMin": -1,
                "quantityMax": -1,
                "minHP": 0.4,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 2,
        "requiredKey": "",
        "keyWearAmount": 0.0,
        "alternativeTool": ""
    }
}
```

### 🔑🔧 Ключ ИЛИ альтернативный инструмент

**AR_Stash_Ammunition_Box_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "AR_Key_1",
        "keyWearAmount": 0.5,
        "alternativeTool": "Crowbar"
    }
}
```

**AR_Stash_Military_Cargo_Case_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "AR_Key_2",
        "keyWearAmount": 0.5,
        "alternativeTool": "Crowbar"
    }
}
```



### 🔧 Только альтернативный инструмент

**AR_Stash_Broken_Wooden_Crate_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "",
        "keyWearAmount": 0.5,
        "alternativeTool": "Hatchet"
    }
}
```

**AR_Stash_Trash_Can_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "",
        "keyWearAmount": 0.5,
        "alternativeTool": "Pickaxe"
    }
}
```



### 🔑 Только ключ

**AR_Stash_Old_Safe_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "AR_Key_3",
        "keyWearAmount": 0.5,
        "alternativeTool": ""
    }
}
```

**AR_Stash_Chest_Config.json**

```json
{
    "AR_Stash_Covered_Wooden_Crate": {
        "items": {	
            "WaterBottle": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "BoxCerealCrunchin": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 100,
                "quantityMax": 100,
                "minHP": 0.7,
                "maxHP": 1.0
            },
            "Canteen": {
                "chance": 0.25,
                "variants": [], 
                "attachments": [],
                "quantityMin": 50,
                "quantityMax": 100,
                "minHP": 0.8,
                "maxHP": 1.0
            }
        },
        "minSpawnCount": 1,
        "maxSpawnCount": 4,
        "requiredKey": "AR_Key_4",
        "keyWearAmount": 0.5,
        "alternativeTool": ""
    }
}
```

---

### Список предметов в `items` можно свободно менять — структура и имена файлов при этом не трогаются.

---

## 6. StashSpawner — настройки точек спавна на карте (`AR_Stash_Spawners\<Класс>_Spawner.json`)

Cистема отвечает за то,
**где на карте** и **сколько штук** каждого типа тайника появляется, и за их респавн после того, как
объект убрали/сломали.

Расположение: `$profile:\AR_Settings\AR_Stash_Spawners\<Класс>_Spawner.json` — по одному файлу на
тип тайника.

| Параметр | Тип | Описание |
|---|---|---|
| `Enable_Spawn` | int (0/1) | Включает/выключает спавн этого типа тайника целиком. `0` — тип полностью пропускается, ни один объект не создаётся, даже если в `Coords` есть точки. |
| `Snap_To_Ground` | int (0/1) | `1` — объект при создании "прилипает" к поверхности земли по высоте (Z из `Position` игнорируется, берётся высота рельефа). `0` — объект ставится ровно на координаты из `Position`, включая высоту (нужно для тайников внутри зданий/на этажах). |
| `Classname` | string | Точный classname объекта, который спавнится. Должен **точно** совпадать с реальным placeable-классом из `AR_Stash_PUBLIC/stash/config.cpp` (регистр важен) — иначе не сможет создать объект. |
| `Min_Objects_In_Map` / `Max_Objects_In_Map` | int | Сколько экземпляров этого типа одновременно живёт на карте. Из всех точек в `Coords` система случайно выбирает и держит активными от `Min` до `Max` штук (остальные точки — "запасной" пул, из которого при необходимости добираются позиции, если ближайшие отклонены по `Min_Distance_Between_Stashes`). **Важно:** должно быть `Min > 0`, `Max > 0` и `Min <= Max` — иначе вся запись для этого типа отбраковывается и тип не заспавнится вообще. Также если `Max` меньше или равен фактическому числу уникальных точек в `Coords`, лишние точки просто не используются. |
| `Coords` | array<{Position, Orientation}> | Пул кандидатов на позиции спавна для этого типа. Каждый элемент: `Position` — координаты `[X, Y, Z]` в мире; `Orientation` — поворот `[yaw, pitch, roll]` в градусах. Чем больше точек в пуле относительно `Max_Objects_In_Map`, тем больше разнообразия (тайники будут появляться в разных местах между рестартами/респавнами), а не всегда в одних и тех же. |
| `Time_Respawn` | float (минуты) | Через сколько минут после того, как активный объект убрали (сломан, разграблен и удалён и т.п.), на освободившуюся точку заново поставят новый экземпляр. Должно быть `> 0`. |
| `Distance_Clean_Undo_Respawn` | float (метры) | Радиус вокруг точки спавна, в котором система перед созданием нового объекта ищет и удаляет старые "зависшие" объекты тайников этого мода (защита от дублей на одной точке). |
| `Min_Distance_Between_Stashes` | float (метры) | Минимальное расстояние между двумя **одновременно активными** тайниками **одного и того же** класса. Точка-кандидат из `Coords`, которая ближе этого расстояния к уже заспавненному тайнику того же типа, отклоняется, и система пробует другую точку из пула. `>= 0`. |

**Логика при старте сервера:** при каждом запуске сервер один раз полностью зачищает существующие
объекты тайников этого мода и заново перевыбирает активные позиции из `Coords` по всем типам.
