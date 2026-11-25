# Lenovo Legion 5 Pro (2025) OpenCore EFI

[中文](#中文) | [English](#english)

---

## 中文

自用的 Lenovo Legion 5 Pro（2025，Arrow Lake-HX）OpenCore EFI，当前运行在 **macOS 26.7.2**，三码已脱敏。

仓库里集成的 `LegionECMode.kext` 源码和单独打包版本维护在：[KSLengYu/LegionECMode-kext](https://github.com/KSLengYu/LegionECMode-kext)。

### 硬件与驱动情况

| 组件 | 硬件 / 驱动 | 备注 |
| --- | --- | --- |
| 系统版本 | macOS 26.7.2 | 当前测试环境 |
| CPU | Intel Core Ultra 7 255HX (Arrow Lake-HX) | `RestrictEvents.kext` + `revpatch=sbvmm,cpuname` |
| GPU | NVIDIA RTX 5070 Laptop (`10de:2d58`) | 配合 **NullMoth** 显卡驱动（`/Library/Extensions/` 下的 `NVRM` / `NVRMFB` / `NVRMAGDC` 3.0，非本仓库自带） |
| 内屏 | `2560x1600 @ 165Hz` (eDP) | HiDPI、亮度调节、睡眠唤醒可用 |
| 睡眠 | S3 | 已处理 XHCI / TXHC / RP09 / RP17 秒醒 |
| EC 功耗模式 | `_SB.GZFD.WMAA` | 集成 `LegionECMode.kext`，终端免密通过 `sysctl -w debug.legionec.mode=1/2/3` 切换 |

NullMoth 的 NVIDIA Blackwell 驱动（`com.nullmoth.NVRM` / `NVRMFB` / `NVRMAGDC`）需要另行安装到 macOS 系统的 `/Library/Extensions/`，不包含在这个 EFI 仓库里。

### 使用前需要填的三码

`EFI/OC/config.plist` 中 `PlatformInfo -> Generic` 已清空为占位值：

- `SystemSerialNumber`: `CHANGE_ME_SERIAL`
- `MLB`: `CHANGE_ME_MLB_17CHARS`
- `SystemUUID`: `00000000-0000-0000-0000-000000000000`
- `ROM`: `<112233445566>`

引导前用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 按 `MacBookPro16,4` 生成一套填进去。建议先写到 FAT32 U 盘测试，确认无误再同步进内置硬盘 EFI 分区。

### 配置改动

- **S3 睡眠防秒醒**：[EFI/OC/ACPI/SSDT-GPRW.aml](EFI/OC/ACPI/SSDT-GPRW.aml) 配合 `config.plist` 里的 `GPRW -> XPRW` 重命名，屏蔽 `0x6D` / `0x6B` / `0x0D` / `0x89` 唤醒源；[EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml](EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml) 在 `_PTS(3)` 和 `_PTS(5)` 前清掉 `PC02.XHCI.PMEE` 与 `PC00.TXHC.PMEE`。
- **CPU 型号显示**：`RestrictEvents.kext` + `boot-args` 里的 `revpatch=sbvmm,cpuname`（配合 `NVRAM -> Delete` 清理旧缓存），系统里显示 `Intel(R) Core(TM) Ultra 7 255HX`。
- **EC 功耗模式**：集成 `EFI/OC/Kexts/LegionECMode.kext`，无需 `sudo` 即可切换安静/均衡/狂暴模式，具体说明见 [LegionECMode-kext](https://github.com/KSLengYu/LegionECMode-kext)。

### 参考与致谢

用到了 Acidanthera（OpenCore、Lilu、VirtualSMC、RestrictEvents、VoodooI2C 等）、NullMoth 的 NVRM 驱动以及黑苹果社区的相关工具和资料。修改 EFI 前记得留好可引导的备份 U 盘。

---

## English

OpenCore EFI for a Lenovo Legion 5 Pro (2025, Arrow Lake-HX) running **macOS 26.7.2**, with SMBIOS redacted.

The bundled `LegionECMode.kext` source and standalone package live in a separate repository: [KSLengYu/LegionECMode-kext](https://github.com/KSLengYu/LegionECMode-kext).

### Hardware & drivers

| Component | Hardware / Driver | Notes |
| --- | --- | --- |
| macOS | macOS 26.7.2 | Tested OS version |
| CPU | Intel Core Ultra 7 255HX (Arrow Lake-HX) | `RestrictEvents.kext` + `revpatch=sbvmm,cpuname` |
| GPU | NVIDIA RTX 5070 Laptop (`10de:2d58`) | Uses **NullMoth** NVIDIA driver (`NVRM` / `NVRMFB` / `NVRMAGDC` 3.0 in `/Library/Extensions/`, not included here) |
| Internal Display | `2560x1600 @ 165Hz` (eDP) | HiDPI, brightness control, and sleep/wake work |
| Sleep | S3 | Instant wake from XHCI / TXHC / RP09 / RP17 addressed |
| EC Power Mode | `_SB.GZFD.WMAA` | Uses `LegionECMode.kext` via `sysctl -w debug.legionec.mode=1/2/3` without `sudo` |

The NullMoth NVIDIA Blackwell kexts (`com.nullmoth.NVRM` / `NVRMFB` / `NVRMAGDC`) must be installed separately into `/Library/Extensions/` and are not shipped in this repository.

### SMBIOS setup before booting

`PlatformInfo -> Generic` in [EFI/OC/config.plist](EFI/OC/config.plist) has been replaced with placeholders:

- `SystemSerialNumber`: `CHANGE_ME_SERIAL`
- `MLB`: `CHANGE_ME_MLB_17CHARS`
- `SystemUUID`: `00000000-0000-0000-0000-000000000000`
- `ROM`: `<112233445566>`

Generate your own values for `MacBookPro16,4` using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) before signing in to Apple services. Test from a FAT32 USB drive first before copying to the internal EFI partition.

### Key changes in this config

- **S3 sleep wake fix**: [EFI/OC/ACPI/SSDT-GPRW.aml](EFI/OC/ACPI/SSDT-GPRW.aml) with the `GPRW -> XPRW` rename patch in `config.plist` blocks wake sources `0x6D`, `0x6B`, `0x0D`, and `0x89`; [EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml](EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml) clears `PC02.XHCI.PMEE` and `PC00.TXHC.PMEE` on `_PTS(3)` and `_PTS(5)`.
- **CPU brand string**: `RestrictEvents.kext` + `revpatch=sbvmm,cpuname` in `boot-args` (with `NVRAM -> Delete` entries) displays `Intel(R) Core(TM) Ultra 7 255HX`.
- **EC power mode**: bundles `EFI/OC/Kexts/LegionECMode.kext` for passwordless Quiet/Balanced/Performance switching; see [LegionECMode-kext](https://github.com/KSLengYu/LegionECMode-kext) for details.

### Credits

Built with OpenCore and kexts from Acidanthera (Lilu, VirtualSMC, RestrictEvents, VoodooI2C, etc.), the NullMoth NVRM driver, and the Hackintosh community. Keep a working bootable USB backup before modifying your EFI.
