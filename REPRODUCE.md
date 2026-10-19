# 固定版本复跑说明 v2

这是 Windows x64 的定向交付复现：从空 profile 验证安装、真实浏览器内置扫描和修复、三个真实原生引擎、缺失引擎、离线依赖与重启持久化。复用原有 20 项内置断言和 18 项原生断言，不将它们声称为新设计的测试，也不重复统计上游 79 项套件。

## 1. 准备外部依赖

需要 Node 24.19.0、DSH 0.2.0-rc.2、pnpm 11.7.0、Python 3、Playwright 1.64.1 与 Chrome，以及 Betterleaks 1.9.0、Semgrep CE 1.181.0、OSV-Scanner 2.7.0。运行前自行准备这些官方工具，本包不携带二进制、不安装系统软件、不修改产品源码。

已准备好工具时直接复用其绝对路径。Windows 上 Semgrep 的启动器绑定原 venv 的 Python 路径；保持完整的原 venv，不要只复制 semgrep.exe。原始 Semgrep 依赖集合在原证据包的 semgrep-freeze.stdout.txt。

需准备 dsh-code-security 0.5.0 的 npm tgz：

```powershell
npm pack dsh-code-security@0.5.0 --pack-destination .
```

固定包 SHA256：
`6da03b715e7ed4b16616842c7c6b07b5bb5dedaf1c64261a53ad1c90363c22c0`

Betterleaks.exe SHA256：
`41a3dc5e75712d52d25aa7203b0b6bc2fa9c796fe5109d3835c60226a9a69352`

OSV-Scanner.exe SHA256：
`9964ed64ec9a387391eb4cc006ad5c531154821aef29b793e02c448de4686438`

## 2. 配置并运行

将 REPRO_KIT_v2.zip 解压到全新目录，保持所有脚本在解压目录根层。以下命令均从该目录执行：

```powershell
Copy-Item -LiteralPath .\config.example.json -Destination .\config.local.json
notepad .\config.local.json
```

填入 JSON 中每一个空字符串。Windows 路径用 `/` 或 `\\`，不要修改脚本。

| 配置项 | 填写内容 |
|---|---|
| dsh_cli | 官方 DSH npm 包的 lib/bin.js |
| chrome_exe | Chrome 的 chrome.exe |
| playwright_module | playwright 包目录中的 index.mjs |
| pnpm_cli | pnpm 包目录中的 bin/pnpm.cjs |
| plugin_archive | 固定版本 dsh-code-security-0.5.0.tgz |
| betterleaks_exe | 已校验的 betterleaks.exe |
| semgrep_exe | 已有独立 venv 中的 Scripts/semgrep.exe |
| osv_exe | 已校验的 osv-scanner.exe |
| source_repo | 默认官方 Git URL，或已存在的本机 Git 仓库绝对路径 |
| profile / host_tag | 保持默认即可；脚本仅在解压目录的独立 DSH_HOME 中使用 |

```powershell
python -X utf8 .\reproduce.py
```

脚本校验驱动文件清单和固定包、工具哈希，然后新建源码 checkout、evidence、fixtures、published、isolated-home 和 run-env.json。它使用固定提交 `0f2c70b14294460ad07f80ad754d0464765e4dbd` 和 `pnpm install --offline --frozen-lockfile --ignore-scripts`，依赖须已在本机 pnpm 缓存中。若离线安装报告缓存缺项，可先在另一个准备目录检出同一提交并执行 `pnpm install --frozen-lockfile --ignore-scripts` 填充缓存，再在新的解压目录运行本流程；失败目录及输出应保留。

source_repo 使用本机仓库时仍执行新 clone，绝不复制原 profile、工作区 catalog 或浏览器存储。本次验收使用这种方式，源码来源在 REPRO_RESULT.json 中明确记录；这不等同于验证在线下载可用性。

## 3. 结果判据与复跑

成功时 evidence/REPRO_RESULT.json 中应满足：

- error 为 null；
- all_recorded_commands_exit_zero、all_monitored_user_configs_unchanged、hosts_stopped 均为 true；
- fresh-builtin 的 passed/total 为 20/20；
- fresh-native 的 passed/total 为 18/18；
- baseline-404.json 实测 404，baseline-200.json 实测 200；
- restart-result.json 记录两个工作区及设置、报告持久化。

38 项断言包括：修复后清零、缺失 Semgrep 时不关闭旧问题、缺失数据库和损坏锁文件不报通过、合成离线公告的匹配与消除。断言数量不等于独立缺陷数量。

每个命令保存 command、cwd、退出码、stdout、stderr、被监控的日常 DSH 配置文件前后哈希。已有记录禁止覆盖；再次复跑需重新解压到另一目录。Host 输出只打印监听 origin，完整带令牌地址仅保存在本机 evidence/*-host-local.json。

不调用模型。项目通过发布包真实的 SecurityWorkbench.remember() 预登记；未验证真实模型会话自动登记。浏览器请求仅允许回环地址；依赖审计只用上游合成离线公告。Windows/Chrome 以外平台、在线 OSV、生产项目检出质量均未验。

## 4. 人工复核与公开交付

不需要人工才能完成本次自动复跑。想检查页面时，须另建复跑目录并按已有记录中的启动命令运行宿主；登录地址只在本机读取，勿公开。

勿上传运行目录。config.local.json、run-env.json、isolated-home、temp、source、fixtures、*-host-local.json、*-process.stdout.txt、DOM 和原始截图都只留本机。公开交付使用另行生成、已检查的 PUBLIC_EVIDENCE_v2.zip，其中脚本按原字节复制；仅日志和报告文本做脱敏，截图使用运行时遮罩版本。不要再对可执行源码做全局 token 正则替换。

原始 PUBLIC_EVIDENCE.zip 保留为历史证据，其中 scripts/ 已知不可直接复跑。v2 替代它的执行入口，不抹去原失败记录。本轮不提交 issue、不发送外部消息，也不声称首次评测或新增产品漏洞。
