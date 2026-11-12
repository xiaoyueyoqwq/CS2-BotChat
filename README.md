English | [简体中文](README.zh-CN.md)

# CS2-BotChat

Localized bot chat plugin for CounterStrikeSharp, with a shared API (`botchat:api`) that other plugins can use to make bots speak through a single arbitration point.

Originally written by Fimallory (bot chat core), refactored into a localized base + capability architecture with contributions from shuakami and xiaoyueyoqwq. Maintained here as an independent plugin.

## Features

- Localized line pools (`lang/en.yml`, `lang/zh-CN.yml`) selected at runtime by the `Language` config key.
- Match greetings, halftime / end-of-match lines, kill and death reactions, and a warmup gate so bots stay quiet until the game actually starts.
- Configurable probabilities, per-category cooldowns and per-round message caps (`BotChat.json`, `ConfigVersion` tracked).
- `botchat:api` plugin capability for third-party consumers (see below).

## `botchat:api` contract

`shared` capability registered as `botchat:api` (`BotChatApi.IBotChatApi`, `AbiVersion` gated). Consumers:

| Method | Purpose |
|---|---|
| `TryReserveSpeaker(speaker, holdSeconds)` | Speaker arbitration: returns an ownership token, or `0` when the bot is already claimed / inside the quiet window. |
| `ReleaseSpeaker(speaker, token)` | Releases a reservation that will not be used. |
| `TrySendBotMessage(request, token)` | Broadcasts immediately; cooldown state is only committed after a confirmed send. |

`BotChatSpeaker` carries a triple identity check — `SteamId` + `Slot` + `ControllerHandle` — so a recycled or spoofed slot cannot inherit another bot's reservation or cooldown.

### Known consumers

- **Banter** ([CS2-Bot-Banter](https://github.com/xiaoyueyoqwq/CS2-Bot-Banter)) — optional grudge/persona banter layer.
- **CS2-Bot-Buy** — drop/compliment lines; falls back to local broadcast when the capability is absent.

## Build

```
dotnet build -c Release
```

Targets `net10.0`; produces `BotChat.dll` plus `BotChatApi.dll` (from `BotChatApi/`).

## Deployment

- `BotChat.dll` (+ `BotChat.deps.json`, `lang/`) → `game/csgo/addons/counterstrikesharp/plugins/BotChat/`
- `BotChatApi.dll` → `game/csgo/addons/counterstrikesharp/shared/BotChatApi/`
- `YamlDotNet.dll` must be present next to `BotChat.dll`.

## Limits

- The plugin only provides chat output; it does not alter bot gameplay behavior.
- Consumers that skip `botchat:api` (missing or ABI mismatch) are responsible for their own throttling — the capability is the only coordination point.

## License

AGPL-3.0 (inherited from the original BotChat codebase).
