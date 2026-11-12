# 分项实操手册

## 1. Pegasus metadata 规范

文件位置 `<TF>/Roms/<平台>/metadata.pegasus.txt`（UTF-8）。头部与游戏块：

```txt
collection: CPS1
shortname: cps1
extensions: zip
launch: am start --user 0 -n com.retroarch.aarch64/com.retroarch.browser.retroactivity.RetroActivityFuture -e ROM "{file.path}" -e LIBRETRO /data/data/com.retroarch.aarch64/cores/fbalpha2012_cps1_libretro_android.so -e CONFIGFILE /storage/emulated/0/Android/data/com.retroarch.aarch64/files/retroarch.cfg

game: 1941 反击战
file: 1941.zip
assets.box_front: Imgs/1941 反击战.png
description: 《1941 反击战》，CPS1 平台街机射击游戏。
  第二行起缩进两空格可续写长简介。
publisher: CAPCOM
release: 1990
genre: 射击
players: 1-2
```

要点：
- `description:` 支持多行，后续行缩进两空格；留空则主题显示 `No description available`。
- `publisher / developer / release / genre / players` 影响 prosperoOS 详情页「More by 厂商」区块；16 个平台原本缺这些字段。
- per-game `launch:` 覆盖集合级 `launch:`，用于换核心（例：`mercsj / samuraia / sonicwi` 换 `fbneo_libretro_android.so`）或独立 APP（PSP 用 PPSSPP intent）。
- **改完必做**：`rm /sdcard/pegasus-frontend/dataCache.json` → 重启天马。

### ANDROID 移植游戏（关键）

`file:` 必须是真实文件，组件名会被丢弃：

```txt
collection: ANDROID
shortname: android
extensions: app

game: 忍者神龟：施莱德的复仇
file: tmnt.app
launch: am start --user 0 -n com.netflix.NGP.TMNTShreddersRevenge/crc64ce797c93931d6e91.MainActivity
assets.box_front: Imgs/忍者神龟：施莱德的复仇.png
```

用 `scripts/kpa_android_apps.sh` 创建占位文件并推送。已整合 7 个：忍者神龟 / 星露谷物语 / 泰拉瑞亚 / 像素足球19 / 小丑牌 / 海之号角 / 口袋博物馆。封面用 `adb exec-out screencap` 截启动画面生成。

## 2. RetroArch 配置分层（优先级从低到高）

| 层 | 路径 | 说明 |
|---|---|---|
| 全局 | `/storage/emulated/0/Android/data/com.retroarch.aarch64/files/retroarch.cfg` | 黑黄 ozone / vulkan / CRT / 96ms / 存档迁 TF / 官方核心源；**设备上存在两份，改要一起改** |
| 核心级 | `/sdcard/RetroArch/config/<核心名>/<核心名>.cfg` | 该核心全部游戏生效 |
| 全局着色器 | `/sdcard/RetroArch/config/global_*.slangp` | 改这一处可覆盖上百个逐游戏预设 |
| per-game | `/sdcard/RetroArch/config/<核心名>/<ROM名>.cfg` | 竖版旋转 / 方向映射写这里 |
| remaps | `/sdcard/RetroArch/config/remaps/<核心>/<核心>.rmp` | **厂商基线，禁止污染**（会作用于该核心全部游戏） |

`libretro_directory` 指向 `/data/user/0/com.retroarch.aarch64/cores/`，RA 从不读 public `RetroArch/cores/`。显示核查基线：`video_filter=""`（滤镜关）、`video_shader_enable=true` 命中 crt-easymode、厂商 `overlays/*_B.cfg` 边框正常、竖版 cfg 统一 `input_overlay_enable=false`。

## 3. 竖版游戏（216 个 cfg）

**最终统一值**（用户实机「1941 测试正常」）：

```ini
video_rotation = "1"          # MAME 2003-Plus / VARCADE；FBNEO 系填 "3"
aspect_ratio_index = "0"
video_scale_integer = "false"
input_overlay_enable = "false"
input_player1_up_btn = "h0right"
input_player1_down_btn = "h0left"
input_player1_left_btn = "h0up"
input_player1_right_btn = "h0down"
input_player1_a_btn = "109"
input_player1_b_btn = "108"
input_player1_start_btn = "97"
input_player1_select_btn = "96"
```

- 厂商原版基线是反的：`up=h0left / down=h0right / left=h0down / right=h0up`（69 个 FBNEO cfg 字节级一致，md5 `44950e1a…`）。**翻转时只精确替换 4 行方向**，其余原样。
- 分布：`MAME 2003-Plus` 140（rot=1）、`FinalBurn Neo` 69（rot=3）、CPS1 3 / CPS2 2 / NEOGEO 2（rot=3）。
- 竖版判定用 `assets/data/mame_orient.json`（6055 条），DB 缺的 4 个补 `gekiridn / tengaij / p47mus / schmeisr`。
- 修已有 cfg 用 `bash scripts/kpa_vertical.sh`（默认 dry-run，`--apply` 才写）。

## 4. MSX blueMSX 核心（本地镜像法）

APK 未内置 bluemsx/fmsx；官方源约 17.5KB/s，手动下载会卡在 14%。步骤：

1. 拿到 `bluemsx_libretro_android.so`（可从已安装设备的私有目录或 RA 官方 zip 解出）。
2. 建最小核心源目录：`zip -j bluemsx_libretro_android.so.zip <.so>`，并造 `.index-extended`（**不是 `.index`**）：
   ```txt
   2026-11-10 3f68b86d bluemsx_libretro_android.so.zip
   ```
   格式每行 `日期 哈希 文件名`。只有 `.index` 时 RA 列表显示「没有可显示的条目」。
3. Mac 起 `python3 -m http.server 8765 --bind 0.0.0.0`（目录即核心源目录）。
4. 临时把**两份** `retroarch.cfg` 的 `core_updater_buildbot_cores_url` 改成 `http://<Mac 局域网 IP>:8765/`。
5. RA 菜单：在线更新 → 核心下载 → BlueMSX（用 `input motionevent` 点）。RA 自己下载并解压到私有 cores 目录。
6. 验证 logcat `avc: granted { execute } .../cores/bluemsx_libretro_android.so`；**装完把两份 cfg 改回官方源 `http://buildbot.libretro.com/nightly/android/latest/arm64-v8a/` 并关掉 HTTP 服务**。
7. MSX metadata 加 launch 行指向该核心；BIOS 在 `/storage/emulated/0/RetroArch/system/`（MSX.rom / MSX2.rom + Machines / Databases）。

MSX 冷启动约 15–18 秒出日文字符界面，不是黑屏。脚本见 `scripts/kpa_msx_core.sh`。

## 5. `.nomedia`（图库隐藏）

加 4 个目录，不删文件、不影响天马按路径读封面：

```txt
/storage/6AF3-384F/Roms/
/storage/emulated/0/RetroArch/
/storage/emulated/0/pegasus-frontend/
/storage/emulated/0/Android/data/com.pocketmuseum.app/
```

重扫：`content call --uri content://media --method scan_volume --arg external_primary`。内部存储立即生效（RetroArch 1105→1、pocketmuseum 20002→1），TF 卡卷 `public:179,129` 重扫不生效，**需重启一次**。脚本见 `scripts/kpa_nomedia.sh`。

## 6. 详情页 `GameDetails.qml` 增强

视觉根因：`gamedetails` 是 mainList 第 2 页（需按 ↓），未选中 `opacity: selected ? 1 : 0.5`、标题 `vpx(24)`、简介 `vpx(18)`、`elide: Text.ElideRight` 截断长文、无背景卡片。

增强版 `assets/golden/GameDetails.qml.improved`：opacity 0.5→0.85、加半透明卡片 + 橙色强调条、标题 24→30、简介 18→25、去掉 elide、`lineHeight: 1.3`、左右 margin。固定色写法（不要用 `theme.highlight || ...`，该主题未定义会回退失败）。

原版回滚点 `assets/golden/GameDetails.qml.before20261113`（设备侧 `.bak20261113`）。推送后清 `dataCache.json` 重启，检查 logcat 无 QML/QQml 报错。**远程输入无法稳定停在详情页那一屏**（方向键/滑动被解释成切换集合），观感必须实机确认。脚本见 `scripts/kpa_gamedetails.sh`。

内容侧：3772 条 description 平均 46.3 字，约 30%（1142 条）是「《X》，X 平台。」空壳句；用 `scripts/bangumi_desc.py` 升级为真简介并回填 16 个平台的 publisher/genre/release。

## 7. 封面修复链路

坏封面三来源：黑屏图（抓图失败）、小图（<8KB）、缺图。按顺序：

1. zip 内部文件名（汉化 ROM 的原版英文名）→ `https://thumbnails.libretro.com/<系统>/Named_Boxarts/<名>.png` 匹配；系统名形如 `MAME`、`SNK - Neo Geo`、`Bandai - WonderSwan`、`Sega - Game Gear`、`Nintendo - Game Boy Color`。
2. 匹配不到 → RA 实机截图：`am start` 带核心参数 → 等 15–40s → `screencap` → PIL 裁掉 RA 边框缩放 480×320。
3. 三核心全黑 → 标题文字占位图。
4. EASYRPG 特例：从游戏目录 `.xyz` 标题画面按 `xyzpal_first / xyzpal_last` 调色板解码，重做 13 张真封面（Mogeko城堡 / 威伦诺尔 / 寂静岭 / 废都物语 / 怪异症候群1-2 / 恐怖美术馆 / 梦幻重装机兵 / 灰色庭院 / 野比大雄系列 4 作），部署到 `Roms/EASYRPG/Imgs/` 并 md5 校验。

## 8. 冷启动等待基准（冒烟判定）

| 平台 / 游戏 | 出画面时间 |
|---|---|
| PS / SATURN / N64 / MD / SFC / NDS | 8–20s |
| `PS/射雕英雄传.chd`（368MB） | 15s |
| ATOMISWAVE | 20–30s（首屏 NOTICE 版权屏，不是不兼容） |
| MSX | 15–18s |
| 泰拉瑞亚 / 星露谷物语 | 15s |

3.2 秒截图会得出大量假黑屏。判定顺序：前台窗口 → `ANR in com.retroarch.aarch64` → 非黑像素 + 外接矩形 → 再下结论。

## 9. 网络与传输实测

- 可用：`api.github.com`、gh-proxy 镜像、libretro thumbnails。thumbnails 目录索引走 `git/trees?recursive=1`，`contents` 分页无效。
- 不可假设可达：GitHub raw、bgm.tv、wikipedia、thegamesdb、screenscraper。
- macOS adb r34.0.5；大传输分块 55MB / 300 文件；`tar -T` 清单只放文件；校验用 `md5sum "路径"`（中文路径必须带双引号）。
