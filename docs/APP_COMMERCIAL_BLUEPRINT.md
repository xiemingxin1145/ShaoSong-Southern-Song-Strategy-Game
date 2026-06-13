# 《绍宋：南渡无悔》APP 商业级开发总蓝图

> 版本：v2.0 / APP Commercial Blueprint  
> 角色定位：总工程师 + 主策划 + 技术美术管线负责人  
> 用途：交给 Claude / Codex / Cursor 后，可直接拆任务、生成代码、建立数据结构、推进客户端原型。

---

## 0. 项目校准

当前仓库原始方向为“南宋深度策略游戏”，偏概念型 GDD。本文档将项目升级为可落地的手机 APP 游戏方案：

- **核心类型**：历史架空策略 SLG + 朝堂抉择 + 大地图战局 + 名臣养成。
- **推荐平台**：Android / iOS。
- **推荐形态**：竖屏优先，横屏可作为后续平板/PC 版本。
- **首发剧本**：靖康危局逆转线，时间从 1125 年末到 1127 年。
- **DLC / 扩展剧本**：建炎南渡、绍兴北伐、蒙元南侵、崖山终局。
- **商业提醒**：若项目商业发行，涉及小说名、角色设定、原作 IP 的部分必须确认授权；无授权时建议改为原创名，例如《天命南渡》《靖康逆转》《大宋无悔》。

### 0.1 一句话定位

玩家不是在“看历史”，而是在靖康风雪里坐上皇位，用每一道朱批、每一次调兵、每一个朝堂选择，决定大宋是被拖进五国城，还是逆风翻盘。

### 0.2 核心体验

1. **奏折式决策**：像皇帝一样批阅奏折，左滑驳回、右滑准奏、点开朱批。
2. **大地图战局**：汴梁、太原、黄河、防线、粮道，以节点地图呈现。
3. **名臣名将养成**：李纲、种师道、岳飞、韩世忠等以卡牌/立绘/动态事件驱动。
4. **AI 动态剧情**：根据玩家当前局势生成朝争、谣言、军报、人物私信。
5. **多结局网络**：靖康之耻、流亡南渡、偏安江南、绍宋复兴、北伐成功。

---

## 1. 技术栈定案

### 1.1 客户端推荐方案

优先选择：**Cocos Creator 3.x + TypeScript**。

原因：

- 2D UI、卡牌、地图、弹窗、粒子表现更轻。
- 移动端包体和性能压力低。
- TypeScript 适合 Claude/Codex 快速生成工程代码。
- 后期接入热更新、资源分包、Web 预览更方便。

备选：**Unity 2D + C#**。

适合场景：

- 后续想做更重的战场演出。
- 需要 Spine / Live2D / Timeline / Shader Graph 深度整合。
- 团队已经熟悉 Unity。

### 1.2 后端推荐方案

首版推荐：**FastAPI + SQLite/PostgreSQL + LLM Adapter**。

职责：

- 保存玩家云存档。
- 管理 AI 大模型 Key 和多模型适配。
- 生成动态事件。
- 记录历史时间线演变日志。
- 提供数据版本更新接口。

### 1.3 AI Key 插拔系统

项目必须设计成“用户输入任意大模型 API Key 后激活 AI 剧情”。

支持模式：

- OpenAI 兼容接口。
- Claude 兼容代理接口。
- Gemini / DeepSeek / Qwen / Moonshot 等兼容接口。
- 本地离线规则引擎兜底。

前端只认统一接口：

```ts
interface AIProviderConfig {
  providerId: string;          // openai-compatible / claude / gemini / custom
  baseUrl: string;             // API 地址
  apiKey: string;              // 用户输入 Key，本地加密保存
  model: string;               // 模型名称
  enabled: boolean;
}
```

后端统一转成：

```ts
interface LLMRequest {
  systemPrompt: string;
  userPrompt: string;
  responseFormat: 'json' | 'text';
  temperature: number;
  maxTokens: number;
}
```

---

## 2. 推荐仓库结构

```txt
ShaoSong-Southern-Song-Strategy-Game/
├── README.md
├── GDD.md
├── docs/
│   ├── APP_COMMERCIAL_BLUEPRINT.md
│   ├── CLAUDE_HANDOFF_PROMPT.md
│   ├── MVP_SPRINT_0_TASKS.md
│   ├── UI_SCREEN_MAP.md
│   ├── AI_EVENT_PROTOCOL.md
│   └── ART_PIPELINE.md
├── client-cocos/
│   ├── assets/
│   │   ├── scripts/
│   │   │   ├── core/
│   │   │   ├── ui/
│   │   │   ├── systems/
│   │   │   ├── data/
│   │   │   └── ai/
│   │   ├── prefabs/
│   │   ├── textures/
│   │   ├── spine/
│   │   ├── live2d/
│   │   └── scenes/
│   └── package.json
├── server/
│   ├── app/
│   │   ├── main.py
│   │   ├── routers/
│   │   ├── services/
│   │   ├── llm/
│   │   ├── schemas/
│   │   └── storage/
│   └── requirements.txt
├── data/
│   ├── officers.json
│   ├── cities.json
│   ├── events_static.json
│   ├── policies.json
│   ├── units.json
│   └── scripts/
├── art/
│   ├── prompts/
│   ├── portraits/
│   ├── cg/
│   ├── icons/
│   └── ui/
└── prototype/
    └── web-demo/
```

---

## 3. APP UI/UX 总结构

### 3.1 交互总原则

- **竖屏优先**：玩家单手批奏折。
- **信息分层**：地图是底层，政务是弹窗，角色是抽屉。
- **国风但清晰**：不能为了水墨牺牲可读性。
- **每次点击都有反馈**：墨晕、纸张翻页、木鱼/印章声。

### 3.2 屏幕层级

#### Layer 0：动态大地图

内容：

- 汴梁、太原、中山、河间、东京周边、黄河渡口。
- 节点式地图，不建议首版做完整 3D 地图。
- 支持拖动、缩放、点击城池。
- 天气粒子：雪、雨、雾、风沙。
- 战线表现：红色为金军压力，青色为宋军控制，黄色为争夺区。

#### Layer 1：顶部 HUD

字段：

- 年月/旬：1126 年正月上旬。
- 国库：钱、粮、军械。
- 皇威：影响朝臣服从。
- 民心：影响城市稳定。
- 军心：影响战斗士气。
- 行动力：每旬可批奏折/调兵/召见次数。

#### Layer 2：底部导航坞

四大入口：

1. **尚书省**：政令、财政、民生。
2. **枢密院**：军令、调兵、防线。
3. **点将录**：官员、武将、忠诚、派系。
4. **内库**：道具、招募、图鉴、AI Key 设置。

#### Layer 3：奏折/事件弹窗

奏折类型：

- 军报：太原告急、黄河渡口失守。
- 朝争：主和派弹劾李纲。
- 民变：京师粮价暴涨。
- 密奏：皇城司发现奸细。
- 私信：皇后、太学生、将领来信。

交互：

- 左滑：驳回。
- 右滑：准奏。
- 长按：御笔朱批。
- 点击角色头像：查看提出奏折的人物信息。

---

## 4. 核心玩法系统

### 4.1 朝会与奏折系统

每旬开始自动生成 3-5 份奏折。

奏折来源：

- 静态历史事件。
- 动态 AI 事件。
- 城市状态触发。
- 官员派系触发。
- 战场压力触发。

奏折字段：

```ts
interface PetitionEvent {
  id: string;
  title: string;
  type: 'military' | 'court' | 'finance' | 'people' | 'secret' | 'diplomacy';
  speakerId: string;
  speakerName: string;
  body: string;
  urgency: number;       // 1-100
  options: PetitionOption[];
  cgTrigger?: string;
  expiresInTurns?: number;
}

interface PetitionOption {
  id: string;
  text: string;
  description?: string;
  cost?: ResourceDelta;
  effect: GameStateDelta;
  aiFollowUpHint?: string;
}
```

### 4.2 大地图与行军系统

地图使用节点图，不做复杂即时战略。

```ts
interface MapNode {
  id: string;
  name: string;
  type: 'capital' | 'city' | 'fort' | 'river_crossing' | 'camp';
  x: number;
  y: number;
  owner: 'song' | 'jin' | 'neutral' | 'rebel';
  defense: number;
  supply: number;
  morale: number;
  connectedNodeIds: string[];
}
```

行军规则：

- 一支军队从一个节点移动到相邻节点。
- 雪天、断粮、敌军骚扰会增加行军时间。
- 补给线断裂会持续扣士气。
- 名将技能可以改变行军或防守效率。

### 4.3 官员与派系系统

字段：

```ts
interface Officer {
  id: string;
  name: string;
  dynastyRole: string;
  type: 'civil' | 'military' | 'royal' | 'eunuch' | 'scholar';
  stats: {
    command: number;
    force: number;
    intelligence: number;
    politics: number;
    charisma: number;
  };
  loyalty: number;
  ambition: number;
  faction: 'pro_war' | 'pro_peace' | 'royalist' | 'neutral' | 'hidden_traitor';
  skills: string[];
  portraitPath: string;
  live2dPath?: string;
  status: 'normal' | 'injured' | 'captured' | 'dead' | 'exiled';
}
```

派系逻辑：

- 主战派支持军费、北伐、防守，但可能消耗财政。
- 主和派降低短期战争压力，但损害军心和忠诚。
- 清流太学生提高民心，但会制造朝堂压力。
- 皇族势力会影响帝位稳定。

### 4.4 科举/招募/卡池系统

为了手游化，需要把“获得人才”做成可持续循环。

类型：

- **恩科**：普通招募，消耗威望。
- **制科**：特殊政策解锁，偏文臣。
- **密诏**：高级招募，偏名臣名将。
- **历史登场**：到指定时间自动触发。

注意：商业化不能只靠抽卡，买断制也可以保留“人才搜寻/科举”玩法，把付费点转成 DLC 剧本、皮肤、CG、战役包。

### 4.5 战时国策树

分类：

- 军事：神臂弓、步人甲、震天雷、军屯、马政。
- 财政：盐铁、海贸、交子、榷货务。
- 政治：清洗六贼、重开言路、整肃台谏。
- 民生：赈济、平粜、修河、迁民。
- 外交：联辽残部、离间金军、安抚西夏。

```ts
interface PolicyNode {
  id: string;
  name: string;
  category: 'military' | 'finance' | 'court' | 'people' | 'diplomacy';
  description: string;
  cost: ResourceDelta;
  requires: string[];
  effects: GameStateDelta;
  iconPath: string;
}
```

---

## 5. 剧情结构

### 5.1 序章：受命于危难

时间：1125 年十二月 - 1126 年正月。

关键事件：

1. 宋徽宗南逃。
2. 主角即位。
3. 金军东路逼近汴梁。
4. 李纲上殿请战。
5. 主和派要求割地赔款。
6. 太学生伏阙上书。
7. 第一次汴梁攻防战。

可能结局：

- 靖康之耻：城防归零。
- 屈辱求和：短期保命，长期埋雷。
- 死守成功：进入第一章。

### 5.2 第一章：风雨飘摇的喘息

时间：1126 年二月 - 1126 年闰十一月。

核心：第一次金军退兵后，玩家要在短暂喘息期完成整军、清洗、修防、筹粮。

事件：

- 六贼余党反扑。
- 李纲被弹劾。
- 勤王军粮草不足。
- 种师道病重但仍愿领兵。
- 皇城司发现奸细。
- 京师粮价暴涨。

### 5.3 第二章：二次南下

时间：1126 年底 - 1127 年。

核心：金军两路再来，太原、黄河、汴梁三线压力爆炸。

结局：

- **流亡南渡**：开启南宋剧本。
- **守住汴梁**：进入架空绍宋线。
- **反攻幽云**：完美隐藏线。

---

## 6. AI 动态事件协议

### 6.1 客户端请求

```json
{
  "request_type": "GENERATE_MONTHLY_EVENTS",
  "player_id": "local_player",
  "save_id": "slot_001",
  "game_state": {
    "current_time": "1126_FEB_UPPER",
    "capital_defense": 80,
    "treasury": 12000,
    "food": 50000,
    "people_support": 62,
    "emperor_traits": ["IronFisted", "ProWar"],
    "frontline_pressure": {
      "taiyuan": 90,
      "yellow_river": 70,
      "kaifeng": 60
    },
    "alive_officers": [
      {"id": "li_gang", "name": "李纲", "loyalty": 95, "faction": "pro_war"},
      {"id": "qin_gui", "name": "秦桧", "loyalty": 30, "faction": "pro_peace"}
    ],
    "recent_decisions": [
      "refused_peace_treaty",
      "increased_military_budget"
    ]
  }
}
```

### 6.2 AI 返回

```json
{
  "events": [
    {
      "event_id": "DYNAMIC_EV_001",
      "title": "秦桧请和",
      "type": "court",
      "npc_name": "秦桧",
      "dialogue": "陛下，臣闻金人又增兵黄河。若再不遣使议和，恐京师百姓先乱。",
      "options": [
        {
          "id": "angry_punish",
          "text": "廷上怒斥，命其退下",
          "effect": {"qin_gui_loyalty": -20, "war_will": 10, "court_tension": 8}
        },
        {
          "id": "pretend_accept",
          "text": "表面安抚，暗令皇城司盯住",
          "effect": {"qin_gui_loyalty": 5, "secret_service_progress": 10, "court_tension": 3}
        }
      ],
      "cg_trigger": "court_argument",
      "risk_tags": ["faction_conflict", "spy_followup"]
    }
  ]
}
```

### 6.3 生成安全规则

AI 必须遵守：

- 只生成 JSON，不夹杂解释。
- 不擅自杀死核心历史人物，除非 `allow_historical_divergence=true`。
- 不生成超出当前时代的科技，除非玩家已解锁架空科技树。
- 每条事件必须有清晰代价，不能全是奖励。
- 每次最多 5 条事件，移动端弹窗不能过载。

---

## 7. 美术资产管线

### 7.1 UI 材质

- 熟宣纸背景。
- 漆器木纹边框。
- 鎏金细边。
- 朱批红印章。
- 宋代纹样暗纹。

### 7.2 图标清单

资源：

- 钱：交子 + 铜钱。
- 粮：稻穗 + 粮袋。
- 军械：箭簇 + 甲片。
- 皇威：金色玉玺。
- 民心：青色万民伞。
- 军心：赤色战鼓。

功能：

- 尚书省：官印 + 文书。
- 枢密院：虎符 + 战刀。
- 点将录：人物卷轴。
- 内库：宝匣。

### 7.3 立绘状态

每个主要人物至少 4 套状态：

1. 常态。
2. 忠诚高：拱手/微笑/坚定。
3. 忠诚低：阴沉/侧目/冷笑。
4. 战损或危机：血污/破甲/疲惫。

### 7.4 大事件 CG

首批必须做：

- 雪夜勤王。
- 李纲请战。
- 太学生伏阙。
- 汴梁城头。
- 黄河冰封。
- 太原孤城。
- 御门宣诏。

---

## 8. Sprint 0：最小可玩闭环

目标：7-14 天内跑通一个“能玩”的核心闭环。

### 8.1 必做内容

1. 主界面大地图：5 个节点。
2. 顶部资源栏：年月、钱、粮、皇威、民心、行动力。
3. 奏折弹窗：支持 3 条事件，左右选择。
4. 官员数据：李纲、秦桧、种师道、完颜宗望。
5. AIManager：先用 Mock JSON，再接真实 LLM。
6. 存档：本地保存当前回合、资源、选择记录。

### 8.2 可验收标准

- 打开游戏进入大地图。
- 点击“朝会”弹出奏折。
- 选择选项后资源变化。
- 点击“下一旬”推进时间。
- 根据上一次选择生成下一条事件。
- 退出重进后存档还在。

---

## 9. Claude / Codex 开发优先级

### 第一优先级：客户端 UI 原型

生成：

- `GameState.ts`
- `AIManager.ts`
- `PetitionPanel.ts`
- `MapController.ts`
- `OfficerPanel.ts`
- `SaveManager.ts`

### 第二优先级：Mock 数据

生成：

- `data/officers.json`
- `data/map_nodes.json`
- `data/petitions_mock.json`
- `data/policies.json`

### 第三优先级：后端 AI 服务

生成：

- `server/app/main.py`
- `server/app/routers/ai_events.py`
- `server/app/llm/provider_router.py`
- `server/app/schemas/events.py`

---

## 10. 第一条开发命令

交给 Claude / Codex 的第一条命令应该是：

> 请基于 `docs/APP_COMMERCIAL_BLUEPRINT.md`，先不要扩写设定，不要写空泛总结，直接创建 `client-cocos/` 的 TypeScript 核心脚本与 `data/` 的 Mock JSON。先完成 Sprint 0：大地图、资源栏、奏折弹窗、AIManager Mock 数据解析、本地存档。代码必须可运行，中文注释清晰，UI 组件用可替换资源占位，不要依赖真实美术资源。

---

## 11. 总工程师结论

当前项目不要再继续堆大而全幻想，第一阶段只做一个铁三角：

```txt
大地图局势变化 → 奏折事件抉择 → 资源/人物/战线反馈
```

只要这个闭环跑通，后面无论接 AI 剧情、名臣 Live2D、抽卡、政策树、战役 DLC，都能往上叠。反过来，如果这个闭环没有跑通，美术和设定写再多也只是纸面项目。
