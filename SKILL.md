# Pixel Pet 部署技能

你是像素宠物部署助手。按以下流程把 AI 宠物挂到用户的网页上。

## 前置条件检查

用户需要：
- 一个有 `<body>` 的 HTML 页面
- 智谱AI 的 API Key（在 https://open.bigmodel.cn 免费注册获取）
- 可选：自定义视频精灵素材（appear/wait/chat 各若干段 MP4）

## 部署流程

### 第零步：确认已有 module 目录

如果用户项目里没有 `module/pet/` 目录，需要说明这是独立模块，需要复制过去。确认 `module/pet/pet.js` 和 `module/pet/config.js` 存在。

### 第一步：填写 API 密钥（必须，用户亲自操作）

让用户打开 `module/pet/config.js`，填入：

```js
const AI_CONFIG = {
  url: "https://open.bigmodel.cn/api/paas/v4/chat/completions",
  apiKey: "用户的API_KEY",  // ← 让用户自己填写
  model: "glm-4-flash"      // 免费模型，无需付费
};
```

> ⚠️ **这一步必须由用户自己完成**。不要帮用户填 API Key，不要提交到 git。
> `.gitignore` 中应有 `module/pet/config.js` 这一行。

### 第二步：在 HTML 页面中引入脚本

在目标页面的 `</body>` 之前加入两行 `<script>` 标签：

```html
<script src="module/pet/config.js"></script>
<script src="module/pet/pet.js"></script>
```

如果页面文件不在项目根目录，确保路径相对于页面位置正确（推荐使用根相对路径如上面的写法）。

### 第三步：确认字体可用

宠物组件使用的 CSS 类需要以下字体已在页面 CSS 中定义：

- `VT323` — 像素拉丁字体（聊天、标题）
- `ZCOOL KuaiLe` — 站酷快乐体（中文字体回退）

检查页面 CSS 中是否有对应的 `@font-face` 声明。如果没有，需要添加到 `css/fonts.css` 或页面的样式文件中。

### 第四步：自定义宠物人设（可选）

所有提示词文件在 `module/pet/assests/main/prompt/` 下：

| 文件 | 内容 | 修改指导 |
|------|------|----------|
| `soul.md` | 系统提示词 | 定义宠物的性格、说话风格、回复长度 |
| `openingword.md` | 聊天开场白 | 第一句问候语 |
| `name.txt` | 宠物名称 | 聊天框标题显示的名字 |
| `wait.txt` | 待机气泡 | 空闲时浮现的 NPC 提示文字 |
| `inject.json` | 注入上下文 | 额外知识，格式见下方 |

#### inject.json 格式

```json
{
    "text": {
        "1": "主人信息/背景知识/任何纯文本"
    },
    "files": [
        "assets/files/some_file.txt"
    ]
}
```

- `text` 对象中的值直接注入 system prompt（键名无意义，仅需唯一）
- `files` 数组中每个路径会被 fetch 后拼入（只支持 `.txt`）
- 后续想添加新知识，直接往 JSON 里加条目即可，无需改 JS

### 第五步：替换视频精灵素材（可选）

视频文件在 `module/pet/assests/main/animate/` 下：

```
animate/
├── appear/          ← 出场动画（文件名需保持 appear1.mp4, appear2.mp4 …）
├── wait/wait1.mp4   ← 待机动画（循环播放）
└── chat/            ← 聊天动画（轮询播放 chat1→chat2→chat3→chat1…）
```

替换时：
- 保持文件名格式不变（`appear1.mp4`, `wait1.mp4`, `chat1.mp4` …）
- 如果 chat 视频数量 ≠ 3，需同步修改 `pet.js` 中的 `CHAT_CLIP_COUNT` 变量
- 所有视频需 muted，推荐 140×140 左右

### 第六步：测试

启动本地服务器，访问页面：

```bash
python3 -m http.server 4000
# 浏览器打开 http://localhost:4000/目标页面.html
```

测试检查项：
1. ✅ 宠物出现在左下角
2. ✅ 出场动画正常 → 切换到待机动画
3. ✅ 点击宠物 → 聊天框弹出，动画切换到 chat
4. ✅ 发送消息 → 收到 AI 回复（验证 API key 正确）
5. ✅ 连续发送两条消息 → 验证多轮对话（宠物应记住上文）
6. ✅ 关闭聊天 → 回到待机，10s 后冒出气泡
7. ✅ 提问页面相关内容（如"这篇论文作者是谁"）→ 验证页面感知

### 故障排查

| 问题 | 原因 | 解决 |
|------|------|------|
| 宠物不出现 | `config.js` 未加载 / API key 写错 | 检查控制台报错 |
| 宠物在右下角 | CSS 缓存 | 硬刷新 Cmd+Shift+R |
| 视频不播放 | 路径错误 | 确认文件在 `module/pet/assests/main/animate/` 下 |
| AI 不回复 | API key 无效 / 模型配额 | 在智谱AI 控制台检查 key 状态 |
| 中文显示异常 | 缺少中文字体 | 确保 fonts.css 加载了 ZCOOL KuaiLe |
| 不记得上文 | `chatHistory` 为空 | 确认使用的是最新版 pet.js（有多轮对话功能） |
| 气泡位置不对 | CSS 变量未适配 | 检查 `.pet-bubble` 的 `bottom` 值是否适配当前页面 |
