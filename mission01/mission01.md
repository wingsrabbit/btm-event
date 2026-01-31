# 📋 mission01_event.xml 逻辑解析文档

> **写给编程小白的教学文档**：本文档详细解释 `mission01_event.xml` 中每一行代码的含义，并告诉你"为什么要这么写"。

---

## 📝 事件概述

| 项目 | 内容 |
|------|------|
| **事件名称** | UMP45派系炸弹恐慌任务 |
| **事件ID** | `ump45_bombpanic_mission` |
| **触发地点** | 前哨站 (Outpost) 或 军事基地 (Military) |
| **声望要求** | UMP45 派系声望 ≥ 100 |
| **核心功能** | 生成NPC → 玩家交互 → 100%概率派发"炸弹恐慌"任务 |

---

## 🔍 第一章：代码逐行精讲

### 1. 文件头声明

```xml
<?xml version="1.0" encoding="utf-8"?>
```

**作用**：告诉计算机这是一个XML文件，使用UTF-8编码（支持中文）。

**为什么需要**：这是XML文件的标准规范，每个XML文件都必须以此开头，否则解析器可能无法正确读取文件。

---

### 2. 根元素

```xml
<RandomEvents>
  <EventPrefabs>
    ...
  </EventPrefabs>
</RandomEvents>
```

**作用**：
- `<RandomEvents>`：最外层的"容器"，所有事件内容都放在里面
- `<EventPrefabs>`：存放事件模板的区域

**为什么用这个结构**：这是Barotrauma事件系统的标准格式。游戏在加载mod时会寻找这个结构来识别事件定义。

---

### 3. 事件定义头

```xml
<ScriptedEvent 
  identifier="ump45_bombpanic_mission" 
  commonness="100" 
  requiredDestinationTypes="outpost,military"
>
```

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `identifier` | `"ump45_bombpanic_mission"` | 事件的唯一身份证号，全局不能重复 |
| `commonness` | `"100"` | 触发权重，100是基准值。数字越大越容易触发 |
| `requiredDestinationTypes` | `"outpost,military"` | **限制触发地点**：只能在前哨站或军事基地触发 |

**为什么用 `<ScriptedEvent>` 而不是别的？**
- `<ScriptedEvent>` 是用于**有脚本逻辑的剧情事件**
- `<TraitorEvent>` 是专门给叛徒模式用的
- 我们要做的是一个有NPC、有对话、有任务派发的完整事件，所以用 `<ScriptedEvent>`

**原版出处**：参考 `events/FactionEvents.xml` 中所有派系事件都使用 `<ScriptedEvent>` 标签。

---

### 4. 地点检测逻辑

```xml
requiredDestinationTypes="outpost,military"
```

**作用**：这是事件定义中的一个属性，用于限制事件只能在特定类型的地点触发。

**可选值**：
| 值 | 含义 |
|----|------|
| `outpost` | 前哨站 |
| `military` | 军事基地 |
| `city` | 城市 |
| `research` | 研究站 |
| `mine` | 矿场 |
| `anyoutpost` | 任何前哨站类型 |

**原版出处**：
> 我是参考了 `new_guys_guide/MissionEvents_Analysis.md` 和原版 `events/MissionEvents.xml` 中的写法：
> ```xml
> <ScriptedEvent identifier="missionevent_clownoutbreak" commonness="40" requiredDestinationTypes="anyoutpost">
> ```
> 以及 `new_guys_guide/EventSets_Analysis.md` 中的 `locationtype` 属性说明。

---

### 5. 声望检测逻辑

```xml
<CheckReputationAction targettype="faction" identifier="UMP45" condition="gte 100">
  <Success>
    <!-- 声望≥100时执行的内容 -->
  </Success>
  <Failure>
    <!-- 声望不足时执行的内容（通常留空） -->
  </Failure>
</CheckReputationAction>
```

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `targettype` | `"faction"` | 检测目标类型是派系 |
| `identifier` | `"UMP45"` | 要检测的派系ID |
| `condition` | `"gte 100"` | 条件：≥ 100（gte = greater than or equal） |

**条件运算符**：
| 运算符 | 含义 |
|--------|------|
| `eq` | 等于 (equal) |
| `gt` | 大于 (greater than) |
| `gte` | 大于等于 (greater than or equal) |
| `lt` | 小于 (less than) |
| `lte` | 小于等于 (less than or equal) |

**原版出处**：
> 我是参考了 `new_guys_guide/FactionEvents_Analysis.md` 中的声望检测示例：
> ```xml
> <CheckReputationAction targettype="faction" identifier="coalition" condition="gte 80">
> ```
> 原版用这个逻辑来判断玩家是否达到联盟派系的信任门槛（≥80），才能解锁特殊雇佣事件。
> 我把派系改成 `UMP45`，门槛改成 `100`，逻辑完全一致。

---

### 6. 标记玩家

```xml
<TagAction criteria="player" tag="player" />
```

**作用**：找到所有玩家，给他们贴上"player"标签。

**为什么需要**：后续的很多操作（如TriggerAction、ConversationAction）需要通过标签来找到目标。如果不先贴标签，系统就不知道"player"指的是谁。

**原版出处**：
> 这是几乎所有事件的标准开头。参考 `new_guys_guide/OutpostEvents_Analysis.md` 和 `new_guys_guide/Barotrauma_Event_Guide.md` 中的万能模板。

---

### 7. 生成NPC

```xml
<SpawnAction 
  npcsetidentifier="outpostnpcs1" 
  npcidentifier="securitynpc[faction]" 
  targettag="ump45_contact" 
  spawnlocation="Outpost" 
  targetmoduletags="admin,adminmodule"
  allowduplicates="false"
/>
```

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `npcsetidentifier` | `"outpostnpcs1"` | 使用前哨站NPC模板集 |
| `npcidentifier` | `"securitynpc[faction]"` | 生成安保人员，`[faction]`会自动替换为当前派系外观 |
| `targettag` | `"ump45_contact"` | 给生成的NPC贴上这个标签 |
| `spawnlocation` | `"Outpost"` | 在前哨站内生成 |
| `targetmoduletags` | `"admin,adminmodule"` | 优先在管理模块区域生成 |
| `allowduplicates` | `"false"` | 不允许生成重复的NPC |

**关于 `[faction]` 动态替换**：
- 如果玩家在联盟控制的前哨站 → 生成 `securitynpccoalition`
- 如果玩家在分离派控制的前哨站 → 生成 `securitynpcseparatists`

**原版出处**：
> 我是参考了 `new_guys_guide/FactionEvents_Analysis.md` 中关于NPC生成的说明，以及原版 `events/FactionEvents.xml` 中的 `coalitionspecialhire1` 事件：
> ```xml
> <SpawnAction npcsetidentifier="customnpcs1" npcidentifier="ignatiusmay" targettag="ignatiusmay" allowduplicates="false" spawnlocation="Outpost" targetmoduletags="admin,adminmodule" />
> ```

---

### 8. 等待玩家交互

```xml
<TriggerAction 
  target1tag="ump45_contact" 
  target2tag="player" 
  applytotarget2="triggerer_player" 
  radius="150" 
  waitforinteraction="true"
/>
```

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `target1tag` | `"ump45_contact"` | 第一个目标（NPC） |
| `target2tag` | `"player"` | 第二个目标（玩家） |
| `applytotarget2` | `"triggerer_player"` | 触发交互的玩家会被额外标记为这个标签 |
| `radius` | `"150"` | 触发距离（像素） |
| `waitforinteraction` | `"true"` | 需要玩家主动按E互动（不是靠近就自动触发） |

**原版出处**：
> 参考 `new_guys_guide/OutpostEvents_Analysis.md` 中的 "问路事件"：
> ```xml
> <TriggerAction target1tag="givingdirections_captain" target2tag="player" applytotarget2="triggerer_player" radius="100" waitforinteraction="true"/>
> ```

---

### 9. NPC跟随

```xml
<NPCFollowAction npctag="ump45_contact" targettag="triggerer_player" follow="true" />
```

**作用**：让NPC跟随触发对话的玩家，增加沉浸感。

**原版出处**：
> 参考 `new_guys_guide/FactionEvents_Analysis.md` 中的 NPCFollowAction 说明：
> ```xml
> <!-- 开始跟随 -->
> <NPCFollowAction npctag="npc标签" targettag="目标标签" follow="true" />
> ```

---

### 10. 对话与选项

```xml
<ConversationAction 
  targettag="triggerer_player" 
  text="对话内容..."
  speakertag="ump45_contact"
  dialogtype="Regular"
>
  <Option text="选项1文字">
    <!-- 选择后执行的内容 -->
  </Option>
  <Option text="选项2文字" endconversation="true">
    <!-- 选择后结束对话 -->
  </Option>
</ConversationAction>
```

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `targettag` | `"triggerer_player"` | 对话目标（看到对话框的人） |
| `text` | 对话内容 | NPC说的话 |
| `speakertag` | `"ump45_contact"` | 说话人 |
| `dialogtype` | `"Regular"` | 对话框类型（Regular=标准大小，Small=小型） |

**选项嵌套**：
- 每个 `<Option>` 是一个对话选项
- 选项可以嵌套更多的 `<ConversationAction>`，形成多层对话
- `endconversation="true"` 表示选择后直接结束对话

**原版出处**：
> 参考 `new_guys_guide/Barotrauma_Event_Guide.md` 中的万能模板，以及 `events/FactionEvents.xml` 中的多层对话结构。

---

### 11. 派发任务

```xml
<MissionAction missiontag="bombscare" />
```

**作用**：触发一个任务。

**参数说明**：
| 属性 | 用法 |
|------|------|
| `missiontag` | 通过任务标签触发（可触发多个符合标签的任务） |
| `missionidentifier` | 通过具体任务ID触发（精确触发某一个任务） |

**原版出处**：
> 参考 `new_guys_guide/MissionEvents_Analysis.md` 中的说明：
> ```xml
> <MissionAction missiontag="cargo" />        <!-- 通过标签触发 -->
> <MissionAction missionidentifier="beacon" /> <!-- 通过具体ID触发 -->
> ```
> 以及原版 `events/FactionEvents.xml` 中的炸弹恐慌相关任务派发。

---

### 12. 增加声望

```xml
<ReputationAction targettype="Faction" identifier="UMP45" increase="5" />
```

**作用**：增加玩家与指定派系的声望。

**参数解析**：

| 属性 | 值 | 含义 |
|------|-----|------|
| `targettype` | `"Faction"` | 目标类型是派系 |
| `identifier` | `"UMP45"` | 派系ID |
| `increase` | `"5"` | 增加的数值（负数=减少） |

**原版出处**：
> 参考 `new_guys_guide/FactionEvents_Analysis.md` 中的示例：
> ```xml
> <ReputationAction targettype="Faction" identifier="coalition" increase="5" />
> ```

---

## 🔗 第二章：逻辑映射与溯源

### 问题1：如何检测地点（Outpost/Military）？

**我的写法**：
```xml
<ScriptedEvent ... requiredDestinationTypes="outpost,military">
```

**原版出处**：
- **文件**：`events/MissionEvents.xml` 和 `new_guys_guide/MissionEvents_Analysis.md`
- **原版示例**：
  ```xml
  <ScriptedEvent identifier="missionevent_clownoutbreak" commonness="40" requiredDestinationTypes="anyoutpost">
  ```
- **理由**：`requiredDestinationTypes` 是 Barotrauma 原版用来限制事件触发地点的标准属性。我把 `anyoutpost` 改成了具体的 `outpost,military` 以满足需求。

---

### 问题2：如何检测声望？

**我的写法**：
```xml
<CheckReputationAction targettype="faction" identifier="UMP45" condition="gte 100">
```

**原版出处**：
- **文件**：`events/FactionEvents.xml` 和 `new_guys_guide/FactionEvents_Analysis.md`
- **原版示例**：
  ```xml
  <CheckReputationAction targettype="faction" identifier="coalition" condition="gte 80">
  ```
- **理由**：这是原版用来检测玩家与派系声望关系的标准做法。原版用于判断玩家是否达到联盟的信任等级（≥80）才能触发特殊雇佣事件。我只是把派系ID改成了 `UMP45`，把门槛值改成了 `100`。

---

### 问题3：如何生成NPC？

**我的写法**：
```xml
<SpawnAction 
  npcsetidentifier="outpostnpcs1" 
  npcidentifier="securitynpc[faction]" 
  targettag="ump45_contact" 
  spawnlocation="Outpost" 
  targetmoduletags="admin,adminmodule"
  allowduplicates="false"
/>
```

**原版出处**：
- **文件**：`events/FactionEvents.xml` 和 `new_guys_guide/FactionEvents_Analysis.md`
- **原版示例**（联盟特殊雇佣事件）：
  ```xml
  <SpawnAction npcsetidentifier="customnpcs1" npcidentifier="ignatiusmay" targettag="ignatiusmay" allowduplicates="false" spawnlocation="Outpost" targetmoduletags="admin,adminmodule" />
  ```
- **理由**：我使用了相同的结构来生成NPC。唯一的改动是把 `npcidentifier` 改成了通用的安保人员模板 `securitynpc[faction]`（而不是特定角色），这样NPC会自动匹配当前前哨站的派系外观。

---

### 问题4：如何给予任务？

**我的写法**：
```xml
<MissionAction missiontag="bombscare" />
```

**原版出处**：
- **文件**：`events/MissionEvents.xml` 和 `new_guys_guide/MissionEvents_Analysis.md`
- **原版示例**：
  ```xml
  <MissionAction missiontag="killmonster_set1" />
  ```
- **理由**：`MissionAction` 是触发任务的标准方式。`missiontag="bombscare"` 会触发所有带有 `bombscare` 标签的任务。如果你想触发自定义任务，只需要把标签改成你自己定义的任务标签。

---

## 🛠️ 第三章：新手修改建议

### 修改1：把"声望100"改成"声望50"

找到这一行：
```xml
<CheckReputationAction targettype="faction" identifier="UMP45" condition="gte 100">
```

改成：
```xml
<CheckReputationAction targettype="faction" identifier="UMP45" condition="gte 50">
```

就这么简单！把 `100` 改成 `50` 即可。

---

### 修改2：把"炸弹恐慌"改成"运送货物"

找到这一行（出现了两处）：
```xml
<MissionAction missiontag="bombscare" />
```

改成：
```xml
<MissionAction missiontag="cargo" />
```

`cargo` 是原版运货任务的标签。如果你有自定义任务，把它改成你自己的任务标签或ID。

---

### 修改3：更换NPC类型

如果你想把安保人员换成普通平民，找到：
```xml
npcidentifier="securitynpc[faction]"
```

改成：
```xml
npcidentifier="commoner"
```

常用NPC类型：
| ID | 类型 |
|----|------|
| `commoner` | 普通平民 |
| `securitynpc[faction]` | 安保人员（自动匹配派系） |
| `captain` | 船长 |
| `outpostmanager` | 前哨站管理员 |

---

### 修改4：更改对话内容

直接找到 `text="..."` 的部分，把引号里的中文内容改成你想要的对话。

**注意**：对话内容要用**英文双引号** `"` 包裹，不要用中文引号 `""`。

---

### 修改5：调整触发概率

找到：
```xml
commonness="100"
```

- 想让事件更常出现？把数字调大，比如 `200`
- 想让事件更稀有？把数字调小，比如 `25`

---

## 📌 常见问题解答

### Q：为什么 `<Failure>` 里面是空的？

A：这是标准做法。当声望检测不通过时，我们希望事件"静默结束"，不执行任何操作。这样玩家不会感知到有一个事件被跳过了，游戏体验更自然。

### Q：`applytotarget2="triggerer_player"` 是什么意思？

A：在多人游戏中，可能有多个玩家。这个属性会把"实际触发交互的那个玩家"额外标记为 `triggerer_player`，确保后续的对话只显示给这个玩家，而不是所有人。

### Q：能不能同时检测多个派系的声望？

A：可以，但需要嵌套多个 `<CheckReputationAction>`。不过这会让逻辑变复杂，建议新手先掌握单派系检测。

---

## 📚 参考文档

| 文档 | 主要内容 |
|------|----------|
| `new_guys_guide/Barotrauma_Event_Guide.md` | 事件系统全局指南、万能模板 |
| `new_guys_guide/FactionEvents_Analysis.md` | 声望检测、NPC雇佣、派系事件 |
| `new_guys_guide/MissionEvents_Analysis.md` | 任务触发、地点限制 |
| `new_guys_guide/OutpostEvents_Analysis.md` | NPC交互、对话分支、技能检定 |
| `new_guys_guide/EventSets_Analysis.md` | 事件集定义、地点类型 |

---

*本文档基于对原版事件文件的学习与实践编写，希望能帮助你快速上手Barotrauma事件开发！*
