# QWRT-AX6-Hermes

Redmi AX6(IPQ807x,IPQ8072A 硬改 1G RAM/240M NAND)的 QSDK 5.4 定制固件。
底座 = 大雕 QWRT R25.12.2(真闭源 qca-wifi),自行精简 + 组件升级。个人研究用。

> ⚠️ 刷机有风险。个人设备定制件,无任何保证。刷前备份,确认 uboot 分区布局匹配。

## v1.1 更新内容(2026-09-14)

**v1.0 → v1.1 变化:**

| 项 | v1.0 | v1.1 |
|---|---|---|
| 镜像 | QWRT_ax6_hermes_factory.ubi / sysupgrade.tar 两个文件名混用 | 统一为 `.bin` 命名(factory.bin / sysupgrade.bin),与刷机工具默认期望一致 |
| NSS 固件 | 12.2-156 | **12.5-210**(高通 IPQ807x 最后公开发布版,与 host 栈 SPF 12.5 官方配套;大雕原包都没升到这) |
| xray | 26.6.1 | **26.9.9** |
| sing-box | 无 | **1.14.0**(新增第二代理核心,官方 musl 静态,与 xray 可并存切换) |
| curl | 7.83.1(CVE 累积) | **8.22 静态版**(usr/bin/curl8,不覆盖系统 curl,零风险并存) |
| ca-bundle | 2021 年证书 | 2026 新版 |
| 内核硬化 | 无 | **首启自动硬化脚本**(SYN flood 防护/关 ICMP 重定向/kptr_restrict=2/禁 unprivileged BPF/SysRq 关闭/SSH 强化) |
| PassWall | 26.9.9 | 26.9.9(不变,已是官方最新) |
| 内核/无线 | 5.4.213 + qca-wifi 闭源 | 同 v1.0(QSDK 13.0 已核查:官方放弃 IPQ807x + 固件 blob 无公开渠道,5.4 线就是天花板) |

## v1.1 相对大雕原版的完整区别

| 层 | 大雕原版 | Hermes v1.1 |
|---|---|---|
| 内核 | 5.4.213 QSDK | 同(闭源无线焊死,不可升) |
| 无线驱动 | qca-wifi 闭源 SPF 12.5 | 同(逐字节未动) |
| NSS 固件 | 12.2-156 | **12.5-210** |
| NSS 模块 | 27 个全家桶 | 16 个核心(删 11 个调试/无用) |
| 软件包 | 610 个(广告插件一堆) | 563 个(删 47 包/716 文件) |
| xray | 26.6.1 | 26.9.9 |
| sing-box | 无 | 1.14.0 |
| curl | 7.83.1 | 8.22 静态并存 |
| 证书 | 2021 | 2026 |
| 内核硬化 | 无 | 首启自动 |
| 五件套 | 无 | PassWall/AGH/SmartDNS/Tailscale/ddnsto 全配 |

## 文件

| 文件 | 大小 | 刷法 |
|---|---|---|
| `QWRT_ax6_hermes_sysupgrade.bin` | 32.9MB | LuCI → System → Backup/Flash(**取消"保留配置"**) |
| `QWRT_ax6_hermes_factory.bin` | 34.4MB | uboot web(救砖/全新刷) |
| `overlay_final_v2.tar.gz` | 101.6MB | 刷完后 scp 到 /tmp,`tar -xzf /tmp/overlay_final_v2.tar.gz -C /` |

sha256 见 `SHA256SUMS`。

## 刷后步骤

```sh
tar -xzf /tmp/overlay_final_v2.tar.gz -C /
for s in passwall smartdns adguardhome tailscale ddnsto; do /etc/init.d/$s enable; done
# 恢复个人配置(六件套备份): PassWall/AGH yaml/tailscale state/ddnsto token
```

注:overlay 是裸文件形式(非 ipk),opkg 数据库不登记,升级=重新解包新版 overlay。

## 已知限制

- 内核 5.4 LTS(EOL)——QSDK 13.0 官方已放弃 IPQ807x(target 列表无 HK),且配套 WLAN.HK 固件 blob 无公开发布渠道,5.4+SPF12.5 线即为闭源 qca-wifi 世界天花板
- NSS Wi-Fi offload(12.5-210)不支持 802.11s mesh(高通文档明示,11.4 支持但旧)
- overlay 二进制在 6.18 环境提取(musl aarch64),5.4 内核 syscall 兼容,静态/纯 musl 可跑,实机验证待做
- 无线为 qca-wifi 私有配置体系,与 ath11k 的 wireless 配置语法不互通,需重新配 SSID

## 回滚

刷回任一已知好镜像(主线 NSS / ImmortalWrt)。uboot/mibib/appsbl 未动,救砖路径常在。

## 溯源与合规

底座固件为作者合法购买件;QSDK 内核/NSS/SSDK 源码为 CodeLinaro GPL 公开;qca-wifi 驱动源码属高通 SPF 授权渠道(已核查 13.0 分支存在但固件 blob 缺失)。本仓库不含任何第三方源码,仅含二进制改造产物与自写脚本。
