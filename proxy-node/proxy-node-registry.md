# 代理节点总登记表 + 扫描日志

> 手册：`~/memory/proxy-node-playbook.md`（Torres 2026-11-07 存入）。本表按手册第 6 节维护：每次交付 / 判死 / 新开账号当场更新。

## 进实测候选（第 2 轮，2026-11-07 15:59 CST 筛出）

| 平台 | 五关结论 | 备注 |
|---|---|---|
| switched.cloud（法国 noprofit.space，Scaleway/Dedibox 机房） | 免费关：免卡，首 3 天免费，之后 Cloud Token 续期（做 Linkvertise 链接赚取，1 token=1 天，可提前续 6 天）；能力关：root VPS（LXC 容器 / KVM），Debian 13/Alpine，Docker 能跑；入口关：共享 IPv4 上 5 个 TCP/UDP 端口 + 独立 IPv6 全端口；条款关：**条款第 8 节原文 "Personal proxies, VPNs, and tunnels are allowed"**，只禁挖矿/DDoS/扫描/垃圾邮件/钓鱼/C2 等；门槛关：noprofit.space 账号登录（用户名/密码/passkey/Discord），未见要手机号/实名 | 待实测核实三点：① 注册是否要邮箱验证（条款称登录不用邮箱）；② Linkvertise 赚 token 是否要人机验证、自动化能不能过（过不去则按手册判死）；③ 出站带宽上限未公布，xray 流量够不够用。另：共享 IPv4 + 端口转发形态，VLESS+WS+TLS 需挂他自己的域名（订阅链接不暴露真实地址）；VPS 过期 3 天后删数据，需自备备份/重建脚本。 |

## 在跑节点

无。

## 平台账号

无。

## 身份资产

无（未新建任何一次性账号；未用任何邮箱注册）。

## 已出局平台及死因

| 平台 | 死因（一句话） | 核查时间 | 依据 |
|---|---|---|---|
| blitz.cloud（德国，2026-09-26 出 beta） | 条款第 8 节点名禁 Xray/V2Ray/sing-box，自动检查发现即暂停应用 + 封免费账号 | 2026-11-07 14:59 CST | 条款原文 https://blitz.cloud/terms/ L253、L262-263；免费 0 欧元免卡、5 应用/512MB、无流量限额见 https://blitz.cloud/faq/ |
| SnapDeploy（AWS Fargate） | 免费档明确不通 WebSocket，VLESS+WS 没法跑；另 100 小时/月、脚本唤不醒 | 2026-11-07 14:57 CST | 官网免费页 https://snapdeploy.dev/free-container-hosting L103、L55、L26 |
| Koyeb | 免费 Starter 已不对新用户开放（Mistral 收购后），新账号 Pro $29/月起 + 要卡 | 2026-11-07 14:57 CST | Koyeb 自身 FAQ 原话（多家 2026-09 独立核查引用）+ 定价页只列付费档；旧攻略说的免费档已过期 |
| Hugging Face Spaces | Docker Space 建号就要 PRO $9/月（2026-08 起），免费只剩 Gradio ZeroGPU 每天 5 分钟 | 2026-11-07 14:57 CST | Hub 文档原话，多家 2026-09/11 核查一致 |
| Render | 免费 web service 也要绑卡 | 2026-11-07 14:57 CST | 2026 年实测对照（flywp / xcloud） |
| Fly.io | 新组织已无免费档，只剩试用 | 2026-11-07 14:57 CST | 2026 年实测对照 |
| Railway | 免费只剩 $1/月额度，约只够跑 6 天，跑不满月 | 2026-11-07 14:57 CST | 2026 年实测对照 |
| Oracle / Google 免费 VM | 要绑卡，违反免卡硬线 | 2026-11-07 14:57 CST | 2026 年实测对照 |
| alwaysdata（法国） | 免费档明文禁 24/7 无人值守程序（help 页原话 "hosting generic programs that need to run 24/7 without any user interaction"），且免费只能用 *.alwaysdata.net 域名、不能挂自己域名 | 2026-11-07 15:58 CST | https://help.alwaysdata.com/en/docs/admin-billing/profile/suspension/；免费 1GB/256MB/四分之一 CPU、SSH 本来都符合，死在 24/7 常驻这一条 |
| HostPeppy 免费 VPS 试用 | 只有 72 小时，不足约 7 天硬线 | 2026-11-07 15:57 CST | https://hostpeppy.com/free-vps-hosting-trial |
| Azure for Students | 只给全日制大学生（学校邮箱验证），无证据表明 Torres 符合该身份 | 2026-11-07 15:57 CST | https://alexwebhosting.com/free-vps/（2026-11-02 核查） |
| AccuWebHosting 试用 | 要身份验证，且 30 天后按 $14.97/月收费，不是免费档 | 2026-11-07 15:57 CST | 同上 |
| EFN（Experimental Free Network） | 免费壳/Docker 名额 2026-07-26 已满 300 人，之后免费只留给捐赠者，另有区域注册限制 | 2026-11-07 15:57 CST | https://shells.efn.lt/start（2026-07-26 公告） |

## 已知缺口

- 第一轮筛完，**没有平台能进实测**。免费容器 PaaS 这条线在 2026-11 已基本走死：要卡、关免费档、明文禁代理、免费档不通 WS，四种死法占全了。
- 第二轮（2026-11-07 15:57–15:59 CST）换了变量类别，按小众 SSH/VPS 型筛：**switched.cloud 进实测候选**（见上表），alwaysdata 等 5 个新名字判死。GitHub 线仍单独立项、未动。
- 待核查（未判死也未进实测）：Serv00（波兰，免费 3GB/SSH/3 个端口/512MB，看着符合，但条款/AUP 原文搜不到，需直接读条款再判）；domcloud.co（免费 1GB 存储 + 1GB 流量/月，流量太小，且要走 GitHub，归 GitHub 线）。
- blitz.cloud 的教训：免费、免卡、新平台三样全占也可能条款点名禁 Xray——以后筛查必须先读条款 acceptable use，全文搜 proxy/vpn/tunnel/Xray，再看免费档。switched.cloud 是反例：条款明文允许个人代理。

## 扫描日志

- 2026-11-07 第 1 轮：搜免费容器 PaaS（snapdeploy/xcloud/flywp 对照 + Koyeb/HF 专项核查），读 blitz.cloud FAQ 与条款全文、SnapDeploy 免费页；新名字 blitz.cloud、SnapDeploy；死因见上表；结论：无进实测候选。
- 2026-11-07 第 2 轮：换变量类别，搜小众 SSH/VPS 型免费主机（free VPS no card / Serv00 / free shell / alwaysdata 四路）；读 switched.cloud 首页 + noprofit.space 条款全文、alwaysdata 停用页、EFN 公告；新名字 switched.cloud、Serv00、alwaysdata、EFN、HostPeppy、AccuWebHosting；结论：switched.cloud 进实测候选，alwaysdata 等 5 个判死，Serv00 待读条款。
