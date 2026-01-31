# 📋 ChainingEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了**跨关卡的连锁剧情事件**——比如在前哨站遇到一个NPC，然后需要在下一关或者信标站完成后续任务的故事。

**适合新手程度**：⭐⭐ （难度较高，涉及存档数据和多阶段剧情）

---

## 🎯 文件结构

```
ChainingEvents.xml
├── <Randomevents>               ← 根元素
│   └── <EventPrefabs>           ← 事件模板容器
│       ├── miketheidiot1/2      ← "傻瓜麦克"系列（多阶段NPC事件）
│       ├── shockjock/shockjock2 ← "震撼DJ"系列（广播+遭遇）
│       ├── assassinationofjacovsubra ← "刺杀雅科夫"任务线
│       └── ...更多连锁事件
```

---

## 🔍 关键逻辑拆解

### 核心概念：事件状态保存

连锁事件最重要的特点是：**它们需要"记住"玩家做过什么**。

这是通过 `SetDataAction` 和 `CheckDataAction` 实现的：

```
玩家第一次遇到Mike → SetDataAction: timesmikefound = 1
                            ↓
下一个前哨站 → CheckDataAction: timesmikefound == 1?
                            ↓ Yes
                    第二阶段事件触发
```

---

### 典型事件1："傻瓜麦克"第一阶段

```xml
<ScriptedEvent identifier="miketheidiot1" commonness="25">
  <TagAction criteria="player" tag="player" />
  <TagAction criteria="itemtag:chair" tag="chair" RequiredModuleTag="CrewModule" />
  <SpawnAction npcsetidentifier="customnpcs1" npcidentifier="miketheidiot" targettag="mike" TargetModuleTags="CrewModule" />
  <NPCOperateItemAction npctag="mike" targettag="chair" Operate="true" />
  <TriggerAction target1tag="player" target2tag="mike" applytotarget1="triggerer_player" waitforinteraction="true"/>
  <ConversationAction targettag="triggerer_player" text="EventText.miketheidiot1.c1" eventsprite="dorm">
    <Option text="EventText.miketheidiot1.o1">
      <ConversationAction targettag="triggerer_player" text="EventText.miketheidiot1.o1.c1">
        <Option text="EventText.miketheidiot1.o1.o1">
          <CheckMoneyAction Amount="5">
            <Success>
              <MoneyAction amount="-5" />
              <SetDataAction identifier="timesmikefound" value="1" />
              <!-- 后续对话... -->
            </Success>
            <Failure>
              <ConversationAction targettag="triggerer_player" text="eventtext.miketheidiot.nomoney"/>
            </Failure>
          </CheckMoneyAction>
        </Option>
        <Option text="EventText.miketheidiot1.o1.o2" endconversation="true" />
      </ConversationAction>
    </Option>
    <Option text="EventText.miketheidiot1.o2" endconversation="true" />
  </ConversationAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- `commonness="25"`：较低的触发概率
- 无前置条件，随机遇到

#### 执行过程 (Action)
1. 找到椅子，在船员模块生成Mike
2. 让Mike坐在椅子上（`NPCOperateItemAction`）
3. 等待玩家主动互动
4. 展开对话，玩家可以选择给Mike 5块钱
5. **关键步骤**：`SetDataAction identifier="timesmikefound" value="1"` → 记录"已遇到Mike一次"

#### 结束条件 (Finish)
- 对话结束后事件完成
- 但数据 `timesmikefound=1` 会保存到存档

---

### 典型事件2："傻瓜麦克"第二阶段

```xml
<ScriptedEvent identifier="miketheidiot2" commonness="25">
  <CheckDataAction identifier="timesmikefound" condition="eq 1">
    <Success>
      <!-- 如果之前遇到过Mike（值=1），执行第二阶段 -->
      <TagAction criteria="player" tag="player" />
      <!-- ... 第二次见面的对话 ... -->
      <SetDataAction identifier="timesmikefound" value="2" />
    </Success>
    <Failure>
      <!--Do nothing-->
    </Failure>
  </CheckDataAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- 首先检查 `timesmikefound` 是否等于 1
- 如果不等于1（从没遇到过，或者已经是2了），什么都不做

#### 执行逻辑
- 只有玩家之前给过Mike钱（触发了第一阶段并付款），才会进入第二阶段
- 第二阶段结束后，数据变成2，防止重复触发

---

### 典型事件3："震撼DJ"双阶段事件

```xml
<!-- 第一阶段：在前哨站听到广播 -->
<ScriptedEvent identifier="shockjock" commonness="50">
  <CheckDataAction identifier="shockjock_broadcast" condition="eq 0">
    <Success>
      <!-- 播放广播内容 -->
      <SetDataAction identifier="shockjock_broadcast" value="1"/>
    </Success>
    <Failure>
      <!-- 已经听过了，不重复 -->
    </Failure>
  </CheckDataAction>
</ScriptedEvent>

<!-- 第二阶段：在信标站遭遇DJ本人 -->
<ScriptedEvent identifier="shockjock2" commonness="50">
  <CheckDataAction identifier="shockjock_broadcast" condition="eq 1">
    <Success>
      <TagAction criteria="submarine:BeaconStation" tag="beaconstation" />
      <TriggerAction target1tag="player" target2tag="beaconstation" radius="4000" />
      <SetDataAction identifier="shockjock_broadcast" value="2"/>
      <SpawnAction npcsetidentifier="customnpcs1" npcidentifier="shockjock" targettag="okelly" spawnpointtag="saferoom" spawnlocation="beaconstation" requirespawnpointtag="true"/>
      <!-- 生成宠物怪物 -->
      <SpawnAction speciesname="orangeboy" spawnpointtag="saferoom" targettag="maurice" spawnlocation="beaconstation" requirespawnpointtag="true"/>
      <!-- DJ会攻击玩家 -->
      <CombatAction combatmode="Offensive" npctag="okelly" enemytag="triggerer_player" isinstigator="true"/>
    </Success>
  </CheckDataAction>
</ScriptedEvent>
```

这个事件展示了**跨地点的连锁**：
1. 前哨站听广播 → 状态变1
2. 信标站遇到真人 → 状态变2（防止重复）

---

## 💡 学习重点

### 1️⃣ 学会用 `SetDataAction` 保存进度

```xml
<!-- 设置一个数值 -->
<SetDataAction identifier="我的变量名" value="1" />

<!-- 设置布尔值 -->
<SetDataAction identifier="已完成任务" value="true" />
```

**重要**：这个数据会保存在存档里，下次加载还在！

### 2️⃣ 学会用 `CheckDataAction` 检查进度

```xml
<CheckDataAction identifier="我的变量名" condition="eq 1">
  <Success>
    <!-- 变量等于1时执行 -->
  </Success>
  <Failure>
    <!-- 变量不等于1时执行 -->
  </Failure>
</CheckDataAction>
```

常用条件运算符：
| 运算符 | 含义 | 示例 |
|--------|------|------|
| `eq` | 等于 | `condition="eq 1"` |
| `gt` | 大于 | `condition="gt 5"` |
| `lt` | 小于 | `condition="lt 3"` |

### 3️⃣ 学会用 `CheckMoneyAction` 检查金钱

```xml
<CheckMoneyAction Amount="100">
  <Success>
    <!-- 玩家有≥100块钱时执行 -->
    <MoneyAction amount="-100" />  <!-- 扣100块 -->
  </Success>
  <Failure>
    <!-- 钱不够时执行 -->
    <ConversationAction text="你的钱不够！" />
  </Failure>
</CheckMoneyAction>
```

### 4️⃣ 学会用 `NPCOperateItemAction` 让NPC使用物品

```xml
<!-- 让NPC坐下 -->
<NPCOperateItemAction npctag="npc标签" targettag="椅子标签" Operate="true" />

<!-- 让NPC站起来 -->
<NPCOperateItemAction npctag="npc标签" targettag="椅子标签" Operate="false" />
```

### 5️⃣ 学会用 `CombatAction` 让NPC战斗

```xml
<CombatAction 
  combatmode="Offensive"        <!-- 进攻模式 -->
  npctag="攻击者标签" 
  enemytag="目标标签" 
  isinstigator="true"           <!-- 是挑衅者（会被守卫攻击） -->
  guardreaction="none"          <!-- 守卫反应 -->
  witnessreaction="retreat"     <!-- 目击者反应 -->
/>
```

---

## 📊 状态机设计模式

这个文件大量使用了"状态机"设计：

```
状态0（未触发）
    ↓ 触发第一阶段
状态1（已完成第一阶段）
    ↓ 触发第二阶段
状态2（已完成所有阶段）
    ↓
不再触发任何事件
```

这种设计确保：
- 玩家不会重复遇到同一个事件
- 事件按正确顺序发生
- 跨关卡的剧情能够延续

---

## 🎮 实战练习

设计一个两阶段的"神秘商人"事件：

1. **第一阶段**：在前哨站遇到商人，他说"下次见面给你好东西"
2. **第二阶段**：再次在前哨站遇到商人，他送给玩家一把武器

<details>
<summary>点击查看参考答案</summary>

```xml
<!-- 第一阶段 -->
<ScriptedEvent identifier="mysterymerchant1" commonness="30">
  <CheckDataAction identifier="mysterymerchant_stage" condition="eq 0">
    <Success>
      <TagAction criteria="player" tag="player" />
      <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="commoner" targettag="merchant" spawnlocation="Outpost" />
      <TriggerAction target1tag="player" target2tag="merchant" waitforinteraction="true"/>
      <ConversationAction text="嘿，朋友！下次见面我有好东西给你！" speakertag="merchant">
        <Option text="好的，下次见">
          <SetDataAction identifier="mysterymerchant_stage" value="1" />
        </Option>
        <Option text="不感兴趣" endconversation="true" />
      </ConversationAction>
    </Success>
  </CheckDataAction>
</ScriptedEvent>

<!-- 第二阶段 -->
<ScriptedEvent identifier="mysterymerchant2" commonness="30">
  <CheckDataAction identifier="mysterymerchant_stage" condition="eq 1">
    <Success>
      <TagAction criteria="player" tag="player" />
      <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="commoner" targettag="merchant" spawnlocation="Outpost" />
      <TriggerAction target1tag="player" target2tag="merchant" waitforinteraction="true"/>
      <ConversationAction text="我们又见面了！这是我答应你的东西！" speakertag="merchant">
        <Option text="太感谢了！">
          <SpawnAction itemidentifier="smg" targetinventory="player" />
          <SetDataAction identifier="mysterymerchant_stage" value="2" />
        </Option>
      </ConversationAction>
    </Success>
  </CheckDataAction>
</ScriptedEvent>
```

</details>

---

## 📌 设计启示

连锁事件是制作**有深度剧情的mod**的关键技术：

1. 用状态变量控制剧情进展
2. 不同关卡/地点触发不同阶段
3. 玩家的选择会影响后续发展
4. 记得设置终止状态，避免无限重复
