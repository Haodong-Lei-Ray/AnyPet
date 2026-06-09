# AnyPet/按你网宠

https://github.com/user-attachments/assets/ec1c64ad-39a7-4112-b398-9a2f0d1a9ea2

> 🤖 **如果你是 AI Agent，请直接阅读 [SKILL.md](SKILL.md)，不要读本文件。**

一个纯前端 AI 宠物组件，为任意网页添加像素风 NPC 聊天伙伴。

## 目录结构

```
module/
├── README.md                          ← 你正在看
├── SKILL.md                           ← Agent 部署指南
└── pet/
    ├── config.js                      ← [需用户填写] API 密钥与模型配置
    ├── pet.js                         ← 核心引擎（纯 JS，零依赖）
    └── assests/main/
        ├── animate/                   ← 视频精灵素材
        │   ├── appear/appear1.mp4     ←  出场动画
        │   ├── wait/wait1.mp4         ←  待机动画（循环）
        │   └── chat/                  ←  聊天动画
        │       ├── chat1.mp4
        │       ├── chat2.mp4
        │       └── chat3.mp4          ←  轮询播放（chat1→2→3→1…）
        └── prompt/
            ├── soul.md                ←  宠物人设（系统提示词）
            ├── openingword.md         ←  开场白
            ├── name.txt               ←  宠物名字（聊天框标题）
            ├── wait.txt               ←  待机气泡文本（每 10s 闪现）
            └── inject.json            ←  额外注入上下文（文本 + 文件）
```

## 核心机制

### 动画状态机

```
appear → wait ⇄ chat
   ↓        ↓      ↓
 入场一次  循环待机  轮询 chat1~3
```

- **appear**: 页面加载后播放一次，结束后自动切换到 wait
- **wait**: 循环播放待机动画；每 10s 弹出 NPC 气泡（内容来自 `wait.txt`）
- **chat**: 点击宠物打开聊天框，动画在 chat1/2/3 之间轮询；关闭后回到 wait

### 上下文注入（多轮对话 + 页面感知）

每次 API 请求携带的消息结构：

```
[system: soul.md]
  + inject.json 注入内容  （如有）
  + 当前网页文本内容       （自动抓取，≤6000 字）
[user: 第1轮用户消息]
[assistant: 第1轮回复]
[user: 第2轮用户消息]
...  ← 完整对话历史保留
```

### inject.json 格式

```json
{
    "text": {"1": "纯文本内容", "2": "可以有多条"},
    "files": [
        "assets/files/something.txt"
    ]
}
```

- `text` 中的值直接拼入 system prompt
- `files` 中的每个路径会被 fetch 后拼入（仅支持 `.txt`）
- 后续只需要往 JSON 里加条目，无需改 JS

### 页面感知（自动跟随所在网页）

`pet.js` 在初始化时调用 `capturePageContext()`：克隆 `<main>` 或 `<body>`，移除宠物自身/script/style/svg，提取纯文本。这样宠物能回答关于所在页面的问题（"我的导师是谁？""这篇论文用的是哪个数据集？"）。

## 部署

两步即可：

```html
<!-- 1. 填好 module/pet/config.js 中的 API key -->
<!-- 2. 在任意页面的 </body> 前加入： -->
<script src="module/pet/config.js"></script>
<script src="module/pet/pet.js"></script>
```

## 文件说明

| 文件               | 用途                       | 是否必须                   |
| ------------------ | -------------------------- | -------------------------- |
| `config.js`      | API endpoint + key + model | **必须**             |
| `pet.js`         | 核心逻辑                   | **必须**             |
| `soul.md`        | 宠物性格                   | 必须                       |
| `openingword.md` | 第一句话                   | 可选                       |
| `name.txt`       | 聊天框标题                 | 必须                       |
| `wait.txt`       | 待机气泡                   | 必须                       |
| `inject.json`    | 额外知识                   | 可选                       |
| `animate/*.mp4`  | 视频精灵                   | 可选（无视频则纯文字聊天） |

## 依赖

- **API**: 智谱AI GLM-4-Flash（免费模型），通过 `config.js` 配置
- **字体**: VT323（像素拉丁）+ ZCOOL KuaiLe（中文字体），需在页面中加载
- **其他**: 零外部 JS 依赖，纯原生 DOM API
