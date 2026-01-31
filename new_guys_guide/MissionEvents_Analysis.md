# 📋 MissionEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了玩家在前哨站可以接到的各种**随机任务事件**——比如有人请你运货、猎杀怪物、挖矿等等。

**适合新手程度**：⭐⭐⭐⭐⭐ （非常适合入门！结构清晰简单）

---

## 🎯 文件结构

```
MissionEvents.xml
├── <Randomevents>          ← 根元素
│   └── <EventPrefabs>      ← 事件模板容器
│       ├── 随机NPC任务事件（commoner给的任务）
│       └── 管理者任务事件（outpostmanager给的任务）
```

---

## 🔍 关键逻辑拆解

### 典型事件1：简单的猎杀怪物任务

```xml
<ScriptedEvent identifier="missionevent_killmonster" commonness="100">
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="commoner" targettag="missionnpc" spawnlocation="Outpost" spawnpointtype="Path" />
  <ConversationAction text="EventText.missionevent_killmonster.c1" speakertag="missionnpc" endeventifinterrupted="false" dialogtype="Small"/>
  <MissionAction missiontag="killmonster_set1" />
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- **无特殊触发条件**，游戏会根据 `commonness="100"` 的概率随机触发
- 这个数值是"基准值"，和其他事件的commonness相比较决定出现概率

#### 执行过程 (Action)
1. **第一步**：在前哨站的路径上生成一个普通平民NPC，给他贴上 `missionnpc` 标签
2. **第二步**：这个NPC说一段话（对话内容由本地化ID `EventText.missionevent_killmonster.c1` 决定）
3. **第三步**：触发一个猎杀怪物的任务（`killmonster_set1`）

#### 结束条件 (Finish)
- 事件本身在 `MissionAction` 执行后就结束了
- 任务是否完成取决于玩家是否击杀了怪物

---

### 典型事件2：带选项的任务（小丑事件）

```xml
<ScriptedEvent identifier="missionevent_clownoutbreak" commonness="40" requiredDestinationTypes="anyoutpost">
  <TagAction criteria="player" tag="player" />
  <TagAction criteria="itemidentifier:opdeco_hrflyers" tag="potentialflyers" submarinetype="outpost" />
  <TriggerAction target1tag="potentialflyers" target2tag="player" applytotarget2="triggerer_player" radius="150" waitforinteraction="true"/>
  <ConversationAction targettag="triggerer_player" text="EventText.missionevent_clownoutbreak.c1">
    <Option text="EventText.missionevent_clownoutbreak.o1">
      <ConversationAction targettag="triggerer_player" text="EventText.missionevent_clownoutbreak.o1.c1" />
      <MissionAction missiontag="cargoclown" />
    </Option>
    <Option text="EventText.missionevent_clownoutbreak.o2">
      <ConversationAction targettag="triggerer_player" text="EventText.missionevent_clownoutbreak.o2.c1" />
      <MissionAction missiontag="cargoclown" />
    </Option>
    <Option text="EventText.missionevent_clownoutbreak.o3" />
  </ConversationAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- `commonness="40"`：比基准值低，触发概率相对较小
- `requiredDestinationTypes="anyoutpost"`：必须是前哨站关卡
- 玩家需要靠近某个传单物品（`opdeco_hrflyers`）并主动互动

#### 执行过程 (Action)
1. 找到所有玩家，贴上 `player` 标签
2. 找到前哨站里的传单物品，贴上 `potentialflyers` 标签
3. 等待玩家靠近传单（距离150以内）并按E互动
4. 弹出对话，玩家可以选择：
   - 选项1/2：都会接受小丑运货任务
   - 选项3：什么都不做（拒绝任务）

#### 结束条件 (Finish)
- 玩家选择任一选项后对话结束，事件完成

---

### 典型事件3：带声望检测的任务

```xml
<ScriptedEvent identifier="missionevent_escort4separatists" commonness="60" requiredDestinationTypes="anyoutpost">
  <CheckReputationAction targettype="faction" identifier="separatists" condition="gte 30">
    <Success>
      <ConversationAction text="EventText.missionevent_escort4separatists.c1" speakertag="outpostmanager" endeventifinterrupted="false" dialogtype="Small" />
      <MissionAction missiontag="escortterroristsseparatists" />
    </Success>
    <Failure>
      <!-- do nothing -->
    </Failure>
  </CheckReputationAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- 必须在前哨站
- 事件开始后会检查玩家在**分离主义者**势力的声望是否 ≥ 30

#### 执行过程 (Action)
- **如果声望够**（≥30）：前哨站管理员会给你一个护送任务
- **如果声望不够**：什么都不发生，事件静默结束

#### 结束条件 (Finish)
- 根据声望检测结果，要么触发任务，要么直接结束

---

## 💡 学习重点

### 1️⃣ 学会用 `MissionAction` 触发任务

这是最简单的"给玩家发任务"方式：

```xml
<MissionAction missiontag="cargo" />        <!-- 通过标签触发 -->
<MissionAction missionidentifier="beacon" /> <!-- 通过具体ID触发 -->
```

### 2️⃣ 学会用 `TriggerAction` 等待玩家

```xml
<TriggerAction 
  target1tag="某物品"      
  target2tag="player" 
  radius="150"              <!-- 触发距离 -->
  waitforinteraction="true" <!-- true=需要按E，false=靠近即触发 -->
/>
```

### 3️⃣ 学会用 `CheckReputationAction` 做声望判断

```xml
<CheckReputationAction targettype="faction" identifier="coalition" condition="gte 50">
  <Success>
    <!-- 声望≥50时执行 -->
  </Success>
  <Failure>
    <!-- 声望<50时执行 -->
  </Failure>
</CheckReputationAction>
```

---

## 📊 本文件中出现的常用属性

| 属性 | 出现次数 | 说明 |
|------|---------|------|
| `commonness` | 几乎每个事件 | 触发权重 |
| `speakertag` | 频繁 | 对话说话人 |
| `missiontag` | 频繁 | 任务标签 |
| `requiredDestinationTypes` | 多次 | 限制关卡类型 |
| `dialogtype="Small"` | 多次 | 小型对话框 |

---

## 🎮 实战练习

试着自己写一个简单的任务事件：

```xml
<ScriptedEvent identifier="mymission_test" commonness="50">
  <!-- 你的代码 -->
</ScriptedEvent>
```

**目标**：让前哨站管理员给玩家布置一个运货任务

<details>
<summary>点击查看参考答案</summary>

```xml
<ScriptedEvent identifier="mymission_test" commonness="50" requiredDestinationTypes="anyoutpost">
  <ConversationAction text="我需要你帮我运一批货物到下一站。" speakertag="outpostmanager" dialogtype="Small"/>
  <MissionAction missiontag="cargo" />
</ScriptedEvent>
```

</details>
