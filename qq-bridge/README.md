# qq-bridge — QQ 小号“小满”

## 一句话

Torres 的 QQ 小号 2750579528（小满）通过 NapCat + 一个 Python 桥接程序，
在 QQ 里跟人聊天。模型在云端调，人在 QQ 里说话。

## 服务器

- 腾讯云 Lighthouse，广州，实例 lhins-7y2px0v5
- 公网 IP：139.199.204.177，2 核 2G，Docker CE，登录用户 ubuntu
- 免费试用，**到期：2026-12-04 00:02:55**，到期前要定办法
- 两个容器：`napcat` 和 `qq-bridge`
- NapCat 监听 3000 / 6099，桥监听 8787/event，端口都只绑 127.0.0.1，
  从公网直接测是通不了的，这不是故障

## 最重要的一条

**绝不能重启 napcat 容器。** 重启会丢 QQ 登录状态。
登录自己也掉过一次（2026-11-04 16:05）。
只重建 qq-bridge，而且要 Torres 同意了再重建。
重建命令示例：`docker compose up -d --build --no-deps bridge`

## 文件都在哪

服务器上：`/home/ubuntu/qq-bridge/`

- `bridge.py` — 桥的程序，唯一真本在服务器上。这个仓库里没有它的副本，
  因为本地拿不到它（OrcaTerm 下载拿不到文件）。要改它，先从服务器上取，
  改完备份再重建。详见 `BRIDGE-SPEC.md`。
- `system-prompt.txt` / `system-prompt-mom.txt` — 两个人格的原文，也在服务器上。
  这个仓库 `personas/` 里的是按记忆重建的版本，不是逐字原文，
  部署前以服务器上的文件为准。
- `.env` — 真配置在服务器上，有真 key，**绝不进仓库**。
  这个仓库里只有 `.env.example`，是模板。
- `stickers/` — 9 张表情包在服务器上。`stickers/legs.jpg` 是本地有的那张，
  另外 8 张要从服务器上取，清单见 `stickers/MANIFEST.md`。
- 备份文件别删：`.env.bak-20261105-glms`、`.env.bak-20261105-deepseek`、
  `.env.bak-20261105-kamente`，还有 `bridge.py.bak-20261105-*` 那几个。

## 账号

- 小号：2750579528，小满
- Torres 大号：1323841589，长温，私聊白名单
- 黑名单：2482076278，写在 `.env` 的 `BLACKLIST_QQ_IDS` 里

## 另外三份文档

- `BRIDGE-SPEC.md` — 桥的行为规格：回复规则、人格切换、称呼、黑名单
- `VOICE-STICKERS.md` — 语音和表情包的触发规则
- `BACKEND.md` — 模型后端：中转、GLM 备胎、2026-11-05 那次故障
