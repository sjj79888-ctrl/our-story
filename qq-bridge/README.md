# qq-bridge — QQ 小号“小满”

## 一句话

Torres 的 QQ 小号 2750579528（小满）通过 SnowLuma + 一个 Python 桥接程序，
在 QQ 里跟人聊天。模型在云端调，人在 QQ 里说话。
SnowLuma 是真 QQ 客户端，2026-11-07 起替代 NapCat。

## 服务器

- 腾讯云 Lighthouse，广州，实例 lhins-7y2px0v5
- 公网 IP：139.199.204.177，2 核 2G，Docker CE，登录用户 ubuntu
- 免费试用，**到期：2026-12-04 00:02:55**，到期前要定办法
- QQ 协议端：SnowLuma（napcat 已停，数据留着当回退，别删）
- 部署文件在 qq-bridge 仓库里，不在这个包里

## 最重要的一条

**不要随便重建 snowluma 容器**，登录状态会丢。
只重建 qq-bridge，而且要 Torres 同意了再重建。
扫码用服务器上的 QR 页 http://139.199.204.177/（码约 2 分钟过期）。

## 文件都在哪

服务器上：`/home/ubuntu/qq-bridge/`

- `bridge.py` — 桥的程序，部署版在 qq-bridge 仓库里，服务器上的是同一份。
  详见 `BRIDGE-SPEC.md`。
- `system-prompt.txt` / `system-prompt-mom.txt` / `system-prompt-maid.txt` —
  三个人格的原文，也在服务器上。
  这个仓库 `personas/` 里的是按记忆重建的版本，不是逐字原文，
  部署前以服务器上的文件为准。
- `.env` — 真配置在服务器上，有真 key，**绝不进仓库**。
  这个仓库里只有 `.env.example`，是模板。
- `stickers/` — 23 张表情包在服务器上（2026-11-07 03:05 换的新一批）。
  `stickers/legs.jpg` 只有本地这份有（2026-11-05 那版），
  推送工具推不了二进制文件，所以仓库里没有它，只有 `stickers/MANIFEST.md`。
  清单见 `stickers/MANIFEST.md`。
- 2026-11-05 的 `.env.bak-20261105-*` 在 2026-11-06 wipe 里丢了，
  别当成还在。

## 账号

- 小号：2750579528，小满
- Torres 大号：1323841589，长温，私聊白名单
- 黑名单：2482076278、428762607，写在 `.env` 的 `BLACKLIST_QQ_IDS` 里

## 另外五份文档

- `BRIDGE-SPEC.md` — 桥的行为规格：回复规则、人格切换、称呼、黑名单、自主管号
- `VOICE-STICKERS.md` — 语音和表情包的触发规则
- `VOICE-TRAINING.md` — 声音的来龙去脉：Genie 训练、SparkTTS 朋友音色
- `VOICE-OPS.md` — 语音运维手册：启动、重启、部署、查日志都在这一篇
- `BACKEND.md` — 模型后端：中转、GLM 备胎、2026-11-05 那次故障
