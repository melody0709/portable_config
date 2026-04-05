# mpv + uosc IPTV 播放器开发指南

基于 **mpv** + **uosc 5.12** 的 IPTV 播放器，支持直播、EPG 回看、远程订阅。

## 核心功能

- **四级菜单 (F8)**：分组 > 频道 > 日期桶 > EPG；搜索支持中文/拼音/首字母
- **组内切台 (PgUp/PgDn)**：直播时在当前组内切换，回看状态不执行
- **EPG 回看**：XMLTV 节目单，时间跳转回看；F9 跨频道搜索节目
- **远程 M3U 订阅**：`m3u_download_url` 缓存启动 + 后台刷新
- **强制刷新 (Shift+F9)**：忽略缓存重新下载 EPG/M3U
- **历史记录**：自动保存/恢复上次频道，快速切台合并写盘

## 文件结构

```
portable_config/
├── scripts/
│   ├── epg/              # 核心业务逻辑 (main.lua 为入口)
│   │   ├── main.lua      # 全局状态 + 事件/按键绑定
│   │   ├── utils.lua     # 工具函数 (纯函数，无 state 依赖)
│   │   ├── data.lua      # M3U/EPG 解析 + 历史记录
│   │   ├── menu.lua      # 四级菜单构建
│   │   └── playback.lua  # 直播/回看/切台逻辑
│   └── uosc/             # ⚠️ 已修改的 uosc 源码 (见 UOSC_MODIFY_DIFF.md)
│       ├── main.lua      # 【修改】新增消息处理器
│       ├── elements/Menu.lua       # 【修改】菜单项/搜索/交互
│       ├── elements/TopBar.lua     # 【修改】顶部标题追加 IPTV 信息
│       └── lib/menus.lua           # 【修改】toggle_menu_with_items
└── script-opts/
    ├── epg.conf          # 发布版默认配置
    └── myepg.conf        # 本地私有覆盖 (不发布)
```

## uosc 修改规范

修改 uosc 源码必须：
1. 用中文标记 `-- 【修改】`/`-- 【新增】`/`-- 【删除】`
2. 保留原始代码注释，在附近添加说明
3. 新增修改点时同步更新 `UOSC_MODIFY_DIFF.md`（迁移清单格式：文件 + 位置 + 用途）

## 开发要点

- **优先扩展 epg/**，仅在 UI 交互需要时修改 uosc
- **多平台兼容**：Windows/Linux/macOS，注意 gzip 解压差异
- **错误处理**：网络请求、文件读写、数据解析均需容错

## 环境工具

- **Lua 5.4**：已通过 scoop 安装，用于语法检查和脚本测试
- **mpv.exe**：位于 `../mpv.exe`
- **Shell**：默认使用 `pwsh` (PowerShell 7)，禁止使用 `powershell.exe` (5.1)；需要 Unix 工具链时使用 Git Bash

## API 参考

```lua
-- uosc 扩展接口
mp.commandv("script-message-to", "uosc", "open-menu", json_data, submenu_id)
mp.commandv("script-message-to", "uosc", "expand-submenu", menu_id)
mp.commandv("script-message-to", "uosc", "select-menu-item", menu_type, index, parent_id)

-- epg 关键函数
parse_m3u(path)                  -- 解析 M3U
fetch_and_parse_epg_async()      -- 异步获取 EPG
build_main_menu()                -- 构建菜单数据
switch_channel_in_current_group(direction)  -- 组内切台
```

## Git 规范

- 提交格式：`feat:` / `fix:` / `refactor:` / `style:` / `docs:`
- 版本：fix → z+1，feat → y+1，重构 → x+1
- 发布前核对：Release 版本、Git Tag、CHANGELOG 三者一致
- `script-opts/epg.conf` 为发布默认配置，私人地址写 `myepg.conf`

## 调试

```lua
mp.msg.info("调试信息")     -- mpv 控制台 (按 ` 查看)
mp.osd_message("提示", 3)   -- OSD 显示 3 秒
```

### 测试启动

```powershell
# 空载启动 (从缓存订阅加载 IPTV)
../mpv.exe
```

## 相关文档

- `UOSC_MODIFY_DIFF.md` - uosc 修改记录
- uosc: https://github.com/tomasklaen/uosc

## 经验记录

> AI 在完成重大修改或解决复杂报错后，可以追加经验记录,更新AGENTS.md。

