# 📋 MainStoryEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了**游戏核心流程相关的事件**——比如解锁新区域、支付通行费、与守卫对话等推动游戏进展的关键事件。

**适合新手程度**：⭐⭐⭐ （中等难度，是理解游戏流程的重要文件）

---

## 🎯 文件结构

```
MainStoryEvents.xml
├── <Randomevents>                    ← 根元素
│   └── <EventPrefabs>                ← 事件模板容器
│       ├── forcebeaconmission        ← 强制信标任务（旧版）
│       ├── unlockpathgeneric         ← 通用路径解锁
│       ├── unlockpathcoldcaverns     ← 寒冷洞穴路径解锁（联盟）
│       ├── unlockpatheuropanridge    ← 欧罗巴山脊路径解锁
│       ├── unlockpathaphoticplateau  ← 深渊高原路径解锁
│       ├── unlockpathgreatsea        ← 大海路径解锁
│       └── unlockpath*separatists    ← 分离派路径解锁
```

---

## 🔍 核心机制：路径解锁系统

### 什么是路径解锁？

在《潜渊症》中，玩家不能随意前往所有区域。某些路径被"锁定"了，需要满足条件才能通过：

```
┌─────────────────┐
│   当前前哨站    │
└────────┬────────┘
         │
    🔒 需要解锁 🔒
         │
         ▼
┌─────────────────┐
│   下一个区域    │
└─────────────────┘
```

解锁方式：
1. **声望足够** → 守卫直接放行
2. **声望不够** → 可以花钱贿赂

---

## 🔍 关键逻辑拆解

### 典型事件：寒冷洞穴路径解锁（联盟）

```xml
<ScriptedEvent identifier="unlockpathcoldcaverns" commonness="1" unlockpathevent="true" unlockpathtooltip="lockedpathtooltipcoalitionreputation" unlockpathreputation="30" faction="coalition" biome="coldcaverns">
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="watchman" targettag="unlockpathnpc" spawnlocation="Outpost" spawnpointtag="unlockpath" spawnpointtype="human" />

  <Label name="retry" />
  <ConversationAction text="EventText.unlockpathgeneric.start" speakertag="unlockpathnpc" invokertag="player" dialogtype="Regular" eventsprite="captain" />

  <CheckReputationAction targettype="faction" identifier="coalition" condition="gte 30">
    <Success>
      <ConversationAction text="EventText.unlockpathcoldcaverns.c1" speakertag="unlockpathnpc" targettag="player" waitforinteraction="false" dialogtype="Regular" eventsprite="captain"/>
      <UnlockPathAction />
    </Success>
    <Failure>
      <ConversationAction text="EventText.unlockpathcoldcaverns.c2" speakertag="unlockpathnpc" targettag="player" waitforinteraction="false" dialogtype="Regular" eventsprite="captain">
        <Option text="EventText.unlockpathcoldcaverns.o1">
          <CheckMoneyAction amount="2000">
            <Success>
              <MoneyAction amount="-2000" />
              <UnlockPathAction />
              <ConversationAction speakertag="unlockpathnpc" waitforinteraction="false" targettag="player" text="EventText.unlockpathcoldcaverns.o1.c1" eventsprite="captain" />
            </Success>
            <Failure>
              <ConversationAction speakertag="unlockpathnpc" waitforinteraction="false" targettag="player" text="EventText.unlockpathcoldcaverns.o1.nomoney" eventsprite="captain" />
              <GoTo name="retry" />
            </Failure>
          </CheckMoneyAction>
        </Option>
        <Option text="EventText.unlockpath.refusebribe">
          <ConversationAction speakertag="unlockpathnpc" waitforinteraction="false" targettag="player" text="EventText.unlockpathcoldcaverns.o2.c1" eventsprite="captain" />
          <GoTo name="retry" />
        </Option>
      </ConversationAction>
    </Failure>
  </CheckReputationAction>
</ScriptedEvent>
```

#### 事件属性解析

| 属性 | 值 | 含义 |
|------|-----|------|
| `unlockpathevent` | `"true"` | 标记这是一个路径解锁事件 |
| `unlockpathtooltip` | `"lockedpathtooltipcoalitionreputation"` | 锁定时显示的提示文本ID |
| `unlockpathreputation` | `"30"` | 需要的声望值 |
| `faction` | `"coalition"` | 关联的派系 |
| `biome` | `"coldcaverns"` | 适用的生态区 |

#### 执行流程图

```
┌──────────────────────────────────┐
│    生成守卫NPC在解锁点           │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│    守卫开始对话                  │
└──────────────┬───────────────────┘
               │
     ┌─────────┴─────────┐
     │ 检查联盟声望≥30？ │
     └─────────┬─────────┘
         │           │
        Yes         No
         │           │
         ▼           ▼
┌─────────────┐ ┌────────────────────┐
│ 直接放行！   │ │ 提供两个选择：     │
│ UnlockPath  │ │ 1. 花2000块贿赂   │
└─────────────┘ │ 2. 拒绝贿赂       │
               └─────────┬──────────┘
                         │
            ┌────────────┴────────────┐
           选1                      选2
            │                         │
    ┌───────┴───────┐                 │
    │ 检查钱≥2000？ │                 │
    └───────┬───────┘                 │
       │         │                    │
      Yes       No                    │
       │         │                    │
       ▼         ▼                    ▼
┌──────────┐ ┌──────────┐ ┌──────────────┐
│ 扣钱     │ │ 钱不够   │ │ 返回对话开头 │
│ 解锁路径 │ │ 返回开头 │ │ (GoTo retry) │
└──────────┘ └──────────┘ └──────────────┘
```

---

### 难度递进：不同区域的解锁条件

| 区域 | 声望要求 | 贿赂金额 |
|------|----------|----------|
| 寒冷洞穴 (Cold Caverns) | ≥30 | 2,000 |
| 欧罗巴山脊 (Europan Ridge) | ≥40 | 4,000 |
| 深渊高原 (Aphotic Plateau) | ≥50 | 8,000 |
| 大海 (Great Sea) | ≥75 | 16,000 |

**设计思路**：
- 越往后的区域，要求越高
- 贿赂金额翻倍增长（2000 → 4000 → 8000 → 16000）
- 逼迫玩家要么提高声望，要么花大价钱

---

### 分离派版本的路径解锁

```xml
<ScriptedEvent identifier="unlockpathcoldcavernsseparatists" ... faction="separatists" biome="coldcaverns">
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="watchmanseparatist" ... />
  
  <CheckReputationAction targettype="faction" identifier="separatists" condition="gte 30">
    <!-- 分离派声望检测 -->
  </CheckReputationAction>
</ScriptedEvent>
```

**关键区别**：
- NPC是 `watchmanseparatist` 而不是 `watchman`
- 检测的是 `separatists` 派系的声望
- 同一个区域有两套解锁事件（联盟和分离派各一套）

---

## 💡 学习重点

### 1️⃣ 学会用 `UnlockPathAction` 解锁路径

```xml
<UnlockPathAction />
```

这是一个**无参数的动作**，执行后会解锁当前被锁定的路径。

### 2️⃣ 学会用 `Label` + `GoTo` 实现"再来一次"

```xml
<Label name="retry" />
<!-- ... 对话内容 ... -->
<GoTo name="retry" />  <!-- 跳回开头，让玩家可以再次选择 -->
```

这种模式常用于：
- 钱不够时让玩家去赚钱再回来
- 玩家犹豫时可以反复选择

### 3️⃣ 理解 `spawnpointtag` 的作用

```xml
<SpawnAction ... spawnpointtag="unlockpath" ... />
```

`spawnpointtag` 指定了NPC应该出生在哪个**预设点位**。
- 这些点位是在关卡设计时放置的
- 通常标记为 `unlockpath`、`saferoom` 等

### 4️⃣ 理解事件的特殊标记属性

```xml
<ScriptedEvent 
  identifier="..." 
  unlockpathevent="true"         <!-- 告诉游戏这是路径解锁事件 -->
  unlockpathtooltip="..."        <!-- 锁定时的提示文本 -->
  unlockpathreputation="30"      <!-- 地图上显示的声望要求 -->
/>
```

这些属性让游戏知道：
- 这是个特殊的系统事件
- 在地图上应该怎么显示锁定提示

---

## 📊 本文件中的重要属性

| 属性 | 说明 | 用途 |
|------|------|------|
| `unlockpathevent` | 标记为路径解锁事件 | 系统识别 |
| `unlockpathtooltip` | 锁定提示的本地化ID | 玩家提示 |
| `unlockpathreputation` | 显示的声望要求 | 地图显示 |
| `faction` | 关联派系 | 决定检测哪方声望 |
| `biome` | 适用生态区 | 决定在哪里生效 |
| `spawnpointtag` | NPC出生点标签 | 控制NPC位置 |

---

## 🎮 设计思考

### 为什么要设计路径解锁系统？

1. **引导玩家** → 不能一开始就去最难的区域
2. **提供目标** → 声望成为玩家努力的方向
3. **增加选择** → 贿赂还是刷声望？
4. **增加紧张感** → 钱不够时的焦虑

### 为什么联盟和分离派要分开做？

1. **派系对立** → 两个派系不会共用同一个守卫
2. **玩家选择** → 走哪条路决定你"站哪边"
3. **多样性** → 同一个地图可能有不同的"门"

---

## 🎮 实战练习

设计一个简单的"门票"系统：需要花100块钱才能进入某个区域。

<details>
<summary>点击查看参考答案</summary>

```xml
<ScriptedEvent identifier="ticket_gate" commonness="100">
  <TagAction criteria="player" tag="player" />
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="commoner" targettag="gatekeeper" spawnlocation="Outpost" />
  <TriggerAction target1tag="player" target2tag="gatekeeper" applytotarget1="buyer" waitforinteraction="true" />
  
  <Label name="askticket" />
  <ConversationAction text="进入这个区域需要购买门票，100块。" speakertag="gatekeeper" targettag="buyer">
    <Option text="好的，我买一张">
      <CheckMoneyAction amount="100">
        <Success>
          <MoneyAction amount="-100" />
          <ConversationAction text="请进！" speakertag="gatekeeper" targettag="buyer" />
          <!-- 这里可以加 UnlockPathAction 或其他解锁逻辑 -->
        </Success>
        <Failure>
          <ConversationAction text="你的钱不够，去赚点钱再来吧。" speakertag="gatekeeper" targettag="buyer" />
          <GoTo name="askticket" />
        </Failure>
      </CheckMoneyAction>
    </Option>
    <Option text="太贵了，我不进去了" endconversation="true" />
  </ConversationAction>
</ScriptedEvent>
```

</details>

---

## 📌 小贴士

这个文件是游戏**核心流程**的一部分。如果你想做一个完整的战役mod，理解路径解锁系统是必须的。

建议学习顺序：
1. 先理解基本的对话流程
2. 再理解声望检测机制
3. 最后理解路径解锁的系统属性
