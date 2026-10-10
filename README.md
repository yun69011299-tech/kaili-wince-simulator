# kaili-wince-simulator

2010 年代凯立德（Kaili）车机系统模拟器 —— 面向测试用途的单文件 HTML 实现。

模拟对象：老款丰田（卡罗拉 / 凯美瑞 2007–2011 等）后装 / 专车专用车机，
操作系统 **Windows Embedded CE 6.0**，800×480 电阻屏，凯立德地图以 SD 卡
`\NaviOne` 目录装载，入口 `NaviOne.exe`，GPS 端口 / 波特率由
`NaviOne\naviconfig.txt` 控制（常见 COM1/4800）。

## 快速开始

无需构建，双击 `kaili-wince-simulator.html` 在浏览器打开即可；
或起本地服务（避免个别浏览器的本地文件限制）：

```powershell
npx serve .
# 或
python -m http.server 8000
```

打开后会播放 WinCE 冷启动动画 → 进入主界面。

## 功能

- **车机仿真**：丰田 2DIN 外壳、实体按键（POWER / NAV / MENU / EJECT / TEST / VOL）、
  WinCE 6.0 启动流程（开机 LOGO 可自定义）、10 图标主界面。
- **凯立德导航**：网格道路地图、POI 查找 → 路线规划 → 逐向导航提示、
  正北 / 车头朝上视角与缩放；GPS 模拟器驱动车辆行驶，
  实时生成带校验和的 `$GPGGA` / `$GPRMC` NMEA 语句。
- **收音机**：FM1/2/3 + AM1/2 共 5 波段 × 6 预置台（30 台）、RDS 电台名/PTY、
  手动步进 / 搜台 / 自动扫描存台、长按预置台存台、立体声切换（v0.2）。
- **蓝牙**：配对（设备名 TOYOTA-HU / PIN 0000）、拨号盘、PBAP 电话本、通话记录、
  模拟来电接听挂断、A2DP 蓝牙音乐播放（v0.2）。
- **倒车影像**：挂 R 挡全屏强制切入，动态三色轨迹线 + 后 4 探头 OPS 距离模拟（v0.2）。
- **行车电脑 / CANBUS**：P/R/N/D 挡位、车速 / 里程 / 油耗 / 水温 / 油量、
  车门与灯光示意（可点击）、500ms 采样的十六进制 CAN 报文监视器（v0.2）。
- **方向盘按键学习（SWC）**：8 个方控键（音量 / 切台 / MODE / 静音 / TEL / NAV）
  可学习重映射，屏幕下方有实体按键模拟条（v0.2）。
- **音效 DSP**：EQ 预设（流行/摇滚/古典/影院/用户）、高低音、前后 / 左右声场、Loud（v0.2）。
- **媒体播放**：读取 SD 音轨（mp3/wav/wma/flac 等）、播放控制、进度跳转、
  列表 / 单曲 / 随机、断点续播持久化（v0.2）。
- **个性化**：壁纸 5 套、按键灯 7 色、屏幕亮度、12/24 小时制、
  刹车检测线（D 挡禁看视频，可开关）（v0.2）。
- **虚拟 SD 卡**：FAT32 目录树，`localStorage` 持久化；支持拖拽 / 文件选择器
  安装软件到 `/Installed/`、在线编辑、导出还原、删除、格式化、弹出插入；
  `run /NaviOne/NaviOne.exe` 可从 SD 启动导航。
- **测试通道**：见下文。

## 测试通道

### 1. 调试控制台（页面内按 `F12`）

| 命令 | 说明 |
| --- | --- |
| `help` | 命令列表 |
| `dir [路径]` / `cat <路径>` | 列目录 / 查看文件 |
| `write <路径> <内容>` / `del <路径>` | 写入 / 删除文件（自动持久化） |
| `install` | 打开文件安装对话框 |
| `mount` / `eject` / `format` | SD 卡插入 / 弹出 / 格式化 |
| `gps on\|off` / `gps speed <m/s>` / `gps fix <lat,lon>` | GPS 模拟器控制 |
| `nmea <语句>` | 注入原始 NMEA（自动补校验和） |
| `port COM7 9600` | 写入 `naviconfig.txt` 端口配置 |
| `run <路径>` | 运行 SD 上的 exe |
| `route <POI序号\|名称>` | 规划路线 |
| `gear p\|r\|n\|d` | 换挡（R 挡切入倒车影像）（v0.2） |
| `bt pair\|ring\|answer\|hangup\|music` | 蓝牙控制（v0.2） |
| `eq <关闭\|流行\|摇滚\|古典\|影院>` | EQ 预设（v0.2） |
| `vol <0-40>` | 音量（v0.2） |
| `media play\|pause\|next\|prev\|list` | 媒体播放（v0.2） |
| `can` | CAN 数据快照（v0.2） |
| `swc learn\|status` | 方控学习 / 映射（v0.2） |
| `radio seek\|scan\|FM1..AM2` | 收音机控制（v0.2） |
| `log export` | 导出 JSON 测试日志 |
| `reboot` / `home` / `nav` / `sysinfo` | 系统控制 |

### 2. JS API（浏览器 DevTools 或自动化脚本）

```js
kld.cmd('port COM7 9600')      // 执行控制台命令
kld.files()                    // 列出 SD 文件
kld.read('/NaviOne/naviconfig.txt')
kld.write('/Installed/foo.txt', 'hello')
kld.run('/NaviOne/NaviOne.exe')
kld.nmea()                     // 当前 NMEA 语句
kld.gps                        // GPS 状态对象 {on,x,y,speed,sats,fix,...}
kld.setGear('R')               // 换挡（触发倒车影像）
kld.bt                         // 蓝牙状态 {paired,calling,ringing,playing,...}
kld.can                        // CAN 状态 {gear,doors,odo,fuel,water,...}
kld.radio                      // 收音机状态 {band,freq,presets,rds,...}
kld.mp                         // 媒体播放状态 {path,playing,pos,mode}
kld.prefs                      // 音效/方控/个性化偏好（localStorage 持久化）
kld.swcPhysical('vol+')        // 模拟方控按键
kld.exportLog()                // 导出日志
```

自动化测试可直接驱动 `kld` API 断言 SD 读写、端口配置生效、导航启动链路、
挡位与倒车影像联动、蓝牙状态机、方控映射等。

## 代码结构（单文件分区）

`kaili-wince-simulator.html` 内部按区块组织，均带注释标题：

| 区块 | 职责 |
| --- | --- |
| 虚拟 SD 卡 | `SEED` 初始目录、`loadSD/saveSD` 持久化、目录/文件操作 |
| 页面路由 | `PAGES` 映射与 `show()` / `openApp()`（含刹车检测线拦截） |
| 启动流程 | `boot()` 冷启动动画（支持自定义 LOGO） |
| GPS / 导航引擎 | `ROUTE/POIS/ROADS` 世界坐标、`gpsTick` 推进、`toNMEA` 语句生成、`drawMap` canvas 绘制、`navAction` 菜单覆盖层 |
| SD 管理界面 | 文件树渲染、安装/编辑/导出/格式化 |
| 收音机 | `BANDS/STATIONS`、RDS、搜台扫描、长按存台（v0.2） |
| 蓝牙 | 配对/拨号/电话本/通话状态机/A2DP（v0.2） |
| CAN & 倒车 | `setGear`、`drawRev` 轨迹线与 OPS、车身示意、CAN 监视器（v0.2） |
| 方控/音效/个性化 | `prefs` 持久化、SWC 学习流程、EQ 与声场（v0.2） |
| 媒体 | SD 音轨扫描、播放状态机、断点续播（v0.2） |
| 测试控制台 | `runCmd()` 命令分发、日志导出 |
| 对外 API | `window.kld` |

坐标系：以启动点为原点的局部米坐标 (x 东, y 北)，
`meters2latlon()` 换算为经纬度生成 NMEA。

## 开发与协作约定

- **分支**：`main` 为稳定分支，直接可运行；新功能请开
  `feature/<简述>` 分支，完成后合回 `main`。
- **提交**：语义化前缀（`feat:` / `fix:` / `docs:` / `chore:`），一个提交只做一件事。
- **版本**：语义化标签 `vMAJOR.MINOR.PATCH`，发布打 tag 并 `git push --follow-tags`。
- **保持单文件**：整个模拟器就是 HTML 一个文件，无构建步骤，改动后浏览器刷新即可验证。
- **验证清单**（提交前）：冷启动 → 主界面 → 导航查找并规划路线 → 收音机搜台/存台 →
  蓝牙配对/来电 → 挂 R 倒车影像可退出 → 方控 MODE 循环 → 控制台
  `dir /NaviOne`、`port COMx <baud>`、`gear r/p` 生效 → SD 管理可安装/导出文件 → 无 JS 报错。

## 路线图（欢迎认领）

- [ ] 导航层：电子眼/限速播报、路线偏好、夜间配色、3D 实景路口放大图
- [ ] 地图数据包（`.cld`）模拟加载 / 校验流程
- [ ] 一机多图通道（多导航软件共存切换）
- [ ] 扩展模块：CMMB 数字电视 / DVR / TPMS / OBD 页面（设置页已留入口位）
- [ ] 更多车型皮肤（卡罗拉 / 凯美瑞 UI 差异）
- [ ] Playwright 自动化测试脚本驱动 `window.kld`

## License

MIT
