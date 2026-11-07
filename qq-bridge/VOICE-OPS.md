# VOICE-OPS — 小满语音运维手册

> 给新接手的 Muse 看的：按这篇就能跑起来、修好、再部署。
> 事实和来龙去脉见 `../MEMORY.md` 和 `VOICE-TRAINING.md`，那两篇是“为什么”，
> 这篇是“怎么做”。本篇只写操作，不重复历史。
> 写于 2026-11-08 03:05 左右（直连隧道桥部署进行中，见末节“当前状态”）。

## 1. 东西都在哪

**语音服务在 Muse 的虚拟机上（不在服务器上）：**

- 目录：`~/workspace/voice-reference/sparktts/service/`
  - `server.py` — FastAPI 服务，只监听 `127.0.0.1:8001`
  - `token.txt`（权限 600）— 桥调用时要带的 token，**绝不进任何仓库**，
    也不贴进聊天。丢了就重新生成一个，同步改服务器 `.env` 里的
    `SPARKTTS_TOKEN`，再重建桥。
  - `run_tunnel.sh` — localhost.run 免费隧道（备胎）
  - `run_direct.sh` — 直连隧道（主用）
  - `sync_url.sh` — 隧道地址同步到公共存储库
  - `install.sh` — 重建全部 systemd 单元（虚拟机换了之后跑它）
- 模型：`~/workspace/voice-reference/sparktts/pretrained_models/Spark-TTS-0.5B`
- 参考音：`~/workspace/voice-reference/friend-voice-10s/ref.wav`（朋友女声，4.7 秒）
- 日志：服务目录下的 `requests.log`（每次合成一行：字数、合成秒数、音频秒数）、
  `direct-run.log`、`localhost-run.log`、`urlsync.log`

**systemd 单元（都在 `/etc/systemd/system/`，虚拟机换了就没了，见第 3 节）：**

| 单元 | 干什么 |
|---|---|
| `sparktts.service` | 语音服务本身，Restart=always |
| `sparktts-direct.service` | 直连隧道（主用），Restart=always |
| `sparktts-tunnel.service` | localhost.run 隧道（备胎），Restart=always |
| `sparktts-urlsync.service` | 备胎隧道地址同步到公共存储库，Restart=always |

**桥的代码在另一个仓库：**

- 仓库：私有 `sjj79888-ctrl/qq-bridge`，`main` 分支，`bridge.py`
- 服务器上的真本：`/home/ubuntu/qq-bridge/bridge.py`（**不是 git 仓库，
  绝不用 `git pull` 更新**，只按第 5 节用 OrcaTerm 上传）

## 2. 语音现在怎么走

- **主用：固定直连地址** `http://172.18.0.1:18001`（代码默认值，
  `.env` 里 `SPARKTTS_DIRECT_URL` 可覆盖，一般不用设）。
  虚拟机用 SSH 反向隧道把服务挂到服务器的 docker 网关地址上，
  桥在容器里就用这个固定地址访问，不再走公网随机地址。
- **备胎：公共隧道地址**，存在公共存储库 `sjj79888-ctrl/sparktts-url`
  （**必须保持公开**，`url.txt` 里只有地址、没有密钥）。
  桥启动时和出错时自己从
  `https://raw.githubusercontent.com/sjj79888-ctrl/sparktts-url/main/url.txt`
  取新地址；服务器 `.env` 里的 `SPARKTTS_URL` 只是取不到时的备用。
- **再往下回落**：直连失败 → 备胎隧道上重发一次任务 → 还失败 →
  edge-tts（Xiaoyi，妈妈模式 Xiaoxuan）→ 再失败 → 回文字。语音永远不丢回复。
- 服务接口（任务式，因为 CPU 合成要几十秒，同步等不到）：
  `POST /tts {"text": "..."}`（带 `X-Auth-Token` 头）→ 202 `{"job_id": ...}`，
  每 10 秒轮询 `GET /tts/<job_id>` 直到 200 返回音频（16kHz 单声道 wav），
  超时 300 秒。`/health` 不用鉴权。
- **自动语音规则（2026-11-08 Torres 定的）**：回 Torres 本人
  （1323841589，私聊和群里他 @ 她时）的每句话限 20 字以内，
  括号（...）里的字不算字数、也不会被念出来；说不完就按标点分成几段发，
  绝不在句中掐断，段与段之间固定等 5 秒；每段 ≤20 字自动发语音
  （朋友音色）代替文字，不用说“说句话”。朋友音色只对他一个人：
  回别人的照旧（不超 20 字也不发朋友音色、不限字数、不分段），
  妈妈模式仍是 edge-tts，女仆模式不发语音。
- 注意：桥是单线程的，一次 SparkTTS 轮询会卡住事件循环 30–50 秒
  （每多一段再加 5 秒），这段时间别的消息会排队，不是卡死了。

## 3. 启动、重启、恢复

**都在虚拟机上跑（要用 sudo，单元在 /etc 里）：**

```bash
# 看四个单元是不是都在跑
sudo systemctl status sparktts.service sparktts-direct.service sparktts-tunnel.service sparktts-urlsync.service

# 重启某一个（模型重载约 10 秒）
sudo systemctl restart sparktts.service          # 语音服务本身
sudo systemctl restart sparktts-direct.service   # 直连隧道（带自恢复，一般不用手动重启）
sudo systemctl restart sparktts-tunnel.service   # 备胎隧道

# 服务本身通不通（虚拟机上）
curl -s http://127.0.0.1:8001/health
# 预期：{"status":"ok","model_loaded":true,...}

# 备胎隧道现在是什么地址
cat ~/workspace/voice-reference/sparktts/service/tunnel_url.txt
curl -s https://raw.githubusercontent.com/sjj79888-ctrl/sparktts-url/main/url.txt
```

**直连隧道通不通（要从服务器侧测，见第 4 节怎么连服务器）：**

```bash
ssh <见第 4 节的完整命令> ubuntu@139.199.204.177 \
  "curl -s -m 10 http://172.18.0.1:18001/health"
# 预期：{"status":"ok","model_loaded":true}

# 同样在服务器 shell 里，看端口是不是在监听
ssh <同上> ubuntu@139.199.204.177 "ss -tln | grep 18001"
# 预期：172.18.0.1:18001 LISTEN
```

**直连隧道挂了怎么办：**

- 先等：`sparktts-direct.service` 有看门狗，服务器侧 `/health`
  连续 3 次失败（约 90 秒）就自己杀掉重连，重连后地址不变（固定地址），
  不用改任何配置。看 `direct-run.log` 里的重连记录确认。
- 还没好：`sudo systemctl restart sparktts-direct.service`，再测一次上面
  的服务器侧 `/health`。
- **虚拟机被换掉/重装之后**：`/etc` 下的单元全没了，代码、模型、参考音
  都在 `~/workspace/` 里还在。重跑一次就全回来：
  ```bash
  bash ~/workspace/voice-reference/sparktts/service/install.sh
  ```
  然后按上面测一遍四个单元 + `/health`。

## 4. 从虚拟机连服务器（SSH）

- 服务器：腾讯云 Lighthouse，广州，`139.199.204.177`，22 端口，用户 `ubuntu`。
- **必须走 egress 代理，直连会被 reset。**完整命令：
  ```bash
  ssh -i /home/hatch/.ssh/id_ed25519 \
    -o BatchMode=yes -o StrictHostKeyChecking=accept-new \
    -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
    -o ProxyCommand="nc -X connect -x hatch-egress-proxy:3128 %h %p" \
    ubuntu@139.199.204.177
  ```
  代理偶尔抖动一次，超时了重试一次就行。
- 公钥已经在服务器 `/home/ubuntu/.ssh/authorized_keys` 里了
  （2026-11-08 查过，只有一行）。**先检查再决定加不加，别重复加：**
  ```bash
  ssh <同上> ubuntu@139.199.204.177 "grep -c hatch ~/.ssh/authorized_keys"
  # 预期输出 1。输出 0 才需要追加公钥（走 OrcaTerm，见第 5 节）。
  ```
- 服务器 sshd 配置里已有 `GatewayPorts clientspecified`
  （2026-11-08 02:56 改的，只 reload、没重启），没有它反向隧道
  只能绑 `127.0.0.1`，桥容器（172.18.0.3）够不着。
  如果哪天被改回去了：OrcaTerm 上改 `/etc/ssh/sshd_config` 这一行，
  `sudo sshd -t` 校验，`sudo systemctl reload ssh`（只 reload 别 restart），
  再用 `sudo sshd -T | grep -i gatewayports` 确认生效。

## 5. 部署新桥（服务器操作）

> 服务器 `/home/ubuntu/qq-bridge` **不是 git 仓库**，更新只走 OrcaTerm 上传，
> 绝不 `git pull`。每次部署只重建 `bridge` 容器，绝不动 SnowLuma/NapCat。

1. 腾讯云 Lighthouse 控制台进 OrcaTerm（TAT 免密登录，用户 ubuntu）。
   **登录时可能弹出微信扫码验证——只有 Torres 能扫，出现就停下问他，
   不要自己处理。**
2. **OrcaTerm 终端一次只发一行命令**（输入框会吞换行，多行粘贴会被拼成一行报错）。
3. 先核对现版本，再备份（日期和事由写进备份名）：
   ```bash
   cd /home/ubuntu/qq-bridge
   md5sum bridge.py
   cp bridge.py bridge.py.bak-20261108-<事由>
   md5sum bridge.py.bak-20261108-<事由>
   ```
   两次 md5 必须一致；和要换的版本 md5 不一样才继续，不一样就停下查。
4. OrcaTerm 文件管理器上传新 `bridge.py`，再挪过去，校验 + 编译：
   ```bash
   md5sum bridge.py
   python3 -m py_compile bridge.py
   ```
   md5 必须和要部署的版本一致，编译必须通过。
5. **只重建 bridge**（两个 compose 文件都要，缺一个会报错）：
   ```bash
   sudo docker compose -f docker-compose.yml -f docker-compose.bridge.yml up -d --build --no-deps bridge
   ```
   `restart` 不会加载新镜像，必须是上面这条重建命令。
6. 验证：
   ```bash
   sudo docker ps
   sudo docker logs --tail 30 qq-bridge
   ```
   `qq-bridge` 和 `snowluma` 都要是 Up；日志里 grep
   `error|traceback|exception|failed` 应该没有命中（`sparktts poll failed`
   除外——那是语音回落的正常记录，见第 6 节）；启动行应有：
   `[bridge] sparktts on: direct http://172.18.0.1:18001 (primary), tunnel https://... (fallback) (short replies <= 20 chars, poll timeout 300s)`
   看不到这行（只看到 `sparktts off: ...`）说明 token 或地址没配好。
7. `.env` 里的语音相关键（只在服务器 `.env` 里，绝不进仓库）：
   `SPARKTTS_TOKEN`（43 个字符）、`SPARKTTS_URL`（备胎隧道地址）、
   `SPARKTTS_DIRECT_URL`（一般不设，用代码默认值 `http://172.18.0.1:18001`）。
   部署新桥一般不用改 `.env`。
8. 部署完让他本人（1323841589）在 QQ 说句短话真测一遍；
   其他 QQ 测试消息一律要他点头才发。

## 6. 查问题：日志怎么看

```bash
# 服务器 OrcaTerm 里
sudo docker logs --tail 150 qq-bridge
```

关键行（桥的日志没有时间戳，按事件顺序找）：

| 日志行 | 意思 |
|---|---|
| `auto-voice: sender=1323841589 persona=xiaoman counted=19 segments=1` | 自动语音触发了：发消息的是 Torres，小满人格，19 个字，1 段 |
| `sparktts url updated: <旧> -> <新>` | 桥从公共存储库取到了新隧道地址（正常） |
| `sparktts ok: job <id> via <地址>, <n> chars, <s>s, <b> bytes` | SparkTTS 合成成功了，看 `via` 知道走的是直连还是备胎 |
| `sparktts poll failed: job <id> via <地址> HTTP 502` | 任务建好了，但轮询时隧道断了（502 "no tunnel"）→ 回落 |
| `sparktts direct failed, trying fallback tunnel` | 直连失败，正在备胎隧道上重发任务 |
| `auto-voice segment 1: sparktts fallback to edge-tts` | 这段 SparkTTS 没走通，回落到 edge-tts |
| `auto-voice segment 1/1: voice sent (spark_ok=True)` | 发出了一条语音。**注意：`spark_ok=True` 只表示这段适合走语音，不表示用了 SparkTTS**，看上面几行才知道是哪个引擎 |
| `sparktts timeout: job <id> via <地址>, <n> chars` | 300 秒还没出音频，回落（长句容易这样） |

常见定位：

- 她回的不是朋友音色 → 先找这句回复对应的 `auto-voice:` 行，
  再看后面是 `sparktts ok:`（成功了，但音色不对？那就是参考音/模型问题）
  还是 `sparktts poll failed` / `direct failed`（隧道问题，走备胎或兜底去了）。
- 隧道中途 502 是最常见的失败：localhost.run 边缘节点自己断，
  一晚断过 9 次。直连隧道在的话，桥会先走直连，不受它影响。
- 服务器 shell 直接 `curl` 公共存储库偶尔会超时（2026-11-08 02:27
  连超时 3 次），是瞬时的，桥自己取地址一般能成，不用因此改配置。
- `BrokenPipeError`（172.18.0.2 断开、do_POST 写失败）是桥自身 8787
  端口的客户端断连噪音，和语音无关。

## 7. 当前状态（写于 2026-11-08 03:05 左右，以实际部署结果为准）

- 线上桥：md5 `a2500f01`（提交 057ca0e1，自动语音版，02:27 部署）。
- 直连版桥：md5 `9b22563d`（提交 8336e955），实现完成 03:02
  （SSH 登录通、直连 `/health` 通、全流程合成测过、重启自恢复测过、
  零断开），**部署进行中——不要写成已上线**，部署完按第 5 节第 6 步
  核对启动行，再让他真测一句。
- 首次真测 02:43 回落到了 edge-tts（隧道中途 502），等他再说一句短话确认。
- 还开着、没堵事：朋友本人是否同意用她的声音、原消息的准确文字稿
  （问过、他还没回；合成不需要文字稿）。

## 8. 硬红线（没他点头 + 底层没变，别重试）

- 服务器上永远不装重东西：Genie 实测峰值 5.4GB，2G 试用机和 4G 升级
  都装不下，仍搁置；SparkTTS 峰值 3.3–5.26GB，同样装不上服务器，
  只能放虚拟机上跑。部署重东西之前必须他先听结果、再点头。
- Fish Audio 从服务器连不上，是死路；INT8 量化试过失败，是死路；
  唱歌先等等（他 2026-11-07 23:20 说的），别去训 RVC。
- 密钥、token、密码绝不进聊天和任何仓库。
- 服务器 2026-12-04 00:02:55 试用到期，到期前定续费或迁移的办法，
  到时候一次性问他，别天天催。
