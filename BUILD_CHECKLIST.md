# 编译与测试强制流程（S20P-WIFI-YES）

> 本仓库只保留 `S20P-WIFI-YES` 一个设备。每次改动后，必须按以下顺序执行，禁止跳过。

## 1. 编译前：本地 `config_preview`
```bash
./build.sh S20P-WIFI-YES config_preview
```
检查点：
- 设备识别正确
- fragments 组合正确（默认 `docker_deps`）
- 没有意外的 fragment 被加入/移除

## 2. 编译前：本地 `debug`（强制）
```bash
./build.sh S20P-WIFI-YES debug
```
检查点：
- 必须成功执行到 `make defconfig` 并写出 `.config`
- 日志中不能出现与**已选包**相关的 `has a dependency/build dependency ... does not exist`
- 出现包名/依赖警告时，必须先修正再继续

本机依赖（Ubuntu）：
```bash
sudo apt install -y gawk libncurses-dev
```

## 3. 云编译
只有第 1、2 步通过后，才允许触发 GitHub Actions：
- Actions → **Build WRT** → `S20P-WIFI-YES`

## 4. 编译后：本地模拟器测试（强制）
使用 `router-emulator` 对产物做启动与基础网络验证。

前提：
- 已准备 S20P 的分区备份：`boot0`、`fip`、`factory` 等
- 已编译 emulator：`router-emulator/build.sh`

建议流程：
```bash
# 1) 准备 S20P 板级数据目录
mkdir -p ~/s20p-dump
# 放入 boot0.bin / fip.bin / factory.bin

# 2) 生成 eMMC 镜像
tools/prepare-nand.sh --stock ~/s20p-dump --emmc --soc mt7986 \
    --uboot cmcc_rax3000m-emmc-ddr4 clx_s20p snapshot ./emmc-s20p

# 3) 启动模拟器
./emulator.sh -P <preset> ...
```

模拟器检查点：
- 能进入 U-Boot / 内核启动
- 能进入 OpenWrt 系统（串口或 LuCI）
- LAN/WAN 基本连通、DHCP 正常
- 至少验证 Clash/代理/穿透等已内置服务可启动

限制：
- Wi-Fi 射频在模拟器中不发射，Wi-Fi 仍需真机验证
- MTK HNAT/WARP、Full Cone NAT 的最终行为需真机确认

## 5. 真机验证（最终）
- 刷机前备份配置
- 先验证基础联网 + LuCI
- 再逐项启用 fullconenat / mwan3 / syncdial，确认无断网

## 规则
- **未跑 `debug` 不允许云编译。**
- **未做模拟器/真机测试不允许发布 Release。**
- 任何包名/依赖修正都必须同步写入 `custom_feed.sh` 或设备 `.config`，避免下次再犯。
