# 📋 OutpostEventsLegacy.xml 解析文档

## 📝 文件概览

**一句话总结**：这是一个**旧版兼容文件**，里面存放的是游戏早期版本使用的一些简单任务事件，现在保留下来是为了让老存档不会出错。

**适合新手程度**：⭐⭐⭐⭐⭐ （结构极其简单，适合第一个学习）

---

## 🎯 文件结构

```
OutpostEventsLegacy.xml
├── <RandomEvents>              ← 根元素（注意这里是大写的E）
│   └── <EventPrefabs>          ← 事件模板容器
│       ├── missionevent_killmonstercommon
│       ├── missionevent_killswarm
│       └── missionevent_killmonsterrare
```

这个文件只有**3个事件**，非常精简！

---

## 🔍 关键逻辑拆解

### 完整事件解析：击杀普通怪物任务

```xml
<ScriptedEvent identifier="missionevent_killmonstercommon" commonness="85">
  <NPCWaitAction npctag="outpostmanager" wait="true" />
  <ConversationAction text="EventText.missionevent_killmonstercommon.c1" speakertag="outpostmanager" endeventifinterrupted="false" dialogtype="Small" />
  <MissionAction missiontag="killmonster" />
  <NPCWaitAction npctag="outpostmanager" wait="false" />
</ScriptedEvent>
```

#### 触发条件 (Trigger)
- `commonness="85"`：相对正常的触发概率
- 无其他特殊条件，系统随机触发

#### 执行过程 (Action)

1. **`<NPCWaitAction npctag="outpostmanager" wait="true" />`**
   - 让前哨站管理员停下来（不再四处走动）
   - 这样玩家可以和他对话

2. **`<ConversationAction ...>`**
   - 管理员说一段话（内容由本地化文件决定）
   - `endeventifinterrupted="false"` → 即使对话被打断，事件也不会结束

3. **`<MissionAction missiontag="killmonster" />`**
   - 触发一个猎杀怪物的任务

4. **`<NPCWaitAction npctag="outpostmanager" wait="false" />`**
   - 让管理员恢复正常行动

#### 结束条件 (Finish)
- 所有Action依次执行完毕后，事件自然结束
- 任务本身是否完成不影响事件结束

---

### 事件对比：普通怪物 vs 稀有怪物

| 属性 | killmonstercommon | killmonsterrare |
|------|-------------------|-----------------|
| `commonness` | 85 | 25 |
| 任务标签 | `killmonster` | `killmonster_set4` |

**学习点**：通过调整 `commonness` 值来控制事件的稀有度。稀有怪物任务（`commonness="25"`）出现概率只有普通任务的约1/3。

---

## 💡 学习重点

### 1️⃣ 学会用 `NPCWaitAction` 控制NPC行动

这个技巧在很多事件中都会用到：

```xml
<!-- 让NPC停下来 -->
<NPCWaitAction npctag="某NPC的标签" wait="true" />

<!-- ... 做一些事情（比如对话） ... -->

<!-- 让NPC恢复行动 -->
<NPCWaitAction npctag="某NPC的标签" wait="false" />
```

**为什么需要这个？**
- 前哨站的NPC默认会到处走
- 如果不让他停下来，可能玩家还没走过去他就跑开了
- 对话结束后要记得让他恢复行动，否则他会一直站着不动

### 2️⃣ 理解 `outpostmanager` 这个特殊标签

游戏内置了一些预定义的标签：

| 内置标签 | 指向 |
|---------|------|
| `outpostmanager` | 前哨站管理员NPC |
| `player` | 当前玩家 |

你不需要用 `TagAction` 创建这些标签，直接使用即可。

### 3️⃣ 事件的线性执行

在这个文件的事件中，所有Action都是**按顺序一个一个执行**的：

```
第1步 → 第2步 → 第3步 → 第4步 → 结束
```

这是最简单的事件流程，适合入门学习。

---

## 📊 代码模板提取

从这个文件中，我们可以提取出一个**管理员发布任务**的标准模板：

```xml
<ScriptedEvent identifier="你的事件ID" commonness="你的概率值">
  <!-- 1. 让管理员停下来 -->
  <NPCWaitAction npctag="outpostmanager" wait="true" />
  
  <!-- 2. 管理员说话 -->
  <ConversationAction 
    text="你的对话内容" 
    speakertag="outpostmanager" 
    endeventifinterrupted="false" 
    dialogtype="Small" 
  />
  
  <!-- 3. 触发任务 -->
  <MissionAction missiontag="任务标签" />
  
  <!-- 4. 让管理员恢复行动 -->
  <NPCWaitAction npctag="outpostmanager" wait="false" />
</ScriptedEvent>
```

---

## 🤔 为什么叫 "Legacy"（遗留）？

- "Legacy" 在编程中通常指"旧版本的代码"
- 这些事件在新版游戏中已经不常用了
- 但为了**向后兼容**（让老存档能正常加载），它们被保留了下来
- 对于学习来说，这些"老代码"反而更简单易懂！

---

## 🎮 实战练习

基于这个文件的模板，写一个让管理员发布"护送任务"的事件：

<details>
<summary>点击查看参考答案</summary>

```xml
<ScriptedEvent identifier="my_escort_mission" commonness="50">
  <NPCWaitAction npctag="outpostmanager" wait="true" />
  <ConversationAction 
    text="有一群难民需要护送到下一个前哨站，你能帮忙吗？" 
    speakertag="outpostmanager" 
    endeventifinterrupted="false" 
    dialogtype="Small" 
  />
  <MissionAction missiontag="escortcommoners" />
  <NPCWaitAction npctag="outpostmanager" wait="false" />
</ScriptedEvent>
```

</details>

---

## 📌 小贴士

这个文件展示了**最简洁的事件写法**。如果你刚开始学习，建议：

1. 先完全理解这3个事件
2. 试着模仿写一个类似的
3. 然后再去看更复杂的事件文件
