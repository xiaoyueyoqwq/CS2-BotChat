[English](README.md) | 简体中文

# CS2-BotChat

CounterStrikeSharp 本地化 bot 聊天插件，附带共享 API（`botchat:api`），让其他插件通过统一仲裁点让 bot 发言。

原作者 Fimallory（bot 聊天核心），后与 shuakami、xiaoyueyoqwq 一起重构为"本地化基础插件 + 能力契约"架构。在此仓库独立维护。

## 功能

- 本地化台词池（`lang/en.yml`、`lang/zh-CN.yml`），由 `Language` 配置键运行时选择。
- 比赛开场问候、半场/终场消息、击杀与死亡反应；开局热身期内 bot 保持安静。
- 可配置概率、分类冷却、每回合发言上限（`BotChat.json`，带 `ConfigVersion` 追踪）。
- 对外暴露 `botchat:api` 插件能力（见下）。

## `botchat:api` 契约

以 `botchat:api` 注册的共享能力（`BotChatApi.IBotChatApi`，`AbiVersion` 门禁）。消费方接口：

| 方法 | 作用 |
|---|---|
| `TryReserveSpeaker(speaker, holdSeconds)` | 发言仲裁：返回所有权 token；bot 已被占用或在静默窗口内返回 `0`。 |
| `ReleaseSpeaker(speaker, token)` | 释放未使用的预约。 |
| `TrySendBotMessage(request, token)` | 立即广播；冷却状态在确认发出后才提交。 |

`BotChatSpeaker` 携带三重身份校验——`SteamId` + `Slot` + `ControllerHandle`——回收或伪造的 slot 无法继承他人预约或冷却。

### 已知消费方

- **Banter**（[CS2-Bot-Banter](https://github.com/xiaoyueyoqwq/CS2-Bot-Banter)）——可选恩怨/人设垃圾话层。
- **CS2-Bot-Buy**——掉落/夸赞台词；能力缺失时降级为本地广播。

## 构建

```
dotnet build -c Release
```

目标框架 `net10.0`；产出 `BotChat.dll` 与 `BotChatApi.dll`（来自 `BotChatApi/`）。

## 部署

- `BotChat.dll`（+ `BotChat.deps.json`、`lang/`）→ `game/csgo/addons/counterstrikesharp/plugins/BotChat/`
- `BotChatApi.dll` → `game/csgo/addons/counterstrikesharp/shared/BotChatApi/`
- `YamlDotNet.dll` 需与 `BotChat.dll` 同目录。

## 限制

- 插件只负责聊天输出，不改变 bot 的游戏行为。
- 绕过 `botchat:api`（缺失或 ABI 不匹配）的消费方需自行节流——该能力是唯一协调点。

## 许可证

AGPL-3.0（继承自原始 BotChat 代码库）。
