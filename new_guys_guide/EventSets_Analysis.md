# 📋 EventSets.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了**事件的组织结构和触发规则**——决定在什么难度、什么地点、触发哪些类型的任务组合。

**适合新手程度**：⭐⭐ （偏系统设计，需要理解事件调度逻辑）

---

## 🎯 文件结构

```
EventSets.xml
├── <Randomevents>                        ← 根元素
│   └── <EventSet>                        ← 主事件集（按关卡类型分）
│       ├── <EventSet> (嵌套)             ← 子事件集（按难度/派系分）
│       │   ├── <ScriptedEvent>           ← 具体事件引用
│       │   └── <EventSet> (更深嵌套)     ← 更细分的事件集
│       └── ...
```

---

## 🔍 核心概念解释

### 什么是 EventSet？

**EventSet** 就像一个"事件抽屉"：
- 里面装着一组相关的事件
- 游戏会根据规则从抽屉里抽取事件来触发
- 可以嵌套（抽屉里还有小抽屉）

```
📦 outpostevents（主抽屉：前哨站事件）
├── 📁 missionevents.coldcaverns（寒冷洞穴任务）
│   ├── 📁 .start（新手阶段）
│   ├── 📁 .basic（基础阶段）
│   └── 📁 .advanced（进阶阶段）
├── 📁 missionevents.europanridge（欧罗巴山脊任务）
└── 📁 missionevents.greatsea（大海任务）
```

---

## 🔍 关键逻辑拆解

### 代码片段1：主事件集定义

```xml
<EventSet 
  identifier="outpostevents" 
  leveltype="outpost" 
  locationtype="outpost,city,research,military,mine" 
  allowatstart="true" 
  minleveldifficulty="0" 
  maxleveldifficulty="80" 
  chooserandom="false" 
  ignorecooldown="true"
>
```

#### 属性解析

| 属性 | 值 | 含义 |
|------|-----|------|
| `identifier` | `"outpostevents"` | 事件集的唯一ID |
| `leveltype` | `"outpost"` | 适用于前哨站类型关卡 |
| `locationtype` | `"outpost,city,..."` | 具体可用的地点类型 |
| `allowatstart` | `"true"` | 关卡开始时就可以触发 |
| `minleveldifficulty` | `"0"` | 最低难度要求 |
| `maxleveldifficulty` | `"80"` | 最高难度限制 |
| `chooserandom` | `"false"` | 不随机选择（按顺序/全部） |
| `ignorecooldown` | `"true"` | 忽略冷却时间 |

---

### 代码片段2：难度分层的任务集

```xml
<EventSet identifier="missionevents.coldcaverns.start" 
  minleveldifficulty="0" 
  maxleveldifficulty="1" 
  allowatstart="true" 
  eventcount="3" 
  chooserandom="false" 
  exhaustible="true"
>
  <ScriptedEvent identifier="missionevent_cargoany" />
  <ScriptedEvent identifier="missionevent_killmonster_set1" />
  <ScriptedEvent identifier="missionevent_collectminerals_mainpath" />
</EventSet>
```

#### 属性解析

| 属性 | 值 | 含义 |
|------|-----|------|
| `minleveldifficulty` | `"0"` | 难度0-1时触发（最简单） |
| `maxleveldifficulty` | `"1"` | |
| `eventcount` | `"3"` | 触发3个事件 |
| `exhaustible` | `"true"` | 用完后不再重复 |

**翻译成大白话**：
> "在难度0-1的寒冷洞穴关卡，给玩家发3个任务：运货、杀怪、挖矿。这些任务只出现一次。"

---

### 代码片段3：随机选择的任务集

```xml
<EventSet identifier="missionevents.coldcaverns.basic" 
  minleveldifficulty="1" 
  maxleveldifficulty="5" 
  allowatstart="true" 
  setcount="3" 
  chooserandom="false" 
  exhaustible="true"
>
  <EventSet identifier="missionevents.coldcaverns.basic.cargo" chooserandom="true" allowatstart="true">
    <ScriptedEvent identifier="missionevent_cargoany" commonness="50" />
    <ScriptedEvent identifier="missionevent_cargoweaponscoalition" commonness="25" faction="coalition" />
    <ScriptedEvent identifier="missionevent_cargoexplosiveseparatists" commonness="25" faction="separatists" />
  </EventSet>
  <EventSet identifier="missionevents.coldcaverns.basic.monster" chooserandom="true" allowatstart="true">
    <ScriptedEvent identifier="missionevent_killswarm_set1" commonness="25" />
    <ScriptedEvent identifier="missionevent_killmonster_set1" commonness="75" />
  </EventSet>
  <EventSet identifier="missionevents.coldcaverns.basic.mining" chooserandom="true" allowatstart="true">
    <ScriptedEvent identifier="missionevent_collectminerals_mainpath" commonness="50" />
    <ScriptedEvent identifier="missionevent_collectminerals_set1" commonness="50" />
  </EventSet>
</EventSet>
```

#### 结构解析

```
missionevents.coldcaverns.basic（父集：选3个子集）
├── .cargo（随机选1个运货任务）
│   ├── 普通运货 (50%)
│   ├── 联盟武器 (25%) 
│   └── 分离派爆炸物 (25%)
├── .monster（随机选1个猎杀任务）
│   ├── 杀虫群 (25%)
│   └── 杀怪物 (75%)
└── .mining（随机选1个挖矿任务）
    ├── 主路矿物 (50%)
    └── 矿物集1 (50%)
```

**翻译成大白话**：
> "在难度1-5时，给玩家发3种任务（运货、猎杀、挖矿各一个）。每种任务从几个选项中随机抽一个。"

---

### 代码片段4：派系任务

```xml
<EventSet identifier="missionevents.coldcaverns.advanced.faction" chooserandom="true" eventcount="1" allowatstart="true">
  <ScriptedEvent identifier="missionevent_escort1coalition" commonness="100" faction="coalition" />
  <ScriptedEvent identifier="missionevent_escort1separatists" commonness="100" faction="separatists" />
</EventSet>
```

**重点**：`faction="coalition"` 或 `faction="separatists"` 限制了任务只在对应派系控制的前哨站出现。

---

## 💡 学习重点

### 1️⃣ 理解 `chooserandom` 的作用

| 值 | 行为 |
|----|------|
| `true` | 随机选择一个（按commonness权重） |
| `false` | 不随机，执行全部或按顺序 |

### 2️⃣ 理解 `eventcount` 和 `setcount`

| 属性 | 作用 |
|------|------|
| `eventcount` | 从子事件中选几个 |
| `setcount` | 从子事件集中选几个 |

### 3️⃣ 理解 `exhaustible`

```xml
exhaustible="true"   <!-- 用完后不再出现 -->
exhaustible="false"  <!-- 可以重复出现 -->
```

### 4️⃣ 理解难度分层

```
难度 0-1   → .start 事件集（教学关）
难度 1-5   → .basic 事件集（基础关）
难度 5-15  → .advanced 事件集（进阶关）
难度 15+   → 更难的事件集...
```

---

## 📊 事件集属性速查表

| 属性 | 类型 | 说明 |
|------|------|------|
| `identifier` | 文本 | 事件集唯一ID |
| `leveltype` | 文本 | 关卡类型（outpost/locationconnection等） |
| `locationtype` | 文本 | 地点类型（可用逗号分隔多个） |
| `minleveldifficulty` | 数字 | 最低难度 |
| `maxleveldifficulty` | 数字 | 最高难度 |
| `allowatstart` | 布尔 | 关卡开始时是否可触发 |
| `chooserandom` | 布尔 | 是否随机选择 |
| `eventcount` | 数字 | 选择的事件数量 |
| `setcount` | 数字 | 选择的子集数量 |
| `exhaustible` | 布尔 | 是否一次性 |
| `ignorecooldown` | 布尔 | 是否忽略冷却 |
| `faction` | 文本 | 限定派系 |

---

## 🎮 设计思考

### 为什么要用 EventSet 嵌套？

1. **模块化**：把相似的事件放在一起管理
2. **概率控制**：确保每种类型的任务都能出现
3. **难度递进**：不同阶段给不同难度的任务
4. **可维护性**：修改一组任务时不影响其他组

### 实际例子

假设没有EventSet，直接把所有任务放一起：
```
随机选3个 → 可能选到3个运货任务，没有猎杀任务
```

使用EventSet嵌套后：
```
运货任务集选1个 + 猎杀任务集选1个 + 挖矿任务集选1个
→ 保证任务类型多样性
```

---

## 🎮 实战练习

设计一个简单的事件集，在难度10-20的关卡中，随机触发2个任务（从4个任务中选）：

<details>
<summary>点击查看参考答案</summary>

```xml
<EventSet 
  identifier="my_custom_missions" 
  leveltype="outpost" 
  minleveldifficulty="10" 
  maxleveldifficulty="20" 
  allowatstart="true" 
  eventcount="2" 
  chooserandom="true"
>
  <ScriptedEvent identifier="missionevent_cargoany" commonness="50" />
  <ScriptedEvent identifier="missionevent_killmonster_set2" commonness="30" />
  <ScriptedEvent identifier="missionevent_salvageartifact" commonness="15" />
  <ScriptedEvent identifier="missionevent_escort1coalition" commonness="5" />
</EventSet>
```

**说明**：
- `eventcount="2"`：选2个任务
- `chooserandom="true"`：随机选择
- `commonness` 不同：运货最容易出现，护送最少出现

</details>

---

## 📌 小贴士

这个文件主要是**系统配置**，而不是具体的事件逻辑。

如果你只是想写一个简单的事件，可以暂时跳过这个文件。当你需要**控制事件的触发条件和频率**时，再回来学习。
