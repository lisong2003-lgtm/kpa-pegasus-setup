# Changelog

## 2.0.0 — 2026-11-13

名称定为「KPA掌机天马安装与优化助手」，slug 统一为 `kpa-pegasus-setup`，发布到 SkillHub / ClawHub / GitHub。

### 修正（会导致实机故障的错误）

- 竖版方向映射：`up=h0left / down=h0right / left=h0down / right=h0up`（厂商基线）→ `up=h0right / down=h0left / left=h0up / right=h0down`（实机验证值）。同步修正 `SKILL.md` 与 `scripts/step4_metadata.sh` 两处。
- ANDROID 集合 `file:` 由 Android 组件名改为 `.app` 占位真实文件 + `extensions: app` + per-game `launch:`；重写 `assets/meta_latest/ANDROID.txt`，并补 `assets/golden/game_dirs.txt` 的 ANDROID 行。
- 机型名由误写的 "AYANEO Pocket Advance" 改为正式名 KONKR Pocket ADVANCE（AYANEO 旗下）。
- MSX 核心安装方式由「下载 .so 到 public 路径」改为「本地 HTTP 最小核心源 + `.index-extended`，让 RA 自己装进私有 cores 目录」。
- `scripts/bangumi_desc.py` 移除硬编码的 adb 绝对路径，改为 `KPA_ADB` / PATH / 自动发现；TF 卡路径支持 `KPA_TF`。
- `scripts/step6_cleanup.sh` 移除两处通配 `rm -rf`，改为只盘点、不自动删。

### 新增

- `scripts/kpa_vertical.sh`：修存量竖版 cfg，默认 dry-run，只精确替换 4 行方向。
- `scripts/kpa_msx_core.sh`：MSX 本地镜像 prepare / restore。
- `scripts/kpa_android_apps.sh`：ANDROID 占位文件与 metadata 部署。
- `scripts/kpa_nomedia.sh`：4 处 `.nomedia` + MediaStore 重扫。
- `scripts/kpa_gamedetails.sh`：详情页 QML 增强与回滚点。
- `scripts/kpa_health_check.sh`：全库静态校验 + 离线自检 `--local`。
- `references/PROCESS_REVIEW.md`：7 阶段完整工程复盘。
- `references/HARD_RULES.md`：22 条硬性禁令与正确做法。
- `references/PLAYBOOK.md`：9 个分项实操手册。
- `references/VERIFICATION.md`：健康校验 SOP、达标基线、遗留项、清理清单、回滚点。
- `assets/golden/GameDetails.improved.qml.txt` 与 `.before20261113`。
- `README.md`、`.gitignore`、本文件；GitHub 仓库 `archive/` 归档工程报告与回滚点（已脱敏）。

### 记录

- 补充「禁止 `am force-stop com.ayaneo.home`」等系统按键层禁令。
- 补充核心批量更新风险评估（3% 中断无影响，不建议更新 nightly）。
- 补充冷启动等待基准（8–26 秒）与 toybox `unzip` 中文条目 CRC 误报说明。
- 明确遗留项：`WS/恐怖传说.zip` 坏 ROM、`_backup_20261112_2347` 幽灵目录、1142 条空壳简介、详情页观感待实机确认、设备侧 17 项待清理。

## 1.0.0 — 2026-11-12

上架版：7 步幂等安装脚本（APP / 主题 / RA 主配置 / metadata + 竖版 cfg / PPSSPP / 清理 / 验证）、`sync_from_device.sh`、`bangumi_desc.py`、36 平台 metadata 与方向库、prosperoOS 主题包。约 3.4MB，不含 ROM / BIOS / APK。
