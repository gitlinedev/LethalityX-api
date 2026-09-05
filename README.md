# LethalityX Lua API

Полный справочник Lua API для скриптов LethalityX (LthX).

---

## Содержание

1. [Быстрый старт](#1-быстрый-старт)
2. [Ограничения среды](#2-ограничения-среды)
3. [Lifecycle](#3-lifecycle)
4. [Core — LthX / Notify / Script](#4-core--lthx--notify--script)
5. [Input / Key](#5-input--key)
6. [ImGui / Ui](#6-imgui--ui)
7. [CVector](#7-cvector)
8. [LocalPlayer](#8-localplayer)
9. [Player](#9-player)
10. [Vehicle](#10-vehicle)
11. [Skin](#11-skin)
12. [Render](#12-render)
13. [Game](#13-game)
14. [SAMP](#14-samp)
15. [World](#15-world)
16. [Camera](#16-camera)
17. [Objects](#17-objects)
18. [Path / Radar](#18-path--radar)
19. [Events / Raknet](#19-events--raknet)
20. [File / Storage](#20-file--storage)
21. [Cheat](#21-cheat)
22. [Net / Memory / Raw (UNSAFE)](#22-net--memory--raw-unsafe)
23. [Примеры скриптов](#23-примеры-скриптов)
24. [Samples в репозитории](#24-samples-в-репозитории)

---

## 1. Быстрый старт

```lua
function onStart()
  SAMP.addMessage("[MyScript] loaded", 0xFF7FFFD4)
  Notify.success("MyScript", "ready")
end

function onFrame(dt)
  if Input.wasKeyPressed(Key.F3) then
    local p = LocalPlayer.getPos()
    SAMP.addMessage(string.format("pos %.1f %.1f %.1f", p.x, p.y, p.z), 0xFFFFFFFF)
  end
end

function onStop()
  Game.setScriptUi(false)
end
```

Альтернатива через `LthX.on` / `Events.on`:

```lua
LthX.on("frame", function(dt) end)
Events.on("onCreateObject", function(id) end)
```

---

## 2. Ограничения среды

| Доступно | Недоступно |
|----------|------------|
| `base`, `string`, `math`, `table`, `utf8` | `io`, `os`, `package`, `require` |
| `File` / `Storage` (sandbox) | `dofile` / `loadfile` / `load` |
| | `coroutine` |

- Файлы только через `File`/`Storage` (max **64 KiB**, без `..` и absolute path).
- Цвет чата: **ARGB** `0xAARRGGBB` (пример: `0xFFFFD700`).
- `Net` / `Memory` / `Raw` / sync Z — только при включённом **UNSAFE** в настройках.
- ImGui только внутри `onDrawMenu` / `onDrawOverlay`.

---

## 3. Lifecycle

| Имя | Alias | Args | Когда |
|-----|-------|------|-------|
| `onStart` | `start` | — | после загрузки скрипта |
| `onStop` | `stop` | — | перед выгрузкой |
| `onFrame` | `frame` | `dt: number` (сек) | каждый тик |
| `onDrawMenu` | `drawMenu` | — | меню LthX открыто; ImGui OK |
| `onDrawOverlay` | `drawOverlay` | — | оверлей; ImGui OK |

```lua
function onFrame(dt)
  -- dt ≈ 1/fps
end

function onDrawOverlay()
  ImGui.Text("hello")
end
```

Регистрация без глобальных `function on*`:

```lua
LthX.on("start", function() end)
LthX.on("frame", function(dt) end)
Events.on("onDrawMenu", function() end)
```

---

## 4. Core — LthX / Notify / Script

### Script

| Поле | Тип | Описание |
|------|-----|----------|
| `Script.name` | `string` | id / имя скрипта |

### LthX

| API | Сигнатура | Возврат | Заметки |
|-----|-----------|---------|---------|
| `log` | `(msg)` | — | notify warning |
| `chat` | `(msg)` | — | локальный чат, белый |
| `notify` | `(title, text [, type])` | — | `"success"` / `"error"` / иначе warning |
| `on` | `(name, cb)` | `cb` | алиасы `onFrame`→`frame` и т.д. |
| `off` | `(name, cb)` | `bool` | имя как stored: `"frame"` |
| `emit` | `(name, …)` | — | emit с `dt=0` |
| `after` / `setTimeout` | `(ms, cb)` | `timerId` | one-shot, `ms >= 0` |
| `every` / `setInterval` | `(ms, cb)` | `timerId` | `ms >= 1` |
| `cancelTimer` / `clearTimer` | `(id)` | — | |
| `menuOpen` | `()` | `bool` | |
| `scriptName` | `()` | `string` | |
| `save` / `reload` | `()` | `bool` | |
| `setSyncZOffset` | `(meters)` | — | **UNSAFE** |
| `getSyncZOffset` | `()` | `float` | **UNSAFE** |

Globals: `on`, `emit`, `setTimeout`, `setInterval`, `clearTimer`.

### Notify

```lua
Notify.show("T", "text", "success")
Notify.info("T", "i")
Notify.success("T", "ok")
Notify.error("T", "fail")
```

### Пример

```lua
local h = LthX.on("frame", function(dt) end)
LthX.off("frame", h)

local id = LthX.after(1500, function()
  Notify.success("Timer", "done")
end)
-- LthX.cancelTimer(id)
```

---

## 5. Input / Key

### Key constants

`LBUTTON`, `RBUTTON`, `MBUTTON`, `XBUTTON1`, `XBUTTON2`,  
`SHIFT`, `CTRL`, `ALT`, `SPACE`, `ENTER`, `ESC`, `TAB`,  
`INSERT`, `DELETE`, `HOME`, `END`, `PRIOR`, `NEXT`,  
`LEFT`, `RIGHT`, `UP`, `DOWN`, `A`–`Z`, `F1`–`F12`.

### Input

| API | Args | Returns | Notes |
|-----|------|---------|-------|
| `isKeyDown` | `vk` | `bool` | при открытом меню — только F1–F12 |
| `wasKeyPressed` | `vk` | `bool` | edge; меню → не-F = false |
| `pressKey` | `vk` | — | synthetic down+up |
| `isMouseDown` | `button` | `bool` | `0` L, `1` R, `2` M |
| `mousePos` | — | `{x,y}` | client coords |

```lua
if Input.wasKeyPressed(Key.F6) then
  SAMP.addMessage("F6", 0xFFFFFFFF)
end

if Input.isKeyDown(Key.CTRL) and Input.wasKeyPressed(Key.C) then
  -- Ctrl+C
end

local m = Input.mousePos()
-- m.x, m.y
```

---

## 6. ImGui / Ui

Только в `onDrawMenu` / `onDrawOverlay`.

### ImGui

| Method | Сигнатура | Returns |
|--------|-----------|---------|
| `Text` | `(text)` | — |
| `TextColored` | `(r,g,b,a, text)` | floats 0–1 |
| `Button` | `(label [, w=0 [, h=0]])` | `bool` |
| `SmallButton` | `(label)` | `bool` |
| `Checkbox` | `(label, value)` | `changed, value` |
| `SliderFloat` | `(label, value, min, max)` | `changed, value` |
| `SliderInt` | `(label, value, min, max)` | `changed, value` |
| `InputText` | `(label, value)` | `changed, value` (~511) |
| `Combo` | `(label, current, items)` | `changed, current` (0-based) |
| `ColorEdit3` / `ColorEdit4` | `(label, r,g,b[,a])` | `changed, …` |
| `Begin` / `End` | `(name)` / `()` | `bool` / — |
| `BeginChild` / `EndChild` | `(id [, w [, h]])` | `bool` / — |
| `SameLine`, `Separator`, `Spacing`, `NewLine` | | |
| `SetNextItemWidth` / `PushItemWidth` / `PopItemWidth` | | |
| `ProgressBar` | `(frac)` | — |
| `IsItemHovered` | `()` | `bool` |
| `SetTooltip` | `(text)` | — |
| `Dummy` | `(w,h)` | — |
| `SetNextWindowPos` / `Size` | `(x,y)` / `(w,h)` | FirstUseEver |

### Ui (короткие алиасы)

`checkbox`, `slider`, `sliderInt`, `button`, `text`, `separator`, `sameLine`,  
`begin(name)`, `close` / `finish`, `menuOpen()`.

```lua
local enabled = false
local speed = 1.0

function onDrawMenu()
  local ch, v = ImGui.Checkbox("Enabled", enabled)
  if ch then enabled = v end

  local ch2, s = ImGui.SliderFloat("Speed", speed, 0.1, 5.0)
  if ch2 then speed = s end

  if ImGui.Button("Notify", 120, 0) then
    Notify.info("UI", "clicked")
  end
end
```

Своё окно + курсор:

```lua
local uiOpen = false

function onFrame()
  if Input.wasKeyPressed(Key.F10) then
    uiOpen = not uiOpen
    Game.setScriptUi(uiOpen)
  end
end

function onDrawOverlay()
  if not uiOpen then return end
  if ImGui.Begin("My Panel") then
    ImGui.Text("Hello")
  end
  ImGui.End()
end
```

---

## 7. CVector

```lua
local a = CVector(1, 2, 3)
local b = CVector()
b.x, b.y, b.z = 4, 5, 6

print(a:length())
print(a:distance(b))
local c = a:add(b):scale(0.5):normalized()
```

Поля: `x`, `y`, `z`.  
Методы: `length()`, `distance(other)`, `add`, `sub`, `scale`, `normalized`.

---

## 8. LocalPlayer

| Method | Args | Returns |
|--------|------|---------|
| `getPos` / `setPos` | — / `CVector` | `CVector` / — |
| `setPosition` | `x,y,z` | — |
| `getHealth` / `setHealth` | — / `float` | `float` / — |
| `getArmor` / `setArmor` | — / `float` | `float` / — |
| `isInVehicle` | — | `bool` |
| `getVehicleID` | — | id или `-1` |
| `getVehicleHealth` / `setVehicleHealth` | | |
| `getVelocity` / `setVelocity` | — / `x,y,z` | `CVector` |
| `changeSkin` / `getSkin` | id / — | — / model |
| `getPointer` / `getSampPointer` | — | `int` |
| `getRotation` / `setRotation` | — / heading | `float` |
| `getWeapon` / `getAmmo` | — | `int` |
| `getInterior` / `getMoney` / `getSpecialAction` | — | `int` |
| `isSpectating` / `isSpawned` | — | `bool` |
| `getScore` / `getPing` / `getId` / `getName` | — | |
| `getBone` | `bone: int` | `{x,y,z}` / `nil` |

```lua
local p = LocalPlayer.getPos()
LocalPlayer.setHealth(100)
LocalPlayer.setArmor(50)

if LocalPlayer.isSpawned() then
  local head = LocalPlayer.getBone(8)
  if head then
    -- head.x, head.y, head.z
  end
end
```

---

## 9. Player

| Method | Args | Returns |
|--------|------|---------|
| `get(id)` | | table (см. ниже) |
| `all()` | | `{ids…}` |
| `closest([maxDist=10000])` | | `{id, distance}`; `id=-1` если нет |
| `getPos` / `distanceTo` / `getName` | id | |
| `isConnected` / `isStreamed` / `isAfk` / `isNpc` | id | `bool` |
| `getScore` / `getPing` / `getColor` / `getTeam` / `getState` / `getSpecialAction` / `getVehicleId` | id | |
| `getWeaponName(weaponId)` | | `string` |
| `getBone(id, bone)` | | `{x,y,z}` / `nil` |

`Player.get(id)` поля:  
`id`, `connected`, `name`, `streamed`, `pos`, `x`,`y`,`z`, `health`, `armor`, `weapon`, `weaponName`, `color`, `team`, `score`, `ping`, `afk`, `npc`, `state`, `specialAction`, `vehicleId`, `distance`.

```lua
local near = Player.closest(50)
if near.id >= 0 then
  local info = Player.get(near.id)
  SAMP.addMessage(info.name .. " dist=" .. string.format("%.1f", near.distance), 0xFFFFFFFF)
end

for _, id in ipairs(Player.all()) do
  if Player.isStreamed(id) then
    -- …
  end
end
```

---

## 10. Vehicle

| Method | Args | Returns |
|--------|------|---------|
| `getById(id)` | | `{id,model,health,pos,driver=-1,passengers={}}` |
| `getSpeed` / `setSpeed` / `multiplySpeed` | local veh | `CVector` / — |
| `all()` | | до 48: `id,x,y,z,health,engineOn,model,pointer,color1,color2` |
| `exists` / `count` / `nearest` | | |
| `getPos` / `getModel` / `getHealth` | id | |

```lua
local id = Vehicle.nearest()
if id and Vehicle.exists(id) then
  local m = Vehicle.getModel(id)
  local hp = Vehicle.getHealth(id)
end

if LocalPlayer.isInVehicle() then
  Vehicle.multiplySpeed(1.05)
end
```

---

## 11. Skin

| API | Args | Returns |
|-----|------|---------|
| `Skin.change` | `id` `0..311` | — включает skin changer |
| `Skin.current` | — | `int` (настройка) |
| `Skin.name` | `id` | `string` |

```lua
Skin.change(0)
Cheat.skinChanger = true
Cheat.skinId = 230
```

---

## 12. Render

Рисование на оверлее (каждый кадр, обычно из `onFrame`).

| Method | Сигнатура | Notes |
|--------|-----------|-------|
| `color(r,g,b[,a])` / `rgb(r,g,b)` | 0–255 | → `ImU32`, a default 255 |
| `worldToScreen(pos)` | `CVector` или `{x,y,z}` | → `ok, sx, sy, 0` |
| `drawLine(x1,y1,x2,y2,col[,thickness[,layer]])` | | thickness default 1 |
| `drawRect` / `drawRectFilled` | | |
| `drawCircle` / `drawCircleFilled` | | segments optional |
| `drawText` / `drawTextCentered` | `(x,y,col,text[,size[,layer]])` | |
| `measureText(text)` | | `{x,y,z=0}` |

`layer`: обычно `"background"` (default).

```lua
function onFrame()
  local ok, sx, sy = Render.worldToScreen(LocalPlayer.getPos())
  if ok then
    local col = Render.rgb(255, 220, 80)
    Render.drawTextCentered(sx, sy - 20, col, LocalPlayer.getName())
    Render.drawCircle(sx, sy, 12, col, 2)
  end
end
```

---

## 13. Game

| API | Returns / Notes |
|-----|-----------------|
| `isReady` / `ready` | `bool` — netgame + local ped |
| `tick` | `number` — `GetTickCount64` |
| `fpsDelta` | `float` |
| `screenWidth` / `screenHeight` | |
| `getScreenSize` | `{x,y}` |
| `screenSize` | `w, h` |
| `isMenuOpen` | `bool` |
| `setMenuOpen` | stub |
| `setScriptUi(open)` | захват курсора под UI скрипта |
| `isScriptUi` | `bool` |
| `getActiveTab` / `setActiveTab(0..5)` | |
| `getAccent` | `{r,g,b,a}` |
| `setAccent` | stub |
| time/weather/gravity/findGroundZ/interior/money/gameText | как World/SAMP |

```lua
if not Game.isReady() then return end

local w, h = Game.screenSize()
Game.setScriptUi(true)
```

---

## 14. SAMP

### Properties

`isConnected`, `localId`, `localName`, `playerCount`.

### Chat / dialog / server

| API | Notes |
|-----|-------|
| `sendChat(text)` | **локально** в чат |
| `say` / `sendCommand` | на сервер |
| `addMessage(text[, color])` | |
| `addChatMessage(prefix, text[, color])` | |
| `lastMessage`, `isChatOpen`, `getChatInput` | |
| `openChat` / `closeChat` | |
| `isDialogOpen`, `getDialogId/Type/Caption/Text` | |
| `hideDialog`, `closeDialog([button=0])`, `sendDialogResponse([button=1])` | |
| `isScoreboardOpen` | |
| `getHost` / `getHostname` / `getPort` | |
| `getGameState` | см. ниже |
| `gameText(text[, time=3000[, style=3]])` | |
| `version()` | строка клиента |
| `maxPlayers()` | |
| `objectExists` / `objectCount` | живой SAMP **pool** |
| `actorExists` / `textDrawExists` | |
| `getCheckpoint` | `{x,y,z,size}` / nil |
| `getRaceCheckpoint` | `{x,y,z,size,type,next}` / nil |
| `registerCommand` / `registerChatCommand` | `(name, fn(args))` → bool, max **16**, без `/` |

### Player helpers

`isPlayerConnected`, `isPlayerStreamed`, `getPlayerName`, `getPlayerId`,  
`getPlayerPing`, `getPlayerScore`, `getMaxPlayerId`.

### GameState

| Value | Meaning |
|------:|---------|
| 0 | none |
| 1 | wait connect |
| 2 | await join |
| 3 | connected |
| 4 | restarting |
| 5 | disconnected |

```lua
SAMP.registerChatCommand("hp", function()
  SAMP.addMessage("HP " .. LocalPlayer.getHealth(), 0xFFFFD700)
end)

SAMP.say("hello")          -- на сервер
SAMP.sendCommand("/gps")   -- на сервер
SAMP.addMessage("local only", 0xFFFFFFFF)

local cp = SAMP.getCheckpoint()
if cp then
  Path.walkTo(cp.x, cp.y, cp.z)
end
```

---

## 15. World

| API | Notes |
|-----|-------|
| `getTime` / `setTime(h,m)` | → `h, m` |
| `getWeather` / `setWeather` | |
| `getGravity` / `setGravity` | |
| `findGroundZ(x,y)` | `0` вне game thread |
| `getInterior` / `getMoney` | |
| `getHour` / `getMinute` | clock |
| `lineOfSight` / `los(x1,y1,z1,x2,y2,z2)` | `true` = clear |

```lua
local h, m = World.getTime()
World.setWeather(1)

local z = World.findGroundZ(0, 0)
local clear = World.los(0, 0, 5, 10, 10, 5)
```

---

## 16. Camera

Freecam API.

| API | Notes |
|-----|-------|
| `isActive` | `bool` |
| `setActive(bool)` | |
| `toggle()` | → new state |
| `fly([dt[, speed=18]])` | min speed 0.5 |
| `getPos` | `{x,y,z}` |
| `getFov` / `setFov` | clamp 30..120 |

```lua
if Input.wasKeyPressed(Key.F4) then
  Camera.toggle()
end

function onFrame(dt)
  if Camera.isActive() then
    Camera.fly(dt, 22)
  end
end
```

---

## 17. Objects

Трекер **входящих** RPC объектов сервера (create/destroy/move/material/RemoveBuilding).

- База копится после загрузки скрипта (net bridge).
- Сбрасывается при disconnect/restart (`gameState` 0/1/4/5).
- Текстуры/тексты (`libraryName`, `textureName`, material text) появляются только после RPC **84**.
- `poolExists` / `SAMP.objectExists` — живой клиентский pool, **не** полный снимок материалов.

### Methods

| API | Args | Returns |
|-----|------|---------|
| `exists` | id | `bool` |
| `count` | — | `int` |
| `clear` | — | — |
| `get` | id | table / `nil` |
| `ids` | — | `{id…}` |
| `all` | — | `{obj…}` |
| `forEach` | `fn(id, modelId, x, y, z)` | — |
| `inRadius` | `cx,cy,cz,radius` | `{obj…}` |
| `inCuboid` | `x1,y1,z1,x2,y2,z2` | `{obj…}` |
| `removed` / `removedBuildings` | — | list |
| `poolExists` / `poolCount` | id / — | SAMP pool |

### Object table (`get` / `all`)

```text
id, objectId, modelId, model
position = {x,y,z}
rotation = {x,y,z}
drawDistance, cameraCol (0/1), isDynamic
attachedVehicleId, attachedObjectId
attachOffset, attachRotation, syncRotation
materials = { { … }, … }
mat_num, mat_num_txt, mat_have
```

Material entry:

```text
materialId, matType / mat_type   -- 1 = texture, 2 = text
modelId, libraryName, textureName, color
materialSize, fontName, fontSize, bold
fontColor, backGroundColor, align, text
```

Removed building:

```text
modelId / model, position={x,y,z}, radius
```

### Events

| Hook | Args |
|------|------|
| `onCreateObject` | `id` |
| `onDestroyObject` | `id` |
| `onSetObjectPosition` | `id` |
| `onSetObjectRotation` | `id` |
| `onMoveObject` | `id` |
| `onSetObjectMaterial` | `id` |
| `onSetObjectMaterialText` | `id` |
| `onRemoveBuilding` | `modelId, x, y, z, radius` |

Полный снимок после события: `Objects.get(id)`.

```lua
Events.onCreateObject(function(id)
  local o = Objects.get(id)
  if not o then return end
  SAMP.addMessage(string.format("create #%d model %d mats=%d",
    id, o.modelId, (o.mat_num or 0) + (o.mat_num_txt or 0)), 0xFF7FFFD4)
end)

Events.onRemoveBuilding(function(model, x, y, z, r)
  -- …
end)

-- рядом с игроком
local p = LocalPlayer.getPos()
local list = Objects.inRadius(p.x, p.y, p.z, 80)
SAMP.addMessage("near objects: " .. #list, 0xFFFFD700)
```

---

## 18. Path / Radar

### Path

`find` / `findFrom` → `{ok, distance, count, nodes={{x,y,z}…}}`.  
`kind`: `"ped"` / `"walk"` / `"foot"` / `"boat"` / иначе car.

| Method | Notes |
|--------|-------|
| `walkTo(x,y[,z])` | on foot, speed ~0.18 → `bool` |
| `runTo(x,y[,z])` | ~0.28 |
| `driveTo(x,y[,z[,speed]])` | in veh; default 0.42; если `>2` → km/h÷80; clamp 0.12..0.85 |
| `stop` / `isActive` / `getProgress` | progress: `{active,index,count,x,y,z,mode}` |
| `goToWaypoint` / `goToCheckpoint` | `bool` |

### Radar

| API | Returns |
|-----|---------|
| `getWaypoint` | `{x,y,z}` / nil |
| `hasWaypoint` | `bool` |
| `clearWaypoint` | — |

```lua
local wp = Radar.getWaypoint()
if wp then
  Path.runTo(wp.x, wp.y, wp.z)
end

local route = Path.find(1000, 1000, nil, "ped")
if route.ok then
  SAMP.addMessage("nodes=" .. route.count .. " dist=" .. route.distance, 0xFFFFFFFF)
end
```

---

## 19. Events / Raknet

### Регистрация

```lua
Events.onSendChat(function(text) end)
Events.on("onIncomingRpc", function(id, bytes) end)
Events.on("frame", function(dt) end)  -- lifecycle тоже можно
```

### Network hooks

| Hook | Args | Return |
|------|------|--------|
| `onSendChat` / `onSendCommand` | `(text)` | `false` block; `string` rewrite |
| `onSendSpawn` | — | `false` block |
| `onOutgoingRpc` / `onSendRpc` | `(rpcId, bytes{})` | `false` block |
| `onIncomingRpc` / `onReceiveRpc` | `(rpcId, bytes{})` | `false` block |
| `onOutgoingPacket` / `onSendPacket` | `(packetId, bytes)` | `false` block |
| `onIncomingPacket` / `onReceivePacket` | observe | |
| `onIncomingBullet` | observe | |
| object events | см. [Objects](#17-objects) | |

```lua
Events.onSendChat(function(text)
  if text:find("secret") then
    return false -- block
  end
  return "[tag] " .. text -- rewrite
end)

Events.onIncomingRpc(function(rpcId, bytes)
  if rpcId == Raknet.RPC.CREATEOBJECT then
    -- observe / optionally return false to block
  end
end)
```

### Raknet.RPC

| Name | ID |
|------|---:|
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

| Name | ID |
|------|---:|
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

## 20. File / Storage

Одинаковый API на `File` и `Storage`.

Sandbox: max **64 KiB**, без `..` / absolute.

| API | Notes |
|-----|-------|
| `resolve(path)` | absolute sandbox path / `""` |
| `exists(path)` | `bool` |
| `read(path)` | `string` |
| `write(path, text)` | `bool` (создаёт parent dirs) |
| `append(path, text)` | `bool` |
| `remove(path)` | `bool` |
| `mkdir(path)` | `bool` |
| `list([path])` | `{names…}` |

```lua
File.mkdir("FSO")
File.write("FSO/note.txt", "hello\n")
local t = File.read("FSO/note.txt")
for _, name in ipairs(File.list("FSO")) do
  -- …
end
```

---

## 21. Cheat

Свойства get/set. Bools + числа с clamp. Цвета `{r,g,b,a}` в диапазоне `0..1`.

### ESP

`espBox`, `espBox2D`, `espBox3D`, `espBoxStyle` (0–1), `espBoxColor`,  
`espHealthBar`, `espHealthText`, `espSkeleton`, `espIgnoreTeam`, `espRainbow`,  
`nametags`, `tracers`

### Aim

`silentAim`, `smoothAim`, `smoothAimSmoothness` (1–20),  
`silentFov` (10–300), `silentMaxDist` (0–150), `silentHitChance` (0–100),  
`silentBone` (0–6), `silentTarget` (0–1),  
`silentMinHp` / `smoothMinHp` (0–176)

### Weapon

`rapidFire`, `noReload`, `autoCBug`,  
`cbugBulletDelay` / `cbugCrouchDelay` (0–200),  
`noSpread`, `fastCrosshair`

### Player / veh / skin

`airbreak`, `airbreakSpeed` (1–5), `noFallDamage`, `noAnims`, `godmode`,  
`carGodMode`, `speedHack`, `speedHackSpeed` (1–10),  
`skinChanger`, `skinId` (0–311)

```lua
Cheat.espBox2D = true
Cheat.nametags = true
Cheat.silentFov = 120

local c = Cheat.espBoxColor  -- {r,g,b,a}
Cheat.espBoxColor = { r = 1, g = 0.2, b = 0.2, a = 1 }
```

---

## 22. Net / Memory / Raw (UNSAFE)

Доступны только если в настройках разрешён unsafe Lua.

### Net

Константы: `HIGH_PRIORITY`, `MEDIUM_PRIORITY`, `LOW_PRIORITY`,  
`RELIABLE`, `RELIABLE_ORDERED`, `UNRELIABLE`, `UNRELIABLE_SEQUENCED`  
(на `send*` сейчас **игнорируются**).

| API | Notes |
|-----|-------|
| `fromHex` / `toHex` | |
| `sendPacket(bytes\|hex)` | |
| `sendRpc(id, bytes\|hex)` | |
| `setSyncZOffset` / `getSyncZOffset` | |
| `emulPacket` / `emulRpc` / sync helpers | stub → `false` |

```lua
Net.sendRpc(Raknet.RPC.SERVERCOMMAND, Net.fromHex("…"))
```

### Memory

`readI8/U8/I16/U16/I32/U32/Float/Double(addr)` → value / nil  
`read(addr, type)` — type: `"i8"|"u8"|"i16"|"u16"|"i32"|"u32"|"float"|"double"`  
`writeI8`…`writeDouble` → `bool`  
`readString(addr[, max=256≤4096])`, `writeString`  
`readBytes` / `writeBytes` (≤4096)

### Raw

`module([name])`, `moduleSize`, `proc(mod, export)`, `addr(base, off)`  
`readPtr` / `writePtr`, `getLastError`  
`PAGE_*` constants  
`pattern` / `protect` / `alloc` / `call*` / `sleep` — stubs

```lua
local base = Raw.module("samp.dll")
local size = Raw.moduleSize("samp.dll")
local ptr = Memory.readU32(Raw.addr(base, 0x1234))
```

---

## 23. Примеры скриптов

### ESP имён (Render)

```lua
function onFrame()
  for _, id in ipairs(Player.all()) do
    if Player.isStreamed(id) then
      local pos = Player.getPos(id)
      pos.z = pos.z + 1.0
      local ok, sx, sy = Render.worldToScreen(pos)
      if ok then
        Render.drawTextCentered(sx, sy, Render.rgb(255, 255, 255), Player.getName(id))
      end
    end
  end
end
```

### Чат-команды + дамп объектов

```lua
SAMP.registerChatCommand("objcount", function()
  SAMP.addMessage(string.format(
    "tracked=%d pool=%d removed=%d",
    Objects.count(), Objects.poolCount(), #Objects.removed()
  ), 0xFFFFD700)
end)

SAMP.registerChatCommand("objnear", function(arg)
  local r = tonumber(arg) or 50
  local p = LocalPlayer.getPos()
  local n = #Objects.inRadius(p.x, p.y, p.z, r)
  SAMP.addMessage("objects in " .. r .. "m: " .. n, 0xFF7FFFD4)
end)
```

### Блок исходящего чата / RPC

```lua
Events.onSendCommand(function(text)
  if text:lower():find("quit") then
    Notify.error("Block", "command blocked")
    return false
  end
end)

Events.onOutgoingRpc(function(id, bytes)
  if id == Raknet.RPC.GIVETAKEDAMAGE then
    return false -- NOP GiveTakeDamage
  end
end)
```

### Сохранение конфига

```lua
local cfg = { fov = 90, tag = "LX" }

function onStart()
  local raw = File.read("my_cfg.txt")
  if raw and #raw > 0 then
    cfg.fov = tonumber(raw:match("fov=(%d+)")) or cfg.fov
    cfg.tag = raw:match("tag=([^\n]+)") or cfg.tag
  end
end

function onStop()
  File.write("my_cfg.txt", string.format("fov=%d\ntag=%s\n", cfg.fov, cfg.tag))
end
```

### Мини-меню на F10

```lua
local open = false
local aim = false

function onFrame()
  if Input.wasKeyPressed(Key.F10) then
    open = not open
    Game.setScriptUi(open)
  end
end

function onDrawOverlay()
  if not open then return end
  if ImGui.Begin("Quick") then
    local ch, v = ImGui.Checkbox("Silent Aim", aim)
    if ch then
      aim = v
      Cheat.silentAim = v
    end
  end
  ImGui.End()
end

function onStop()
  Game.setScriptUi(false)
end
```

### Следование к ближайшему игроку

```lua
SAMP.registerChatCommand("follow", function()
  local n = Player.closest(80)
  if n.id < 0 then
    SAMP.addMessage("nobody nearby", 0xFFFF0000)
    return
  end
  local p = Player.getPos(n.id)
  Path.runTo(p.x, p.y, p.z)
end)

SAMP.registerChatCommand("stopfollow", function()
  Path.stop()
end)
```

---

## 24. Samples в репозитории

| File | Описание |
|------|----------|
| `Internal/Features/Scripting/samples/objects.lua` | Objects API — `/objcount`, `/objdump` |
| `Internal/Features/Scripting/samples/nops.lua` | блок RPC/packets + F10 UI |
| `Internal/Features/Scripting/samples/camhack.lua` | freecam |
| `Internal/Features/Scripting/samples/weapon_id.lua` | weapon helpers |

---

## Краткая шпаргалка

```lua
-- lifecycle
function onStart() end
function onFrame(dt) end
function onStop() end

-- chat / notify
SAMP.addMessage("hi", 0xFFFFFFFF)
Notify.success("T", "ok")

-- input
Input.wasKeyPressed(Key.F5)

-- player
local me = LocalPlayer.getPos()
local near = Player.closest(30)

-- objects
local o = Objects.get(id)
local list = Objects.inRadius(me.x, me.y, me.z, 50)

-- events
Events.onCreateObject(function(id) end)
Events.onIncomingRpc(function(rpcId, bytes) end)

-- files
File.write("out.txt", "data")

-- UI
Game.setScriptUi(true)
-- ImGui.* only in onDrawMenu / onDrawOverlay
```

---

*LethalityX Lua API*
