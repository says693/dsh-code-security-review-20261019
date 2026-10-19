# dsh-code-security 0.5.0：Windows 独立复核证据

本仓只发布经过脱敏检查的测评材料，不是插件源码仓库，也不声明首次评测或新产品漏洞。

- 被测仓库：STARDUSTLC666/dsh-code-security。
- 固定提交：`0f2c70b14294460ad07f80ad754d0464765e4dbd`。
- 固定版本：npm 0.5.0，DSH 0.2.0-rc.2，Windows x64。
- 新目录复跑：内置浏览器断言 20/20，原生引擎浏览器断言 18/18，重启后 2 个工作区持久化。
- 两次驱动启动失败的记录也在证据包中；驱动问题没有当作插件漏洞申报。
- 未验：Linux/WSL、真实模型会话、生产项目准确率、在线 OSV、跨机器安装。

## 文件

- [复核报告](REPORT_v2.md)
- [复跑说明](REPRODUCE.md)
- [公开证据 ZIP](https://github.com/says693/dsh-code-security-review-20261019/releases/download/review-0.5.0-20261019/PUBLIC_EVIDENCE_v2.zip)
- [可复跑脚本 ZIP](https://github.com/says693/dsh-code-security-review-20261019/releases/download/review-0.5.0-20261019/REPRO_KIT_v2.zip)
- [Release 及完整文件清单](https://github.com/says693/dsh-code-security-review-20261019/releases/tag/review-0.5.0-20261019)

公开证据 ZIP SHA256：
`783e2ea66401a553fdb12f4cb85c767311acad61e0b4288da81e1251be39d762`

可复跑脚本 ZIP SHA256：
`55913e58f3bf39aa6c6a5406ff088e2b5d7b9443876a6cb5f2eb66994495cfca`

运行脚本与验收时执行的文件逐字节一致。所有本机配置、真实登录令牌、原始 DOM 和未遮罩截图均未发布。

工具与依赖复用既有本机安装，源码在新目录 clone 后固定提交；不是另一台机器从网络全新安装验证。
报告中的历史 79 项上游测试属于早先一轮，本轮没有重跑，也不把重复执行相加为新增覆盖。

## 提交状态

本证据仓库用于向 FuRongJun-1999/dsh-bounty 提交一件独立复核。受理和贡献等级由维护者判定。
