# QWRT-AX6-Hermes

Redmi AX6(IPQ807x,IPQ8072A 硬改 1G RAM/240M NAND)的 QSDK 5.4 定制固件。
底座 = 大雕 QWRT R25.12.2(付费购买,真闭源 qca-wifi),自行精简 + 组件升级,个人研究用。

> ⚠️ 刷机有风险。这是个人设备定制件,不提供任何保证。刷前务必备份,确认 uboot 分区布局匹配。

## 构成(对比大雕原版)

| 层 | 内容 |
|---|---|
| 内核 | 5.4.213 QSDK(原样,无源码不可重编) |
| 无线驱动 | qca-wifi 闭源 SPF 12.5(umac/qca_ol/qdf,原样) |
| 无线固件 | WLAN.HK.2.9.r4-00018(原样) |
| NSS | host 栈 12.5 线 + **固件升级 12.2-156 → 12.5-210**(与 host 官方配套) |
| 用户态 | 删 47 插件包(openclash/ssr-plus/ruby/mihomo/zerotier/samba 等 716 文件) |
| 应用 | overlay 分发:PassWall 26.9.9 / xray 26.9.9 / AdGuardHome / SmartDNS / Tailscale / ddnsto |

## 文件

| 文件 | 大小 | 刷法 |
|---|---|---|
| `QWRT_ax6_hermes_sysupgrade.bin` | 32.9MB | LuCI → System → Backup/Flash(取消"保留配置")|
| `QWRT_ax6_hermes_factory.bin` | 34.4MB | uboot web(救砖/全新刷)|
| `overlay_final.tar.gz` | 68.3MB | 刷完后 scp 到 /tmp,`tar -xzf /tmp/overlay_final.tar.gz -C /` |

sha256 记录见 `SHA256SUMS`。

## 刷后步骤

```sh
# overlay 展开五件套
tar -xzf /tmp/overlay_final.tar.gz -C /
# 启用服务
for s in passwall smartdns adguardhome tailscale ddnsto; do /etc/init.d/$s enable; done
# 恢复各自配置(/etc/config/* + AGH yaml + tailscale state)
```

## 回滚

刷回任一已知好镜像(主线 NSS / ImmortalWrt / 小米官方经过渡包)。uboot 未动,mibib/appsbl 未动,救砖路径常在。

## 已知限制

- 内核 5.4 LTS(EOL)——QSDK 线天花板,闭源 qca-wifi kmod 锁死此内核
- NSS FW 12.5-210 为高通最后公开发布的 IPQ807x 固件,SPF 13.0 源码公开但配套固件 blob 无公开渠道
- overlay 二进制在 6.18 环境提取(musl aarch64),5.4 内核 syscall 兼容,静态/纯 musl 可跑,实机验证待做

## 溯源与合规

- 底座固件为作者合法购买件;QSDK 内核/NSS/SSDK 源码本为 CodeLinaro GPL 公开;qca-wifi 驱动源码属高通 SPF 授权渠道
- 本仓库不含任何大雕源码(其未提供),仅含二进制改造产物与自写脚本
