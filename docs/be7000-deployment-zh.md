# BE7000 刷机与恢复手册

此手册对应 Xiaomi BE7000（RC06）与此仓库发布的 MiWRT。以下例子使用 `v2.3.1.2`，管理地址为 `10.10.10.1`。

## 下载与核对

从同一个发行版获取 ZIP、`factory.ubi`、`sysupgrade.bin` 和 `sha256sums.txt`。核对机型 `xiaomi_be7000`、平台 `qualcommbe/ipq95xx` 及 SHA‑256。此版本的构建提交为 `b64734ea857a193c5253056639f043b97c785fbb`。 [发行版](https://github.com/xtaoops/MiWRT/releases/tag/v2.3.1.2) · [构建记录](https://github.com/xtaoops/MiWRT/actions/runs/44957704331)

在 Windows 可用 `Get-FileHash -Algorithm SHA256 <镜像文件>`，在 Linux 或 macOS 可用 `shasum -a 256 <镜像文件>`。把结果与对应文件的摘要逐字核对。

## 首次安装

1. 使用有线 LAN 接入。在原厂固件已有合法管理员 SSH 访问的前提下，备份本机的两个固件槽、引导相关分区、ART 和 bdata，并在另一台设备核对备份完整性。ART 和 bdata 含有设备自身的校准或识别数据，只归本机使用。
2. 从发行 ZIP 解出 `install-from-stock.sh`，和同版本 `factory.ubi` 一起复制到路由器的 `/tmp`。上传后再核对镜像 SHA‑256。
3. 按发行包内第 4 步运行安装脚本。脚本识别当前原厂槽，向另一个槽写入 MiWRT。另一种 OpenWrt 路径由 ZIP 内 `install-from-openwrt.sh` 提供。 [安装步骤](instruction/1-install-zh.txt)
4. 启动后让电脑重新获取 DHCP 地址，打开 `http://10.10.10.1/`。检查 LAN、WAN、无线国家和 SSID，并设置管理密码。需要直连排查时，电脑可临时用 `10.10.10.2/24`。

## 升级、重置与原厂回退

- 现有 MiWRT 用 `sysupgrade.bin` 升级。先导出配置并留在电脑上，使用 LuCI 的固件升级，或先以 `sysupgrade -T` 做兼容性检查，再按维护窗口升级。启动后核对版本、WAN、管理及所有额外服务。 [升级步骤](instruction/2-update-zh.txt)
- Reset 恢复当前 MiWRT 镜像的出厂配置。此构建的 MiWRT 默认管理地址为 `10.10.10.1`。恢复整个原厂固件时，从 LuCI **系统 → 槽位**选择已核实的原厂槽并重启。 [原厂回退与故障排查](instruction/3-rollback-and-problems-zh.txt)

## 恢复部署配置

1. 从私有备份下载同一版本、同一设备的 `sysupgrade` 配置包及 `.sha256`，核对摘要，并查看 `tar -tzf <配置包>`。另取此版本所需的软件包清单和安装包。
2. 在相同构建的 MiWRT 上用有线 LAN 管理。先保存目标路由器当前配置，准备备用管理入口和维护窗口。
3. 用 LuCI **系统 → 备份/升级 → 恢复配置**导入配置包。也可在有线 SSH 会话中把包上传到 `/tmp`，执行 `sysupgrade -r /tmp/<配置包>`，成功后安排重启。若配置包含额外服务，按备份记录先安装它们及其依赖。
4. 重启后用本地网线确认管理页，然后验证 DHCP、DNS、访客隔离、两条 WAN、内外网 HTTPS、VPN 管理以及相关服务的开机状态。使用另一台冷备机时，先确认设备专用配置及 VPN 身份，并避免两台机器同时使用相同的地址或身份。

日常配置备份可用 `sysupgrade -k -b /tmp/be7000-config.tar.gz` 生成。自建服务须把配置、启动脚本和防火墙引用文件加入 `/etc/sysupgrade.conf`，并用 `sysupgrade -l` 核对。配置备份及完整恢复记录应放在受控的私有存储中。
