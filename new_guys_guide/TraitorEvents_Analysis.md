# 📋 TraitorEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了**多人模式中的叛徒任务**——当你被选为叛徒时，会收到的各种破坏任务，比如搞砸信标站、偷窃货物、破坏采矿设备等。

**适合新手程度**：⭐⭐ （较复杂，涉及多人游戏机制和条件检测）

---

## 🎯 文件结构

```
TraitorEvents.xml
├── <RandomEvents>                    ← 根元素
│   ├── <EventSprites>                ← 叛徒事件专用图片
│   │   ├── MentalBreakdown
│   │   ├── Duffelbag
│   │   ├── Magazine
│   │   └── ...
│   │
│   └── <EventPrefabs>                ← 叛徒事件模板
│       ├── bakingstation             ← 搞炸信标站
│       ├── cargotheft                ← 货物盗窃
│       ├── miningsabotage            ← 采矿破坏
│       ├── naturevsnurture           ← 偷泥猛龙蛋
│       └── ...更多叛徒任务
```

---

## 🔍 核心概念：TraitorEvent

### 什么是叛徒事件？

在《潜渊症》多人模式中，系统可能会随机选择一名玩家成为**叛徒**。叛徒会收到一个秘密任务，需要在不被发现的情况下完成。

```
┌──────────────────────────────────┐
│    系统随机选择一名叛徒           │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│    叛徒收到秘密任务               │
│    例如：让信标站反应堆熔毁       │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│    任务完成？                     │
│    ├─ 是 → 叛徒获得奖励          │
│    └─ 否 → 任务失败              │
└──────────────────────────────────┘
```

---

## 🔍 关键标签：TraitorEvent

```xml
<TraitorEvent 
  identifier="bakingstation" 
  dangerlevel="2"                          <!-- 危险等级 -->
  RequiredPreviousDangerLevel="1"          <!-- 需要完成的前置危险等级 -->
  RequirePreviousDangerLevelCompleted="false"
  commonness="10"                          <!-- 触发概率 -->
  StealPercentageOfExperience="25"         <!-- 偷取其他玩家经验的百分比 -->
  WrongVotedAsTraitorMoneyPenalty="100"    <!-- 被误判时的金钱惩罚 -->
>
```

### TraitorEvent 专属属性

| 属性 | 说明 |
|------|------|
| `dangerlevel` | 任务危险等级（1-3） |
| `RequiredPreviousDangerLevel` | 需要先完成的危险等级 |
| `StealPercentageOfExperience` | 成功后偷取他人经验的比例 |
| `WrongVotedAsTraitorMoneyPenalty` | 被冤枉时对方扣的钱 |

---

## 🔍 关键逻辑拆解

### 典型事件1：搞炸信标站

```xml
<TraitorEvent identifier="bakingstation" dangerlevel="2" RequiredPreviousDangerLevel="1" commonness="10">
  <Icon texture="Content/UI/MissionIcons.png" sourcerect="0,0,256,256" color="200,0,0,255" />
  <MissionRequirement missiontag="beacon" />
  <LevelRequirement LevelType="LocationConnection" mindifficulty="0" mindifficultyincampaign="5">
    <ItemConditional HasTag="reactor" InBeaconStation="true" />
  </LevelRequirement>
  
  <!-- 找到信标站的反应堆 -->
  <TagAction criteria="itemtag:reactor" tag="beaconreactor" SubmarineType="BeaconStation" />
  <!-- 告诉叛徒任务内容 -->
  <ConversationAction targettag="traitor" text="EventText.bakingstation.c1" eventsprite="MentalBreakdown" />
  <!-- 添加任务日志 -->
  <EventLogAction targettag="traitor" id="initialinfo" text="EventText.bakingstation.objective1" />
  <!-- 显示任务目标 -->
  <EventObjectiveAction targettag="traitor" type="add" id="traitorobjective.bakingstation.meltdown" />
  
  <!-- 循环检测反应堆是否熔毁 -->
  <Label name="waitforexplosion" />
  <WaitAction time="5.0" />
  <CheckConditionalAction targettag="beaconreactor" targetitemcomponent="Reactor" MeltedDownThisRound="true">
    <Failure>
      <GoTo name="waitforexplosion" />
    </Failure>
  </CheckConditionalAction>
  
  <!-- 任务完成！ -->
  <SetTraitorEventStateAction state="Completed" />
  <EventLogAction targettag="traitor" id="success" text="EventText.bakingstation.objective2" />
  <EventObjectiveAction targettag="traitor" type="complete" id="traitorobjective.bakingstation.meltdown" />
  
  <!-- 回合结束时的处理 -->
  <OnRoundEndAction>
    <EventLogAction targettag="nontraitor" id="initialinfo" text="EventText.bakingstation.objective1" />
    <CheckConditionalAction targettag="beaconreactor" targetitemcomponent="Reactor" MeltedDownThisRound="true">
      <Success>
        <GiveExpAction targettag="traitor" amount="500" />
        <EventLogAction targettag="nontraitor" id="success" text="EventText.bakingstation.objective2" />
      </Success>
      <Failure>
        <EventLogAction id="failure" text="EventText.bakingstation.objective3" />
      </Failure>
    </CheckConditionalAction>
    <EventObjectiveAction targettag="traitor" type="remove" id="traitorobjective.bakingstation.meltdown" />
  </OnRoundEndAction>
</TraitorEvent>
```

#### 执行流程

1. **任务分配**：检查是否有信标任务，是否有反应堆
2. **告知叛徒**：显示任务说明和目标
3. **持续检测**：每5秒检查一次反应堆是否熔毁
4. **任务完成**：熔毁时标记成功
5. **回合结算**：给叛徒发经验，向所有人公布结果

---

### 典型事件2：货物盗窃

```xml
<TraitorEvent identifier="cargotheft" dangerlevel="2" commonness="5">
  <MissionRequirement missiontype="cargo" />
  <LevelRequirement LevelType="LocationConnection" mindifficultyincampaign="15" />
  
  <ConversationAction targettag="traitor" text="EventText.cargotheft.c1" eventsprite="Silhouette" />
  <EventLogAction targettag="traitor" id="initialinfo" text="EventText.cargotheft.objective1" />
  <EventObjectiveAction targettag="traitor" type="add" id="traitorobjective.cargotheft.missingcargo" />
  
  <!-- 标记所有货物任务物品 -->
  <TagAction criteria="itemtag:cargomission" tag="targetitem" SubmarineType="Player" />
  
  <!-- 检测货物是否被处理掉了 -->
  <Label name="waitforitemsdisposed" />
  <WaitAction time="1.0" />
  <CheckItemAction targettag="targetitem" RequiredConditionalMatchPercentage="50" CompareToInitialAmount="true">
    <Conditional InPlayerSubmarine="true" />
    <Success>
      <!-- 50%以上的货物还在玩家潜艇上，继续等待 -->
      <GoTo name="waitforitemsdisposed" />
    </Success>
    <Failure>
      <!-- 不到50%的货物在潜艇上了，任务完成！ -->
      <SetTraitorEventStateAction state="Completed" />
      <ConversationAction targettag="traitor" text="EventText.cargotheft.c2" eventsprite="Silhouette" />
      <EventObjectiveAction targettag="traitor" type="complete" id="traitorobjective.cargotheft.missingcargo" />
    </Failure>
  </CheckItemAction>
  
  <!-- 继续监控（防止货物被找回） -->
  <Label name="itemsstillgone" />
  <WaitAction time="1.0" />
  <CheckItemAction targettag="targetitem" RequiredConditionalMatchPercentage="50" CompareToInitialAmount="true">
    <Conditional InPlayerSubmarine="true" />
    <Success>
      <!-- 货物被找回了，任务失败 -->
      <EventObjectiveAction targettag="traitor" type="Fail" id="traitorobjective.cargotheft.missingcargo" />
      <SetTraitorEventStateAction state="Failed" />
      <GoTo name="waitforitemsdisposed" />
    </Success>
    <Failure>
      <SetTraitorEventStateAction state="Completed" />
      <GoTo name="itemsstillgone" />
    </Failure>
  </CheckItemAction>
  
  <OnRoundEndAction>
    <!-- 结算奖励或惩罚 -->
    <CheckTraitorEventStateAction state="Completed">
      <Success>
        <MoneyAction targettag="traitor" amount="800" />
        <GiveExpAction targettag="traitor" amount="800" />
      </Success>
    </CheckTraitorEventStateAction>
  </OnRoundEndAction>
</TraitorEvent>
```

#### 任务机制

- **目标**：让至少50%的货物"消失"（丢掉、销毁等）
- **持续检测**：每秒检查一次货物状态
- **可逆性**：如果货物被找回，任务会失败

---

## 💡 学习重点

### 1️⃣ 叛徒系统专用标签

```xml
<!-- "traitor" 标签：自动指向当前叛徒玩家 -->
<ConversationAction targettag="traitor" text="..." />

<!-- "nontraitor" 标签：自动指向非叛徒玩家 -->
<EventLogAction targettag="nontraitor" ... />
```

### 2️⃣ 任务状态管理

```xml
<!-- 设置任务状态 -->
<SetTraitorEventStateAction state="Completed" />  <!-- 完成 -->
<SetTraitorEventStateAction state="Failed" />     <!-- 失败 -->

<!-- 检查任务状态 -->
<CheckTraitorEventStateAction state="Completed">
  <Success><!-- 已完成 --></Success>
  <Failure><!-- 未完成 --></Failure>
</CheckTraitorEventStateAction>
```

### 3️⃣ 任务要求

```xml
<!-- 需要特定类型的任务存在 -->
<MissionRequirement missiontag="beacon" />
<MissionRequirement missiontype="cargo" />

<!-- 需要特定类型的关卡 -->
<LevelRequirement LevelType="LocationConnection" mindifficulty="0">
  <!-- 额外条件：需要有反应堆 -->
  <ItemConditional HasTag="reactor" InBeaconStation="true" />
</LevelRequirement>
```

### 4️⃣ 任务目标显示

```xml
<!-- 添加任务目标 -->
<EventObjectiveAction targettag="traitor" type="add" id="目标ID" />

<!-- 完成任务目标 -->
<EventObjectiveAction targettag="traitor" type="complete" id="目标ID" />

<!-- 失败任务目标 -->
<EventObjectiveAction targettag="traitor" type="Fail" id="目标ID" />

<!-- 移除任务目标 -->
<EventObjectiveAction targettag="traitor" type="remove" id="目标ID" />
```

### 5️⃣ 日志系统

```xml
<EventLogAction targettag="traitor" id="日志ID" text="日志内容" />
```

日志会记录在游戏的事件日志中，回合结束时可以查看。

### 6️⃣ 检查物品组件状态

```xml
<CheckConditionalAction 
  targettag="目标标签" 
  targetitemcomponent="Reactor"      <!-- 检查的组件类型 -->
  MeltedDownThisRound="true"         <!-- 检查的条件 -->
>
```

### 7️⃣ 回合结束处理

```xml
<OnRoundEndAction>
  <!-- 这里的内容在回合结束时执行 -->
  <!-- 常用于：发放奖励、公布结果 -->
</OnRoundEndAction>
```

---

## 📊 叛徒任务设计模式

### 模式1：破坏类任务

```
找到目标 → 循环检测 → 目标被破坏? → 任务完成
                         ↓
                       继续等待
```

### 模式2：偷窃类任务

```
标记目标物品 → 循环检测 → 物品还在? → 继续等待
                           ↓
                         任务完成
                           ↓
                     继续监控（防止找回）
```

### 模式3：收集类任务

```
告知目标 → 等待玩家找到 → 带回潜艇? → 任务完成
```

---

## 🎮 设计思考

### 为什么叛徒任务有难度等级？

1. **循序渐进**：新叛徒先做简单任务
2. **风险收益**：高难度任务高回报
3. **技能门槛**：复杂任务需要游戏经验

### 为什么要持续检测？

1. **实时反馈**：玩家知道任务进度
2. **可逆性**：支持任务状态改变
3. **准确判定**：避免错误判断

---

## 🎮 实战练习

设计一个叛徒任务：偷走船上所有的手枪

<details>
<summary>点击查看参考答案</summary>

```xml
<TraitorEvent identifier="guntheft" dangerlevel="1" commonness="20">
  <LevelRequirement LevelType="LocationConnection" mindifficulty="0" />
  
  <ConversationAction targettag="traitor" text="你的任务：把船上所有的手枪都处理掉（丢掉或藏起来）" eventsprite="Silhouette" />
  <EventLogAction targettag="traitor" id="initial" text="秘密任务：处理掉所有手枪" />
  <EventObjectiveAction targettag="traitor" type="add" id="traitorobjective.guntheft" />
  
  <!-- 标记所有手枪 -->
  <TagAction criteria="itemidentifier:revolver" tag="targetguns" SubmarineType="Player" />
  
  <Label name="checkguns" />
  <WaitAction time="3.0" />
  <!-- 检查是否还有手枪在玩家潜艇上 -->
  <CheckItemAction targettag="targetguns" Amount="1">
    <Conditional InPlayerSubmarine="true" />
    <Success>
      <!-- 还有手枪，继续等待 -->
      <GoTo name="checkguns" />
    </Success>
    <Failure>
      <!-- 没有手枪了，任务完成！ -->
      <SetTraitorEventStateAction state="Completed" />
      <ConversationAction targettag="traitor" text="所有手枪都被处理掉了，干得好！" />
      <EventObjectiveAction targettag="traitor" type="complete" id="traitorobjective.guntheft" />
    </Failure>
  </CheckItemAction>
  
  <OnRoundEndAction>
    <CheckTraitorEventStateAction state="Completed">
      <Success>
        <MoneyAction targettag="traitor" amount="500" />
        <GiveExpAction targettag="traitor" amount="300" />
      </Success>
    </CheckTraitorEventStateAction>
    <EventObjectiveAction targettag="traitor" type="remove" id="traitorobjective.guntheft" />
  </OnRoundEndAction>
</TraitorEvent>
```

</details>

---

## 📌 小贴士

叛徒事件是《潜渊症》多人游戏的核心玩法之一。设计叛徒任务时要注意：

1. **可行性**：任务必须是玩家能完成的
2. **隐蔽性**：不能太容易被发现
3. **平衡性**：不能太破坏游戏体验
4. **趣味性**：要有足够的挑战和奖励
