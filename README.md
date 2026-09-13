# LethalityX Lua API

## Содержание

1. [Старт](#1-старт)
2. [Среда](#2-среда)
3. [Lifecycle](#3-lifecycle)
4. [LthX / Notify / Script](#4-lthx--notify--script)
5. [Input / Key](#5-input--key)
6. [ImGui / Ui](#6-imgui--ui)
7. [CVector](#7-cvector)
8. [LocalPlayer](#8-localplayer)
9. [Player](#9-player)
10. [Vehicle](#10-vehicle)
11. [Skin](#11-skin)
12. [Render](#12-render)
13. [Game](#13-game)
14. [Util / Json](#14-util--json)
15. [SAMP](#15-samp)
16. [World](#16-world)
17. [Camera](#17-camera)
18. [Objects](#18-objects)
19. [Path / Radar](#19-path--radar)
20. [Events / Raknet](#20-events--raknet)
21. [File / Storage](#21-file--storage)
22. [Cheat](#22-cheat)
23. [Net / Memory / Raw](#23-net--memory--raw)
24. [MoonLoader](#24-moonloader)
25. [Примеры](#25-примеры)

---

## 1. Старт

```lua
function onStart()
  SAMP.addMessage("[MyScript] loaded", 0xFF7FFFD4)
end

function onFrame(dt)
  if Input.wasKeyPressed(Key.F3) then
    local p = LocalPlayer.getPos()
    SAMP.addMessage(string.format("%.1f %.1f %.1f", p.x, p.y, p.z), 0xFFFFFFFF)
  end
end

function onStop()
  Game.setScriptUi(false)
end
```

Регистрация без глобалов: `LthX.on("frame", fn)`, `Events.on("onCreateObject", fn)`.

---

## 2. Среда

| Есть | Нет |
|------|-----|
| `base`, `string`, `math`, `table`, `utf8` | `io`, `os`, `package`, `require` |
| `File` / `Storage` | `dofile`, `load`, `loadfile`, `coroutine` |

- Файл: до **64 KiB**, без `..` и абсолютных путей.
- Цвет чата: ARGB `0xAARRGGBB`.
- Скрипт — UTF-8. Чат (`SAMP.addMessage`, `sendChat`, `LthX.chat`) → CP1251. `say` / `sendCommand` без конвертации. Notify / ImGui — UTF-8.
- ImGui только в `onDrawMenu` / `onDrawOverlay`.
- `Net` / `Memory` / `Raw` — только после `LthX.setUnsafe(true)` (перезагружает скрипты).
- Нет `wait`. Таймеры: `LthX.after` / `LthX.every`.

---

## 3. Lifecycle

| Функция | Alias | Аргументы | Когда |
|---------|-------|-----------|-------|
| `onStart` | `start` | — | загрузка |
| `onStop` | `stop` | — | выгрузка |
| `onFrame` | `frame` | `dt` (сек) | каждый кадр |
| `onDrawMenu` | `drawMenu` | — | открыто меню LthX |
| `onDrawOverlay` | `drawOverlay` | — | оверлей |

---

## 4. LthX / Notify / Script

### Script

| Поле | Тип |
|------|-----|
| `Script.name` | `string` |

### LthX

| API | Сигнатура | Описание |
|-----|-----------|----------|
| `log` | `(msg)` | notify warning |
| `chat` | `(msg)` | локальный чат |
| `notify` | `(title, text [, type])` | `"success"` / `"error"` / иначе warning |
| `on` | `(name, fn)` → `fn` | подписка |
| `off` | `(name, fn)` → `bool` | |
| `emit` | `(name)` | |
| `after` / `setTimeout` | `(ms, fn)` → `id` | один раз |
| `every` / `setInterval` | `(ms, fn)` → `id` | повтор |
| `cancelTimer` / `clearTimer` | `(id)` | |
| `menuOpen` | `()` → `bool` | |
| `scriptName` | `()` → `string` | |
| `save` / `reload` | `()` → `bool` | |
| `isUnsafe` | `()` → `bool` | |
| `setUnsafe` | `(bool)` → `bool` | вкл. Net/Memory/Raw |
| `setSyncZOffset` / `getSyncZOffset` | | только UNSAFE |

Globals: `on`, `emit`, `setTimeout`, `setInterval`, `clearTimer`.

### Notify

| API | Сигнатура |
|-----|-----------|
| `show` | `(title, text [, type])` |
| `info` / `success` / `error` | `(title, text)` |

---

## 5. Input / Key

### Key

`LBUTTON`, `RBUTTON`, `MBUTTON`, `XBUTTON1`, `XBUTTON2`,  
`SHIFT`, `CTRL`, `ALT`, `SPACE`, `ENTER`, `ESC`, `TAB`,  
`INSERT`, `DELETE`, `HOME`, `END`, `PRIOR`, `NEXT`,  
`LEFT`, `RIGHT`, `UP`, `DOWN`, `A`–`Z`, `F1`–`F12`.

### Input

| API | Сигнатура | Описание |
|-----|-----------|----------|
| `isKeyDown` | `(vk)` → `bool` | меню открыто → только F1–F12 |
| `wasKeyPressed` | `(vk)` → `bool` | нажатие |
| `wasKeyReleased` | `(vk)` → `bool` | отпускание |
| `getMouseWheelDelta` | `()` → `float` | |
| `pressKey` | `(vk)` | tap |
| `holdKey` / `releaseKey` | `(vk)` | удержание |
| `releaseKeys` | `()` | |
| `isMouseDown` | `(button)` → `bool` | `0` L, `1` R, `2` M |
| `mousePos` | `()` → `{x,y}` | |
| `setPad` | `(lr, ud)` | ±128 |
| `clearPads` | `()` | |
| `setGameKeyState` | `(key, state)` | как MoonLoader: `1,-255` вперёд, `16,255` sprint, `14,255` jump |

---

## 6. ImGui / Ui

Только в `onDrawMenu` / `onDrawOverlay`.

### Виджеты

| API | Сигнатура |
|-----|-----------|
| `Text` | `(text)` |
| `TextColored` | `(r,g,b,a, text)` |
| `Button` | `(label [, w [, h]])` → `bool` |
| `SmallButton` | `(label)` → `bool` |
| `InvisibleButton` | `(id, w, h)` → `bool` |
| `Checkbox` | `(label, value)` → `changed, value` |
| `SliderFloat` / `SliderInt` | `(label, v, min, max)` → `changed, v` |
| `InputText` | `(label, value)` → `changed, value` |
| `InputTextMultiline` | `(label, value [, w [, h]])` → `changed, value` |
| `Combo` | `(label, current, items)` → `changed, current` |
| `ColorEdit3` / `ColorEdit4` | → `changed, …` |
| `Selectable` | `(label [, selected [, flags]])` → `bool` |
| `CollapsingHeader` | `(label [, flags])` → `bool` |
| `TreeNode` / `TreePop` | `(label)` / `()` |
| `ProgressBar` | `(frac)` |
| `IsItemHovered` / `IsItemClicked` | `([button])` → `bool` |
| `SetTooltip` | `(text)` |

### Окна / layout

| API | Описание |
|-----|----------|
| `Begin(name)` | → `visible` |
| `Begin(name, open [, flags])` | → `open, visible` |
| `End` | |
| `BeginChild` / `EndChild` | |
| `SameLine` / `Separator` / `Spacing` / `NewLine` / `Dummy` | |
| `Indent` / `Unindent` / `BeginGroup` / `EndGroup` | |
| `PushID` / `PopID` | |
| `GetCursorPos` / `SetCursorPos` | |
| `GetContentRegionAvail` | |
| `SetNextItemWidth` / `PushItemWidth` / `PopItemWidth` | |
| `SetNextWindowPos` / `SetNextWindowSize` | `(x,y [, cond])` |
| `PushStyleColor` / `PopStyleColor` | |
| `PushStyleVar` / `PopStyleVar` | |
| `GetDisplaySize` / `GetMousePos` / `GetDeltaTime` | |

Константы: `Col_*`, `StyleVar_*`, `Cond_*`, `WindowFlags_*`.

### Ui

Обёртки: `checkbox`, `slider`, `sliderInt`, `button`, `text`, `separator`, `sameLine`, `begin`, `close` / `finish`, `menuOpen`.

---

## 7. CVector

```lua
local a = CVector(1, 2, 3)
print(a.x, a:length(), a:distance(CVector(0, 0, 0)))
```

Поля: `x`, `y`, `z`.  
Методы: `length()`, `distance(other)`, `add`, `sub`, `scale(s)`, `normalized()`.

---

## 8. LocalPlayer

| API | Сигнатура |
|-----|-----------|
| `getPos` | `()` → `CVector` |
| `setPos` | `(CVector)` |
| `setPosition` | `(x,y,z)` |
| `teleport` | `(x,y,z [, opts])` — `ground`, `stream`, `interior`, `cameraBehind` |
| `getHealth` / `setHealth` | `()` / `(float)` |
| `getArmor` / `setArmor` | `()` / `(float)` |
| `getArmour` / `setArmour` | алиасы |
| `addArmor` / `addArmour` | `(points)` |
| `isInVehicle` | `()` → `bool` |
| `getVehicleID` | `()` → `int` / `-1` |
| `getVehicleHealth` / `setVehicleHealth` | |
| `getVelocity` / `setVelocity` | `()` → `CVector` / `(x,y,z)` |
| `changeSkin` / `getSkin` | `(id)` / `()` |
| `getPointer` / `getSampPointer` | `()` → `int` |
| `getRotation` / `setRotation` | |
| `getHeading` / `setHeading` | `setHeading` пишет Rotation + AimRotation |
| `getWeapon` / `getAmmo` | |
| `getInterior` / `setInterior` | |
| `setCameraBehind` | `()` |
| `setControllable` | `(bool)` |
| `getMoney` | `()` → `int` |
| `getSpecialAction` / `setSpecialAction` / `clearSpecialAction` | `2` = jetpack |
| `applyAnimation` | `(anim, lib [, delta, loop, lockX, lockY, freeze, time])` |
| `clearAnimations` | `()` |
| `isSpectating` / `isSpawned` | `()` → `bool` |
| `getScore` / `getPing` / `getId` / `getName` | |
| `getBone` | `(bone)` → `{x,y,z}` / `nil` |

---

## 9. Player

| API | Сигнатура |
|-----|-----------|
| `get` | `(id)` → table |
| `all` | `()` → `{ids…}` |
| `closest` | `([maxDist])` → `{id, distance}` |
| `getPos` / `distanceTo` / `getName` | `(id)` |
| `isConnected` / `isStreamed` / `isAfk` / `isNpc` | `(id)` → `bool` |
| `getScore` / `getPing` / `getColor` / `getTeam` | `(id)` |
| `getState` / `getSpecialAction` / `getVehicleId` | `(id)` |
| `getWeaponName` | `(weaponId)` → `string` |
| `getBone` | `(id, bone)` → `{x,y,z}` / `nil` |

`Player.get(id)`: `id`, `connected`, `name`, `streamed`, `pos`, `x`,`y`,`z`, `health`, `maxHealth`, `armor`, `weapon`, `weaponName`, `color`, `team`, `score`, `ping`, `afk`, `npc`, `state`, `specialAction`, `vehicleId`, `distance`.

---

## 10. Vehicle

| API | Сигнатура |
|-----|-----------|
| `getById` | `(id)` → `{id,model,health,pos,driver,passengers}` |
| `getSpeed` / `setSpeed` / `multiplySpeed` | локальный транспорт |
| `warpInto` | `(id [, seat=0])` → `bool` |
| `all` | до 48 streamed |
| `exists` / `count` / `nearest` | |
| `getPos` / `getModel` / `getHealth` | `(id)` |

---

## 11. Skin

| API | Сигнатура |
|-----|-----------|
| `change` | `(id)` `0..311` |
| `current` | `()` → `int` |
| `name` | `(id)` → `string` |

---

## 12. Render

| API | Сигнатура |
|-----|-----------|
| `color` / `rgb` | `(r,g,b [, a])` → `ImU32` |
| `worldToScreen` | `(pos)` → `ok, sx, sy` |
| `drawLine` / `drawRect` / `drawRectFilled` | |
| `drawCircle` / `drawCircleFilled` | |
| `drawTriangle` | `(x1,y1,x2,y2,x3,y3,col [, filled])` |
| `drawPolygon` | `(points, col [, filled])` |
| `drawBox3D` | `(min, max, col [, thickness])` |
| `drawBones` | `(playerId, col [, thickness])` — `playerId < 0` = local |
| `drawText` / `drawTextCentered` | `(x,y,col,text [, size [, layer]])` |
| `measureText` | `(text)` → `{x,y}` |

`layer`: `"background"` (default) / `"foreground"`.

---

## 13. Game

| API | Описание |
|-----|----------|
| `isReady` / `ready` | netgame + ped |
| `tick` | `GetTickCount64` |
| `fpsDelta` | |
| `screenWidth` / `screenHeight` | |
| `getScreenSize` | `{x,y}` |
| `screenSize` | `w, h` |
| `isMenuOpen` / `setMenuOpen` | меню LthX |
| `isForeground` | окно игры в фокусе |
| `isPaused` | pause GTA |
| `setScriptUi` / `isScriptUi` | захват курсора |
| `getActiveTab` / `setActiveTab` | `0..5` |
| `getAccent` / `setAccent` | `{r,g,b,a}` / `(r,g,b [, a])` |
| `getTime` / `setTime` | как World |
| `getWeather` / `setWeather` | |
| `getGravity` / `setGravity` | |
| `findGroundZ` | `(x,y)` |
| `getInterior` / `getMoney` | |
| `gameText` | `(text [, time=3000 [, style=3]])` |

---

## 14. Util / Json

### Util

| API | Сигнатура |
|-----|-----------|
| `getClipboard` | `()` → `string` |
| `setClipboard` | `(text)` → `bool` |

### Json

| API | Сигнатура |
|-----|-----------|
| `encode` | `(value)` → `string` |
| `decode` | `(string)` → value |

---

## 15. SAMP

### Свойства

`isConnected`, `localId`, `localName`, `playerCount`.

### Чат / диалог / сервер

| API | Описание |
|-----|----------|
| `sendChat` | локально в чат |
| `say` / `sendCommand` | на сервер |
| `addMessage` | `(text [, color])` |
| `addChatMessage` | `(prefix, text [, color])` |
| `lastMessage` / `isChatOpen` / `getChatInput` | |
| `openChat` / `closeChat` | |
| `isDialogOpen` | |
| `getDialogId` / `getDialogType` / `getDialogCaption` / `getDialogText` | |
| `hideDialog` | |
| `closeDialog` | `([button=0])` |
| `sendDialogResponse` | `([button=1])` |
| `isScoreboardOpen` | |
| `getHost` / `getHostname` / `getPort` | |
| `getGameState` | см. ниже |
| `gameText` | `(text [, time [, style]])` |
| `version` | клиент |
| `maxPlayers` | |
| `objectExists` / `objectCount` | pool |
| `actorExists` / `textDrawExists` | |
| `getCheckpoint` | `{x,y,z,size}` / `nil` |
| `setCheckpoint` / `clearCheckpoint` | |
| `getRaceCheckpoint` | `{x,y,z,size,type,next}` / `nil` |
| `setRaceCheckpoint` / `clearRaceCheckpoint` | |
| `registerCommand` / `registerChatCommand` | `(name, fn)` → `bool`, max 16 |

### Игроки

`isPlayerConnected`, `isPlayerStreamed`, `getPlayerName`, `getPlayerId`,  
`getPlayerPing`, `getPlayerScore`, `getMaxPlayerId`.

### GameState

| Значение | Смысл |
|---------:|-------|
| 0 | none |
| 1 | wait connect |
| 2 | await join |
| 3 | connected |
| 4 | restarting |
| 5 | disconnected |

---

## 16. World

| API | Описание |
|-----|----------|
| `getTime` / `setTime` | `h, m` |
| `getWeather` / `setWeather` | |
| `getGravity` / `setGravity` | |
| `findGroundZ` | `(x,y)` |
| `requestCollision` | `(x,y)` |
| `getInterior` / `getMoney` | |
| `getHour` / `getMinute` | |
| `lineOfSight` / `los` | `(x1,y1,z1,x2,y2,z2)` → `bool` |

---

## 17. Camera

| API | Описание |
|-----|----------|
| `isActive` / `setActive` / `toggle` | freecam |
| `fly` | `(_, speed?=15)` |
| `getPos` | `{x,y,z}` |
| `getFov` / `setFov` | `30..120` |

---

## 18. Objects

| API | Описание |
|-----|----------|
| `exists` / `count` / `clear` | tracked |
| `poolExists` / `poolCount` | SAMP pool |
| `get` | `(id)` → info / `nil` |
| `ids` / `all` | |
| `forEach` | `(fn(id, modelId, x,y,z))` |
| `inRadius` | `(cx,cy,cz,r)` |
| `inCuboid` | `(…)` |
| `removed` / `removedBuildings` | |

Поля объекта: `id`, `modelId`, `position`, `rotation`, `drawDistance`, `cameraCol`, `isDynamic`, `attachedVehicleId`, `attachedObjectId`, `attachOffset`, `attachRotation`, `materials`, …

---

## 19. Path / Radar

### Path

| API | Описание |
|-----|----------|
| `find` | `(x,y [, z [, kind]])` → маршрут |
| `findFrom` | `(sx,sy,sz, dx,dy,dz [, kind])` |
| `walkTo` / `runTo` | `(x,y [, z])` → `bool` |
| `driveTo` | `(x,y [, z [, speed]])` → `bool` |
| `stop` / `isActive` | |
| `getProgress` | table |
| `setEmulation` / `getEmulation` | `"auto"` / `"keys"` / `"pad"` / `"velocity"` |
| `getResolvedEmulation` | |
| `setAutoJump` / `getAutoJump` | default `false` |
| `setNoClip` / `getNoClip` | default `false` |
| `setAvoidObstacles` / `getAvoidObstacles` | default `false` |
| `jump` | `([durationSec=0.35])` |
| `goToWaypoint` / `goToCheckpoint` | → `bool` |

Событие `pathArrive` — цель достигнута.

### Radar

| API | Описание |
|-----|----------|
| `getWaypoint` | `{x,y,z}` / `nil` |
| `hasWaypoint` | `bool` |
| `setWaypoint` / `placeWaypoint` | `(x,y,z)` → `bool` |
| `clearWaypoint` / `removeWaypoint` | |

Сэмплы: `Temp/samples/farmbot_malinovka.lua`, `checkpoint_bot.lua`.

---

## 20. Events / Raknet

### Регистрация

```lua
Events.onSendChat(function(text) end)
Events.on("onIncomingRpc", function(id, bytes) end)
```

### Хуки

| Hook | Аргументы | Возврат |
|------|-----------|---------|
| `onSendChat` / `onSendCommand` | `(text)` | `false` — блок; `string` — замена |
| `onSendSpawn` | — | `false` — блок |
| `onOutgoingRpc` / `onSendRpc` | `(rpcId, bytes)` | `false` — блок |
| `onIncomingRpc` / `onReceiveRpc` | `(rpcId, bytes)` | `false` — блок |
| `onOutgoingPacket` / `onSendPacket` | `(packetId, bytes)` | `false` — блок |
| `onIncomingPacket` / `onReceivePacket` | `(packetId, bytes)` | `false` — блок |
| `onIncomingBullet` | | |
| `onCreateObject` / `onDestroyObject` | | |
| `onSetObjectPosition` / `onSetObjectRotation` | | |
| `onMoveObject` / `onSetObjectMaterial` / `onSetObjectText` | | |
| `onRemoveBuilding` | | |

### Raknet.RPC

| Имя | ID |
|-----|---:|
| `CHAT` | 101 |
| `SERVERCOMMAND` | 50 |
| `DIALOGRESPONSE` | 62 |
| `ENTERVEHICLE` | 26 |
| `EXITVEHICLE` | 154 |
| `MAPMARKER` | 119 |
| `SPAWN` | 52 |
| `DEATH` | 53 |
| `GIVETAKEDAMAGE` | 115 |
| `CLICKPLAYER` | 23 |
| `PICKEDUPPICKUP` | 131 |
| `REQUESTCLASS` | 128 |
| `REQUESTSPAWN` | 129 |
| `SETINTERIORID` | 118 |
| `CLIENTJOIN` | 25 |
| `CLICKTEXTDRAW` | 83 |
| `REMOVEBUILDING` | 43 |
| `CREATEOBJECT` | 44 |
| `SETOBJECTPOS` | 45 |
| `SETOBJECTROT` | 46 |
| `DESTROYOBJECT` | 47 |
| `SETOBJECTMATERIAL` | 84 |
| `MOVEOBJECT` | 99 |

### Raknet.PACKET

| Имя | ID |
|-----|---:|
| `VEHICLE_SYNC` | 200 |
| `AIM_SYNC` | 203 |
| `WEAPONS_UPDATE` | 204 |
| `STATS_UPDATE` | 205 |
| `BULLET_SYNC` | 206 |
| `PLAYER_SYNC` | 207 |
| `MARKERS_SYNC` | 208 |
| `UNOCCUPIED_SYNC` | 209 |
| `TRAILER_SYNC` | 210 |
| `PASSENGER_SYNC` | 211 |
| `SPECTATOR_SYNC` | 212 |
| `RPC` | 20 |

---

## 21. File / Storage

Одинаковый API. Лимит **64 KiB**.

| API | Описание |
|-----|----------|
| `resolve` | `(path)` → absolute / `""` |
| `exists` | `(path)` → `bool` |
| `read` | `(path)` → `string` |
| `write` | `(path, text)` → `bool` |
| `append` | `(path, text)` → `bool` |
| `remove` / `mkdir` | `(path)` → `bool` |
| `list` | `([path])` → `{names…}` |

---

## 22. Cheat

Свойства get/set. Цвета `{r,g,b,a}` в `0..1`.

### ESP

`espBox`, `espBox2D`, `espBox3D`, `espBoxStyle` (0–1), `espBoxColor`,  
`espHealthBar`, `espHealthText`, `espSkeleton`, `espIgnoreTeam`, `espRainbow`,  
`nametags`, `tracers`

### Aim

`silentAim`, `smoothAim`, `smoothAimSmoothness` (1–20),  
`silentFov` (10–300), `silentMaxDist` (0–150), `silentHitChance` (0–100),  
`silentBone` (0–6), `silentTarget` (0–1),  
`silentMinHp` / `smoothMinHp` (0–176)

### Оружие

`rapidFire`, `noReload`, `autoCBug`,  
`cbugBulletDelay` / `cbugCrouchDelay` (0–200),  
`noSpread`, `fastCrosshair`

### Движение / транспорт / скин

`airbreak`, `airbreakSpeed` (1–5), `noFallDamage`, `noAnims`,  
`godmode` / `carGodMode`,  
`fastSprint`, `fastSprintSpeed` (1–3), `infiniteEnergy`,  
`bigJump`, `jumpHeight` (1–5),  
`speedHack`, `speedHackSpeed` (1–10),  
`skinChanger`, `skinId` (0–311)

---

## 23. Net / Memory / Raw

Только после `LthX.setUnsafe(true)`.

### Net

| API | Описание |
|-----|----------|
| `fromHex` / `toHex` | |
| `sendPacket` | `(bytes\|hex)` → `bool` |
| `sendRpc` | `(id, bytes\|hex)` → `bool` |
| `setSyncZOffset` / `getSyncZOffset` | |

Константы: `HIGH_PRIORITY`, `MEDIUM_PRIORITY`, `LOW_PRIORITY`,  
`RELIABLE`, `RELIABLE_ORDERED`, `UNRELIABLE`, `UNRELIABLE_SEQUENCED` (сейчас игнорируются).

### Memory

| API | Описание |
|-----|----------|
| `readI8` … `readDouble` | `(addr)` → value / `nil` |
| `read` | `(addr, type)` — `"i8"`…`"double"` |
| `writeI8` … `writeDouble` | → `bool` |
| `readString` / `writeString` | |
| `readBytes` / `writeBytes` | ≤4096 |

### Raw

| API | Описание |
|-----|----------|
| `module` | `([name])` → base |
| `moduleSize` | `([name])` |
| `proc` | `(mod, export)` |
| `addr` | `(base, offset)` |
| `readPtr` / `writePtr` | |
| `getLastError` | |
| `PAGE_*` | константы |

---

## 24. MoonLoader

| MoonLoader | LthX |
|------------|------|
| `addArmourToChar` | `LocalPlayer.addArmor` |
| `setGameKeyState` | `Input.setGameKeyState` |
| `isKeyDown` / `wasKeyPressed` / `wasKeyReleased` | `Input.*` |
| `getMousewheelDelta` | `Input.getMouseWheelDelta` |
| `processLineOfSight` | `World.lineOfSight` |
| `convert3DCoordsToScreen` | `Render.worldToScreen` |
| `printString*` | `SAMP.gameText` |
| `placeWaypoint` / `removeWaypoint` | `Radar.setWaypoint` / `clearWaypoint` |
| `getTargetBlipCoordinates` | `Radar.getWaypoint` |
| `showCursor` / lock | `Game.setScriptUi` / `LocalPlayer.setControllable` |
| clipboard | `Util.*` |
| `isGamePaused` | `Game.isPaused` |
| `encodeJson` / `decodeJson` | `Json.*` |
| packet/RPC events | `Events.*` |
| `wait` / opcodes / CLEO | нет |

---

## 25. Примеры

### Чат-команда

```lua
SAMP.registerChatCommand("hp", function()
  SAMP.addMessage("HP " .. LocalPlayer.getHealth(), 0xFFFFD700)
end)
```

### Блок чата

```lua
Events.onSendChat(function(text)
  if text:find("secret") then return false end
  return "[tag] " .. text
end)
```

### Path к чекпоинту

```lua
Path.setEmulation("auto")
Path.setAutoJump(false)
Path.setAvoidObstacles(false)
Path.goToCheckpoint()
```

### Json + файл

```lua
local cfg = { sprint = 1.4 }
function onStart()
  local raw = File.read("cfg.json")
  if raw and #raw > 0 then
    local ok, t = pcall(Json.decode, raw)
    if ok and type(t) == "table" then cfg = t end
  end
end
function onStop()
  File.write("cfg.json", Json.encode(cfg))
end
```

### Броня

```lua
LocalPlayer.addArmor(25)
Util.setClipboard(tostring(LocalPlayer.getArmor()))
```

### UI на F10

```lua
local open = false
function onFrame()
  if Input.wasKeyPressed(Key.F10) then
    open = not open
    Game.setScriptUi(open)
  end
end
function onDrawOverlay()
  if not open then return end
  if ImGui.Begin("Quick") then
    local ch, v = ImGui.Checkbox("Silent", Cheat.silentAim)
    if ch then Cheat.silentAim = v end
  end
  ImGui.End()
end
function onStop()
  Game.setScriptUi(false)
end
```
