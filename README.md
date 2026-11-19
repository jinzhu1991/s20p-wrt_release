# CLX S20P（AX6000 Wi-Fi）固件构建仓库

本仓库基于 [dqsq2e2/wrt_release](https://github.com/dqsq2e2/wrt_release) 二次开发，仅保留 **`S20P-WIFI-YES`** 一个设备配置，用于构建 CLX S20P（MT7986 / AX6000）的 ImmortalWrt 25.12 固件。

- 上游源码：`chasey-dev/immortalwrt-mt798x-rebase`（分支 `25.12`，稳定版）
- 设备配置：`wrt_core/compilecfg/S20P-WIFI-YES.ini` + `wrt_core/deconfig/S20P-WIFI-YES.config`
- 默认片段：`docker_deps`（Docker 运行依赖）
- 已移除 `honk` 片段，以精简体积与编译时间

## 主要内置软件

基础配置已包含：OpenClash、iStore、Samba4、UPnP、ttyd、Argon/Aurora 主题、TurboACC (MTK)、WOL Ultra、云闪付/axonhub 等。

本次额外启用（以配置为准）：

- 代理/穿透：tailscale、frp、nps、ddns-go、easytier、lucky
- 远程访问：ddnsto
- DNS：smartdns
- IPTV：msd_lite、udpxy、igmpproxy
- 定时任务：taskplan
- 多拨/负载均衡：mwan3、syncdial
- 全锥 NAT：fullconenat-nft + luci-app-fullconenat（NAT1，需与硬件 offload 实测共存）
- Web 服务：无 Caddy（已移除）

## 本地编译

```bash
sudo apt -y update
sudo apt -y full-upgrade
sudo apt install -y dos2unix libfuse-dev jq dwarves
sudo bash -c 'bash <(curl -sL https://build-scripts.immortalwrt.org/init_build_environment.sh)'

git clone https://github.com/jinzhu1991/s20p-wrt_release.git
cd s20p-wrt_release
./build.sh S20P-WIFI-YES
```

调试/预览：

```bash
./build.sh S20P-WIFI-YES config_preview
./build.sh S20P-WIFI-YES debug
./build.sh S20P-WIFI-YES container
```

## GitHub Actions 云编译

1. 打开仓库 **Actions → Build WRT**。
2. 点击 **Run workflow**，`Device Model` 选择 **`S20P-WIFI-YES`**。
3. 编译完成后在 Artifacts 下载固件（保留 3 天）。

## 编译与测试强制流程

每次改动必须按顺序执行（详见 [BUILD_CHECKLIST.md](BUILD_CHECKLIST.md)）：

1. `./build.sh S20P-WIFI-YES config_preview`
2. `./build.sh S20P-WIFI-YES debug`（**强制**，未通过不得云编译）
3. GitHub Actions 云编译
4. 本地模拟器测试（**强制**，router-emulator + S20P 分区备份）
5. 真机验证（Wi-Fi / HNAT / Full Cone 最终确认）

## 重要说明

- **硬件 NAT（MTK HNAT/WARP）** 与 **Full Cone NAT** 可能冲突：若断网或 fullcone 不生效，请二选一。
- **mwan3 / syncdial** 默认不启用多拨；配置错误可能破坏默认路由，建议先单 WAN 验证。
- 上游使用 `25.12` 稳定分支，避免 snapshot 的内核/驱动 breakage。
- OAF（应用过滤）使用前需在「系统 → 启动项」启用 `appfilter`。

## 项目结构（精简后）

- `wrt_core/compilecfg/S20P-WIFI-YES.ini`：源码与片段配置
- `wrt_core/deconfig/S20P-WIFI-YES.config`：设备与软件包配置
- `wrt_core/deconfig/compile_base.config`：公共配置
- `wrt_core/deconfig/fragments/`：可组合片段（含 `docker_deps`）
- `wrt_core/modules/`：构建阶段脚本
- `wrt_core/patches/`：补丁与注入文件

## 许可证

GPL-3.0（继承自上游项目）。再发布时请保留版权、作者署名与来源信息。
