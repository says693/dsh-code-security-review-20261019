# dsh-code-security 0.5.0：交付复现复核 v2

2026-10-19，Windows x64。本次从新解压的修订包、新源码 checkout、新 DSH_HOME/profile 和新浏览器 context 完成定向复现。结果：内置浏览器断言 **20/20**，原生浏览器断言 **18/18**，重启后 **2 个工作区**的设置与修复报告保持一致。

本次发现并修复的是我此前交付材料中的复现问题；没有发现新的插件产品缺陷。本机复用现有工具和缓存，不冒充另一人、另一台机器或独立安装全套依赖的验证。

## 一、锚定与输入

- 产品：STARDUSTLC666/dsh-code-security，npm 0.5.0。
- 提交：`0f2c70b14294460ad07f80ad754d0464765e4dbd`，2026-10-17 23:15:35 +08:00。
- npm tgz SHA256：`6da03b715e7ed4b16616842c7c6b07b5bb5dedaf1c64261a53ad1c90363c22c0`。
- 原始公开包 SHA256：`33dbec8db47e9b40dfc1ea9f24dbc0dc4b40161f81baa2c916413136962ed0d3`；原包及原记录保持不变。108 个原包清单条目已逐个校验。
- 修订复跑包 `REPRO_KIT_v2.zip` SHA256：`55913e58f3bf39aa6c6a5406ff088e2b5d7b9443876a6cb5f2eb66994495cfca`。
- Node 24.19.0、DSH 0.2.0-rc.2、pnpm 11.7.0、Playwright 1.64.1、系统 Chrome。
- Betterleaks 1.9.0、Semgrep CE 1.181.0、OSV-Scanner 2.7.0。复用原来已校验的实际可执行文件；Semgrep 保持原 venv 路径。
- 从原本机 Git 仓库新 clone 后检出固定提交，冻结锁文件并离线安装依赖；没有复制原 profile、catalog、扫描状态或浏览器数据。

## 二、原交付包的问题与修正

| 项 | 原始观察 | v2 修正 | 证据性质 |
|---|---|---|---|
| evidence 目录缺失 | 原导出 record.py 直接运行退出 1，FileNotFoundError | 记录器主动建目录；记录名冲突时拒绝覆盖 | 实际执行，delivery-audit/original-recorder.stderr.txt |
| 脱敏改坏正则 | `/token=.*/` 被改成 `/token=<REDACTED>*/`，合成测试串中的 token 未被移除 | host 只输出 URL origin；源码逐字节打包，不做文本脱敏 | 合成串实际验证，无真实令牌，delivery-audit/original-redaction-probe.stdout.txt |
| 环境路径和状态文件名固定 | CLI/Chrome 路径是占位符，UI 仍读 installed-host 而说明启动 fresh-host | 所有本机值在 config.local.json；统一运行根、profile 和 host_tag | 静态对照＋修订包实际执行 |
| 缺少配置与准备步骤 | run-env.json、目录准备、源码目录返回步骤不完整 | reproduce.py 显式准备、设置 cwd、校验版本/哈希并执行 | 静态对照＋修订包实际执行 |
| 首次引导等待状态错误 | 第一次尝试等待 disabled 的“继续”按钮；第二次在两弹窗过渡间隙提前点设置；两次均超时，0 项产品断言执行 | 显式完成欢迎页→API Key“稍后配置”→弹窗消失的状态流程，再打开设置 | 两次失败日志、截图保留于 delivery-audit/attempt1/ 和 attempt2/ |

两次修订尝试分别使用 acceptance-v2、acceptance-v2-final，原始结果均为失败，已保留。最终重新解压到 acceptance-v2-verified；本报告的成功读数仅来自后者。没有在失败目录补跑后混写成首次成功。

## 三、新目录实际读数

| 验证面 | 实测 | 公开证据 |
|---|---|---|
| 包与固定提交 | 50 个文件一致；package.json 语义比较，其余 Git blob 字节比较 | fresh-run/package-integrity.stdout.txt |
| 无插件基线 | HTTP 404 | fresh-run/baseline-404.json |
| 安装后空 profile | HTTP 200，空工作区 | fresh-run/baseline-200.json |
| 内置浏览器流程 | 20/20 | fresh-run/fresh-builtin-checks.json |
| 原生引擎浏览器流程 | 18/18 | fresh-run/fresh-native-checks.json |
| 重启持久化 | 2 个工作区，settings/reports 不变 | fresh-run/restart-result.json |
| 记录完整性 | 24 个命令记录退出 0；各记录覆盖的日常配置前后哈希不变 | fresh-run/REPRO_RESULT.json 与各命令 .json |
| 驱动交付一致性 | 验收运行脚本与最终包脚本逐字节一致 | reproduce-kit/MANIFEST.json；本机 DELIVERY_VALIDATION.json |

内置流程包含 SEC-001/SEC-304 检出、真实浏览器下载 SARIF/Markdown、合成令牌脱敏与修复复查。原生流程实际调用三个引擎，检出五个预期项；使 Semgrep 路径失效时 complete=false、passed=false、closedCount=0 且原报告保留；恢复路径并修复后关闭五项。离线 OSV 用合成 DSH-TEST-0001 公告验证 1.0.0→1.1.0，以及缺库和损坏锁文件不误判为通过。

未重跑上游完整 79 项套件。此前 79 pass / 0 fail / 0 skipped 仅是第一轮的历史读数，放在 historical-v1/。本轮 38 项是同一组浏览器断言在新状态下的重复执行，不与历史次数相加为“新增覆盖”。

## 四、复跑入口与交付

解压公开包内的 REPRO_KIT_v2.zip 到新目录，按 REPRODUCE.md 填 config.local.json，执行：

```powershell
python -X utf8 .\reproduce.py
```

不需要改脚本正文。每个阶段有原始命令、cwd、退出码和 stdout/stderr；失败立即终止，保留证据，随后需换新目录再跑。源码、依赖准备、引擎版本和本机环境值均显式声明。

本机完整原始记录在 acceptance-v2-verified/evidence/；公开副本在 fresh-run/。公开包不含真实宿主令牌、本机配置、原始 DOM、未遮罩截图或虚拟环境。截图在浏览器采集时遮罩路径字段，文本脱敏不作用于可执行脚本。

## 五、范围与贡献边界

- **已验**：上述固定版本在 Windows/Chrome 的安装、内置和原生扫描、修复、异常路径、导出和重启；修订包在空状态可执行。
- **未验**：Linux/WSL、桌面原生窗口、另一台电脑从网络全新安装依赖、在线 OSV、真实模型自动登记/自动扫描、生产项目检出准确率、性能和费用。
- **不确定**：维护者是否认定本件有增量贡献，以及网站与既有 issue 展示差异的原因。
- **不做**：将自有交付脚本问题报成插件漏洞，宣称首次评测、总分或排名；发送 issue、推送、外部消息或费用支出。

项目登记使用发布包 SecurityWorkbench.remember()，属于测试准备；未伪称为真实模型会话。所有目标是自建合成项目，依赖公告也是合成数据。

查重仍引用 2026-10-19 19:01（UTC+8）的既有记录：悬赏仓 #984、#1001，上游 #1 与作者 VALIDATION.md。此次未重新在线检索，不宣称查重结果覆盖此后的变化。若准备公开提交，需要刷新查重并由维护者判定是否合并。本轮依据是已保存的工作流文档，Git blob `15658d890c80e559c4f41918171945ffe27dd874`。


## 公开提交前更新（2026-10-19）

2026-10-19 19:25:58（UTC+8）重新查重：插件关键词搜索仍为 6 条，incomplete_results=false，
当前账号此前同插件申报 0 条；重新读取的工作流文档 Git blob 仍为
`15658d890c80e559c4f41918171945ffe27dd874`。对照 #984/#1001 未见同范围真实宿主原生流程复核。

用户随后明确授权直接提交。历史“本轮不提交”描述的是测试阶段；本次将公开证据上传至
[证据 Release](https://github.com/says693/dsh-code-security-review-20261019/releases/tag/review-0.5.0-20261019)，并向 dsh-bounty 提交 intake。无需以人工点击代替已完成的自动验证；
维护者仍需人工判断是否计入贡献。ZIP 中原报告保留提交前状态，没有改写历史证据。
