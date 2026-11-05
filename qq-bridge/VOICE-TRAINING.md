# VOICE-TRAINING — 给小满训一个专属声音

状态：2026-11-05 深夜，参考音频已备好，训练还没开始，卡在租 GPU。

## 为什么做

- Torres 说小满的 AI 味还很重，语音和措辞都有。
- 措辞那半（MaiBot：人设重写 + 少量示例 + 拆条发）还没动，先做的声音。
- 现在的语音是 edge-tts：平时 Xiaoyi，妈妈模式 Xiaoxuan。

## 选的路：Genie-TTS + GPT-SoVITS

- 训 GPT-SoVITS **V2ProPlus**（只有 V2 和 V2ProPlus 能转 ONNX），
  转 ONNX 后在服务器上跑 Genie 伴生服务，桥只改 TTS 调用，
  edge-tts 留着当备胎。
- 备胎：Fish Audio API（约 $3/月，不用训练）；Kokoro 免费但只有预设音色。
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
- 4090D ¥1.88/小时，重庆 A 区 2026-11-05 21:44 已售罄（整个区都没卡），
  还没租到实例，钱一分没花。三个选择还没定：
  换区租 4090D、等重庆 A 区、或换 3090（¥1.20/小时，训练慢约半小时，
  一次约 ¥2.40）。2026-11-12 AutoDL 要调价，调价前下单更划算。
- 2026-11-05 21:49 他叫停，先做女仆人设。租卡的决定等他回来再做。
- 租到之后：重庆 A 区优先，4090D，按量计费，Miniconda 镜像，
  浏览器里开 JupyterLab，不用 SSH。参考音频和文字稿传到 `/root/data/`。
  环境：`conda create -n GPTSoVITS python=3.10`，克隆 GPT-SoVITS，
  `bash install.sh --device CU126 --source ModelScope`，
  装完验证 torch 能用 CUDA、验证 `s2Gv2ProPlus.pth` 在。

## 部署之后

- 部署完，小满给他大号发一句测试语音，是他 2026-11-05 20:13
  提前同意过的。其他 QQ 测试消息一律要他点头才发。
- 他只负责登录，登录页出现就叫他；其他事 Muse 全包。
