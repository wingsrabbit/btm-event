# 📋 PvPEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了**PvP死斗模式**中的武器空投和物资刷新逻辑——比如每隔一段时间在地图上刷新更强的武器。

**适合新手程度**：⭐⭐⭐ （中等难度，涉及循环和随机逻辑）

---

## 🎯 文件结构

```
PvPEvents.xml
├── <RandomEvents>                   ← 根元素
│   └── <EventPrefabs>               ← 事件模板容器
│       ├── deathmatchweapondrop1    ← 死斗模式武器空投
│       ├── alienruindmweapondrop1   ← 外星遗迹死斗武器空投
│       ├── alienruindmdivingsuits   ← 潜水服刷新
│       ├── trashitems               ← 垃圾桶物资刷新
│       └── cleanup_weapondrops      ← 清理空武器箱
```

---

## 🔍 关键逻辑拆解

### 核心机制：分层武器空投

这个文件最重要的设计是**武器分层递进系统**：

```
时间线 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━→
        ↓           ↓           ↓           ↓
      Tier 1     Tier 2     Tier 3     Tier 4
     (基础武器)  (进阶武器)  (高级武器)  (顶级武器)
     撬棍、扳手  手枪、霰弹  步枪、SMG   外星武器
```

### 代码片段1：基础补给刷新循环

```xml
<Label name="basicsupplies" />
<ClearTagAction tag="selectedsupplycab" />
<!-- 随机选择一个补给柜 -->
<TagAction criteria="itemidentifier:suppliescabinet" tag="selectedsupplycab" submarinetype="outpost" ChooseRandom="true" ChooseRandomExcludingTag="usedsupplycab" ContinueIfNoTargetsFound="true" />
<!-- 标记为已使用 -->
<TagAction criteria="eventtag:selectedsupplycab" tag="usedsupplycab" submarinetype="outpost" ContinueIfNoTargetsFound="true" />

<CountTargetsAction TargetTag="selectedsupplycab" minamount="1">
  <Success>
    <RNGAction chance="0.5">
      <Success>
        <SpawnAction itemidentifier="antibleeding1" amount="4" targetinventory="selectedsupplycab" SpawnIfInventoryFull="false" ContinueIfFailedToSpawn="true" />
      </Success>
      <Failure>
        <SpawnAction itemidentifier="revolver" TargetTag="spawnedrevolver" amount="1" targetinventory="selectedsupplycab" SpawnIfInventoryFull="false" ContinueIfFailedToSpawn="true" />
      </Failure>
    </RNGAction>
  </Success>
</CountTargetsAction>

<GoTo name="basicsupplies" maxtimes="10" />
```

#### 执行逻辑解析

1. **`<Label name="basicsupplies" />`**
   - 定义一个"书签"，后面可以跳回这里

2. **`<ClearTagAction tag="selectedsupplycab" />`**
   - 清除之前的标签，准备选择新的柜子

3. **`<TagAction ... ChooseRandom="true" ChooseRandomExcludingTag="usedsupplycab">`**
   - 随机选择一个补给柜
   - 但排除已经被选过的（通过 `usedsupplycab` 标签排除）

4. **`<CountTargetsAction TargetTag="selectedsupplycab" minamount="1">`**
   - 检查是否找到了至少1个柜子
   - 如果找到了（Success），就往里面放东西

5. **`<RNGAction chance="0.5">`**
   - 50%的概率走 Success 分支
   - 50%的概率走 Failure 分支
   - 用来实现"随机刷新不同物品"

6. **`<GoTo name="basicsupplies" maxtimes="10" />`**
   - 跳回 `basicsupplies` 标签的位置
   - 但最多只重复10次
   - 这样就实现了"循环填充10个补给柜"

---

### 代码片段2：分层武器空投

```xml
<!-- TIER 1 -->
<SpawnAction itemtag="weapondroptier1" SpawnPointTag="deathmatchweapondrop" ignorebyai="true" offset="200" />
<SpawnAction itemtag="weapondroptier1" SpawnPointTag="deathmatchweapondrop" ignorebyai="true" offset="200" />
<RNGAction chance="0.3">
  <Success>
    <SpawnAction itemtag="weapondroptier1_special" SpawnPointTag="deathmatchweapondrop" ignorebyai="true" offset="200" />
  </Success>
</RNGAction>

<WaitAction time="120" />

<!-- TIER 2 -->
<Label name="tier2" />
<SpawnAction itemtag="weapondroptier2" SpawnPointTag="deathmatchweapondrop" ignorebyai="true" offset="200" />
<WaitAction time="120" />
<GoTo name="tier2" maxtimes="2" />

<!-- TIER 3 ... -->
```

#### 执行逻辑解析

1. **Tier 1 开始**：
   - 刷新4个基础武器
   - 30%概率额外刷新1个特殊武器

2. **等待120秒**：
   - 玩家有2分钟时间抢这波武器

3. **Tier 2 开始**：
   - 刷新更好的武器
   - 这个阶段会循环3次（初始1次 + maxtimes="2"）

4. **逐级递进**：
   - 直到 Tier 4，武器越来越强

---

### 代码片段3：清理空武器箱

```xml
<ScriptedEvent identifier="cleanup_weapondrops">
  <Label name="repeat" />
  <WaitAction time="10" />
  <ClearTagAction tag="weaponcrate" />
  <TagAction criteria="itemtag:weapondrop" tag="weaponcrate" submarinetype="outpost" ContinueIfNoTargetsFound="true" />
  <!-- 检查箱子是否为空 -->
  <CheckConditionalAction targettag="weaponcrate" targetitemcomponent="ItemContainer" ContainedItemCount="0" ApplyTagToTarget="emptycrate">
    <Success>
      <!-- 删除空箱子 -->
      <RemoveItemAction targettag="emptycrate" />
    </Success>
  </CheckConditionalAction>
  <GoTo name="repeat" />
</ScriptedEvent>
```

这是一个**持续运行的清理脚本**：
- 每10秒检查一次
- 找出所有空的武器箱
- 把它们删除
- 永远循环（没有 `maxtimes` 限制）

---

## 💡 学习重点

### 1️⃣ 学会用 `Label` + `GoTo` 实现循环

```xml
<Label name="循环开始" />
<!-- 要重复的操作 -->
<GoTo name="循环开始" maxtimes="5" />  <!-- 最多重复5次 -->
```

**注意**：如果不设置 `maxtimes`，会无限循环！

### 2️⃣ 学会用 `RNGAction` 实现随机事件

```xml
<RNGAction chance="0.3">  <!-- 30%概率 -->
  <Success>
    <!-- 30%走这里 -->
  </Success>
  <Failure>
    <!-- 70%走这里 -->
  </Failure>
</RNGAction>
```

### 3️⃣ 学会用 `CountTargetsAction` 检查目标数量

```xml
<CountTargetsAction TargetTag="目标标签" minamount="1">
  <Success>
    <!-- 至少有1个目标时执行 -->
  </Success>
</CountTargetsAction>
```

### 4️⃣ 学会排除已选择的目标

```xml
<TagAction 
  criteria="itemidentifier:某物品" 
  tag="selected" 
  ChooseRandom="true" 
  ChooseRandomExcludingTag="alreadyused"  <!-- 排除已标记的 -->
/>
<TagAction criteria="eventtag:selected" tag="alreadyused" />  <!-- 标记为已使用 -->
```

---

## 📊 本文件中的重要参数

| 参数 | 说明 | 示例值 |
|------|------|--------|
| `ChooseRandom` | 随机选择一个目标 | `"true"` |
| `ChooseRandomExcludingTag` | 排除带某标签的目标 | `"usedsupplycab"` |
| `ContinueIfNoTargetsFound` | 找不到目标时是否继续 | `"true"` |
| `SpawnIfInventoryFull` | 背包满时是否生成 | `"false"` |
| `ContinueIfFailedToSpawn` | 生成失败时是否继续 | `"true"` |
| `ignorebyai` | AI是否忽略此物品 | `"true"` |
| `maxtimes` | GoTo最大重复次数 | `"10"` |

---

## 🎮 实战思考题

1. 为什么武器空投要用 `ignorebyai="true"`？
   <details>
   <summary>点击查看答案</summary>
   因为这是PvP模式，如果AI会去捡这些武器，就会干扰玩家的游戏体验。
   </details>

2. 如果删掉 `maxtimes="10"`，会发生什么？
   <details>
   <summary>点击查看答案</summary>
   循环会无限进行，直到找不到更多的补给柜为止（因为有 ContinueIfNoTargetsFound）。
   </details>

3. 为什么要先 `ClearTagAction` 再 `TagAction`？
   <details>
   <summary>点击查看答案</summary>
   因为每次循环都要选择新的目标，如果不清除旧标签，新选择的目标会和旧的混在一起。
   </details>

---

## 📌 设计启示

这个文件展示了一个完整的**PvP游戏循环设计**：

1. **初始补给** → 让所有玩家有基础装备
2. **分层递进** → 游戏越往后武器越强，增加紧张感
3. **随机性** → 不同位置刷新不同物品，增加策略性
4. **自动清理** → 保持地图整洁，避免物品堆积
