# 📋 OutpostEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件是**最丰富的前哨站事件库**——包含了各种NPC互动、随机遭遇、支线任务等，是学习事件编写的宝库。

**适合新手程度**：⭐⭐⭐ （事件种类多，适合进阶学习）

---

## 🎯 文件结构

```
OutpostEvents.xml
├── <Randomevents>                    ← 根元素
│   ├── <EventSprites>                ← 事件图片定义区
│   │   └── <Sprite identifier="..." .../> × 很多
│   │
│   └── <EventPrefabs>                ← 事件模板区
│       ├── testevent                 ← 测试事件
│       ├── givingdirections          ← 问路事件
│       ├── soundinthevent            ← 通风管声音事件
│       ├── trashfind                 ← 垃圾桶寻宝
│       ├── gambleryoulose/youwin     ← 赌博事件
│       ├── aNPCaHusk                 ← 感染者事件
│       └── ...更多事件
```

---

## 🔍 特色内容：EventSprites 事件图片

这个文件包含了大量的**事件图片定义**，用于在对话框左上角显示插图：

```xml
<EventSprites>
  <Sprite identifier="gambler" texture="Content/Map/Outposts/Art/Event_pic2.png" sourcerect="559,6,165,240" origin="0.5,0.5"/>
  <Sprite identifier="cultist" texture="Content/Map/Outposts/Art/Event_pic1.png" sourcerect="37,507,182,249" origin="0.5,0.5"/>
  <Sprite identifier="captain" texture="Content/Map/Outposts/Art/Event_pic1.png" sourcerect="34,0,182,251" origin="0.5,0.5"/>
  <!-- ...更多图片... -->
</EventSprites>
```

### 图片属性解析

| 属性 | 说明 | 示例 |
|------|------|------|
| `identifier` | 图片的唯一ID，在ConversationAction中引用 | `"captain"` |
| `texture` | 图片文件路径 | `"Content/Map/.../Event_pic1.png"` |
| `sourcerect` | 在大图中的位置 (x,y,宽,高) | `"34,0,182,251"` |
| `origin` | 图片中心点 (0-1) | `"0.5,0.5"` |

使用方式：
```xml
<ConversationAction text="..." eventsprite="captain" />
```

---

## 🔍 关键逻辑拆解

### 典型事件1：问路事件（Giving Directions）

```xml
<ScriptedEvent identifier="givingdirections" commonness="100">
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="captain" targettag="givingdirections_captain" spawnlocation="Outpost" spawnpointtype="Path" targetmoduletags="crewmodule" />
  <TagAction criteria="player" tag="player" />
  <TriggerAction target1tag="givingdirections_captain" target2tag="player" applytotarget2="triggerer_player" radius="100" waitforinteraction="true"/>
  <NPCWaitAction npctag="givingdirections_captain" wait="true" />
  <ConversationAction targettag="triggerer_player" text="EventText.givingdirections.c1" eventsprite="captain">
    <Option text="EventText.givingdirections.o1">
      <!-- 帮忙指路 -->
      <ConversationAction targettag="triggerer_player" text="EventText.givingdirections.o1.c1">
        <Option text="EventText.givingdirections.o1.o1">
          <SkillCheckAction requiredskill="Helm" requiredlevel="50" targettag="triggerer_player">
            <Success>
              <GiveSkillEXPAction skill="helm" amount="5" targettag="triggerer_player" />
              <ConversationAction targettag="triggerer_player" text="EventText.givingdirections.o1.o1.c1" />
            </Success>
            <Failure>
              <ConversationAction targettag="triggerer_player" text="EventText.givingdirections.o1.o1.c2" />
            </Failure>
          </SkillCheckAction>
        </Option>
      </ConversationAction>
    </Option>
    <Option text="EventText.givingdirections.o2">
      <!-- 无礼回应 -->
      <RNGAction chance="0.15">
        <Success>
          <!-- 15%概率对方给你一把枪 -->
          <SpawnAction itemidentifier="revolver" targettag="prizerevolver" targetinventory="triggerer_player" />
          <SpawnAction itemidentifier="revolverround" targetinventory="prizerevolver" />
        </Success>
        <Failure>
          <!-- 85%概率对方攻击你 -->
          <CombatAction combatmode="Offensive" npctag="givingdirections_captain" enemytag="triggerer_player" isinstigator="false"/>
        </Failure>
      </RNGAction>
    </Option>
    <Option text="EventText.givingdirections.o3" />
  </ConversationAction>
  <NPCWaitAction npctag="givingdirections_captain" wait="false" />
</ScriptedEvent>
```

#### 事件亮点

1. **技能检定奖励**：帮忙指路+驾驶技能≥50 = 获得5点驾驶经验
2. **随机结果**：无礼回应有15%概率获得武器，85%概率被攻击
3. **风险与收益**：游戏设计的典型模式

---

### 典型事件2：通风管里的声音

```xml
<ScriptedEvent identifier="soundinthevent" commonness="100">
  <TagAction criteria="player" tag="player" />
  <TagAction criteria="itemtag:mudraptorspawnvent" tag="mudraptorspawnvent" SubmarineType="Outpost" chooserandom="true" />
  <TriggerAction target1tag="mudraptorspawnvent" target2tag="player" applytotarget1="looseventspawnpoint" applytotarget2="triggerer_player" radius="150"/>
  <SpawnAction itemidentifier="loosevent" spawnlocation="Outpost" SpawnPointTag="looseventspawnpoint" TargetTag="selectedmudraptorspawnvent" offset="0"/>
  <StatusEffectAction targettag="selectedmudraptorspawnvent">
    <StatusEffect target="This" spritecolor="0,0,0,0" spritedepth="0.89" setvalue="true">
      <Sound file="Content/Sounds/Damage/Creak4.ogg" range="400" volume="3" frequencymultiplier="2" />
    </StatusEffect>
  </StatusEffectAction>
  <ConversationAction targettag="triggerer_player" text="EventText.soundinthevent.c1" eventsprite="vent"/>
  
  <Label name="UseitemOnVent" />
  <TriggerAction target1tag="selectedmudraptorspawnvent" target2tag="player" radius="80" waitforinteraction="true"/>
  <ConversationAction targettag="triggerer_player" text="eventtext.soundinthevent.whatdoyoudo" eventsprite="vent">
    <!-- 选项1：用螺丝刀 -->
    <Option text="EventText.soundinthevent.o1">
      <CheckItemAction targettag="triggerer_player" itemtags="screwdriveritem">
        <Success>
          <SkillCheckAction targettag="triggerer_player" requiredskill="Mechanical" requiredlevel="60">
            <Success>
              <!-- 成功修理，获得奖励 -->
              <GiveSkillEXPAction skill="mechanical" amount="5" targettag="triggerer_player" />
              <MoneyAction amount="1000" />
            </Success>
            <Failure>
              <!-- 失败，受伤 -->
              <AfflictionAction affliction="lacerations" strength="15" limbtype="LeftHand" targettag="triggerer_player" />
            </Failure>
          </SkillCheckAction>
        </Success>
        <Failure>
          <ConversationAction targettag="triggerer_player" text="eventtext.soundinthevent.getscrewdriver" />
          <Goto name="UseitemOnVent" />
        </Failure>
      </CheckItemAction>
    </Option>
    <!-- 选项2：用手电筒照 -->
    <Option text="EventText.soundinthevent.o2">
      <CheckItemAction targettag="triggerer_player" itemidentifiers="flashlight" RequireEquipped="true">
        <Success>
          <SkillCheckAction targettag="triggerer_player" requiredskill="weapons" requiredlevel="70">
            <Success>
              <!-- 发现怪物幼崽 -->
              <SpawnAction speciesname="Mudraptor_hatchling" spawnlocation="Outpost" spawnpointtag="looseventspawnpoint" />
            </Success>
            <!-- ... -->
          </SkillCheckAction>
        </Success>
      </CheckItemAction>
    </Option>
  </ConversationAction>
</ScriptedEvent>
```

#### 事件亮点

1. **物品检测**：`CheckItemAction` 检查玩家是否有特定物品
2. **装备检测**：`RequireEquipped="true"` 确保物品在手上
3. **多条路径**：不同工具导致不同结果
4. **状态效果**：`StatusEffectAction` 可以播放声音、改变外观
5. **伤害系统**：`AfflictionAction` 可以给玩家造成特定伤害

---

## 💡 学习重点

### 1️⃣ 学会用 `CheckItemAction` 检查物品

```xml
<!-- 检查是否有某类物品 -->
<CheckItemAction targettag="player" itemtags="screwdriveritem">
  <Success><!-- 有这个物品 --></Success>
  <Failure><!-- 没有这个物品 --></Failure>
</CheckItemAction>

<!-- 检查是否装备了特定物品 -->
<CheckItemAction targettag="player" itemidentifiers="flashlight" RequireEquipped="true">
  <!-- ... -->
</CheckItemAction>
```

### 2️⃣ 学会用 `AfflictionAction` 造成伤害/状态

```xml
<AfflictionAction 
  affliction="lacerations"     <!-- 伤害类型：割伤 -->
  strength="15"                <!-- 伤害强度 -->
  limbtype="LeftHand"          <!-- 受伤部位 -->
  targettag="player" 
/>
```

常用伤害类型：
| affliction | 中文 |
|------------|------|
| `lacerations` | 割伤 |
| `bleeding` | 出血 |
| `burn` | 烧伤 |
| `stun` | 眩晕 |
| `oxygenlow` | 缺氧 |

### 3️⃣ 学会用 `StatusEffectAction` 创建特效

```xml
<StatusEffectAction targettag="目标标签">
  <StatusEffect target="This">
    <!-- 播放声音 -->
    <Sound file="音频文件路径" range="400" volume="3" />
    <!-- 创建爆炸 -->
    <Explosion range="200.0" stun="0.2" force="5.0" />
    <!-- 删除物品 -->
    <Remove />
  </StatusEffect>
</StatusEffectAction>
```

### 4️⃣ 学会用 `RNGAction` 创造随机性

```xml
<RNGAction chance="0.15">  <!-- 15%概率 -->
  <Success>
    <!-- 15%走这里 -->
  </Success>
  <Failure>
    <!-- 85%走这里 -->
  </Failure>
</RNGAction>
```

### 5️⃣ 学会定义事件图片

```xml
<!-- 在EventSprites区域定义 -->
<Sprite identifier="mysprite" texture="路径/图片.png" sourcerect="x,y,宽,高" origin="0.5,0.5"/>

<!-- 在对话中使用 -->
<ConversationAction text="..." eventsprite="mysprite" />
```

---

## 📊 本文件中常见的事件模式

### 模式1：技能检定 + 奖励

```
玩家选择 → 检查技能 → 成功?
            ↓         ↓
          给经验     负面结果
```

### 模式2：物品检测 + 循环

```
玩家选择 → 有物品? → 执行
            ↓
          没有 → 提示 → 返回选择
```

### 模式3：随机结果

```
玩家选择 → RNG判定 → 好结果(x%)
                    → 坏结果(100-x%)
```

---

## 🎮 实战练习

设计一个"修理机器"事件：
- 需要玩家有扳手
- 技能检定：机械≥40
- 成功：+1000块，+5机械经验
- 失败：受轻伤

<details>
<summary>点击查看参考答案</summary>

```xml
<ScriptedEvent identifier="repair_machine" commonness="75">
  <TagAction criteria="player" tag="player" />
  <TagAction criteria="itemidentifier:junctionbox" tag="brokenmachine" SubmarineType="Outpost" chooserandom="true" />
  <TriggerAction target1tag="player" target2tag="brokenmachine" applytotarget1="repairer" radius="100" waitforinteraction="true" />
  
  <ConversationAction text="这台机器好像坏了，需要修理。" targettag="repairer" eventsprite="mechanic">
    <Option text="尝试修理">
      <CheckItemAction targettag="repairer" itemtags="wrenchitem">
        <Success>
          <SkillCheckAction requiredskill="mechanical" requiredlevel="40" targettag="repairer">
            <Success>
              <GiveSkillEXPAction skill="mechanical" amount="5" targettag="repairer" />
              <MoneyAction amount="1000" />
              <ConversationAction text="修好了！你获得了1000块报酬。" targettag="repairer" />
            </Success>
            <Failure>
              <AfflictionAction affliction="lacerations" strength="10" limbtype="RightHand" targettag="repairer" />
              <ConversationAction text="出了点意外，你的手被划伤了..." targettag="repairer" />
            </Failure>
          </SkillCheckAction>
        </Success>
        <Failure>
          <ConversationAction text="你需要一把扳手才能修理这个。" targettag="repairer" />
        </Failure>
      </CheckItemAction>
    </Option>
    <Option text="不管它" endconversation="true" />
  </ConversationAction>
</ScriptedEvent>
```

</details>

---

## 📌 小贴士

这个文件是**前哨站事件的大全**，包含了几乎所有常用的事件编写技巧：

1. **对话分支** - 多层嵌套的Option
2. **技能检定** - SkillCheckAction
3. **物品检测** - CheckItemAction
4. **随机事件** - RNGAction
5. **NPC控制** - NPCWaitAction, CombatAction
6. **奖惩系统** - MoneyAction, AfflictionAction, GiveSkillEXPAction

建议把这个文件当作**参考手册**，遇到不会写的功能就来这里找例子！
