# Lenovo Legion 5 Pro (2025) Hackintosh OpenCore EFI

适用于 **Lenovo Legion 5 Pro（2025，Arrow Lake-HX）** 的 OpenCore EFI 配置（已脱敏三码，可直接用于 U 盘引导或同步至内置硬盘 EFI 分区）。

> 本仓库内置的 `LegionECMode.kext` 独立源码与预编译版本维护在：
> 👉 **[KSLengYu/LegionECMode-kext](https://github.com/KSLengYu/LegionECMode-kext)**

---

## 硬件配置与工作状态

| 组件 | 规格 | 状态 |
| --- | --- | --- |
| CPU | Intel Core Ultra 7 255HX (Arrow Lake-HX) | 完整型号已正确识别（`revpatch=sbvmm,cpuname`），核心/线程与调度正常 |
| GPU | NVIDIA GeForce RTX 5070 Laptop (`10de:2d58`) | 配合系统 `/Library/Extensions/NVRM*.kext` (`NVRM`/`NVRMFB`/`NVRMAGDC` 3.0) 硬件加速、高刷与外接显示正常 |
| 内建屏幕 | `2560x1600 @ 165Hz` (eDP, HiDPI) | 亮度调节与休眠唤醒正常 |
| 睡眠（S3） | `SSDT-GPRW.aml` + `SSDT-FIXSHUTDOWN.aml` | 已修复 XHCI/TXHC/RP09/RP17 等导致的睡眠秒醒问题 |
| 整机 EC 模式 | `_SB.GZFD.WMAA` | 已集成自研 `LegionECMode.kext`，终端免密执行 `sysctl -w debug.legionec.mode=1/2/3` 即可切换安静/均衡/狂暴模式 |

> 注：NVIDIA Blackwell (`NVRM` / `NVRMFB` / `NVRMAGDC`) 闭源驱动需安装到 macOS 系统 `/Library/Extensions/` 目录，不包含在本 EFI 仓库内。

---

## ⚠️ 使用前必读（三码已脱敏）

为了隐私安全，[EFI/OC/config.plist](EFI/OC/config.plist) 中的 `PlatformInfo -> Generic` 已全部替换为占位值：

- `SystemSerialNumber`: `CHANGE_ME_SERIAL`
- `MLB`: `CHANGE_ME_MLB_17CHARS`
- `SystemUUID`: `00000000-0000-0000-0000-000000000000`
- `ROM`: `<112233445566>`

**引导前请务必：**
1. 使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 为机型 **`MacBookPro16,4`** 生成一套全新的三码与 `ROM`。
2. 填入 `EFI/OC/config.plist -> PlatformInfo -> Generic` 后保存。
3. 建议先拷贝到 FAT32 U 盘测试引导，确认全部正常后再同步到内置硬盘 EFI 分区。

---

## 主要定制补丁说明

1. **S3 睡眠防秒醒（ACPI）**
   - [EFI/OC/ACPI/SSDT-GPRW.aml](EFI/OC/ACPI/SSDT-GPRW.aml) + `config.plist` 中 `GPRW -> XPRW` 重命名补丁：屏蔽 `0x6D` / `0x6B` / `0x0D` / `0x89` 唤醒源。
   - [EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml](EFI/OC/ACPI/SSDT-FIXSHUTDOWN.aml)：在关机（`_PTS(5)`）与睡眠（`_PTS(3)`）前清理 `PC02.XHCI.PMEE` 和 `PC00.TXHC.PMEE`。
2. **CPU 完整型号名称**
   - `RestrictEvents.kext` + `boot-args` 中 `revpatch=sbvmm,cpuname`（配合 `NVRAM -> Delete` 清理旧值），系统可完整识别 `Intel(R) Core(TM) Ultra 7 255HX`。
3. **免密 EC 功耗模式驱动集成**
   - `EFI/OC/Kexts/LegionECMode.kext`：在内核中暴露 `debug.legionec.mode` 接口直接读写 `_SB.GZFD.WMAA`，详见 [LegionECMode-kext 仓库](https://github.com/KSLengYu/LegionECMode-kext)。

---

## 致谢与免责声明

- 感谢 Acidanthera（OpenCore / Lilu / VirtualSMC / RestrictEvents / VoodooI2C 等）、NVRM Blackwell macOS 驱动生态及拯救者 Hackintosh 社区。
- 本配置按“现状”开源分享，不提供任何担保；修改 EFI 引导前请务必保留可引导的备份 U 盘。
