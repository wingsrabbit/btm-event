# 📋 FactionEvents.xml 解析文档

## 📝 文件概览

**一句话总结**：这个文件定义了与**阵营/派系相关的特殊事件**——比如联盟的特殊雇佣任务、炸弹恐慌事件、以及需要高声望才能触发的剧情。

**适合新手程度**：⭐⭐⭐ （中等难度，涉及声望系统和复杂对话分支）

---

## 🎯 文件结构

```
FactionEvents.xml
├── <RandomEvents>                    ← 根元素
│   └── <EventPrefabs>                ← 事件模板容器
│       ├── coalitionspecialhire1/2   ← 联盟特殊雇佣（高声望奖励）
│       ├── separatistspecialhire1/2  ← 分离派特殊雇佣
│       ├── bombscare                 ← 炸弹恐慌事件
│       ├── stowaway                  ← 偷渡者事件
│       └── ...更多派系相关事件
```

---

## 🔍 关键逻辑拆解

### 典型事件1：联盟特殊雇佣

```xml
<ScriptedEvent identifier="coalitionspecialhire1">
  <CheckReputationAction targettype="faction" identifier="coalition" condition="gte 80">
    <Failure>
      <!-- 声望不够，什么都不做 -->
    </Failure>
    <Success>
      <CheckDataAction identifier="coalitionspecialhire1_hired" condition="eq true">
        <Success>
          <!-- 已经雇佣过了，不重复 -->
        </Success>
        <Failure>
          <!-- 可以雇佣！ -->
          <TagAction criteria="player" tag="player" />
          <WaitAction time="5" />
          <ConversationAction text="eventtext.coalitionspecialhire1.announcement" eventsprite="NoticeBoard" />
          <SpawnAction npcsetidentifier="customnpcs1" npcidentifier="ignatiusmay" targettag="ignatiusmay" allowduplicates="false" spawnlocation="Outpost" targetmoduletags="admin,adminmodule" />
          
          <Label name="beginning" />
          <CheckDataAction identifier="coalitionspecialhire1_declinedonce" condition="eq true">
            <Failure>
              <!-- 第一次对话 -->
              <TriggerAction target1tag="ignatiusmay" target2tag="player" applytotarget2="triggerer_player" waitforinteraction="false" radius="300" />
              <NPCFollowAction npctag="ignatiusmay" targettag="triggerer_player" follow="true" />
              <ConversationAction targettag="triggerer_player" text="eventtext.coalitionspecialhire1.c1" eventsprite="oldman">
                <Option text="eventtext.coalitionspecialhire1.o1">
                  <!-- 详细对话... -->
                  <Option text="eventtext.coalitionspecialhire1.o1.o1.o1">
                    <!-- 同意雇佣 -->
                    <NPCChangeTeamAction npctag="ignatiusmay" teamid="Team1" addtocrew="true" />
                    <SetDataAction identifier="coalitionspecialhire1_hired" value="true" />
                  </Option>
                  <Option text="eventtext.separatistspecialhire2.o1.o1.o2">
                    <!-- 拒绝，但可以下次再谈 -->
                    <SetDataAction identifier="coalitionspecialhire1_declinedonce" value="true" />
                    <GoTo name="beginning" />
                  </Option>
                </Option>
              </ConversationAction>
            </Failure>
            <Success>
              <!-- 之前拒绝过，显示简化对话 -->
              <TriggerAction target1tag="ignatiusmay" target2tag="player" applytotarget2="triggerer_player" waitforinteraction="true" radius="300" />
              <ConversationAction targettag="triggerer_player" text="eventtext.coalitionspecialhire1.repeat" eventsprite="oldman">
                <!-- 简化选项... -->
              </ConversationAction>
            </Success>
          </CheckDataAction>
        </Failure>
      </CheckDataAction>
    </Success>
  </CheckReputationAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
1. 玩家在**联盟**势力的声望必须 ≥ 80
2. 之前没有雇佣过这个NPC（`coalitionspecialhire1_hired ≠ true`）

#### 执行过程 (Action)
1. 显示公告板通知
2. 在管理模块生成特殊NPC
3. NPC会主动走向玩家
4. 展开对话，玩家可以选择：
   - **同意雇佣** → NPC加入船员，设置"已雇佣"标记
   - **拒绝** → 设置"已拒绝一次"标记，NPC留在原地等待玩家回心转意

#### 特殊机制
- **`GoTo name="beginning"`**：拒绝后跳回对话开始，让玩家可以再次选择
- **`allowduplicates="false"`**：防止生成多个同样的NPC

---

### 典型事件2：炸弹恐慌

```xml
<ScriptedEvent identifier="bombscare" commonness="50">
  <TagAction criteria="player" tag="player" />
  <TagAction criteria="itemidentifier:mediumsteelcabinet" tag="bombcabinet" SubmarineType="Outpost" chooserandom="true" />
  <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="securitynpc[faction]" targettag="bombsecurity" spawnlocation="Outpost" />
  
  <Label name="beginning" />
  <TriggerAction target1tag="player" target2tag="bombsecurity" applytotarget1="triggerer_player" waitforinteraction="true" />
  <NPCFollowAction npctag="bombsecurity" targettag="triggerer_player" follow="true" />
  <ConversationAction text="EventText.bombscare.c1" targettag="triggerer_player" eventsprite="BombScare1">
    <Option text="EventText.bombscare.o1">
      <!-- 接受任务 -->
      <SpawnAction itemidentifier="traitortimebomb" targettag="bombscarebomb" targetinventory="bombcabinet" ignorebyai="true" />
      <!-- 等待玩家找到炸弹 -->
      <TriggerAction target1tag="player" target2tag="bombcabinet" applytotarget1="noisehearer" radius="80" />
      <StatusEffectAction targettag="bombscarebomb">
        <StatusEffect target="This" condition="100" IsOn="true" setvalue="true" />
      </StatusEffectAction>
      <ConversationAction text="eventtext.bombscare.tickingnoise" targettag="noisehearer" dialogtype="Small"/>
      
      <!-- 玩家必须拿起炸弹 -->
      <TriggerAction target1tag="player" target2tag="bombscarebomb" applytotarget1="bombfinder" waitforinteraction="true" />
      <ConversationAction text="EventText.bombscare.o1.o1.o1.c1.c1" targettag="bombfinder" eventsprite="BombScare2">
        <Option text="eventtext.bombscare.o1.o1.o1.c1.o1">
          <!-- 尝试拆弹 -->
          <SkillCheckAction requiredskill="electrical" requiredlevel="80" targettag="bombfinder">
            <Success>
              <!-- 拆弹成功！ -->
              <StatusEffectAction targettag="bombscarebomb">
                <StatusEffect target="This" IsOn="false" setvalue="true" />
              </StatusEffectAction>
              <GiveSkillEXPAction skill="electrical" amount="10" targettag="bombfinder" />
              <MoneyAction amount="500" />
              <ReputationAction targettype="Faction" identifier="coalition" increase="5" />
            </Success>
            <Failure>
              <!-- 拆弹失败，炸弹倒计时加速！ -->
            </Failure>
          </SkillCheckAction>
        </Option>
      </ConversationAction>
    </Option>
  </ConversationAction>
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- `commonness="50"`：中等概率随机触发

#### 执行过程 (Action)
1. 找一个随机柜子作为藏炸弹的地方
2. 安保人员请求玩家帮忙找炸弹
3. 玩家靠近柜子时听到滴答声
4. 拿起炸弹后可以选择拆弹
5. **技能检定**：需要电气技能 ≥ 80

#### 结果分支
| 情况 | 结果 |
|------|------|
| 拆弹成功 | +10电气经验，+500块，+5联盟声望 |
| 拆弹失败 | 炸弹倒计时加速，可能爆炸 |
| 不管炸弹 | 炸弹最终爆炸，造成损坏 |

---

## 💡 学习重点

### 1️⃣ 学会用 `CheckReputationAction` 做声望门槛

```xml
<CheckReputationAction targettype="faction" identifier="coalition" condition="gte 80">
  <Success>
    <!-- 声望≥80时执行 -->
  </Success>
  <Failure>
    <!-- 声望<80时执行（通常是什么都不做） -->
  </Failure>
</CheckReputationAction>
```

### 2️⃣ 学会用 `NPCChangeTeamAction` 招募NPC

```xml
<NPCChangeTeamAction 
  npctag="npc标签" 
  teamid="Team1"          <!-- Team1=玩家队伍 -->
  addtocrew="true"        <!-- 加入船员列表 -->
/>
```

### 3️⃣ 学会用 `NPCFollowAction` 让NPC跟随

```xml
<!-- 开始跟随 -->
<NPCFollowAction npctag="npc标签" targettag="目标标签" follow="true" />

<!-- 停止跟随 -->
<NPCFollowAction npctag="npc标签" targettag="目标标签" follow="false" />
```

### 4️⃣ 学会用 `SkillCheckAction` 做技能检定

```xml
<SkillCheckAction requiredskill="electrical" requiredlevel="80" targettag="玩家标签">
  <Success>
    <!-- 技能够高，成功！ -->
    <GiveSkillEXPAction skill="electrical" amount="10" targettag="玩家标签" />
  </Success>
  <Failure>
    <!-- 技能不够，失败！ -->
  </Failure>
</SkillCheckAction>
```

常用技能名：
| 技能ID | 中文名 |
|--------|--------|
| `electrical` | 电气 |
| `mechanical` | 机械 |
| `medical` | 医疗 |
| `weapons` | 武器 |
| `helm` | 驾驶 |

### 5️⃣ 学会用 `ReputationAction` 改变声望

```xml
<ReputationAction 
  targettype="Faction"        <!-- 目标类型：派系 -->
  identifier="coalition"      <!-- 哪个派系 -->
  increase="5"                <!-- 增加多少（负数=减少） -->
/>
```

---

### 6️⃣ 动态NPC标识符 `[faction]`

```xml
<SpawnAction ... npcidentifier="securitynpc[faction]" ... />
```

`[faction]` 会自动替换为当前前哨站的派系：
- 联盟前哨站 → `securitynpccoalition`
- 分离派前哨站 → `securitynpcseparatists`

---

## 📊 本文件中的重要属性

| 属性 | 说明 | 常用值 |
|------|------|--------|
| `targettype` | 声望目标类型 | `"faction"`, `"location"` |
| `identifier` | 派系/地点ID | `"coalition"`, `"separatists"` |
| `condition` | 比较条件 | `"gte 80"`, `"eq true"` |
| `allowduplicates` | 是否允许生成重复NPC | `"false"` |
| `teamid` | 队伍ID | `"Team1"` (玩家), `"Team2"` (敌对) |

---

## 🎮 实战练习

设计一个需要声望≥50才能触发的商人事件：

**要求**：
- 只有分离派声望≥50才能触发
- 商人会卖给玩家一把特殊武器
- 需要花费1000块

<details>
<summary>点击查看参考答案</summary>

```xml
<ScriptedEvent identifier="separatist_arms_dealer" commonness="30">
  <CheckReputationAction targettype="faction" identifier="separatists" condition="gte 50">
    <Failure>
      <!-- 声望不够，静默结束 -->
    </Failure>
    <Success>
      <TagAction criteria="player" tag="player" />
      <SpawnAction npcsetidentifier="outpostnpcs1" npcidentifier="commoner" targettag="dealer" spawnlocation="Outpost" />
      <TriggerAction target1tag="player" target2tag="dealer" applytotarget1="buyer" waitforinteraction="true" />
      <ConversationAction text="嘿，朋友。我有些好东西，你有兴趣吗？" targettag="buyer" speakertag="dealer">
        <Option text="让我看看">
          <ConversationAction text="一把精良的SMG，只要1000块。" targettag="buyer">
            <Option text="我买了">
              <CheckMoneyAction Amount="1000">
                <Success>
                  <MoneyAction amount="-1000" />
                  <SpawnAction itemidentifier="smg" targetinventory="buyer" />
                  <ConversationAction text="成交！祝你用得愉快。" targettag="buyer" />
                </Success>
                <Failure>
                  <ConversationAction text="你的钱不够，下次再来吧。" targettag="buyer" />
                </Failure>
              </CheckMoneyAction>
            </Option>
            <Option text="太贵了，不买" endconversation="true" />
          </ConversationAction>
        </Option>
        <Option text="不感兴趣" endconversation="true" />
      </ConversationAction>
    </Success>
  </CheckReputationAction>
</ScriptedEvent>
```

</details>

---

## 📌 设计启示

派系事件是增加游戏**重复可玩性**的好方法：

1. **声望门槛**：让玩家有动力提高声望
2. **派系独占**：不同派系有不同的奖励
3. **选择后果**：帮一边可能得罪另一边
4. **永久变化**：某些决定会永久改变游戏状态（如雇佣NPC）
