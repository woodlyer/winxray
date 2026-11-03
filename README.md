
WinXray 是一个 Windows 平台上非常好用的轻量代理客户端，原版版本号开发到 3.8（原地址已失效）。  
本项目在原有基础上持续修复与演进，全面适配新版 Xray-core、REALITY、Hysteria 2、TUIC 等主流协议，提升稳定性和交互体验。

---

## 📥 下载方式

👉 **[点击前往 Releases 页面下载最新版本](../../releases)**  
（解压即可直接运行，纯绿色免安装，体积轻巧）

---

## 📝 本版本更新内容 (Current Version - v3.22)

### 1. DNS 防泄漏与智能分流深度重构
- **路由策略全面改为 `AsIs`**：将 Core 路由策略从 `IPIfNonMatch` 全面改为 `AsIs`，彻底杜绝 Xray 为匹配 IP 规则在本地向上游/运营商 DNS 发起境外域名解析导致的本地 DNS 泄漏；
- **生产级国内外智能 DNS 分流**：
  - **内部域名**：绑定 `geosite:cn`，定向经由阿里加密 DoH（`https+local://223.5.5.5/dns-query`）直连解析，并开启 `expectIPs: ["geoip:cn"]` 防止运营商劫持投毒；
  - **外部域名**：绑定 `geosite:geolocation-!cn`，定向走 Cloudflare DoH（`https://1.1.1.1/dns-query`），经由代理出站隧道加密解析，杜绝污染；
- **强制仅查询 IPv4**：显式声明 `"queryStrategy": "UseIPv4"`，有效避免因国内残缺 IPv6 导致的网络卡顿、断流或 IPv6 DNS 泄漏；
- **配置与旧数据热迁移**：首次加载自动检测旧版存储的配置，无损热升级为 `AsIs` 并补全 `dns` 节点；
- **DNS 泄漏在线检测推荐**：开启代理服务后，可访问 [https://who.1ms.dog/zh-cn/](https://who.1ms.dog/zh-cn/) 在线检测 DNS 是否泄漏。

### 2. 内核启动端口占用诊断与运行时看门狗（Watchdog）
- **启动阶段 250ms 存活检测**：解决本地端口（默认 1080/1082）被其他软件占用时，内核崩溃退出而 UI 却误判为运行中并盲目切换系统代理导致断网的顽疾；
- **智能诊断与提示**：启动后即时校验进程状态，如因端口占用异常退出，智能提取 `bind` / `already in use` 错误输出，在控制台和弹窗中明确提示用户排查冲突端口；
- **500ms 进程运行时看门狗**：新增内核存活状态轮询监控，一旦内核中途异常退出，UI 毫秒级重置按钮状态为“启动服务”、清空活动服务器指示，并**自动恢复系统代理为直连模式**，杜绝死锁与断网。


---

## 📜 历史版本更新记录 (Changelog)

### v3.21
- **彻底解耦 AppData，实现真正 100% 绿色便携**：移除所有写入系统 `%LocalAppData%\winXray` 路径的逻辑，配置文件、核心程序、临时缓存、下载目录全部集中于软件同级目录（`./config/`、`./download/`、`./temp/`）；
- **旧配置无缝热迁移**：首次启动自动检测旧版 AppData 配置并完整迁移至 `./config/`，用户订阅链接与已有节点数据零丢失；
- **现代 aardio 运行时与编译兼容全面修复**：适配保留关键字 `switch` 重命名为 `setMode`；修复类数组判断机制；修复 Xray 重复传参报错；内置打包 `qrencode` 扩展库及 DLL；推荐搭配 aardio-v35 稳定版编译；
- **入站默认端口规范与 PAC 规则库重大升级**：默认本地入站端口调整为业界通用的 1080 (SOCKS5) 与 1082 (HTTP)；PAC 解析引擎重构为安全切片算法；同步最新 GFWList 规则库。

### v3.20
- **毫秒级极速退出与进程清理**：重构内核进程终止机制，退出时通过保存的进程句柄直接向内核发送终止信号，退出耗时从数秒缩减至 10ms 以内；配合 Windows Job Object 进程树同生共死特性，保证主程序退出时子内核进程绝无残留；统一托盘右键菜单与侧边栏退出流程；
- **全面支持 Hysteria 2 协议**：支持基于 UDP/QUIC Brutal 拥塞控制的高速代理协议 Hysteria 2，支持标准 `hysteria2://` 及 `hy2://` 分享链接解析与导入导出；接入官方内核，支持认证密码、SNI、ALPN、混淆密码、证书 pinSHA256、上下行带宽限制、多端口跳跃等完整特性，支持配置界面一键在线下载与更新；
- **全面支持 TUIC (v5) 协议**：支持基于 QUIC 的新一代低延迟代理协议 TUIC 与标准 `tuic://` 分享链接导入，接入 tuic-client 内核，支持一键在线获取与更新；
- **节点编辑与交互体验优化**：新增协议模板智能填充、SNI 联动同步自动补齐、ALPN 详细提示与智能预填；修复节点回显缺失用户 UUID 问题；移除强制忽略证书行为，执行严格证书校验。

### v3.15
- **TUIC (v5) 协议深度集成**：完整支持标准 `tuic://` 分享链接解析导入与通用 JSON 格式配置；
- **独立内核调度架构**：接入 [Itsusinn/tuic](https://github.com/Itsusinn/tuic) 内核，winXray 在后台自动启动 `tuic-client.exe` 并由 Xray Core 统一进行路由规则分流与系统代理接管；
- **内核在线下载与自动维护**：配置界面新增「下载 / 更新 TUIC Core」专属按钮，自动匹配 Windows 32/64位平台并高速下载最新版本，优先识别便携路径 `./tuic-core/tuic-client.exe`；
- **配置与日志精简优化**：TUIC 运行时配置精简为标准 `relay` 与 `local` 架构，默认日志级别提升至 `error`，过滤心跳冗余警告并禁用 ANSI 彩色控制乱码；
- **UI 与稳定性细节修复**：修复在线更新核心时传递窗口句柄导致的报错，修复 `tuic-client` 多余位置参数命令行崩溃，为 TUIC 定制专属火箭图标（`\uF135`）。


### v3.12
- **默认路由与分流优化**：默认集成大陆域名 (`geosite:cn`) 以及大陆/局域网 IP (`geoip:cn`, `geoip:private`) 直连规则，避免国内流量绕外网；API 规则置顶优先处理，预置广告拦截备用规则；
- **内核管理机制重构**：直接原生调用 `xray.exe run -c config.json`，优先匹配本地便携内核；
- **节点配置与排序优化**：SOCKS / HTTP / Naïve 支持直观的 `"user"` 与 `"password"` 字段；支持节点自由快捷键排序；
- **测速策略与UI调整**：优化启动测速比对分档机制，微调侧边栏图标与提示排版。

### v3.11
- **REALITY 协议支持**：完整支持 VLESS + REALITY 协议配置（`publicKey` / `shortId` / `spiderX` / `fingerprint`）；
- **uTLS 客户端指纹**：支持模拟 Chrome、Firefox、iOS、Safari 等客户端 TLS 指纹；
- **多协议字段增强**：完善 Trojan-go、NaïveProxy、Shadowsocks-2022 等新特性解析与配置导出；
- **路由规则在线更新**：支持一键在线下载/更新 Loyalsoldier 的最新 `geoip.dat` 与 `geosite.dat` 规则库。

### v3.10
- **全面适配 Xray-core**：支持新一代 Xray 内核特性与 VLESS XTLS 传输流控；
- **订阅管理优化**：支持批量导入 Base64/明文订阅链接，支持节点异常自动刷新订阅源；
- **多路复用支持**：支持配置 Mux 最大并发连接数与动态流控。

### v3.8 及更早（原版特性）
- 极速并发 TCPing 测速与秒级故障自愈重连；
- 内置独立 PAC 代理服务器与全局/PAC 快捷键热键切换；
- Windows 托盘集成、系统代理自动托管、一键同步系统时间、UWP 应用免代理回环工具。

---

(以下内容为原版 WinXray 介绍文档)
--------------------------------------------------------------------------------------------------

# WinXray 
WinXray[:loud_sound:](http://dict.youdao.com/dictvoice?audio=winxray&type=2) 是最简洁轻快的 V2Ray、XRay、Trojan、Trojan-go、Shadowsocks、SSR(ShadowsocksR)、SSRoT、NaïveProxy、Hysteria 2、TUIC，SOCKS，HTTP,HTTPS 全能通用客户端（Windows系统），支持并发检测大量服务器并迅速找到当前最快的服务器，服务器连接异常时可自动寻找其他速度最快的服务器 - 切换速度快如闪电，自订阅源获取的服务器异常时可自动刷新订阅，并且自带一键自动部署服务端工具。

**本软件源码已放弃版权贡献到公共域** ，源码可使用 [aardio](http://www.aardio.com) 编译生成单文件绿色EXE，**[点这里下载](./../../raw/master/release/winXray.7z)** （ [64位版本](./../../raw/master/release/winXray.7z) / [32位版本](./../../raw/master/release/winXray32.7z) ），解压即可直接使用( 体积很小仅  **[6.1 MB](./../../raw/master/release/winXray.7z)** - 已自带 V2Ray Core ）。  

# WinXray 未注册任何域名，谨防钓鱼网站    
WinXray 分为原版、抄袭版。  
抄袭版没有贡献任何功能，仅添加了假冒官网推广链接，然后原版更新任何功能，抄袭版都会复制粘贴改成他自己的名字重新提交，并且乱改版本号，日常踩原作者吹捧自己。原版作者估计是受不了那货已经失踪很久了。

本项目基于原版 WinXray v3.7 基础上继续更新 ，并将保持原版干净、纯净。本项目严禁上述假冒官网的抄袭版抄袭本项目的任何一句代码 。

# 免费服务器   
[网络免费 vmess 服务器订阅链接](https://proxypool.ga/vmess/sub)   
[网络免费 Shadowsocks 服务器订阅链接](https://proxypool.ga/ss/sub)     
[网络免费 clash 服务器订阅链接](https://proxypoolss.tk/clash/proxies?speed=100&type=vmess,trojan)   
可复制上面各种格式订阅链接，在 winXray 中点击「批量导入链接」体验 winXray 有强大的兼容性。  
免费的服务器仅供测试（一定要走 PAC，不要开全局代理不要登录账号更不要长时间使用 ）。

# 关于误报  
现在大多杀毒软件都是白名单查杀，所以新生的EXE都会乱报病毒，因为我更新的速度太快，所以不断的推新EXE上来，所以你可能遇到误报，但是你完全可以使用源代码自己编译出一模一样的EXE，还有人吹牛说其他翻墙软件不误报 - 你去 issues 里以及网上搜一下有多少误报好不好？！遇到误报可以提交给你的杀毒厂商核实，也可以自己编译源码生成EXE后使用，解决和核实问题很容易 - 其他套路都是多余的。


# PAC 代理模式 / 全局代理 + 路由模式 对比

winXray 的 PAC 代理稳定、流畅、易用。  在 PAC 模式下，winXray 会优先启用高效安全的 SOCKS5 协议，并且可以自动兼容在 PAC 模式下仅支持 HTTP代理的应用。winXray 也可以在 PAC 模式下完美支持 Telegram IP 地址库 。

SOCKS5 支持对比：
- [ ] 全局路由模式: 不支持 SOCKS5
- [x] PAC模式: 支持 高效安全的SOCKS5   
  
UWP 应用支持对比：
- [ ] 全局路由模式: UWP 应用全部无法联网。
- [x] PAC模式: UWP 应用可以正常联网，使用 winXray 自带工具也可以为UWP应用开启本地代理。 

DNS 解析对比：
- [ ] 全局路由模式: 使用本机发起 DNS 解析，即使设为国外 DNS 服务器，仍然会返回适用于国内线路地址。
- [x] PAC模式: 使用服务器上的 DNS 解析，安全可靠。

根据客户端自动切换代理协议：
- [ ] 全局路由模式: 不支持
- [x] PAC模式: winXray 里的 PAC 代理可以让目标应用（例如浏览器）优先选择高效安全的 SOCKS5 代理协议，对于不支持 SOCKS 代理的应用（例如谷歌地球），winXray 在 PAC 模式下会自动为这些应用提供 HTTP 代理。

IP 段代理规则：
- [x] 全局路由模式: 比较好的支持 IP 段代理规则
- [x] PAC模式: winXray 里 PAC 可以支持IP 段代理规则（ 完美支持 Telegram ）。

独立性
- [ ] 全局路由模式: 不独立，代理规则集成在翻墙软件内核中
- [x] PAC模式: 完全独立，PAC 代理服务完全独立于翻墙软件，只有 PAC 指定的域名或IP才会与翻墙软件发生交互。

兼容性
- [ ] 全局路由模式: 不是由系统实现的规则，一旦设置全局代理，不管适不适合走代理的软件都被强制使用代理，所以兼容性不太好，会导致上述的 UWP 无法联网等问题。
- [x] PAC模式: 由系统提供的PAC有良好的兼容性，因为历史悠久，一般的软件都会对PAC有良好的兼容，PAC 主要为适合走代理的浏览器等软件而设计，所以其他软件可以较好的识别并判断是不是要使用 PAC 指定的代理还是直连。

简易度
- [ ] 全局路由模式: 配置复杂，有一定门槛。
- [x] PAC模式: 配置非常简单。


一般不建议普通用户去编辑路由规则 - 错误的配置可能会导致敏感的流量误走代理服务器。
专业的事请交给专业的人去做，使用 winXray 可以一键启用、更新 [v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) 提供的最新路由规则。

注意在 winXray 里无论使用 NaïveProxy 还是 SSR，SSRoT 都支持 V2Ray 路由规则。

# 设置系统代理失败怎么办
如果设置代理以后不能正常生效：请首先右键点击 winXray 任务栏的托盘图标，在弹出的右键菜单中点击【查看 Internet 代理设置】，并检查代理设置是否正常。如果 winXray 不能修改代理设置，但是可以手动修改成功，这一般是被安全软件错误地拦截了( 而且安全软件没有正常弹出确认对话框，或者误点了阻止设置 ）。这时候请到安全软件的相关设置中将 winXray 添加到信任列表即可。

如果不是上面的原因，请按下【Win + R】组合键打开系统运行对话框，输入 regedit 点击确定打开注册表路径 
<span style="color:green">HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Connections</span>
然后将“Connections”项删除，注销一下系统即可正常使用代理了。

如果上面的方法仍然不行，请在注册表中打开打开路径
<span style="color:green">Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WinHttpAutoProxySvc</span>
将 start 的值改为 2， 也就是将 WinHttpAutoProxySvc 服务改为自动启动，然后重启计算机即可。

# Core 默认路径与下载地址

可在「 winXray / 配置 / Core配置 」一键下载更新各核心。找不到会自动在线下载，也可以手动下载放入对应目录：

### 1. Xray Core
- **查找目录**：`./xray-core/` 或 `./v2ray-core/` 或 `%localappdata%\winXray\core`
- **最新版本发布**：https://github.com/XTLS/Xray-core/releases
- ⚠️ **特别提示（推荐兼容版本：v25.1.1）**：
  较新版本的 Xray-core 逐步废弃并移除了 TLS 配置中的 `allowInsecure` 选项（改用证书哈希绑定等机制，直接配置可能导致内核启动报错）。如果您的节点使用了自签名证书，或需要保持对 `allowInsecure` 选项的完整兼容，推荐使用经过实测兼容稳定的 **[Xray-core v25.1.1](https://github.com/XTLS/Xray-core/releases/tag/v25.1.1)**：
  - [Xray-windows-64.zip (v25.1.1 64位下载)](https://github.com/XTLS/Xray-core/releases/download/v25.1.1/Xray-windows-64.zip)
  - [Xray-windows-32.zip (v25.1.1 32位下载)](https://github.com/XTLS/Xray-core/releases/download/v25.1.1/Xray-windows-32.zip)
  解压后将 `xray.exe` 放入 `./xray-core/` 目录即可。

### 2. Hysteria 2 Core (hysteria)
- **查找目录**：`./hysteria-core/` 或 `%localappdata%\winXray\hysteria-core`（亦支持直接放在软件同级目录）
- **项目仓库**：https://github.com/apernet/hysteria
- **Releases 发布页**：https://github.com/apernet/hysteria/releases
- **Windows 下载直链**：
  - [64位：hysteria-windows-amd64.exe](https://github.com/apernet/hysteria/releases/latest/download/hysteria-windows-amd64.exe)
  - [32位：hysteria-windows-386.exe](https://github.com/apernet/hysteria/releases/latest/download/hysteria-windows-386.exe)
  下载后重命名为 `hysteria.exe` 保存到 `./hysteria-core/` 目录即可。

### 3. TUIC Core (tuic-client)
- **查找目录**：`./tuic-core/` 或 `%localappdata%\winXray\tuic-core`（亦支持直接放在软件同级目录）
- **项目仓库**：https://github.com/Itsusinn/tuic
- **Releases 发布页**：https://github.com/Itsusinn/tuic/releases
- **Windows 下载直链**：
  - [64位：tuic-client-x86_64-windows.exe](https://github.com/Itsusinn/tuic/releases/download/v2.0.0-dev7/tuic-client-x86_64-windows.exe)
  - [32位：tuic-client-i686-windows.exe](https://github.com/Itsusinn/tuic/releases/download/v2.0.0-dev7/tuic-client-i686-windows.exe)
  下载后重命名为 `tuic-client.exe` 保存到 `./tuic-core/` 目录即可。

### 4. SSR Core
- **查找目录**：`./v2ray-core/ssr-core` 或 `%localappdata%\winXray\ssr-core`

### 5. NaïveProxy Core
- **查找目录**：`./v2ray-core/naive-core` 或 `%localappdata%\winXray\naive-core`
- **Releases 发布页**：https://github.com/klzgrad/naiveproxy/releases

> 提示：没有代理直连访问 Github 可能会很慢或超时，建议在 winXray 的「工具」页中运行自带的【Github 网速优化工具】加速访问。

注意不同的代理协议连接时会调用不同的 Core，例如 NaïveProxy 连接时会启动 naive.exe，Hysteria 连接时会启动 hysteria.exe，TUIC 连接时会启动 tuic-client.exe，此时系统防火墙如弹出提示请点击允许。  


