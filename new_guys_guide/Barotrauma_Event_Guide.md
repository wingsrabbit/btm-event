# 🐙 《潜渊症》事件系统新手完全指南

> **写给完全不懂代码的你！** 本教程会手把手教你如何制作游戏内的剧情事件，就像给游戏"编剧本"一样。

---

## 📚 目录

1. [什么是事件系统？](#什么是事件系统)
2. [全局结构树状图](#全局结构树状图)
3. [万能代码模板](#万能代码模板)
4. [核心参数字典 (Cheat Sheet)](#核心参数字典-cheat-sheet)
5. [新手避坑指南](#新手避坑指南)

---

## 什么是事件系统？

想象一下，你是一个电影导演。**事件系统**就是你的"剧本"工具：

- 🎬 你可以决定**什么时候**发生什么事（比如：玩家靠近某个NPC时）
- 🎭 你可以安排**角色出场**（比如：在前哨站生成一个商人）
- 💬 你可以编写**对话和选项**（比如：玩家可以选择帮助或者拒绝）
- 🎁 你可以给玩家**奖励或惩罚**（比如：给钱、给经验、扣声望）

简单来说：**Event = 游戏里的"如果...就..."脚本**

---

## 全局结构树状图

每一个事件XML文件的结构，就像一棵大树：

```
📁 XML 文件
│
├── 🏷️ <?xml version="1.0" encoding="utf-8"?>  ← 文件头声明（每个XML文件必须有）
│
└── 📦 <RandomEvents> 或 <Randomevents>        ← 最外层的"盒子"（根元素）
    │
    ├── 🖼️ <EventSprites>                      ← [可选] 定义对话框里显示的图片
    │   └── <Sprite identifier="..." ... />    ← 单张图片的设置
    │
    └── 📋 <EventPrefabs>                      ← 存放所有事件"模板"的地方
        │
        ├── 🎭 <ScriptedEvent>                 ← 一个完整的"剧情事件"
        │   │
        │   ├── 属性: identifier="事件唯一ID"    ← 每个事件的"身份证号码"
        │   ├── 属性: commonness="100"          ← 触发概率（数字越大越容易发生）
        │   │
        │   └── 📜 各种 Action 标签             ← 事件的具体"动作"脚本
        │       │
        │       ├── <TagAction .../>           ← 给目标"贴标签"（方便后续操作找到它）
        │       ├── <SpawnAction .../>         ← 生成东西（NPC、物品、怪物）
        │       ├── <ConversationAction>       ← 触发一段对话
        │       │   └── <Option>               ← 对话中的选项
        │       ├── <TriggerAction .../>       ← 设置触发条件
        │       ├── <WaitAction .../>          ← 等待一段时间
        │       ├── <CheckDataAction>          ← 检查游戏数据（分支判断）
        │       │   ├── <Success>              ← 如果检查通过，执行这里面的
        │       │   └── <Failure>              ← 如果检查失败，执行这里面的
        │       └── ...更多动作...
        │
        └── 🎭 <TraitorEvent>                  ← 叛徒专用事件（多人模式）
            └── ...（结构类似 ScriptedEvent）
```

### 🔑 关键概念解释

| 术语 | 大白话解释 |
|------|-----------|
| **根元素** | 就像一个最大的收纳盒，所有内容都要放在里面 |
| **identifier** | 事件的"身份证号"，不能重复，游戏靠它来识别不同的事件 |
| **Action** | "动作指令"，告诉游戏要做什么（生成NPC？播放对话？给奖励？） |
| **Tag** | "标签/便签"，给游戏里的东西贴个名字，方便后续找到并操作它 |
| **Success/Failure** | "如果成功/如果失败"，像是岔路口，根据条件走不同的路 |

---

## 万能代码模板

以下是一个最基础的事件模板，你可以直接复制并修改：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 这一行告诉电脑：这是一个XML文件，使用UTF-8编码（支持中文） -->

<RandomEvents>
<!-- 最外层的"盒子"，所有事件内容都要放在这对标签里面 -->

  <EventPrefabs>
  <!-- 存放事件模板的区域 -->

    <ScriptedEvent identifier="my_first_event" commonness="100">
    <!-- 
      一个事件的开始！
      identifier: 给这个事件起一个独一无二的名字（英文，不能有空格）
      commonness: 触发概率，100是基准值，200就是2倍概率，50就是一半概率
    -->

      <!-- 第一步：找到玩家，给他贴个"player"标签 -->
      <TagAction criteria="player" tag="player" />
      <!-- 
        criteria="player" → 寻找条件：是玩家
        tag="player"      → 给找到的目标贴上"player"标签
        以后用"player"这个标签就能找到这个玩家了
      -->

      <!-- 第二步：在前哨站生成一个NPC -->
      <SpawnAction 
        npcsetidentifier="outpostnpcs1" 
        npcidentifier="commoner" 
        targettag="my_npc" 
        spawnlocation="Outpost" 
      />
      <!-- 
        npcsetidentifier: NPC套装ID（游戏内置的NPC模板集合）
        npcidentifier: 具体是哪种NPC（commoner=普通平民）
        targettag: 给生成的NPC贴个标签，方便后续操作
        spawnlocation: 在哪里生成（Outpost=前哨站）
      -->

      <!-- 第三步：等玩家靠近NPC时触发 -->
      <TriggerAction 
        target1tag="player" 
        target2tag="my_npc" 
        radius="150" 
        waitforinteraction="true" 
      />
      <!-- 
        target1tag/target2tag: 监测这两个目标之间的距离
        radius: 触发距离（150像素）
        waitforinteraction: 是否需要玩家主动互动（true=需要按E）
      -->

      <!-- 第四步：播放一段对话 -->
      <ConversationAction 
        text="对话内容写这里（或者填写本地化文本ID）" 
        speakertag="my_npc"
        dialogtype="Small"
      >
      <!-- 
        text: 对话的文字内容
        speakertag: 谁在说话（用之前贴的标签）
        dialogtype: 对话框类型（Small=小型，Regular=常规）
      -->

        <!-- 选项1：同意帮忙 -->
        <Option text="好的，我来帮你！">
          <MoneyAction amount="500" />
          <!-- 给玩家500块钱作为奖励 -->
          <ConversationAction text="太感谢你了！" />
        </Option>

        <!-- 选项2：拒绝 -->
        <Option text="抱歉，我很忙" endconversation="true" />
        <!-- endconversation="true" 表示选择后直接结束对话 -->

      </ConversationAction>

    </ScriptedEvent>
    <!-- 事件结束 -->

  </EventPrefabs>
</RandomEvents>
```

---

## 核心参数字典 (Cheat Sheet)

### 📌 常用标签（Tags）

| 标签名 | 作用 | 必填属性 | 示例 |
|--------|------|----------|------|
| `<ScriptedEvent>` | 定义一个剧情事件 | `identifier` | `<ScriptedEvent identifier="test">` |
| `<TraitorEvent>` | 定义一个叛徒任务 | `identifier`, `dangerlevel` | `<TraitorEvent identifier="test" dangerlevel="1">` |
| `<TagAction>` | 给目标贴标签 | `criteria`, `tag` | `<TagAction criteria="player" tag="p1"/>` |
| `<SpawnAction>` | 生成NPC/物品/怪物 | 至少一个spawn参数 | 见下方详细表格 |
| `<ConversationAction>` | 触发对话 | `text` | `<ConversationAction text="你好">` |
| `<Option>` | 对话选项 | `text` | `<Option text="选项文字">` |
| `<TriggerAction>` | 设置触发条件 | `target1tag` | `<TriggerAction target1tag="player">` |
| `<WaitAction>` | 等待指定秒数 | `time` | `<WaitAction time="5"/>` |
| `<MoneyAction>` | 给/扣钱 | `amount` | `<MoneyAction amount="-100"/>` |
| `<ReputationAction>` | 改变声望 | `targettype`, `identifier`, `increase` | `<ReputationAction targettype="Faction" identifier="coalition" increase="10"/>` |
| `<MissionAction>` | 触发任务 | `missiontag` 或 `missionidentifier` | `<MissionAction missiontag="cargo"/>` |
| `<CheckDataAction>` | 检查存档数据 | `identifier`, `condition` | `<CheckDataAction identifier="flag1" condition="eq 1">` |
| `<SetDataAction>` | 设置存档数据 | `identifier`, `value` | `<SetDataAction identifier="flag1" value="1"/>` |
| `<GoTo>` | 跳转到标签位置 | `name` | `<GoTo name="retry"/>` |
| `<Label>` | 定义跳转标签 | `name` | `<Label name="retry"/>` |

### 📌 SpawnAction 详细参数

| 属性名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `npcsetidentifier` | 文本 | NPC模板集合ID | `"outpostnpcs1"` |
| `npcidentifier` | 文本 | 具体NPC类型 | `"commoner"`, `"securitynpc"` |
| `itemidentifier` | 文本 | 物品ID | `"revolver"`, `"antibleeding1"` |
| `speciesname` | 文本 | 怪物种类名 | `"Mudraptor"`, `"crawler"` |
| `targettag` | 文本 | 给生成物贴的标签 | `"my_npc"` |
| `spawnlocation` | 文本 | 生成位置 | `"Outpost"`, `"BeaconStation"` |
| `amount` | 数字 | 生成数量 | `"3"` |
| `targetinventory` | 文本 | 放入哪个容器/背包 | `"player"` |

### 📌 ConversationAction 参数

| 属性名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `text` | 文本 | 对话内容/本地化ID | `"你好！"` |
| `speakertag` | 文本 | 说话人的标签 | `"my_npc"` |
| `targettag` | 文本 | 对话目标的标签 | `"player"` |
| `dialogtype` | 文本 | 对话框大小 | `"Small"`, `"Regular"` |
| `eventsprite` | 文本 | 显示的事件图片 | `"captain"` |
| `endconversation` | 布尔 | 是否结束对话 | `"true"` |

### 📌 CheckDataAction 条件运算符

| 运算符 | 含义 | 示例 |
|--------|------|------|
| `eq` | 等于 (equal) | `condition="eq 1"` → 等于1 |
| `gt` | 大于 (greater than) | `condition="gt 5"` → 大于5 |
| `gte` | 大于等于 | `condition="gte 10"` → ≥10 |
| `lt` | 小于 (less than) | `condition="lt 3"` → 小于3 |
| `lte` | 小于等于 | `condition="lte 0"` → ≤0 |

### 📌 常用 spawnlocation 值

| 值 | 说明 |
|----|------|
| `Outpost` | 前哨站 |
| `BeaconStation` | 信标站 |
| `MainPath` | 主航道 |
| `Wreck` | 沉船 |
| `Ruin` | 遗迹 |

### 📌 常用 faction 值

| 值 | 说明 |
|----|------|
| `coalition` | 联盟势力 |
| `separatists` | 分离主义者势力 |

---

## 新手避坑指南

### ❌ 错误1：identifier 重复

```xml
<!-- 错误示例 -->
<ScriptedEvent identifier="my_event">...</ScriptedEvent>
<ScriptedEvent identifier="my_event">...</ScriptedEvent>  <!-- 重复了！ -->
```

**后果**：游戏只会加载第一个，第二个会被忽略，或者报错崩溃。

✅ **解决方法**：每个事件的 `identifier` 必须独一无二，建议用 `模组名_功能名` 的命名方式。

---

### ❌ 错误2：标签没有正确闭合

```xml
<!-- 错误示例 -->
<ConversationAction text="你好">
  <Option text="选项1">
  <!-- 忘记写 </Option> 了！ -->
</ConversationAction>
```

**后果**：XML解析失败，游戏无法加载你的mod。

✅ **解决方法**：
- 每个 `<标签>` 必须有对应的 `</标签>`
- 或者使用自闭合标签 `<标签 .../>`
- 建议使用带XML高亮的编辑器（如 VS Code）

---

### ❌ 错误3：使用了不存在的标签名

```xml
<!-- 错误示例 -->
<TriggerAction target1tag="player" target2tag="npc"/>
<!-- 但是之前根本没有用 TagAction 创建 "npc" 这个标签！ -->
```

**后果**：游戏找不到目标，事件会卡住或者跳过。

✅ **解决方法**：使用某个标签之前，确保你已经用 `<TagAction>` 或 `<SpawnAction targettag="...">` 创建了它。

---

### ❌ 错误4：属性值没有加引号

```xml
<!-- 错误示例 -->
<WaitAction time=5/>  <!-- 5 没有加引号！ -->
```

**后果**：XML语法错误。

✅ **解决方法**：所有属性值都必须用英文双引号包裹：`time="5"`

---

### ❌ 错误5：中英文标点混用

```xml
<!-- 错误示例 -->
<ConversationAction text="你好！"/>  <!-- 这个引号是中文的！ -->
```

**后果**：XML解析失败。

✅ **解决方法**：
- 代码里的所有标点符号必须是**英文半角**
- 只有对话内容文字里可以用中文标点

---

## 📖 推荐学习路径

1. **先看** `MissionEvents.xml` - 结构简单，容易理解
2. **再看** `OutpostEventsLegacy.xml` - 经典的NPC对话事件
3. **进阶** `ChainingEvents.xml` - 学习多阶段连锁事件
4. **高级** `TraitorEvents.xml` - 学习条件检测和分支逻辑

---

## 🎯 下一步

查看 `new_guys_guide` 文件夹里其他的 `*_Analysis.md` 文件，了解每个官方事件文件的详细解析！

---

*本指南基于仓库中 `events/` 文件夹的官方XML文件编写*
