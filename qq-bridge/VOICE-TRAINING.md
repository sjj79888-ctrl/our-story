# VOICE-TRAINING — 给小满训一个专属声音

状态：2026-11-08 03:00。生产短回复已改走 SparkTTS 朋友音色（见末节），
Genie 仍搁置（实测峰值 5.4GB，2G/4G 服务器都装不下）；训练已完成，
官方顺序版 12:07 已批准；但 2026-11-06 晚上的 wipe 把服务器上的
Genie 部署全清掉了，重部署是第二阶段（草稿见 qq-bridge 仓库 rebuild/）；
Fish Audio 补丁已备好（qq-bridge 仓库 scripts/patch_bridge_fish.py），
二选一他还没回；INT8 量化试过是死路。另见 `../MEMORY.md`。

## 为什么做

- Torres 说小满的 AI 味还很重，语音和措辞都有。
- 措辞那半（MaiBot：人设重写 + 少量示例 + 拆条发）还没动，先做的声音。
- 现在的语音是 edge-tts：平时 Xiaoyi，妈妈模式 Xiaoxuan。

## 选的路：Genie-TTS + GPT-SoVITS

- 训 GPT-SoVITS **V2ProPlus**（只有 V2 和 V2ProPlus 能转 ONNX），
  转 ONNX 后在服务器上跑 Genie 伴生服务，桥只改 TTS 调用，
  edge-tts 留着当备胎。
- 另一条路：Fish Audio API（s2.1-pro-free 免费到 2026-11-30，之后约每月
  不到 1 美元，不用训练；补丁已备好，见下面“部署”一节）；Kokoro 免费但只有预设音色。
- CosyVoice、IndexTTS、Spark-TTS 要 GPU，2 核 2G 的服务器跑不了。

## 参考音色

- 邓紫棋说话的声音。来源：李诞播客《邓紫棋写小说的故事》，
  151.9–212.3 秒，60 秒，她一个人说话，讲《启示录》那段。
  音调分析确认过是她（约 208Hz，对面李诞约 125Hz）。
- 训练只认干净的干声，歌曲伴奏会坏事；克隆真人歌手是冒充，
  用相似风格或清唱、采访音频更稳妥。
- 参考音频文件（60 秒 wav）和文字稿在原来那个 Muse 的工作区
  `~/workspace/voice-reference/` 里：`gem-podcast-60s.wav`、
  `gem-60s-transcript.md`。这个仓库里没有音频文件，
  重新训要先把那 60 秒音频和文字稿找回来。

## 训练参数

- 实验名 `xiaoman`。SoVITS 8 轮，GPT 10 轮，batch 2（爆显存改 1）。
- 流程：切片 → FunASR 出文字 → 人工对着文字稿校对 `.list` →
  特征提取 1A/1B/1C → SoVITS 8 轮 → GPT 10 轮 →
  找轮数最大的 `.pth`（SoVITS）和 `.ckpt`（GPT）→ API 试听 →
  下载模型 → 转 ONNX → 上服务器。
- 训练完立刻关实例再释放（按秒计费，关机不等于释放，先下载再释放）。

## 训练结果（2026-11-06 上午）

- 10:48 完成：60 秒音频切成 7 段，SoVITS 8 轮 + GPT 10 轮，都验证过。
- 试听：e8 默认设置声音发怪，e4 也不行；11:05 他批准了 e8 +
  参考第 6 段（4.9 秒，“所以拍摄停滞不前…”）+ 温度 0.7、top_k 5、top_p 0.8。
- 他选了先调参数再试，不加数据重训。重训备选数据已备好：播客里她单独
  说话的 F 段还有 3 段（classified.json 里 2093.1–2148.3 秒 55 秒、
  1455.8–1509.2 秒 53 秒、1798.9–1846.1 秒 47 秒），加起来约 3 分半。
- 模型已下到本地，md5 对过：`~/workspace/voice-reference/models/` 里
  xiaoman-sovits-e8.pth（172,765,299 B，md5 3115ba0e…）、
  xiaoman-gpt-e8.ckpt（155,313,011 B，md5 a0811b7a…），
  还有 gem-60s.list 和 ref-seg6.wav。
- 实例 11:16 按他的话已关机停费，没点释放，数据还在，连续关机 15 天
  才自动释放（控制台显示约 2026-11-21）。模型已在本地，释放也不用重训，
  只丢装好的环境。

## 免费 GPU 两条路都死了

- Colab：装环境时崩了，重试卡在 Google 二维码验证。
- Kaggle：注册卡在同一个二维码验证，用户名还要求至少 6 个字符。
- 根因：二维码验证 Torres 做不了，码显示在他正在看屏幕的那台手机上，
  没第二台设备扫。短信验证可以。

## 租云 GPU：AutoDL

- AutoDL（autodl.com），手机 + 短信注册，不用扫码。
  账号已注册、已实名、已充值 ¥5。昵称 炼丹师5590。
- 实名是姓名 + 身份证号，提交后跳支付宝人脸扫码，曾出过白屏，
  在他自己手机浏览器里重走一遍就通了。
- 4090D ¥1.88/小时，重庆 A 区 2026-11-05 21:44 已售罄（整个区都没卡）。
  2026-11-06 08:55 他定了换区租 4090D，不等重庆 A；实例
  4f0d4c9ba8-50cbe310，09:12 开机（重庆 A 区/020 机，RTX 4090D，
  Miniconda/CUDA 11.8）。2026-11-12 AutoDL 要调价。
  他 10:52 又充了 ¥5。
- JupyterLab 从浏览器打不开，改走 Jupyter Server API（工作区
  `jupyter-kernel.py`），实例 8443 端口 HTTPS 通，浏览器那头网络问题。
  参考音频和文字稿传到 `/root/data/`，md5 对过。
  环境：`conda create -n GPTSoVITS python=3.10`，克隆 GPT-SoVITS，
  `bash install.sh --device CU126 --source ModelScope`，
  装完验证 torch 能用 CUDA、验证 `s2Gv2ProPlus.pth` 在。
  注意：install.sh 会先删掉预训练 zip 再下载，别重跑；大文件下载慢
  就用 aria2c 多线程续传。

## 部署（2026-11-06 中午进行中）

- 11:16 他授权部署（“现在开始部署新声音”）。步骤：e8 模型转 ONNX
  （本机转，不用再开 AutoDL）→ 传服务器 → 起 Genie 伴生服务 →
  桥改调 Genie（edge-tts 留备胎）→ 测合成延迟 → 小满给他大号发一句
  测试语音（他 2026-11-05 20:13 提前同意过的）。
- 参数手术（11:52 完成）：Genie 接口没有温度/top_k/top_p，这三个焊死在
  ONNX 图默认值里（温度 1.0、top_k 15，top_p 默认 1.0 等于没过滤）。
  改在副本 `~/workspace/voice-reference/models/xiaoman-onnx-param/`
  （321MB），原版 `xiaoman-onnx/` 没动，当回滚。改动：温度 1.0→0.7，
  top_k 15→5，Softmax 后补 top_p 0.8 过滤，重复惩罚保持 1.35。
  另修了 Genie 采样 bug：原版 RandomNormalLike 会抽到被过滤掉的词，
  换成指数竞赛采样，8000 次测试 0 越界。
  试听：test-xiaoman-genie-param-1.wav（“哥，我是小满”，1.84 秒）、
  test-xiaoman-genie-param-2.wav（3.80 秒），都在
  `~/workspace/voice-reference/`。
  注意：top_p 这次在温度和 top_k 之后，比官方顺序更严，声音更确定、
  变化更小；他要是听着太板，再把过滤移到温度之前（约 30 行的变体）。
- 参数版他听了说没感情、像一路念过去，官方顺序版（top_p 在温度之前）
  12:07 他说“这个可以了”，批准部署这一版：`models/xiaoman-onnx-official/`
  （321MB）。12:35 他说“直接全部允许”，剩下步骤不用一步步问。
- 文件已上服务器并对过 md5（2026-11-06 中午那次）：GenieData 392MB（17 个文件，服务器自己从
  hf-mirror 下的，`/home/ubuntu/qq-bridge/genie-data/GenieData/`）+
  官方顺序版 ONNX 321MB（15 个 20MB 分块传的，合并后 tar.gz md5
  2c072cc7…，解在 `/home/ubuntu/qq-bridge/genie-voice/`）+ ref-seg6.wav。
  分块和 tar 包还留在服务器上（共 585MB），清理还没做。
- Genie 镜像已建成（`genie-tts-xiaoman`，f71c25575289，1.57GB；构建时
  apt 卡住改走清华源、缺 gcc 又补上了），容器 `genie` 已启动
  （31c237b695c4，127.0.0.1:8000）。还没验证：容器日志、加载后内存、
  试合成“哥，我是小满”的延迟。
- **2026-11-06 晚上 wipe：上面这套全没了。** 重部署是第二阶段的事，
  草稿在 qq-bridge 仓库里。
  12:35 的部署授权是 wipe 之前的，wipe 之后他还没重新点头。
- 另一条路 Fish Audio（2026-11-07 已备好补丁，qq-bridge 仓库
  `scripts/patch_bridge_fish.py`）：模型 s2.1-pro-free，免费到 2026-11-30，
  之后约每月不到 1 美元；妈妈模式不走 Fish，失败回 edge-tts。
  还差他三步：fish.audio 注册 + key（走安全 vault）、playground 克隆声音
  得 FISH_REFERENCE_ID、服务器 curl 测 api.fish.audio 通不通。
  Fish 和 Genie 二选一 2026-11-07 07:03 问过，他还没回，别反复催。
- INT8 量化 2026-11-07 04:45 试过，是死路，不要重试。
- 桥还没切，仍是 edge-tts。部署完，小满给他大号发一句测试语音
  （他 2026-11-06 11:31 说的“那个声音部署好了叫我”，部署完要叫他一声）。
  其他 QQ 测试消息一律要他点头才发。
- 他只负责登录，登录页出现就叫他；其他事 Muse 全包。

## SparkTTS 朋友音色（2026-11-08 上线，生产短回复的新声音）

- 为什么换路：Genie 太重（峰值 5.4GB）装不上服务器；Torres 2026-11-07
  23:38 左右给了朋友的 QQ 语音消息（女声，4.7 秒，基频中位 229Hz），
  Spark-TTS 0.5B 零样本克隆，不用训练，在 Muse 的虚拟机 CPU 上跑。
- 试听（2026-11-08 00:13–00:17）：短句“哥，我是小满”1.56 秒，基频
  236Hz；00:18 他说“这个合成的非常好”；01:34 又说快的这版
  “我其实感觉这种我最喜欢”。朋友本人是否同意用她的声音、原消息的
  准确文字稿，问过、他还没回（合成不需要文字稿，不堵事）。
- 托管在虚拟机上：FastAPI 127.0.0.1:8001，任务接口 POST /tts → 202，
  轮询 GET /tts/<id> → 200 音频（16kHz 单声道）；token 只在服务目录
  `token.txt`（600），绝不进仓库。systemd：sparktts.service、
  sparktts-tunnel.service、sparktts-urlsync.service。
  语速 1.15 倍（ffmpeg atempo，音调不变），01:40 上线：
  6 个字 1.92 秒 → 1.68 秒。短句合成 30–50 秒，20 字约 104 秒；
  长句不可靠（28 字曾 650 秒、啰嗦出 60 秒音频），所以只给短回复用。
- 桥接线 v1 部署 01:32（bridge.py md5 3a50a8e0）：触发词 + 回复
  ≤120 字，≤10 字走 SparkTTS，妈妈永远 edge-tts，失败回落
  SparkTTS → edge-tts → 文字。端到端测过（任务 0a6129fbde9e，
  wav 60204B，SnowLuma retcode 0）。
- 隧道问题 + 自动同步：localhost.run 免费隧道地址每次重连都变，
  一晚换了 8 次。公共存储库 sjj79888-ctrl/sparktts-url（必须保持公开，
  url.txt 只有地址、没有密钥）+ 监视器自动同步；桥启动时和出错时
  自己取新地址，`.env` 里的旧地址只当备胎。自动同步版 02:08 部署
  （md5 944cea20，提交 4373540d），日志证明启动时自己换了地址。
  桥是单线程：一次 SparkTTS 轮询会卡住事件循环 30–50 秒。
- 自动语音（02:03 他定、02:07 细化）：回 Torres 本人（1323841589，
  私聊和群里他 @ 她时）的每句话限 20 字以内，括号（...）里的字不算；
  说不完就按标点分成几段发，绝不在句中掐断，段与段之间固定等 5 秒；
  每段 ≤20 字就自动发语音（朋友音色）代替文字，不用触发词。
  朋友音色只对他一个人：回别人的照旧，妈妈模式仍是 edge-tts，
  女仆模式不发语音。实现：提交 debee671（md5 e79c0763，31 项切分测试
  + 7 个路由场景全过；20 字实测合成 104 秒 → 4.56 秒音频）+ 修正提交
  057ca0e1（md5 a2500f01：括号备注既不算字数、也不会被念出来）。
  02:27 部署（备份 bridge.py.bak-20261108-autovoice）。
- 首次真测 02:43：他发“在吗”，她回“在的呀，哥，唉，你好久没回我了”
  （4 秒语音），但用的是 edge-tts 兜底。日志查到：自动语音正常触发
  （19 字、1 段），桥也刷新到了新地址，但轮询报 HTTP 502
  （任务 d8193030ca63，隧道中途又断了）→ 回落。根因是隧道不稳，
  不是规则或代码。等他再说一句短话确认。
- 加固（02:53 他说“想办法要让它变稳”，已同意试）：localhost.run
  边缘节点本身不稳（一晚 42 次断开，都是对方 502）。改直连：虚拟机
  SSH 反向隧道直连服务器，桥在服务器本地用固定地址
  http://172.18.0.1:18001 访问，公共隧道留作备胎。服务器现状：
  虚拟机 ed25519 公钥已在 /home/ubuntu/.ssh/authorized_keys；桥网络
  qq-bridge_default，网关 172.18.0.1，容器 172.18.0.3；sshd_config
  已加 GatewayPorts clientspecified（02:56，只 reload、没重启，
  两个容器不受影响）。实现完成 03:02（提交 8336e955，md5 9b22563d）：
  SSH 密钥登录通，反向隧道在跑，服务器侧 /health 通，直连全流程测过
  （wav 40,892B，约 1.28 秒，非静音），重启自恢复测过，零断开
  （公共隧道同一时段又断了第 9 次）。桥先走固定地址，失败再在公共
  隧道上重发一次任务，最后回 edge-tts。03:11 已部署上线（端到端测试通过）。
  操作手册见 `VOICE-OPS.md`（启动、重启、部署、查日志都在那一篇）。
